# Grid Search with Scikit-learn and Dask


## Setup
Hyperparameter tuning is a crucial, and often painful, part of building machine learning models. Squeezing out each bit of performance from your model may mean the difference of millions of dollars in ad revenue or life-and-death for patients in healthcare models. Even if your model takes one minute to train, you can end up waiting hours for a grid search to complete (think a 10×10 grid, cross-validation, etc.). Each time you wait for a search to finish breaks an iteration cycle and increases the time it takes to produce value with your model.

This article assumes that you already have a working pipeline with single-node Python packages such as pandas, NumPy, and scikit-learn. This guide will help you take this code and speed it up! There are a few different scenarios that can arise when doing hyperparameter searching:
1. The training data is small, and the parameter space is small
2. The training data is small, and the parameter space is **large**
3. The training data is **large**, and the parameter space is small
4. The training data is **large**, and the parameter space is **large**

For case 1, you don't need Dask! Go ahead and use scikit-learn like usual. 

For case 2, we can keep training data in pandas/NumPy and code in scikit-learn, but scale the parameter testing across a cluster with [joblib and Dask](#joblib-for-large-parameter-spaces).

For cases 3 and 4, Dask has a [Dask-ML package](#dask-ml-for-large-data-andor-large-parameter-spaces) with classes that look and feel like scikit-learn, except all operations are spread across your Dask cluster.

We'll illustrate the two Dask methods of scaling grid search using the famous [NYC Taxi Dataset](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page). This dataset contains information on taxi trips in New York City.

### Connect to Dask cluster

First, let's connect to our Dask cluster. If you are not familiar with how to start a Dask cluster in Saturn Cloud, check out [this page](<docs/Using Saturn Cloud/create_dask_cluster.md>).

```python
from dask_saturn import SaturnCluster
from dask.distributed import Client

cluster = SaturnCluster()
client = Client(cluster)
```

## Joblib for large parameter spaces
In this case, the training data and pipeline code remains in pandas/NumPy/scikit-learn. Scikit-learn has algorithms that support parallel execution via the `n_jobs` parameter, and `GridSearchCV` is one of them. By default, this parallelizes across all cores on a single machine using the [Joblib](https://joblib.readthedocs.io/en/latest/) library. Dask provides a Joblib backend that hooks into these scikit-learn algorithms to parallelize work across a Dask cluster. This enables us to pull in Dask just for the grid search.

```python
import pandas as pd
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.linear_model import ElasticNet
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import GridSearchCV
```

### Create pandas DataFrame for training data
```python
taxi = pd.read_csv(
    'https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2020-05.csv',
    parse_dates=['tpep_pickup_datetime', 'tpep_dropoff_datetime']
)
```

### Specify feature and label column names
```python
raw_features = [
    'tpep_pickup_datetime',
    'passenger_count',
    'tip_amount',
    'fare_amount',
]
features = [
    'pickup_weekday',
    'pickup_weekofyear',
    'pickup_hour',
    'pickup_week_hour',
    'pickup_minute',
    'passenger_count',
]
label = 'tip_fraction'
```

### Clean data
```python
def prep_df(taxi_df):
    '''
    Generate features from a raw taxi dataframe.
    '''
    df = taxi_df[taxi_df.fare_amount > 0][raw_features].copy()  # avoid divide-by-zero
    df[label] = df.tip_amount / df.fare_amount

    df['pickup_weekday'] = df.tpep_pickup_datetime.dt.isocalendar().day
    df['pickup_weekofyear'] = df.tpep_pickup_datetime.dt.isocalendar().week
    df['pickup_hour'] = df.tpep_pickup_datetime.dt.hour
    df['pickup_week_hour'] = (df.pickup_weekday * 24) + df.pickup_hour
    df['pickup_minute'] = df.tpep_pickup_datetime.dt.minute
    df = df[features + [label]].astype(float).fillna(-1)

    return df

taxi_feat = prep_df(taxi)
```

### Set up training grid
```python
pipeline = Pipeline(steps=[
    ('scale', StandardScaler()),
    ('clf', ElasticNet(normalize=False, max_iter=100, l1_ratio=0)),
])

params = {
    'clf__l1_ratio': np.arange(0, 1.1, 0.1),
    'clf__alpha': [0, 0.5, 1, 2],
}

grid_search = GridSearchCV(
    pipeline,
    params,
    cv=3,
    n_jobs=-1,
    verbose=1,
    scoring='neg_mean_squared_error',
)
```

### Run training

To execute the grid search in Dask we need to run inside a context manager for a Joblib backend. Besides that, we call the `grid_search.fit()` method the same way as you would when using scikit-learn in a non-distributed environment. When you run this cell, watch the Dask Dashboard to see the progress.

```python
import joblib

with joblib.parallel_backend('dask'):
    _ = grid_search.fit(
        taxi_sample[features],
        taxi_sample[label],
    )
```
Note that using the Dask Joblib backend requires sending the DataFrame through the scheduler to all the workers, so make sure your scheduler has enough RAM to hold your dataset! If your data is large and you need to handle it with Dask objects, read on to the next section.


## Dask-ML for large data and/or large parameter spaces

This version accelerates the grid search by using Dask DataFrames and Dask-ML's `GridSearchCV` class. [Dask-ML](https://ml.dask.org/) is a package in the Dask ecosystem that has its own parallel implementations of machine learning algorithms, written in a familiar scikit-learn-like API. This includes `GridSearchCV` and other hyperparameter search options. To use it, we load our data into a Dask DataFrame and use Dask ML’s preprocessing and model selection classes.

### Create Dask DataFrame for training data

```python
import dask.dataframe as dd

taxi_dd = dd.read_csv(
    's3://nyc-tlc/trip data/yellow_tripdata_2020-05.csv',
    parse_dates=['tpep_pickup_datetime', 'tpep_dropoff_datetime'],
    storage_options={'anon': True},
    assume_missing=True,
)
```

### Clean data
```python
def prep_df(taxi_df):
    '''
    Generate features from a raw taxi dataframe.
    '''
    df = taxi_df[taxi_df.fare_amount > 0][raw_features].copy()  # avoid divide-by-zero
    df[label] = df.tip_amount / df.fare_amount

    df['pickup_weekday'] = df.tpep_pickup_datetime.dt.isocalendar().day
    df['pickup_weekofyear'] = df.tpep_pickup_datetime.dt.isocalendar().week
    df['pickup_hour'] = df.tpep_pickup_datetime.dt.hour
    df['pickup_week_hour'] = (df.pickup_weekday * 24) + df.pickup_hour
    df['pickup_minute'] = df.tpep_pickup_datetime.dt.minute
    df = df[features + [label]].astype(float).fillna(-1)

    return df

taxi_feat_dd = prep_df(taxi_dd)
```

### Set up training grid

```python
import numpy as np
from dask_ml.pipeline import Pipeline
from sklearn.linear_model import ElasticNet
from dask_ml.preprocessing import StandardScaler
from dask_ml.metrics import mean_squared_error
from dask_ml.model_selection import GridSearchCV
```

Notice that the only thing that changed in the above imports is swapping out `sklearn` for `dask_ml`. We are still using scikit-learn's `ElasticNet` class in this case. This will use Dask to do the pre-processing and grid search work, but use scikit-learn for the model fitting. This means that within a given Dask worker, the processed training dataset will be pulled down to a pandas DataFrame. In most cases, this is probably okay because the data will be small after processing. If the data is still too large, you can use one of Dask-ML's estimators, such as [LinearRegression](https://ml.dask.org/glm.html).

```python
pipeline = Pipeline(steps=[
    ('scale', StandardScaler()),
    ('clf', ElasticNet(normalize=False, max_iter=100, l1_ratio=0)),
])

params = {
    'clf__l1_ratio': np.arange(0, 1.1, 0.1),
    'clf__alpha': [0, 0.5, 1, 2],
}

grid_search = GridSearchCV(
    pipeline,
    params,
    cv=3,
    scoring='neg_mean_squared_error',
)
```
### Run training
Now we can run the grid search using the `grid_search` object defined above. Hint: it works the same way as scikit-learn’s `GridSearchCV` class!

```python
_ = grid_search.fit(
    taxi_sample_dd[features],
    taxi_sample_dd[label],
)
```

These are a couple of ways to speed up grid search for machine learning with Dask. Check out our [other examples](<docs/Examples/MachineLearning/xgboost-training.md>) for more ways to make your machine learning faster on Saturn Cloud!