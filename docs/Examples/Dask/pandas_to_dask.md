# Convert pandas Code to Dask

You can incorporate Dask functionality in your existing pandas code in a number of different ways, all without making major changes or refactoring a whole lot of code.

This tutorial assumes you have a basic understanding of Dask concepts. If you need more information about that, visit [our Dask Concepts documentation first](<docs/Reference/dask_concepts.md>).

***

## Use Delayed Functions
If you are starting with a pandas dataframe, but want to parallelize the code without changing that data type, you can create delayed functions.  To change your custom code to run in a delayed fashion, just apply `@dask.delayed` decoration on your custom functions, as shown below. 

```python
import dask 
import pandas as pd

@dask.delayed
def testfn(df):
    df1 = df.groupby("PULocationID").trip_distance.mean()
    return df1
```
This will return a "Delayed" object, and you'll run `.compute()` to trigger evaluation. The `@dask.delayed` decorator is the only thing that needs to be added.

```python
testfn(df).compute()
```

This function happens to include pandas code, but you can include any combinations of python code you like. The dataframe object continues to be a pandas Dataframe throughout.

Using these functions lets you create delayed objects that you can run later, in parallel, increasing the efficiency and speed of your job.

***

## Use Dask Dataframes
Another option is to switch from your pandas Dataframe objects to Dask Dataframes, which is what we’ll do here. This takes you from one discrete data object to a distributed data object, meaning your data can now be stored in partitions across the cluster of workers. Literally, your Dask Dataframe is a collection of smaller pandas Dataframes that are distributed across your cluster. For the most part, you can use pandas methods on this Dask dataframe, as shown below.

You might create a Dask Dataframe by:

* Converting an existing pandas Dataframe: `dask.dataframe.from_pandas()`
* Loading data directly into a Dask Dataframe: for example, `dask.dataframe.read_csv()`
For the latter, see our page about [loading data with flat files](<docs/Examples/LoadData/load_data_flat.md>).

### Groupby
Here we’ll group the data by a column, then extract the mean of another column. All that is different with Dask is that we run `.compute()` at the end so that computation is triggered and results returned.

```python
pandasDF.groupby("PULocationID").trip_distance.mean()
```

```python
daskDF.groupby("PULocationID").trip_distance.mean().compute()
```

### Analyze a Column
What if we don’t group, but just calculate a single metric of a column? Same situation, our code snippets are identical with the addition of `.compute()`.

```python
pandasDF[["trip_distance"]].mean()
```

```python
daskDF[["trip_distance"]].mean().compute()
```

### Drop Duplicates
Here’s one that we need a lot, and which requires looking across all the data – dropping duplicates. Dask has this too!

```python
pandasDF.drop_duplicates("passenger_count")
```

```python
daskDF.drop_duplicates("passenger_count").compute()
```

To learn more about how Dask Dataframes correspond to the pandas API, see <a href="https://docs.dask.org/en/latest/dataframe-api.html" target='_blank' rel='noopener'>the official Dask documentation</a>.