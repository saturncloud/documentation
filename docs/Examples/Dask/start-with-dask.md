# Start Using Dask

Dask is an open-source framework that enables parallelization of Python code, on one machine or clusters or many machines. You can use Dask with pandas, NumPy, scikit-learn, and other Python libraries. 

For machine learning, there are a couple of areas where Dask parallelization might be useful for making our work faster and better.
* Loading and handling large datasets (especially if they are too large to hold in memory)
* Running time or computation heavy tasks at the same time, quickly (for example, think of hyperparameter tuning or ensembled models)

In this article, we're going to show you the essentials of applying Dask parallelization to your code. If you need more information about the underlying concepts of parallelization with Dask, [visit our deeper dive article](<docs/Reference/dask_concepts.md>).


## Connect to a Cluster

This code imports the Dask libraries and connects to a preexisting Saturn Cloud Dask cluster. If you need help getting this set up, [visit our documentation about Clusters](<docs/Using Saturn Cloud/create_dask_cluster.md>).


```python
import dask
from dask_saturn import SaturnCluster
from dask.distributed import Client

cluster = SaturnCluster()
client = Client(cluster)

#>     [2021-02-19 23:03:10] INFO - dask-saturn | Cluster is ready
#>     [2021-02-19 23:03:10] INFO - dask-saturn | Registering default plugins
#>     [2021-02-19 23:03:11] INFO - dask-saturn | {'tcp://10.0.26.85:42489': {'status': 'repeat'}}
```

## Write a Function

The `@dask.delayed` decorator indicates this will run lazily, and will apply parallelism wherever possible. You can include any kind of Python code or calculations inside this function, not just exponents.

```python
@dask.delayed
def lazy_exponent(args):
    x, y = args
    return x ** y
```

Because this is a delayed function, we can now run it across a distributed set of workers using the `client` functions. In this case we take this list of pairs of numbers and will iterate through them with our function using `.map()`. The function is run one time on each set of numbers, using the cluster's computing resources. These runs of the function will happen simultaneously, as long as the cluster has enough resources.

## Run the Function in Parallel

```python
inputs = [[1,2], [3,4], [5,6], [9, 10], [11, 12]]

example_future = client.map(lazy_exponent, inputs)
futures_gathered = client.gather(example_future)
futures_computed = client.compute(futures_gathered, sync=False)
```

After using `client.compute()` to actually compute the values and return them to our local environment, we can then look at the results by using `x.result()` for each element of the computed list.

## Examine Results

```python
results = [x.result() for x in futures_computed]
results

#>     [1, 81, 15625, 3486784401, 3138428376721]
```

This is the starting point for applying parallelism to your Python code. Any function, with any underlying computations involved, can likely be made parallel using this technique. If you'd like to try some more complex applications of Dask, [visit our other tutorials](/docs)! 


## Housekeeping
When you're done, to avoid excess expenses, you can stop your cluster if you would like to.

```python
cluster.close()
```
