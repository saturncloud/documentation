# Dask Clusters

## Create a Dask Cluster programmatically

For various reasons you may not want to use the UI to create the Dask cluster.
For instance if you have script that runs and you want the code to spin up a different sized Dask cluster depending on the situation. You
might also want to [connect to Dask externally](<docs/using-saturn-cloud/External Connect/azure_external_connect.md>) and avoid the UI entirely.
In these situations you can use Python code to create an appropriate Dask cluster.

You should first make sure your have the `dask-saturn` and `dask.distributed` Python libraries installed where you're running the code. We include these libraries installed in all our Jupyter server environments, and if you’re [building your own images](<docs/using-saturn-cloud/manage-images/build-images/create-images.md>), we strongly recommend you include these libraries as well.

You can create a Dask cluster as follows:

```python
from dask_saturn import SaturnCluster
from dask.distributed import Client

cluster = SaturnCluster()
client = Client(cluster)
client
```

### Adjusting cluster parameters

The default Dask cluster might not be sized correctly for your needs. You can adjust it using the following parameters to the `SaturnCluster()` function

* **n_workers**: how many instances to start with
* **nprocs**: the number of processes per machine (usually the number of cores)
* **nthreads**: the number of threads per process
* **scheduler_size**: the size of machine to use for the Dask scheduler
* **worker_size**: the size of machine to use for each Dask worker
* **worker_is_spot**: whether you wish to use EC2 Spot Instances, which may be less reliable but are cheaper than traditional EC2 Instances

For example, here is one setup for Dask cluster:

```python
from dask_saturn import SaturnCluster
from dask.distributed import Client

cluster = SaturnCluster(
    n_workers=10,
    worker_size='8xlarge',
    scheduler_size='2xlarge',
    nthreads=32,
    worker_is_spot=True,
)
client = Client(cluster)
client
```

This is a lot of stuff, and may be confusing! We have documentation in our Articles section that discusses things like scheduler/worker sizes and the concept of threads. You can also read the details of the <a href="https://github.com/saturncloud/dask-saturn/blob/main/dask_saturn/core.py" target='_blank' rel='noopener'>code on GitHub</a>.

To begin thinking about the sizes of machines you want, it can be very helpful to use the `describe_sizes()` function.

```python
dask_saturn.describe_sizes()

#> {'medium': 'Medium - 2 cores - 4 GB RAM',
#> 'large': 'Large - 2 cores - 16 GB RAM',
#> 'xlarge': 'XLarge - 4 cores - 32 GB RAM',
#> '2xlarge': '2XLarge - 8 cores - 64 GB RAM',
#> '4xlarge': '4XLarge - 16 cores - 128 GB RAM',
#> '8xlarge': '8XLarge - 32 cores - 256 GB RAM',
#> '12xlarge': '12XLarge - 48 cores - 384 GB RAM',
#> '16xlarge': '16XLarge - 64 cores - 512 GB RAM',
#> 'g4dnxlarge': 'T4-XLarge - 4 cores - 16 GB RAM - 1 GPU',
#> 'g4dn4xlarge': 'T4-4XLarge - 16 cores - 64 GB RAM - 1 GPU',
#> 'g4dn8xlarge': 'T4-8XLarge - 32 cores - 128 GB RAM - 1 GPU',
#> 'p32xlarge': 'V100-2XLarge - 8 cores - 61 GB RAM - 1 GPU',
#> 'p38xlarge': 'V100-8XLarge - 32 cores - 244 GB RAM - 4 GPU',
#> 'p316xlarge': 'V100-16XLarge - 64 cores - 488 GB RAM - 8 GPU'}
```
