# Train Random Forest on GPU with RAPIDS

> This example requires a GPU cluster and GPU image on Saturn Cloud to run.

This example will demonstrate how users can run Random Forest machine learning on GPU hardware - allowing larger models to be trained, and increasing the speed of training thanks to the parallelization possible.

If you need more information about how to choose whether GPU is the right thing for your workflow, visit [our reference article](<docs/Reference/choosing_machines.md>).

## Concepts
### Random Forest
Random forest is a machine learning algorithm trusted by many data scientists for its robustness, accuracy, and scalability. The algorithm trains many decision trees through bootstrap aggregation, then predictions are made from aggregating the outputs of the trees in the forest. 

Due to its ensemble nature, a random forest is an algorithm that can be implemented in distributed computing settings. Trees can be trained in parallel across processes and machines in a cluster, resulting in significantly faster training time than using a single process.

### RAPIDS 
RAPIDS is an open-source Python framework that executes data science code on GPUs instead of CPUs. This results in huge performance gains for data science work, similar to those seen for training deep learning models. RAPIDS has interfaces for DataFrames, ML, graph analysis, and more. 

RAPIDS uses Dask to handle parallelizing to machines with multiple GPUs, as well as a cluster of machines each with one or more GPUs. The libraries we're going to use below, including `cudf` and `cuml`, are part of <a href="https://docs.rapids.ai/api" target='_blank' rel='noopener'>the RAPIDS ecosystem</a> and are designed specifically to work on GPU hardware.

***

## Workflow

You should have a GPU Dask cluster running in order to proceed. If you need help creating a cluster, we have [step by step instructions to help](<docs/Using Saturn Cloud/create_dask_cluster.md>).


Set up connection to your cluster, first. For this example, we recommend at least a 4 worker cluster of T4 4XLarge instances.

```python
from dask.distributed import Client
from dask_saturn import SaturnCluster

cluster = SaturnCluster()
client = Client(cluster)
```

### Create Dataframe
In this example, we use the publicly available NYC Taxi dataset and train a random forest regressor to predict the fare amount of a taxi ride. Taxi rides from 2017, 2018, and 2019 are used as the training set, amounting to 300,700,143 instances.

The data files are hosted on a public S3 bucket, so we can read the CSVs directly from there. The S3 bucket has all files in the same directory, so we use `s3fs` to select the files we want.

```python
import s3fs
fs = s3fs.S3FileSystem(anon=True)
files = [f"s3://{x}" for x in fs.ls('s3://nyc-tlc/trip data/')
         if 'yellow' in x and ('2019' in x or '2018' in x or '2017' in x)]
         
cols = ['VendorID', 'tpep_pickup_datetime', 'tpep_dropoff_datetime',
        'passenger_count', 'trip_distance','RatecodeID', 
        'store_and_fwd_flag', 'PULocationID', 'DOLocationID', 
        'payment_type', 'fare_amount','extra', 'mta_tax', 
        'tip_amount', 'tolls_amount', 'improvement_surcharge', 'total_amount']

import dask_cudf
taxi = dask_cudf.read_csv(files, 
                          assume_missing=True,
                          parse_dates=[1,2], 
                          usecols=cols, 
                          storage_options={'anon': True})
```

Notice that we use `dask_cudf` to load these CSV's - this allows us to get a dataframe that works on GPU hardware. For more information about the `dask-cudf` library, visit <a href="https://docs.rapids.ai/api/cudf/stable/10min.html" target='_blank' rel='noopener'>the RAPIDS website</a>.


### Feature Engineering

We’ll generate a few features based on the pickup time and then persist the DataFrame. In both frameworks, this executes all the CSV loading and preprocessing, and stores the results in RAM (in the RAPIDS case, GPU RAM). The features we will use for training are:

```python
features = ['pickup_weekday', 'pickup_hour', 'pickup_minute',
            'pickup_week_hour', 'passenger_count', 'VendorID', 
            'RatecodeID', 'store_and_fwd_flag', 'PULocationID', 
            'DOLocationID']
```

GPU hardware has different requirements for numeric types than CPU, and as a result we need to convert all float values to float32 precision for GPU computing.

```python
from dask import persist
from dask.distributed import wait

taxi['pickup_weekday'] = taxi.tpep_pickup_datetime.dt.weekday
taxi['pickup_hour'] = taxi.tpep_pickup_datetime.dt.hour
taxi['pickup_minute'] = taxi.tpep_pickup_datetime.dt.minute
taxi['pickup_week_hour'] = (taxi.pickup_weekday * 24) + taxi.pickup_hour
taxi['store_and_fwd_flag'] = (taxi.store_and_fwd_flag == 'Y').astype(float)
taxi = taxi.fillna(-1)

X = taxi[features].astype('float32')
y = taxi['total_amount']

X, y = persist(X, y)
_ = wait([X, y])
```

### Train/Test Split
Here we're using a 75/25 split.

```python
from dask_ml.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=.25)

X_train, X_test, y_train, y_test = persist(X_train, X_test, y_train, y_test)
_ = wait([X_train, X_test, y_train, y_test])
```

### Train Model

At this point, training the model takes only a couple of lines of code. We will define the Random Forest Regressor, and use the `.fit()` method to train. Notice that we are specifically using the `cuml.dask` submodule to get the `RandomForestRegressor`; this is important so that our model training is compatible with Dask.

```python
from cuml.dask.ensemble import RandomForestRegressor
cu_rf_params = {
    'n_estimators': 100,
    'max_depth': 10,
    'seed': 42,
    'n_streams': 5,
    'client': client
}

rf = RandomForestRegressor(**cu_rf_params)
rf_trained = rf.fit(X_train, y_train)
```

### Predict on Test Sample

```python
y_pred = rf_trained.predict(X_test)
```

Now you can choose the evaluation metric you prefer to examine the model performance and continue working on the model tuning!
