# Load Data from S3

You may choose to store your data in a cloud service option, such as S3. This means you can keep all kinds of data and have it available whenever you need it! This tutorial will give you foundational information to load data from S3 into Saturn Cloud quickly and easily.

If you have questions about making a connection between Saturn Cloud and S3, visit our [specific documentation about data connectivity](<docs/Using Saturn Cloud/connect_data.md>).
The following sections assume the credentials have been added and saved as described there.

Our users frequently find that the <a href="https://s3fs.readthedocs.io/en/latest/" target='_blank' rel='noopener'>`s3fs` library</a> is a useful place to begin when interacting with S3 data storage from inside Saturn Cloud.

## Set Up Connection
Normally, s3fs will automatically seek your AWS credentials from the environment. Since you have followed our instructions above for adding and saving credentials, this will work for you! (If you don't have credentials, and are accessing a public repository, set `anon=True` in the `s3fs.S3FileSystem()` call.)

```python
import s3fs
s3 = s3fs.S3FileSystem(anon=False)
```

At this point, you can reference the `s3` handle and look at the contents of your S3 bucket as if it were a local file system. For examples, you can visit the <a href="https://s3fs.readthedocs.io/en/latest/#examples" target='_blank' rel='noopener'>s3fs documentation</a> where they show multiple ways to interact with files like this.

## Load a CSV (pandas)
This approach just uses routine pandas syntax.

```python
import pandas as pd
import s3fs
s3 = s3fs.S3FileSystem(anon=False)

file = 'nyc-tlc/trip data/yellow_tripdata_2019-01.csv'
df = pd.read_csv(
    s3.open(file, mode='rb'),
    parse_dates=['tpep_pickup_datetime', 'tpep_dropoff_datetime']
    )
```
For small files, this approach will work fine. For large or multiple files, we recommend using Dask, as described next.

## Load a CSV (Dask)
This syntax is the same as pandas, but produces a distributed data object.
```python
import dask.dataframe as dd
import s3fs
s3 = s3fs.S3FileSystem(anon=False)

file = 'nyc-tlc/trip data/yellow_tripdata_2019-01.csv'
with s3.open(file, mode='rb') as f:
    df = dd.read_csv(f, parse_dates=['tpep_pickup_datetime', 'tpep_dropoff_datetime'])
```

## Load a Folder of CSVs (Dask)
Dask can read and load a whole folder of files if they are formatted the same, using glob syntax.

```python
import dask.dataframe as dd
import s3fs
s3 = s3fs.S3FileSystem(anon=False)

files = s3.glob('s3://nyc-tlc/trip data/yellow_tripdata_2019-*.csv')
taxi = dd.read_csv(
    files,
    parse_dates=['tpep_pickup_datetime', 'tpep_dropoff_datetime'],
    storage_options={'anon': False},
    assume_missing=True,
)
```

## Load Other Files
This is a good approach for things such as image files.

```python
import s3fs
s3 = s3fs.S3FileSystem(anon=False)

with s3.open(path) as f:
    data = f.read()
```

To find out more about loading other kinds of files, or other ways that you can interact with S3 from Saturn Cloud, check out <a href="https://s3fs.readthedocs.io/en/latest/api.html" target='_blank' rel='noopener'>the s3fs documentation</a>.
