# Create Rolling Averages with Dask

> "I need to calculate a rolling average of a numerical column, in time series data. In pandas, I can do this with rolling(x).mean() with sorted values, but what do I do in Dask, with distributed data?"

* Sort by index within AND across partitions
* Know when to compute (convert to pandas DF) or persist (process computations on cluster)
* Run calculations, with attention to our need to cross partitions correctly.

This example will walk you through these specific points, and demonstrate how it's done. We'll use New York City taxi trip data, and get the 30-day rolling average of base fare prices, for our example.

## Single Node
```python

import dask

timeseries = dask.datasets.timeseries()
timeseries.rolling('1D').mean().compute()
```

## Cluster

```python
from dask_saturn import SaturnCluster
from dask.distributed import Client

cluster = SaturnCluster(
    scheduler_size='medium',
    worker_size='xlarge',
    n_workers=3,
    nthreads=4,
)
client = Client(cluster)
client
```

```python
import s3fs

s3 = s3fs.S3FileSystem(anon=True)
files_2019 = 's3://nyc-tlc/trip data/yellow_tripdata_2019-*.csv'
s3.glob(files_2019)
```

```python
import dask.dataframe as dd

taxi = dd.read_csv(
    files_2019,
    parse_dates=['tpep_pickup_datetime', 'tpep_dropoff_datetime'],
    storage_options={'anon': True},
    assume_missing=True,
)
```

```python
taxi = taxi.set_index("tpep_pickup_datetime")
from dask.distributed import wait

taxi = taxi["2019-01-01": "2020-01-01"]
taxi = taxi.persist()
_ = wait(taxi)
```

```python
rolling_fares = taxi.fare_amount.rolling('30D').mean()
rolling_fares_df = rolling_fares.to_frame(name="fare_amount_rolled")
type(rolling_fares_df)
```

```python
taxi_new = taxi.join(rolling_fares_df, how='outer')
```