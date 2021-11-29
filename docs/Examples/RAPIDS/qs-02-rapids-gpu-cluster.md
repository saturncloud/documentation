# Use RAPIDS on a GPU Cluster




## Overview
This example is an extension of the example of using [RAPIDS on a single GPU](<docs/Examples/RAPIDS/qs-01-rapids-single-gpu.md>) to train a random forest model on NYC taxi data, only here we will be using multiple GPUs. We will be using Dask to orchestrate the model training over multiple worker machines, each with a GPU. GPU clusers can be valuable for training models quickly and can be necessary if your data is too large to fit into a single GPU's memory. 

We recommend you skim the single GPU example first if you haven't read it already.

## Modeling Process

### Imports

Compared to the first excercise, this exercise uses a few new packages.

* [`dask_saturn`](https://github.com/saturncloud/dask-saturn) and [`dask_distributed`](http://distributed.dask.org/en/stable/): Set up and run the Dask cluster in Saturn Cloud.
* [`dask-cudf`](https://docs.rapids.ai/api/cudf/stable/basics/dask-cudf.html): Create distributed `cudf` dataframes using Dask.


```python
from dask_saturn import SaturnCluster
from dask.distributed import Client, wait
import dask_cudf

from cuml.dask.ensemble import RandomForestClassifier
from cuml.metrics import roc_auc_score
from sklearn.metrics import roc_curve

import matplotlib.pyplot as plt
```

### Start the Dask Cluster

The template resource you are running has a Dask cluster already attached to it with three workers. The `dask-saturn` code below creates two important objects: a cluster and a client.

* `cluster`: knows about and manages the scheduler and workers
    - can be used to create, resize, reconfigure, or destroy those resources
    - knows how to communicate with the scheduler, and where to find logs and diagnostic dashboards
* `client`: tells the cluster to do things
    - can send work to the cluster
    - can restart all the worker processes
    - can send data to the cluster or pull data back from the cluster


```python
n_workers = 3
cluster = SaturnCluster(n_workers=n_workers)
client = Client(cluster)
```

If you already started the Dask cluster on the resource page, then the code above will run much more quickly since it will not have to wait for the cluster to turn on.

>**Pro tip**: Create and start the cluster in the Saturn Cloud UI before opening JupyterLab if you want to get a head start!

The last command ensures the kernel waits until all the desired workers are online before continuing.


```python
client.wait_for_workers(n_workers=n_workers)
```

### Download and Examine the Dataset

The code below loads the data into a `dask-cudf` dataframe. You can interact with this data structure as if it were just a regular `cudf` dataframe, but it is actually a collection of smaller `cudf` dataframes spread across the workers in the Dask cluster.


```python
taxi = dask_cudf.read_csv(
    "s3://nyc-tlc/trip data/yellow_tripdata_2019-01.csv",
    parse_dates=["tpep_pickup_datetime", "tpep_dropoff_datetime"],
    storage_options={"anon": True},
    assume_missing=True,
).persist()

wait(taxi)
```

Many dataframe operations that you would execute on a pandas dataframe, like `.head()` and `.dtypes`, also work on a `dask-cudf` dataframe.

Simple commands might take longer than you are used to. This is due to the distributed nature of the dataframe.

You can compute the length and memory usage of the dataset using the following code.


```python
num_rows = len(taxi)
memory_usage = taxi.memory_usage(deep=True).sum().compute() / 1e9
print(f"Num rows: {num_rows}, Memory Usage: {memory_usage} GB")
```

>**Note**: Dask is lazily evaluated. The result from a computation is not computed until you ask for it. Instead, a Dask task graph for the computation is produced. Anytime you have a Dask object and you want to get the result, call `.compute()`.

When we say that a `dask-cudf` dataframe is a *distributed* dataframe, that means that it comprises multiple smaller `cudf` dataframes. Run the following to see how many of these pieces (called "partitions") there are.


```python
taxi.npartitions
```

### Preprocess the Data
This code looks nearly identical to the code you ran in the single-node RAPIDS example. `dask-cudf` translates regular `cudf` operations into the corresponding distributed operations.


```python
def prep_df(df: dask_cudf.DataFrame) -> dask_cudf.DataFrame:

    df = df[df["fare_amount"] > 0]  # to avoid a divide by zero error
    df["tip_fraction"] = df["tip_amount"] / df["fare_amount"]
    df["target"] = df["tip_fraction"] > 0.2

    df["pickup_weekday"] = df["tpep_pickup_datetime"].dt.weekday
    df["pickup_hour"] = df["tpep_pickup_datetime"].dt.hour
    df["pickup_week_hour"] = (df["pickup_weekday"] * 24) + df.pickup_hour
    df["pickup_minute"] = df["tpep_pickup_datetime"].dt.minute

    df = df[
        [
            "pickup_weekday",
            "pickup_hour",
            "pickup_week_hour",
            "pickup_minute",
            "passenger_count",
            "PULocationID",
            "DOLocationID",
            "target",
        ]
    ]

    df = df.astype("float32").fillna(-1)
    df["target"] = df["target"].astype("int32")

    return df
```


```python
taxi = prep_df(taxi)
```

Since this is a binary classification task, before proceeding we should examine the proportion of 1s and 0s in the target. Note that we add `.compute()` to ask for the results of the calculation immediately.



```python
taxi["target"].value_counts(normalize=True).compute()
```

Now that the dataframe has been processed, let's check its length and size in memory again. Again, we need to add `.compute()` in order to get the results immediately.


```python
num_rows = len(taxi)
memory_usage = taxi.memory_usage(deep=True).sum().compute() / 1e9
print(f"Num rows: {num_rows}, Memory Usage: {memory_usage} GB")
```

### Train a Random Forest Model

Now that the data has been prepped, it's time to build a model! This code is identical to the first example, except we are using the Dask version of the `cuml RandomForestClassifier`.


```python
X = taxi.drop(columns=["target"])
y = taxi["target"]

rfc = RandomForestClassifier(n_estimators=100, max_depth=10, n_streams=4)
```


```python
%%time
_ = rfc.fit(X, y)
```

As you might expect, this model takes less time to run than the single GPU example!

### Calculate Metrics on a Test Set 

We will use another month of taxi data for the test set and calculate the AUC score.


```python
taxi_test = dask_cudf.read_csv(
    "s3://nyc-tlc/trip data/yellow_tripdata_2019-02.csv",
    parse_dates=["tpep_pickup_datetime", "tpep_dropoff_datetime"],
    storage_options={"anon": True},
    assume_missing=True,
).persist()

wait(taxi_test)
```


```python
taxi_test = prep_df(taxi_test)
```

As of this writing, `cuml.metrics.roc_auc_score` does not support Dask collections as inputs. The code below uses `.compute()` to create `cudf` series instead. 


```python
X_test = taxi_test.drop(columns=["target"])
y_test = taxi_test["target"]

preds = rfc.predict_proba(X_test)[1]

y_test = y_test.compute()
preds = preds.compute()
```


```python
roc_auc_score(y_test, preds)
```

### Graph the ROC Curve


```python
fpr, tpr, _ = roc_curve(y_test.to_array(), preds.to_array())

plt.rcParams["font.size"] = "16"

fig = plt.figure(figsize=(8, 8))

plt.plot([0, 1], [0, 1], color="navy", linestyle="--")
plt.plot(fpr, tpr, color="red")
plt.legend(["Random chance", "ROC curve"])
plt.xlabel("False positive rate")
plt.ylabel("True positive rate")
plt.xlim([0, 1])
plt.ylim([0, 1])
plt.fill_between(fpr, tpr, color="yellow", alpha=0.1)
plt.show()
```

## Conclusion

By only changing a few lines of code, we went from training on a single GPU to a training on a GPU cluster! Wow! 

Feel free to play around with parameters and the volume of data. You could, for instance, read in and train on all of 2019's taxi data (`yellow_tripdata_2019-*.csv`). *Make sure you test on a different test set!*

Take a look at our other [examples](https://saturncloud.io/docs/examples/) for more resources on running models on single and multiple GPUs!
