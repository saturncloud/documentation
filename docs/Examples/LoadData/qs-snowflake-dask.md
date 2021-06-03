# Load Data from Snowflake into Saturn Cloud


Our users have varied needs for loading data, but Snowflake is a very popular choice for data storage. This tutorial will give you foundational information to load data from Snowflake into Saturn Cloud quickly and easily.

If you have questions about making a connection between Saturn Cloud and Snowflake, visit our [documentation about data connectivity](https://saturncloud.io/docs/getting-started/connect_data/).

This notebook shows how to connect to a Snowflake database and do large data manipulations that would require distributed computing using Dask. We will use the NYC Taxi dataset hosted in a Saturn Cloud Snowflake database.

First, we import the necessary libraries:


```python
import os
import snowflake.connector
import pandas as pd
```

> Don't worry if you see a warning about an incompatible version of pyarrow installed. This is because `snowflake.connector` relies on a older version of pyarrow for certain methods. We won't use those methods here so it's not a problem!

Next, we connect to the Snowflake database using the Snowflake connector module. Here we are loading the credentials as environment variables using the Saturn Cloud credential manager, however you could [load them in other ways too](https://www.saturncloud.io/docs/getting-started/credentials/). Be sure not to save them as plaintext in unsafe places!

## Connection Setup


```python
conn_info = {
    "account": os.environ["EXAMPLE_SNOWFLAKE_ACCOUNT"],
    "user": os.environ["EXAMPLE_SNOWFLAKE_USER"],
    "password": os.environ["EXAMPLE_SNOWFLAKE_PASSWORD"],
    "database": os.environ["TAXI_DATABASE"],
}

conn = snowflake.connector.connect(**conn_info)
```

### Snowflake Connector (pandas)

We can run queries directly on a Snowflake database and load them into a pandas DataFrame. In the following cell we run a query to determine which days had a least one taxi ride in January 2019.


```python
dates_query = """
SELECT
    DISTINCT(DATE(pickup_datetime)) as date
FROM taxi_yellow
WHERE
    pickup_datetime BETWEEN '2019-01-01' and '2019-01-31'
"""
dates = pd.read_sql(dates_query, conn)["DATE"].tolist()

print(dates[0:5])
```

<hr>

## Snowflake Connector with Dask

When data sizes exceed what can fit into a single pandas DataFrame, we can read larger datasets into Dask DataFrames.

To use Dask with Snowflake, first we import the required modules and connect to the Dask cluster on Saturn Cloud. For this code to run you need to [start the Dask cluster from the project page of Saturn Cloud](https://saturncloud.io/docs/examples/dask/create_cluster_ui/).

### Start the Cluster


```python
import dask.dataframe as dd
import dask
from dask.distributed import Client
from dask_saturn import SaturnCluster

cluster = SaturnCluster()
client = Client(cluster)
```

Next, we define a function that will query a small part of the data. Here we query information about a single day of taxi rides. We will run this query for each day separately and using all the Dask workers to run the queries and store the results. The `@dask.delayed` indicates that this function will be run over the Dask cluster. (For more information about delayed functions, check out [the Dask documentation](https://docs.dask.org/en/latest/).)


```python
@dask.delayed
def load_from_snowflake(day):
    with snowflake.connector.connect(**conn_info) as conn:
        query = f"""
        SELECT *
        FROM taxi_yellow
        WHERE
            date(pickup_datetime) = '{day}'
        """
        df = pd.read_sql(query, conn)
        # some days have no values for congestion_surcharge, this line ensures
        # that the missing data is properly filled
        df.CONGESTION_SURCHARGE = df.CONGESTION_SURCHARGE.astype("float64")
        return df
```

We create a list of all the results from running this query in a distributed way over all the days. As you can see from the `delayed_obs[:5]` call, these aren't pandas dataframes that are returned, they are Dask objects. The queries haven't actually be run yet, since Dask is lazy they won't be run until they are needed.

The list of delayed observations can be turned into a single Dask Dataframe using `.from_delayed()`. A Dask DataFrame performs just liked a pandas DataFrame, they can use similar function calls and share a similar syntax, however the Dask DataFrame is actually a collection of pandas dataframes all distributed across a Dask cluster.

## Pull Data


```python
delayed_obs = [load_from_snowflake(day) for day in dates]
delayed_obs[:5]

dask_data = dd.from_delayed(delayed_obs)
```

We can see the contents of the DataFrame by using the same `.head()` call on the Dask DataFrame as we would on the pandas DataFrame.


```python
dask_data.head()
```

To actually cause the lazy computations to run, such as when finding the sum of a column in pandas, for Dask we need to end with `.computed()` or `.persist()`


```python
dask_data["TOLLS_AMOUNT"].sum().compute()
```

<hr>
As you can see, by using Dask we are able to store data across multiple machines and run distributed calculations, all while using the same syntax as pandas. Depending on the size and type of data you are working with it may make more sense to either use pandas directly or Dask instead!
