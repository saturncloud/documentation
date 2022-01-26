# Dask DataFrames
If you come across problems where you find that data is too large to fit into pandas DataFrame or computations in pandas are slow, you can transition to Dask DataFrames. Dask DataFrames mimic pandas DataFrames allow you to distribute data across a Dask cluster.

Dask DataFrame can be thought of as multiple pandas DataFames spread over multiple Dask workers. In the diagram below you can see that we have one Dask DataFrame made up of 3 pandas DataFrames, which resides across multiple machines. Pandas being single threaded needs to fit all the data in a single machine, so if the size of the data is more than the size of your machine you may come across an 'out of memory' error. Dask DataFrames on the other hand distribut the data, hence you can use much more data and run commands on it concurrently.



## Creating a Dask DataFrame

Dask DataFrames have a similar set of commands to pandas DataFrames, so creating and using them are fairly similar.

First, start the Dask cluster associated with your Saturn Cloud resource.


```python
from dask_saturn import SaturnCluster
from dask.distributed import Client

client = Client(SaturnCluster())
```

After running the above command, it's recommend that you check on the Saturn Cloud resource page that the Dask cluster as fully online before continuing. Alternatively, you can use the command `client.wait_for_workers(3)` to halt the notebook execution until all three of the workers are ready.

## Create Dask Dataframe from File

In code below, the data file is hosted on a public S3 bucket, so we can read the CSVs directly from there. Using `read_csv` from Dask takes the same form as using that function from pandas. You can also read other file formats like Parquet file, HDF files, JSON files etc. Note that Dask loads the data lazily--it won't read in the full dataset until it is used by later operations.


```python
import dask.dataframe as dd

df = dd.read_csv(
    "s3://saturn-public-data/examples/Dask/revised_house", storage_options={"anon": True}
)
```

## Create Dask Dataframe from a pandas DataFrame

You can create a Dask DataFrame from an existing pandas DataFrame using `from_pandas`. In the code below `npartitions` states how many partitions of the index we want to create. You can also use `chunksize` parameter instead which tells the number of rows per index partition to use.


```python
from dask.dataframe import from_pandas
import pandas as pd

data = [{"x": 1, "y": 2, "z": 3}, {"x": 4, "y": 5, "z": 6}]

# Creates DataFrame.
df = pd.DataFrame(data)
df1 = from_pandas(df, npartitions=1)
df1.compute()
```

## Create a Dask DataFrame from a Dask Array

You can convert a dask array to dask dataframe using `from_dask_array` method. In code below parameter `column` lists column names for DataFrame.


```python
import dask.array as da

df = dd.from_dask_array(da.zeros((9, 3), chunks=(3, 3)), columns=["x", "y", "z"])
df.compute()
```

## Example of using Dask DataFrames

The code below shows how to do group and summary operations with Dask DataFrames. Here we have a formula one laptime dataset taken from [kaggle](https://www.kaggle.com/rohanrao/formula-1-world-championship-1950-2020?select=lap_times.csv). You can see in code below that we have used group by and mean function on a Dask DataFrame the same way as we do with pandas except in the end we have added `compute()`. This is because dask is lazy and won't compute the operation until told to do so. When you use `read_csv`, Dask DataFrame is not going to read the entire data set. It is just going to read the column names and data types. Only when you do call compute function will Dask read all the data and perform computation.  


```python
import dask.dataframe as dd

f1 = dd.read_csv(
    "s3://saturn-public-data/examples/Dask/f1_laptime.csv", storage_options={"anon": True}
)
f1.groupby("driverId").milliseconds.mean().compute()
```

## Best Practices 

1. Be thoughtful about when you use the `compute` command--a powerful part of Dask is that it can be used to avoid unnecessary computations until they're needed.
2. Go simple whenever possible. Use Dask dataset when your data is large, but once you have filtered your data and reached a point where the data can be handled by pandas, use pandas.
3.Choose your partitions wisely. Your data in partition should be small enough to fit in memory but big enough to avoid large overheads during operations

See the Saturn Cloud blog on [differences in Dask and pandas](https://saturncloud.io/blog/dask-is-not-pandas/) for more detailed tips on this subject.
