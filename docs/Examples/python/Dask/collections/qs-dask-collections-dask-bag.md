# Dask Bags
Dask Bags are used for taking a collection of Python objects and processing them in parallel. It can be thought as a bunch of lists, and each list holds some numbers . It's like python map reduce functions. Dask Bag implements operations like map, filter, fold, and groupby on collections of generic Python objects.

Sometimes data is not structured or is semi structured and might not be handled in the most efficient way in dask dataframes, eg Dask dataframes do not support nested json files. In such cases, dask bags are very useful. You can do cleaning, data preprocession of messy data and then convert data to flat columns (like dask dataframes) for implementing complex algorithms like machine learning. 



First, start the Dask cluster associated with your Saturn Cloud resource.


```python
from dask_saturn import SaturnCluster
from dask.distributed import Client

client = Client(SaturnCluster())
```

After running the above command, it's recommend that you check on the Saturn Cloud resource page that the Dask cluster as fully online before continuing. Alternatively, you can use the command `client.wait_for_workers(3)` to halt the notebook execution until all three of the workers are ready.

## Create a Dask Bag from a list

In code below we are loading data from a simple Python iteration. Parameter `npartitions` has been set to 2. By default Dask tries to partition data into around a hundred partitions.


```python
import dask.bag as db

x = db.from_sequence(list(range(1, 9)), npartitions=2)
```

## Create a Dask Bag by loading files

Function `read_text` is used to load data directly from a single file or a bunch of files. In code below we are loading data from 2 files. 


```python
import dask.bag as db

x = db.read_text(
    [
        "s3://saturn-public-data/examples/Dask/1.json",
        "s3://saturn-public-data/examples/Dask/2.json",
    ],
    storage_options={"anon": True},
)
```

## Example using bags

In this example we will load F1 racing data and find the names of F1 drivers whose are ranked 1st through 10th. The data is available from a public URL.

The command below reads the data from the public URL, then takes the first 2 elements.


```python
import dask.bag as db
import json

b = db.read_text(
    "s3://saturn-public-data/examples/Dask/0.json", storage_options={"anon": True}
).map(json.loads)
b.take(2)
```

The data is then filtered to the name of driver those who have a rank of at most 10 and the name is pulled.


```python
result = b.filter(lambda record: record["rank"] <= 10).map(lambda record: record["name"])
result.compute()
```

## Best Practices:

Operations involve communication amongst workers (like groups in the data) can be expensive in Dask Bags. Hence if you want to perform operations which involve shuffling of data, it would be advisable to switch to DataFrame or other data type instead.  
