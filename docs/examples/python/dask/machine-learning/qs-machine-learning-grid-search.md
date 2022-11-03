# Grid Search

Hyperparameter tuning is a crucial, and often painful, part of building machine learning models. Squeezing out each bit of performance from your model may mean the difference of millions of dollars in ad revenue or life-and-death for patients in healthcare models. Even if your model takes one minute to train, you can end up waiting hours for a grid search to complete (think a 10×10 grid, cross-validation, etc.). Each time you wait for a search to finish breaks an iteration cycle and increases the time it takes to produce value with your model. When using pandas, NumPy, and scikit-learn for model training, you can often speed up the grid search with the help of Dask.

There are several different scenarios that can arise when doing hyperparameter searching:

1. The training data is small, and the parameter space is small - You do not need Dask, use scikit-learn.
2. The training data is small, and the parameter space is **large** - Train the data with pandas/NumPy/scikit-learn, and use [joblib and Dask](https://ml.dask.org/joblib.html) for distributed parameter testing.
3. The training data is **large** - Use the [Dask-ML package](https://ml.dask.org/), with classes that look and feel like scikit-learn, to spread operations across your Dask cluster.

We'll illustrate the two Dask methods of scaling grid search using the famous [NYC Taxi Dataset](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page). This dataset contains information on taxi trips in New York City.

First, start the Dask cluster associated with your Saturn Cloud resource.


```python
from dask_saturn import SaturnCluster
from dask.distributed import Client

client = Client(SaturnCluster())
```

After running the above command, it's recommended that you check on the Saturn Cloud resource page that the Dask cluster as fully online before continuing. Alternatively, you can use the command `client.wait_for_workers(3)` to halt the notebook execution until all three of the workers are ready.

## Joblib for small data and large parameter spaces
In this case, the training data and pipeline code remains in pandas/NumPy/scikit-learn. Scikit-learn has algorithms that support parallel execution via the `n_jobs` parameter, and `GridSearchCV` is one of them. By default, this parallelizes across all cores on a single machine using the [Joblib](https://joblib.readthedocs.io/en/latest/) library. Dask provides a Joblib backend that hooks into these scikit-learn algorithms to parallelize work across a Dask cluster. This enables us to pull in Dask just for the grid search.



```python
import pandas as pd
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.linear_model import ElasticNet
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import GridSearchCV
```

The data is loaded into a pandas DataFrame from S3:


```python
taxi = pd.read_parquet("s3://saturn-public-data/nyc-taxi/data/yellow_tripdata_2019-01.parquet")
```

The next chunk of code defines the features and cleans the data:


```python
raw_features = [
    "tpep_pickup_datetime",
    "passenger_count",
    "tip_amount",
    "fare_amount",
]
features = [
    "pickup_weekday",
    "pickup_weekofyear",
    "pickup_hour",
    "pickup_week_hour",
    "pickup_minute",
    "passenger_count",
]
label = "tip_fraction"


def prep_df(taxi_df):
    """
    Generate features from a raw taxi dataframe.
    """
    df = taxi_df[taxi_df.fare_amount > 0][raw_features].copy()  # avoid divide-by-zero
    df[label] = df.tip_amount / df.fare_amount

    df["pickup_weekday"] = df.tpep_pickup_datetime.dt.isocalendar().day
    df["pickup_weekofyear"] = df.tpep_pickup_datetime.dt.isocalendar().week
    df["pickup_hour"] = df.tpep_pickup_datetime.dt.hour
    df["pickup_week_hour"] = (df.pickup_weekday * 24) + df.pickup_hour
    df["pickup_minute"] = df.tpep_pickup_datetime.dt.minute
    df = df[features + [label]].astype(float).fillna(-1)

    return df


taxi_feat = prep_df(taxi)
```

The pipeline needs to be define for how to do the grid search:


```python
pipeline = Pipeline(
    steps=[
        ("scale", StandardScaler()),
        ("clf", ElasticNet(normalize=False, max_iter=100, l1_ratio=0)),
    ]
)

params = {
    "clf__l1_ratio": np.arange(0, 1.1, 0.1),
    "clf__alpha": [0, 0.5, 1, 2],
}

grid_search = GridSearchCV(
    pipeline,
    params,
    cv=3,
    n_jobs=-1,
    verbose=1,
    scoring="neg_mean_squared_error",
)
```

To execute the grid search in Dask we need to run inside a context manager for a Joblib backend. Besides that, we call the `grid_search.fit()` method the same way as you would when using scikit-learn in a non-distributed environment. When you run this cell, watch the Dask Dashboard to see the progress.


```python
import joblib

with joblib.parallel_backend("dask"):
    _ = grid_search.fit(
        taxi_feat[features],
        taxi_feat[label],
    )
```

Note that using the Dask Joblib backend requires sending the DataFrame through the scheduler to all the workers, so make sure your scheduler has enough RAM to hold your dataset.


## Dask-ML for large data and/or large parameter spaces

This version accelerates the grid search by using Dask DataFrames and Dask-ML's `GridSearchCV` class. [Dask-ML](https://ml.dask.org/) is a package in the Dask ecosystem that has its own parallel implementations of machine learning algorithms, written in a familiar scikit-learn-like API. This includes `GridSearchCV` and other hyperparameter search options. To use it, we load our data into a Dask DataFrame and use Dask ML’s preprocessing and model selection classes.

We begin by importing the required libraries. When setting up the training grid notice that we are still using scikit-learn's `ElasticNet` class in this case, but now we are using `dask_ml` versions of some libraries. This will use Dask to do the pre-processing and grid search work, but use scikit-learn for the model fitting. This means that within a given Dask worker, the processed training dataset will be pulled down to a pandas DataFrame. In most cases, this is probably okay because the data will be small after processing. If the data is still too large, you can use one of Dask-ML's estimators, such as [LinearRegression](https://ml.dask.org/glm.html).


```python
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.linear_model import ElasticNet
from dask_ml.preprocessing import StandardScaler
from dask_ml.model_selection import GridSearchCV
```

This time the data is read into a Dask DataFrame instead of a pandas one:


```python
import dask.dataframe as dd

taxi_dd = dd.read_parquet(
    "s3://saturn-public-data/nyc-taxi/data/yellow_tripdata_2019-01.parquet",
    storage_options={"anon": True},
    assume_missing=True,
)
```

The data is cleaned similar to before only now it's a Dask DataFrame being cleaned:


```python
def prep_df(taxi_df):
    """
    Generate features from a raw taxi dataframe.
    """
    df = taxi_df[taxi_df.fare_amount > 0][raw_features].copy()  # avoid divide-by-zero
    df[label] = df.tip_amount / df.fare_amount

    df["pickup_weekday"] = df.tpep_pickup_datetime.dt.isocalendar().day
    df["pickup_weekofyear"] = df.tpep_pickup_datetime.dt.isocalendar().week
    df["pickup_hour"] = df.tpep_pickup_datetime.dt.hour
    df["pickup_week_hour"] = (df.pickup_weekday * 24) + df.pickup_hour
    df["pickup_minute"] = df.tpep_pickup_datetime.dt.minute
    df = df[features + [label]].astype(float).fillna(-1)

    return df


taxi_feat_dd = prep_df(taxi_dd)
```

Again the pipeline is created in a similar manner as before:


```python
pipeline = Pipeline(
    steps=[
        ("scale", StandardScaler()),
        ("clf", ElasticNet(normalize=False, max_iter=100, l1_ratio=0)),
    ]
)

params = {
    "clf__l1_ratio": np.arange(0, 1.1, 0.1),
    "clf__alpha": [0, 0.5, 1, 2],
}

grid_search = GridSearchCV(
    pipeline,
    params,
    cv=3,
    scoring="neg_mean_squared_error",
)
```

Now we can run the grid search using the `grid_search` object defined above. It works the same way as scikit-learn’s `GridSearchCV` class.


```python
_ = grid_search.fit(
    taxi_feat_dd[features],
    taxi_feat_dd[label],
)
```

These are a couple of ways to speed up grid search for machine learning with Dask. Check out our [other examples](https://saturncloud.io/docs/examples/python/) for more ways to use Python on Saturn Cloud!
