# Use RAPIDS on a GPU cluster


We perform the same machine learning exercise as the previous notebook, except on a cluster of multiple GPUs with Dask. This exercise uses the following RAPIDS packages:
    
* [`dask-cudf`](https://docs.rapids.ai/api/cudf/stable/dask-cudf.html): distributed `cudf` dataframes using Dask
* [`cuml.dask`](https://github.com/rapidsai/cuml): distributed `cuml` algorithms using Dask

## Connect to Dask Cluster

This project has a Dask cluster defined for it, which you can start or connect to in the below cell. For more information about Dask clusters in Saturn Cloud, check out [the docs](https://www.saturncloud.io/docs/examples/dask/).



```python
from dask.distributed import Client, wait
from dask_saturn import SaturnCluster

n_workers = 3
cluster = SaturnCluster(n_workers=n_workers)
client = Client(cluster)
client.wait_for_workers(n_workers)
```

## Load data

The code below loads the data into a `dask-cudf` data frame. This is similar to a `pandas` or `cudf` dataframe, but it is distributed across GPUs in the cluster.


```python
import dask_cudf

taxi = dask_cudf.read_csv(
    "s3://nyc-tlc/trip data/yellow_tripdata_2019-01.csv",
    parse_dates=["tpep_pickup_datetime", "tpep_dropoff_datetime"],
    storage_options={"anon": True},
    assume_missing=True,
)
```

Many dataframe operations that you would execute on a `pandas` dataframe also work for a `dask-cudf` dataframe:


```python
len(taxi)
```


```python
taxi.head()
```

When we say that a `dask-cudf` dataframe is a *distributed* data frame, that means that it comprises multiple smaller `cudf` data frames. Run the following to see how many of these pieces (called "partitions") there are.


```python
taxi
```

## Train model

Now that the data have been prepped, it's time to build a model!

For this task, we'll use the `RandomForestClassifier` from `cuml.dask` (notice the `.dask`!). If you've never used a random forest or need a refresher, consult ["Forests of randomized trees"](https://scikit-learn.org/stable/modules/ensemble.html#forest) in the `scikit-learn` documentation. We cast to 32-bit types for compatibility with older versions of `cuml`.Cast to 32-bit types for compatibility with older versions of `cuml`


```python
X = taxi[["PULocationID", "DOLocationID", "passenger_count"]].astype("float32").fillna(-1)
y = (taxi["tip_amount"] > 1).astype("int32")
```

Dask performs computations in a [lazy manner](https://tutorial.dask.org/01x_lazy.html), so we persist the dataframe to perform data loading and feature processing and load into GPU memory.


```python
X, y = client.persist([X, y])
_ = wait([X, y])
```


```python
from cuml.dask.ensemble import RandomForestClassifier

rfc = RandomForestClassifier(n_estimators=100, ignore_empty_partitions=True)
```


```python
_ = rfc.fit(X, y)
```

## Calculate metrics
We'll use another month of taxi data for the test set and calculate the AUC score


```python
taxi_test = dask_cudf.read_csv(
    "s3://nyc-tlc/trip data/yellow_tripdata_2019-02.csv",
    parse_dates=["tpep_pickup_datetime", "tpep_dropoff_datetime"],
    storage_options={"anon": True},
    assume_missing=True,
)

X_test = taxi_test[["PULocationID", "DOLocationID", "passenger_count"]].astype("float32").fillna(-1)
y_test = (taxi_test["tip_amount"] > 1).astype("int32")
```


```python
from cuml.metrics import roc_auc_score

preds = rfc.predict_proba(X_test)[1]
roc_auc_score(y_test.compute(), preds.compute())
```
