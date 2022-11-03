# Load Data From S3 Buckets|
| AWS Access Key ID   |  Environment Variable  | `aws-access-key-id` | `AWS_ACCESS_KEY_ID`
| AWS Secret Access Key | Environment Variable  | `aws-secret-access-key`  | `AWS_SECRET_ACCESS_KEY`
| AWS Default Region  | Environment Variable  | `aws-default-region`  | `AWS_DEFAULT_REGION`

Copy the values from your AWS console into the *Value* section of the credential creation form. The credential names are recommendations; feel free to change them as needed for your workflow. You must, however, use the provided *Variable Names* for S3 to connect correctly.

With this complete, your S3 credentials will be accessible by Saturn Cloud resources! You will need to restart any Jupyter Server or Dask Clusters for the credentials to populate to those resources.

<a id='connect-via-s3fs'></a>

### Connect to Data Via `s3fs`
#### Set Up the Connection
Normally, `s3fs` will automatically seek your AWS credentials from the environment. Since you have followed our instructions above for adding and saving credentials, this will work for you! 

If you don't have credentials and are accessing a public repository, set `anon=True` in the `s3fs.S3FileSystem()` call.


```python
import s3fs

s3 = s3fs.S3FileSystem(anon=False)
```

At this point, you can reference the `s3` handle and look at the contents of your S3 bucket as if it were a local file system. For examples, you can visit the <a href="https://s3fs.readthedocs.io/en/latest/#examples" target='_blank' rel='noopener'>s3fs documentation</a> where they show multiple ways to interact with files like this.

#### Load a Parquet file using pandas
This approach just uses routine pandas syntax.


```python
import pandas as pd

file = "saturn-public-data/nyc-taxi/data/yellow_tripdata_2019-01.parquet
with s3.open(file, mode="rb") as f:
    df = pd.read_parquet(f)
```

For small files, this approach will work fine. For large or multiple files, we recommend using Dask, as described next.

#### Load a Parquet file using Dask
This syntax is the same as pandas, but produces a distributed data object.


```python
import dask.dataframe as dd

file = "saturn-public-data/nyc-taxi/data/yellow_tripdata_2019-01.parquet"
with s3.open(file, mode="rb") as f:
    df = dd.read_parquet(f)
```

#### Load a folder of Parquet files using Dask
Dask can read and load a whole folder of files if they are formatted the same, using glob syntax.


```python
files = s3.glob("s3://saturn-public-data/nyc-taxi/data/yellow_tripdata_2019-*.parquet")
taxi = dd.read_parquet(
    files,
    storage_options={"anon": False},
    assume_missing=True,
)
```
