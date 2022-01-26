# Dask Futures
Dask Futures allows you to use the syntax of the standard Python `concurrent.futures` library but to have the computations run across a distributed cluster of machines. It's an alternative to Dask Delayed which has its own particular syntax and lazy evaluation--here everything mimics the built in Python methods.

This example will show how to have Python code run in parallel using Dask Futures. This will use a Formula One lap time dataset from [kaggle](https://www.kaggle.com/rohanrao/formula-1-world-championship-1950-2020?select=lap_times.csv). We will be finding a range of lap times for a formula one driver. First we create a pandas dataframe out of csv file and filter out data for a particular driver. We create three functions: finding maximum lap time, finding minimum lap time and then getting the range out of these two functions.  

First, start the Dask cluster associated with your Saturn Cloud resource.


```python
from dask_saturn import SaturnCluster
from dask.distributed import Client

client = Client(SaturnCluster())
```

After running the above command, it's recommend that you check on the Saturn Cloud resource page that the Dask cluster as fully online before continuing. Alternatively, you can use the command `client.wait_for_workers(3)` to halt the notebook execution until all three of the workers are ready.

Next, we load the data, filter it, and define the functions:


```python
import pandas as pd

f1 = pd.read_csv(
    "s3://saturn-public-data/examples/Dask/f1_laptime.csv", storage_options={"anon": True}
)

d_20 = f1[f1["driverId"] == 20]


def max_lap(x):
    return max(x)


def min_lap(x):
    return min(x)


def range_lap(a, b):
    return a - b
```

Now let us add scalability to code above using Futures.  We have 3 tasks below , finding the maximum laptime, minimum laptime and range. We have used future's submit method, to submit each task individually. `client.submit` allows us to submit data or tasks directly to the cluster. This method returns a future object and the results stay in remote thread. To convert Future into a concrete value you call result method as shown in last line.


```python
a = client.submit(max_lap, d_20.milliseconds)
b = client.submit(min_lap, d_20.milliseconds)
c = client.submit(range_lap, a, b)
c.result()
```
