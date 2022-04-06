# Use Bodo.ai and iPyParallel in a Jupyter Notebook
This example shows how to do feature engineering and train models using the HPC-like platform Bodo using a notebook on a single node.
New York taxi data is used to predict how much a tip each driver will get. Both the feature engineering
and model training are parallelized across multiple cores using Bodo. This can be a straightforward way to
make Python code run faster that it would otherwise without requiring much change to the code.

The Bodo framework knows when to parallelize code based on the `%%px` at the start of cells and `@bodo.jit` function decorators. Removing those and restarting the kernel will run the code without Bodo.

**The Bodo parallel cluster in this example runs within the same Saturn Cloud resource as the notebook.** Thus, to increase the performance of the Bodo cluster you only need to increase the instance size of the Jupyter Server resource it's running on.

## Start an IPyParallel cluster

Run the following code in a cell to start an IPyParallel cluster. IPyParallel is used to interactively control a cluster of IPython processes. The variable `n` is used to specify the number of clusters based on the number of CPU cores available (up to 8 in the free Bodo Community Edition).


```python
import ipyparallel as ipp
import psutil

n = min(psutil.cpu_count(logical=False), 8)

# command to create and start the local cluster
rc = ipp.Cluster(engines="mpi", n=n).start_and_connect_sync(activate=True)
```

The following code imports bodo and verifies that the IPyParallel cluster is set up correctly:


```python
%%px
import bodo

print(f"Hello World from rank {bodo.get_rank()}. Total ranks={bodo.get_size()}")
```

## Importing the Packages

These are the main packages we are going to work with:
 - Bodo to parallelize Python code automatically
 - Pandas to work with data
 - scikit-learn to build and evaluate regression models
 - xgboost for xgboost regressor model


```python
%%px
import time

import bodo
import pandas as pd
from sklearn.linear_model import Lasso, LinearRegression, Ridge, SGDRegressor
from sklearn.metrics import mean_squared_error
from sklearn.model_selection import train_test_split
```

## Load and clean data

The function below will load the taxi data from a public S3 bucket into a single pandas DataFrame.


>**Note**: Bodo works best for very large datasets, so downloading the data used in the examples can take some time. Please be patient while the datasets download for each example - you will see the speed benefits of Bodo when manipulating the downloaded data.


```python
%%px
@bodo.jit(distributed=["taxi"], cache=True)
def get_taxi_trips():
    start = time.time()
    taxi = pd.read_parquet(
        "s3://bodo-example-data/nyc-taxi/yellow_tripdata_2019_half.pq",
        )
    print("Reading time: ", time.time() - start)
    print(taxi.shape)
    return taxi


taxi = get_taxi_trips()
if bodo.get_rank()==0:
    display(taxi.head())
```

## Feature engineering

The data is then modified to have a set of features appropriate for machine learning. Other than
the Bodo decorators, this is the same function you would use for training without Bodo.


```python
%%px
@bodo.jit(distributed=["taxi_df", "df"], cache=True)
def prep_df(taxi_df):
    """
    Generate features from a raw taxi dataframe.
    """
    start = time.time()
    df = taxi_df[taxi_df.fare_amount > 0][
        "tpep_pickup_datetime", "passenger_count", "tip_amount", "fare_amount"
    ].copy()  # avoid divide-by-zero
    df["tip_fraction"] = df.tip_amount / df.fare_amount

    df["pickup_weekday"] = df.tpep_pickup_datetime.dt.weekday
    df["pickup_weekofyear"] = df.tpep_pickup_datetime.dt.weekofyear
    df["pickup_hour"] = df.tpep_pickup_datetime.dt.hour
    df["pickup_week_hour"] = (df.pickup_weekday * 24) + df.pickup_hour
    df["pickup_minute"] = df.tpep_pickup_datetime.dt.minute
    df = (
        df[
            "pickup_weekday",
            "pickup_weekofyear",
            "pickup_hour",
            "pickup_week_hour",
            "pickup_minute",
            "passenger_count",
            "tip_fraction",
        ]
        .astype(float)
        .fillna(-1)
    )
    print("Data preparation time: ", time.time() - start)
    print(df.head())
    return df


taxi_feat = prep_df(taxi)
```

The data is then split into X and y sets as well as training and testing using the scikit-learn functions. Again bodo is used to increase the speed at which it runs.


```python
%%px
@bodo.jit(distributed=["taxi_feat", "X_train", "X_test", "y_train", "y_test"])
def data_split(taxi_feat):
    X_train, X_test, y_train, y_test = train_test_split(
        taxi_feat[
            "pickup_weekday",
            "pickup_weekofyear",
            "pickup_hour",
            "pickup_week_hour",
            "pickup_minute",
            "passenger_count",
        ],
        taxi_feat["tip_fraction"],
        test_size=0.3,
        train_size=0.7,
        random_state=42,
    )
    return X_train, X_test, y_train, y_test


X_train, X_test, y_train, y_test = data_split(taxi_feat)
```

## Training the model


Below we'll train four distinct linear models on the data, all using Bodo for faster performance.
We'll train the models to predict the `tip_fraction` variable and evaluate these models against the test set using RMSE.

#### Linear regression


```python
%%px
@bodo.jit(distributed=["X_train", "y_train", "X_test", "y_test"], cache=True)
def lr_model(X_train, y_train, X_test, y_test):
    start = time.time()
    lr = LinearRegression()
    lr_fitted = lr.fit(X_train, y_train)
    print("Linear Regression fitting time: ", time.time() - start)

    start = time.time()
    lr_preds = lr_fitted.predict(X_test)
    print("Linear Regression prediction time: ", time.time() - start)
    print(mean_squared_error(y_test, lr_preds, squared=False))


lr_model(X_train, y_train, X_test, y_test)
```

#### Ridge regression


```python
%%px
@bodo.jit(distributed=["X_train", "y_train", "X_test", "y_test"])
def rr_model(X_train, y_train, X_test, y_test):
    start = time.time()
    rr = Ridge()
    rr_fitted = rr.fit(X_train, y_train)
    print("Ridge fitting time: ", time.time() - start)

    start = time.time()
    rr_preds = rr_fitted.predict(X_test)
    print("Ridge prediction time: ", time.time() - start)
    print(mean_squared_error(y_test, rr_preds, squared=False))


rr_model(X_train, y_train, X_test, y_test)
```

#### Lasso regression


```python
%%px
@bodo.jit(distributed=["X_train", "y_train", "X_test", "y_test"])
def lsr_model(X_train, y_train, X_test, y_test):
    start = time.time()
    lsr = Lasso()
    lsr_fitted = lsr.fit(X_train, y_train)
    print("Lasso fitting time: ", time.time() - start)

    start = time.time()
    lsr_preds = lsr_fitted.predict(X_test)
    print("Lasso prediction time: ", time.time() - start)
    print(mean_squared_error(y_test, lsr_preds, squared=False))


lsr_model(X_train, y_train, X_test, y_test)
```

#### SGD regressor model


```python
%%px
@bodo.jit(distributed=["X_train", "y_train", "X_test", "y_test"])
def sgdr_model(X_train, y_train, X_test, y_test):
    start = time.time()
    sgdr = SGDRegressor(max_iter=100, penalty="l2")
    sgdr_fitted = sgdr.fit(X_train, y_train)
    print("SGDRegressor fitting time: ", time.time() - start)

    start = time.time()
    sgdr_preds = sgdr_fitted.predict(X_test)
    print("SGDRegressor prediction time: ", time.time() - start)
    print(mean_squared_error(y_test, sgdr_preds, squared=False))


sgdr_model(X_train, y_train, X_test, y_test)
```

## Stopping the cluster

When you're done using the parallel cluster you can shut it down with a single command. Note that since the cluster is running within the same Jupyter Server resource as the notebook, there is no change to what hardware is running after you stop it (since the resource is still on).


```python
# command to stop the cluster
rc.cluster.stop_cluster_sync()
```
