# Use a Saturn Cloud Dask Cluster Externally

If you'd like to work in your local IDE, but want to have access to Dask clusters at a click, Saturn Cloud is a great tool. This example will show you how to do a data cleaning workflow from your IDE on a remote Dask cluster.

***

This assumes you're running code from your local laptop, in an IDE such as VSCode or PySpark, or in a local Jupyter Notebook installation.

## Connect to Cluster

First, connect your IDE to the cluster. If you need help, [we have detailed instructions for this](<docs/Using Saturn Cloud/External Connect/colab_external_connect.md>). When done, you should have a `client` object in your local workspace that represents the connection to your Dask cluster. In the code below replace the `[PROJECT_ID]` and `[API_TOKEN]` from the values used to connect to your cluster.


```python
from dask_saturn.external import ExternalConnection
from dask_saturn import SaturnCluster
import dask_saturn
from dask.distributed import Client, progress

conn = ExternalConnection(
    project_id=[PROJECT_ID],
    base_url='https://app.community.saturnenterprise.io',
    saturn_token=[API_TOKEN]
)

cluster = SaturnCluster(
    external_connection=conn,
    n_workers=4,
    worker_size='4xlarge',
    scheduler_size='large')


client = Client(cluster)
client.wait_for_workers(4)
client
```

## Load Data

At this point, we can load in our dataset, which for me is a set of just over 60 CSV files in an S3 repository. The total dataset represents more than 12 million rows of purchase records, with 23 columns. For this demo, we are going to load just one file from this set.

```python
import os
import pandas as pd
import dask.dataframe as dd
from dask.distributed import wait
import s3fs

s3 = s3fs.S3FileSystem(anon=True)
s3fpath = 's3://saturn-public-data/ia_data/ia_10.csv'

iowa = dd.read_csv(
    s3fpath,
    parse_dates = ['Date'],
    engine = 'python',
    error_bad_lines = False,
    warn_bad_lines = False,
    storage_options={'anon': True},
    assume_missing=True
)

iowa = iowa.repartition(npartitions = 4)
iowa = iowa.persist()
_ = wait(iowa)
```

By using persist on the data, we can ensure that the queued tasks on the Dask Dataframe are processed, while the data object remains distributed. 

## Process Data
To demonstrate an analysis on the cluster, I'll do a couple of analyses that you might want to run for business.

The first task to do aggregations across dataframes effectively with Dask is to **set the index of the dataframe**. This lets Dask easily organize the data that is partitioned across the cluster, while still keeping it distributed. 

> This is sometimes a slow task, but it only needs to be done once.

```python
iowa = iowa.set_index("Date")
iowa = iowa.persist()
_ = wait(iowa)
```

### Rolling Average
From here, we can treat the dataframe very much like a pandas dataframe, while it remains distributed.   
We'll calculate a new series, which is the 30 day rolling average of items sold (bottles), then shape it into a dataframe.

```python
bottles_sold_roll = iowa['Bottles Sold'].rolling('30D').sum()
bottles_sold_roll = bottles_sold_roll.to_frame(name="bottles_sold_roll")
bottles_sold_roll = bottles_sold_roll.persist()
```

### Group and Summarize
For a second example of calculations over the dataset on the cluster, I'll group by store and date, and calculate the store level daily sales in dollars.

```python
iowa['Sale (Dollars)'] = iowa['Sale (Dollars)'].str.lstrip('$').astype('float')

sum_store_sales = iowa.groupby(['Date', "Store Number"])["Sale (Dollars)"].sum()
sum_store_sales = sum_store_sales.to_frame(name="sum_store_sales")
sum_store_sales = sum_store_sales.persist()
```

## Rejoin Dataframes
If you want to, from here you can rejoin those new columns to your existing data using the indices.

```python
iowa_new = dd.concat([iowa, bottles_sold_roll], axis=1)
iowa_new = iowa_new.persist()
_ = wait(iowa_new)

iowa_final = iowa_new.merge(sum_store_sales, how="left",
                            on=['Date', "Store Number"])
iowa_final = iowa_final.persist()
_ = wait(iowa_final)
```
