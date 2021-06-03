# Use RAPIDS on a single GPU


This notebook describes a machine learning training workflow using the famous [NYC Taxi Dataset](https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page). That dataset contains information on taxi trips in New York City.

In this exercise, we attempt to answer this classification question:

> based on characteristics that can be known at the beginning of a trip, will this trip result in a high tip?

RAPIDS is a collection of libraries which enable you to take advantage of NVIDIA GPUs to accelerate machine learning workflows. This exercise uses the following RAPIDS packages to execute code on a GPU, rather than a CPU:
    
* [`cudf`](https://github.com/rapidsai/cudf): data frame manipulation, similar to `pandas`
* [`cuml`](https://github.com/rapidsai/cuml): machine learning training and evaluation, similar to `scikit-learn`



## Load data

The code below loads the data into a `cudf` data frame. This is similar to a `pandas` dataframe, but it lives in GPU memory and most operations on it are done on the GPU.


```python
import cudf

taxi = cudf.read_csv(
    "https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2019-01.csv",
    parse_dates=["tpep_pickup_datetime", "tpep_dropoff_datetime"],
)
```

Many dataframe operations that you would execute on a `pandas` dataframe also work for a `cudf` dataframe:


```python
len(taxi)
```


```python
taxi.head()
```

## Train model

Now that the data have been prepped, it's time to build a model!

For this task, we'll use the `RandomForestClassifier` from `cuml`. If you've never used a random forest or need a refresher, consult ["Forests of randomized trees"](https://scikit-learn.org/stable/modules/ensemble.html#forest) in the `scikit-learn` documentation. We cast to 32-bit types for compatibility with older versions of `cuml`.


```python
X = taxi[["PULocationID", "DOLocationID", "passenger_count"]].astype("float32").fillna(-1)
y = (taxi["tip_amount"] > 1).astype("int32")
```


```python
from cuml.ensemble import RandomForestClassifier

rfc = RandomForestClassifier(n_estimators=100)
```


```python
_ = rfc.fit(X, y)
```

## Calculate metrics
We'll use another month of taxi data for the test set and calculate the AUC score


```python
taxi_test = cudf.read_csv(
    "https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2019-02.csv",
    parse_dates=["tpep_pickup_datetime", "tpep_dropoff_datetime"],
)

X_test = taxi_test[["PULocationID", "DOLocationID", "passenger_count"]].astype("float32").fillna(-1)
y_test = (taxi_test["tip_amount"] > 1).astype("int32")
```


```python
from cuml.metrics import roc_auc_score

preds = rfc.predict_proba(X_test)[1]
roc_auc_score(y_test, preds)
```
