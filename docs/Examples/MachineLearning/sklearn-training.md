# Train a Model with scikit-learn and Dask

Many data scientists use scikit-learn as the framework for running machine learning tasks. Conveniently, Dask is intentionally easy to integrate with scikit-learn and has strong API similarities in the `dask-ml` library. In this example, we'll show you how to create a machine learning pipeline that has all the convenience of scikit-learn but adds the speed and performance of Dask. For more information about dask-ml, visit <a href="https://ml.dask.org/" target='_blank' rel='noopener'>the official docs</a>.

To follow along, [create a project](<docs/Getting Started/start_project.md>) and spin up a Jupyter server. Open the Jupyter server, and then follow the instructions below.

### Set up cluster
Creating a Dask machine cluster in Saturn Cloud takes only a few clicks. To learn how to create yours, [visit our cluster setup documentation](<docs/Using Saturn Cloud/Create Cluster/create_cluster_ui.md>). Once your cluster has been created, to initialize a Dask client pointing to your cluster, you can run the following code in your Jupyter Notebook.

```python
from dask_saturn import SaturnCluster
from dask.distributed import Client

cluster = SaturnCluster()
client = Client(cluster)
client
```


### Create Dask dataframe for training data

We use the publicly available NYC Taxi dataset, which contains a lot of information about taxi rides taken in the city. The data files are hosted on a public S3 bucket, so we can read the CSVs directly from there.

Using `read_csv` from Dask takes the same form as using that function from pandas, and the arguments will be familiar to pandas users.

```python
import dask.dataframe as dd

taxi = dd.read_csv(
    's3://nyc-tlc/trip data/yellow_tripdata_2019-*.csv',
    parse_dates=['tpep_pickup_datetime', 'tpep_dropoff_datetime'],
    storage_options={'anon': True},
    assume_missing=True,
)
taxi
```

### Specify feature and label column names

We’ll generate a few features based on the pickup time and then cache/persist the DataFrame. In both frameworks, this executes all the CSV loading and preprocessing, and stores the results in RAM. The features we will use for training are:


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

Notice that this feature engineering code is exactly the same as what we do in pandas. Dask’ DataFrame API matches pandas’ API in many places.

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

### Split train and test samples
Notice that this function works the same as `sklearn.model_selection.train_test_split`! For more information about dask-ml, visit <a href="https://ml.dask.org/" target='_blank' rel='noopener'>the official docs</a>.

```python
from dask_ml.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    taxi_feat[features],
    taxi_feat[label],
    test_size=0.3,
    random_state=42
)
```
Due to Dask’s lazy evaluation, these arrays have not been computed yet. To ensure the rest of our ML code runs quickly, lets kick off computation on the cluster by calling `persist()` on the arrays. Note that there is a `dask.persist` function that accepts multiple objects rather than calling `.persist()` individually. This is helpful for objects that share upstream tasks - Dask will avoid re-computing the shared tasks. If you want to learn more about how Dask handles these sorts of tasks, visit our [documentation about parallelism in Python](<docs/Reference/dask_concepts.md>).

```python
from distributed import wait

X_train, X_test, y_train, y_test = dask.persist(
    X_train, X_test, y_train, y_test,
)
_ = wait(X_train)
```

### Run training

We’ll train a linear model to predict tip_fraction. We define a `Pipeline` to encompass both feature scaling and model training. This will be useful later when performing a grid search - notice that this is from scikit-learn, not dask-ml, but we can still use it together with dask-ml objects.

Evaluate the model against the test set using RMSE. We’ll also save out the model for later use.

```python
from sklearn.pipeline import Pipeline
from dask_ml.linear_model import LinearRegression
from dask_ml.preprocessing import StandardScaler
from dask_ml.model_selection import GridSearchCV

lr = Pipeline(steps=[
    ('scale', StandardScaler()),
    ('clf', LinearRegression(penalty='l2', max_iter=100)),
])
```

Now we are ready to train our model. Before we train, we’ll coerce our testing and training sets from `dask.dataframe` objects to `dask.array` objects. We’ll also take this chance to precompute the chunksize of our arrays.

```python
X_train_arr = X_train.to_dask_array(lengths=True)
y_train_arr = y_train.to_dask_array(lengths=True)
X_test_arr = X_test.to_dask_array(lengths=True)
y_test_arr = y_test.to_dask_array(lengths=True)

lr_fitted = lr.fit(
    X_train_arr,
    y_train_arr,
)
```

### Calculate MSE for evaluation

```python
from dask_ml.metrics import mean_squared_error

lr_preds = lr_fitted.predict(X_test_arr)
mean_squared_error(y_test_arr, lr_preds, squared=False)
```

### Save trained model object

```python
import cloudpickle

with open('/tmp/model.pkl', 'wb') as f:
    cloudpickle.dump(lr_fitted, f)
```
