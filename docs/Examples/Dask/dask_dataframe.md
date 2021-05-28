# Create Dask Dataframes

A Dask Dataframe contains a number of pandas Dataframes, which are distributed across your cluster. The API to interact with these objects is very much like the pandas API.

<img src="/images/docs/dask_df.png" alt="Diagram showing how a Dask dataframe is comprised of multiple underlying pandas dataframes" class="doc-image">

This example assumes you have a basic understanding of Dask concepts. If you need more information about that, visit [our Dask Concepts documentation first](<docs/Reference/dask_concepts.md>).

For more reference about Dask Dataframes, see the <a href="https://docs.dask.org/en/latest/dataframe.html" target='_blank' rel='noopener'>official Dask documentation</a>.

***

## From CSV on Disk
```python
dask.dataframe.read_csv('filepath.csv')
```

For more details, see our page about [loading data from flat files](<docs/Examples/LoadData/load_data_flat.md>).

## From pandas Dataframe
```python
dask.dataframe.from_pandas(pandas_df)
```

For more details about adapting from pandas to Dask, [see our tutorial](<docs/Examples/Dask/pandas_to_dask.md>).

## From parquet

```python
dask.dataframe.read_parquet('s3://bucket/my-parquet-data')
```

## From JSON
This adopts the behavior of `pandas.read_json()`.

```python
dask.dataframe.read_json()
```

There are many other ways to read in data as Dask Dataframes as well. <a href="https://docs.dask.org/en/latest/dataframe-create.html" target='_blank' rel='noopener'>Dask official documentation</a> gives thorough details of the Dask Dataframes API, if you have further questions!