# Starting with Dask
Dask is an open-source framework that enables parallelization of Python code, on one machine or clusters of many machines. You can use Dask with pandas, NumPy, scikit-learn, and other Python libraries.
In this article, we’re going to show you the essentials of spinning up and using Dask clusters on Saturn Cloud. If you need more information about creating and attaching a Dask cluster to a Saturn Cloud resource see the [Saturn Cloud docs](https://saturncloud.io/docs/user-guide/create_dask_cluster/).


## Connect to a Dask Cluster

The code below imports connects the client to the Saturn Cloud Dask cluster attached to the resource. The Python library `dask_saturn` is pre-installed on Saturn Cloud resources and used for connecting to Saturn Cloud Dask clusters. By default this will start a Dask cluster with the same settings that you have already set in the Saturn UI (and the specific number of workers). 


```python
from dask_saturn import SaturnCluster
from dask.distributed import Client

client = Client(SaturnCluster())
```

After running the above command, it's recommended that you check on the Saturn Cloud resource page that the Dask cluster as fully online before continuing. Alternatively, you can use the command `client.wait_for_workers(3)` to halt the notebook execution until all three of the workers are ready.

You can also adjust the size of dask cluster by explicitly specifying parameters in the `SaturnCluster()` call. The additional parameters are:

* **`n_workers`:** Number of workers to provision for the cluster.
* **`worker_size`:** the size of machine to use for each Dask worker
* **`scheduler_size`:** the size of machine to use for the Dask scheduler
* **`nthreads`:** The number of threads available to each dask-worker process.
* **`worker_is_spot`:** Flag to indicate if workers should be started on Spot Instances nodes.

Once your Dask cluster is ready, you can use Dask commands in the same way you would with a local Dask cluster. Below is an example of using Dask to compute some exponents


```python
import dask


@dask.delayed
def lazy_exponent(args):
    x, y = args
    return x**y


inputs = [[1, 2], [3, 4], [5, 6], [9, 10], [11, 12]]
outputs = [lazy_exponent(i) for i in inputs]
futures = client.compute(outputs, sync=False)

results = [x.result() for x in futures]
results
```

Once you are done using Dask, you can shut down the cluster using the following command: `client.cluster.close()`.

For more on the different capabilities of Dask you can use on Saturn Cloud, check out our other examples:

* Dask Collections - use Dask to manipulate data across a distributed cluster
  * [Dask DataFrames](<docs/user-guide/examples/python/dask/collections/qs-dask-collections-dask-dataframe.md>)
  * [Dask Arrays](<docs/user-guide/examples/python/dask/collections/qs-dask-collections-dask-array.md>)
  * [Dask Bags](<docs/user-guide/examples/python/dask/collections/qs-dask-collections-dask-bag.md>)
* Dask Concurrency - parallelize your code directly
  * [Dask Delayed](<docs/user-guide/examples/python/dask/concurrency/qs-dask-concurrency-dask-delayed.md>)
  * [Dask Futures](<docs/user-guide/examples/python/dask/concurrency/qs-dask-concurrency-dask-futures.md>)
* Machine Learning - train machine learning models with multiple machines
  * [Model training](<docs/user-guide/examples/python/dask/machine-learning/qs-machine-learning-model-training.md>)
  * [Grid search](<docs/user-guide/examples/python/dask/machine-learning/qs-machine-learning-grid-search.md>)
* Special topics - other ways to use Dask on Saturn Cloud
  * [CLI calls](<docs/user-guide/examples/python/dask/special-topics/qs-special-topics-cli-calls.md>)
  * [Logging](<docs/user-guide/examples/python/dask/special-topics/qs-special-topics-logging.md>)
  * [Computing rolling averages](<docs/user-guide/examples/python/dask/special-topics/qs-special-topics-rolling-average.md>)

