# Use RAPIDS on a Single GPU


## Overview

[RAPIDS](https://rapids.ai/) is a collection of libraries that enable you to take advantage of NVIDIA GPUs to accelerate machine learning workflows. Minimal changes are required to transition from familiar pandas and scikit-learn code! For more information on RAPIDS, see ["Getting Started"](https://rapids.ai/start.html) in the RAPIDS docs.

This example describes how to run a machine learning training workflow using the famous NYC Taxi Dataset. This dataset contains information about taxi trips in New York City. For the purposes of this example, we will be looking at the yellow taxi data from January 2019.

We will use this data to answer the following question:
> Based on characteristics that can be known at the beginning of a trip, will this trip result in a good tip? A tip is good if it's over 20% of the fare.

We are going to use RAPIDS to train a random forest model that takes features known at the beginning of taxi trips as inputs and tries to predict the boolean value of if the tip is >20% or not.

## Modeling Process

### Imports

This exercise uses the following RAPIDS packages to execute code on a GPU, rather than a CPU:

* [`cudf`](https://docs.rapids.ai/api/cudf/stable/): data frame manipulation, similar to pandas
* [`cuml`](https://docs.rapids.ai/api/cuml/stable/): machine learning training and evaluation, similar to scikit-learn


```python
import cudf

from cuml.ensemble import RandomForestClassifier
from cuml.metrics import roc_auc_score
from sklearn.metrics import roc_curve

import matplotlib.pyplot as plt
```

### Download and Examine the Dataset

The first thing we want to do is load in the NYC Taxi Trip dataset. The code below loads the data into a `cudf` data frame. This is similar to a pandas dataframe, but it lives in GPU memory and most operations on it are done on the GPU.


```python
taxi = cudf.read_parquet(
    "s3://saturn-public-data/nyc-taxi/data/yellow_tripdata_2019-01.parquet",
    storage_options={"anon": True},
)
```

Many dataframe operations that you would execute on a pandas dataframe, like `.head()` or `.dtypes`, also work for a `cudf` dataframe.

You can compute the length and memory usage of the dataset using the following code.


```python
num_rows = len(taxi)
memory_usage = round(taxi.memory_usage(deep=True).sum() / 1e9, 2)
print(f"Num rows: {num_rows}, Memory Usage: {memory_usage} GB")
```

### Preprocess the Data
The raw data we downloaded needs to be processed before we can use it to train our machine learning model. We need to do things like create a target column, add additional features, and remove unnecessary columns. We will wrap everything in a function so we can use it later when we need to prepare data for testing or implementation.


```python
def prep_df(df: cudf.DataFrame) -> cudf.DataFrame:

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

Since this is a binary classification task, before proceeding we should examine the proportion of 1s and 0s in the target. This can be done with the `value_counts()` method.



```python
taxi["target"].value_counts(normalize=True)
```

Now that the dataframe has been processed, let's check its length and size in memory again.


```python
num_rows = len(taxi)
memory_usage = round(taxi.memory_usage(deep=True).sum() / 1e9, 2)
print(f"Num rows: {num_rows}, Memory Usage: {memory_usage} GB")
```

Removing unused columns dropped the size of the training data to about one-third the size of the raw data. You can also see that the dataset lost a few rows with zero fare amounts.

### Train a Random Forest Model

Now that the data has been prepped, it's time to build a model!

For this task, we'll use the `RandomForestClassifier` from `cuml`. If you've never used a random forest or need a refresher, consult ["Forests of randomized trees"](https://scikit-learn.org/stable/modules/ensemble.html#forest) in the scikit-learn documentation.

First, we define the X and y variables for the model.


```python
X = taxi.drop(columns=["target"])
y = taxi["target"]
```

Next, we define the model with the following parameters:
- `n_estimators=100` = create a 100-tree forest
- `max_depth=10` = stop growing a tree once it contains a leaf node that is 10 levels below the root
- `n_streams=4` - create four decision trees at a time


```python
rfc = RandomForestClassifier(n_estimators=100, max_depth=10, n_streams=4)
```

Changing any of these parameters will change the training time, memory requirements, and model accuracy. Feel free to play around with these parameters!

And, finally, we train the model.


```python
_ = rfc.fit(X, y)
```

### Calculate Metrics on a Test Set 

We will use another month of taxi data for the test set and calculate the AUC score.


```python
taxi_test = cudf.read_parquet(
    "s3://saturn-public-data/nyc-taxi/data/yellow_tripdata_2019-02.parquet",
    storage_options={"anon": True},
)
```

Before creating predictions on this new dataset, it has to be transformed in exactly the way that the original training data were prepared. Thankfully you have already wrapped that transformation logic in a function!


```python
taxi_test = prep_df(taxi_test)
```

`cuml` comes with many functions for calculating metrics that describe how well a model's predictions match the actual values. This tutorial uses the `roc_auc_score()` to evaluate the model. This metric measures the area under the receiver operating characteristic curve. Values closer to 1.0 are desirable.


```python
X_test = taxi_test.drop(columns=["target"])
y_test = taxi_test["target"]

preds = rfc.predict_proba(X_test)[1]
roc_auc_score(y_test, preds)
```

### Graph the ROC Curve

Finally, let's look at the ROC curve. `cuml` does not have a ROC curve function, so we convert the target column and predictions to numpy arrays and use the `sklearn` `roc_curve` function.


```python
y_test.to_numpy()
```


```python
fpr, tpr, _ = roc_curve(y_test.to_numpy(), preds.to_numpy())

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

The graph shows that our results were only moderately better than random chance. It is possible that tuning hyperparameters and giving the model additional features and training data will improve this outcome.

## Conclusion
As you can see, RAPIDS is great for making models like random forest train faster with a GPU. Since it mimics pandas, you can convert existing code to RAPIDS for an immediate performance boost. These techniques are especially helpful for situations like this NYC taxi dataset where the data is potentially quite large. 

At some point, however, a single GPU may not be powerful enough for your problem, and thus you will need to use multiple GPUs at once. To do so, check out our example on using [using a GPU cluster](<docs/user-guide/examples/python/rapids/qs-02-rapids-gpu-cluster.md>)!
