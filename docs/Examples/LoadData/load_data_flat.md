# Load Data from Local Files

Flat files are a very common data storage option, and lots of our customers will use them at some time. This tutorial will show you how to load a dataset from a file (for example, a CSV or JSON file) on disk or on a cloud storage space into Saturn Cloud using pandas or Dask.

Before starting this, you should create a Jupyter server resource and start it. See our [startup documentation](<docs/Getting Started/start_resource.md>) if you don't know how to do this yet.

## Upload Files
If you want to place a flat file in the Saturn Cloud Jupyter server workspace, there's a simple UI option.

<img src="/images/docs/upload_file.png" alt="Jupyter Lab interface showing a red arrow pointing to the Upload button" class="doc-image">

Alternately, you can store your data in a cloud storage tool, such as S3 or Snowflake, and access it directly from there. We have [tutorials to help you work with data in those storage tools](/docs).

## Loading Files

Once files are uploaded into your workspace, you can treat them like local files.  For many files, using pandas tools will be perfectly adequate. If your data is too large to work in pandas, Dask tools are also available. For reference, here are some useful sections of pandas and Dask documentation for loading files.

* <a href="https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_csv.html" target='_blank' rel='noopener'>pandas read_csv</a>
* <a href="https://docs.dask.org/en/latest/dataframe-api.html?#dask.dataframe.read_csv" target='_blank' rel='noopener'>Dask read_csv</a>
* <a href="https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_json.html" target='_blank' rel='noopener'>pandas read_json</a>
* <a href="https://docs.dask.org/en/latest/dataframe-api.html?#dask.dataframe.read_json" target='_blank' rel='noopener'>Dask read_json</a>

These Dask options work best with one file, or with folders full of files that are formatted exactly alike (same column names, same data types, etc). They make assumptions about the data characteristics because Dask doesn't read all the data immediately, but creates a pointer to it that allows "delayed" loading later.

If your data doesn't fit this description, but you have many files to load, check out the next section.

## Dask.delayed Options

Data files aren’t always provided in a clean, tabular form that's readable with a `read_*` method from pandas or Dask. With `dask.delayed` functions, we can write a function that processes a single chunk of raw data and then tell Dask to collect these into a Dask DataFrame.

We’ll illustrate that now with the CSV files, but it's always better to use a `dd.read_*` method if your data supports it. When you need it though, `dask.delayed` is very flexible and powerful - chances are you will use it for some of your workloads.

We’ll define a function, `load_csv`, which will return a pandas DataFrame for a given  file path. In here, you could include any number of custom data processing steps that you want done at the time of data loading! Then, call this for the folder full of CSV files, and create a Dask DataFrame with `dd.from_delayed`.

```python
import pandas as pd
import dask
import dask.dataframe as dd

@dask.delayed
def load_csv(file):
    df = pd.read_csv(file, mode='rb')

    # We are selecting only the columns all the files have in common
    df = df[['Column1', 'Column3', 'Column5']]
    return df

dfs = []
for f in s3.glob('s3://bucket_name/data-file-*.csv'):
    df = load_csv(f)
    dfs.append(df)

df_delayed = dd.from_delayed(dfs)
```
