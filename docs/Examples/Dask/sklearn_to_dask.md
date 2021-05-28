# Convert scikit-learn Code to Dask

This tutorial assumes you have a basic understanding of Dask concepts. If you need more information about that, visit [our Dask Concepts documentation first](<docs/Reference/dask_concepts.md>).


scikit-learn code is used for training and applying machine learning models, and Dask has a specific module called `dask-ml` that replicates the features of `scikit-learn` accelerated with parallelization. We'll demonstrate a few examples here of scikit-learn code, and the `dask-ml` library equivalents.

***

## Train-test Split

### scikit-learn
```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    dataframe[features], 
    dataframe[label], 
    test_size=0.3,
    random_state=42
)
X_train.shape, y_train.shape
X_test.shape, y_test.shape

```

### Dask
```python
from dask_ml.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    dataframe[features], 
    dataframe[label], 
    test_size=0.3,
    random_state=42
)
len(X_train), len(y_train)
len(X_test), len(y_test)
```

***

## Create Pipeline

### scikit-learn
```python
from sklearn.pipeline import Pipeline
from sklearn.linear_model import ElasticNet
from sklearn.preprocessing import StandardScaler

pipeline = Pipeline(steps=[
    ('scale', StandardScaler()),
    ('clf', ElasticNet(normalize=False, max_iter=100, l1_ratio=0)),
])
```

### Dask
```python
from sklearn.pipeline import Pipeline
from dask_ml.linear_model import LinearRegression
from dask_ml.preprocessing import StandardScaler

pipeline = Pipeline(steps=[
    ('scale', StandardScaler()),
    ('clf', LinearRegression(penalty='l2', max_iter=100)),
])
```

***
## Fit and Evaluate Model

### scikit-learn
```python
from sklearn.metrics import mean_squared_error

fitted = pipeline.fit(X_train, y_train)
preds = fitted.predict(X_test)
mean_squared_error(y_test, preds, squared=False)
```

### Dask
`Dask-ml`'s `LinearRegression` model accepts only arrays rather than dataframes, so we convert them to arrays. Similar to the relationship between a pandas dataframe and a Dask dataframe, Dask arrays are collections of numpy arrays that conform to a similar API.

With pandas dataframes, we could call `.values` to get a numpy array. Dask needs more information about chunk sizes of the underlying numpy arrays, so we need to do `.to_dask_array(lengths=True)`.

```python
X_train_arr = X_train.to_dask_array(lengths=True)
X_test_arr = X_test.to_dask_array(lengths=True)
y_train_arr = y_train.to_dask_array(lengths=True)
y_test_arr = y_test.to_dask_array(lengths=True)

```

Due to Dask's lazy evaluation, these dataframes have not been computed yet. To ensure the rest of our ML code runs quickly, let's kick off computation on the cluster by calling `.persist()` on the arrays. Note that there is a `dask.persist` function that accepts multiple objects rather than calling `.persist()` individually. This is helpful for objects that share upstream tasks - Dask will avoid re-computing the shared tasks.

```python
X_train_arr, X_test_arr, y_train_arr, y_test_arr = dask.persist(
    X_train_arr, X_test_arr, y_train_arr, y_test_arr,
)
_ = wait(X_train_arr)
```

```python
from dask_ml.metrics import mean_squared_error

pipeline_fitted = pipeline.fit(X_train_arr, y_train_arr)
pipeline_preds = pipeline_fitted.predict(X_test_arr)
mean_squared_error(y_test_arr, pipeline_preds, squared=False)
```