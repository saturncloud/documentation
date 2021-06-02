# Create a Dask Cluster in Jupyter

In Saturn Cloud, Dask clusters are groups of worker machines created for specific purposes, such as to use with an existing Jupyter server or to power a scheduled Prefect job.

To start a Dask cluster, you may either set it up inside the UI, or create it programmatically with code in Jupyter. This tutorial will show you how to do it programmatically.

First, no matter how you are creating your Dask Cluster, you must start a project. [You may need to create it if you haven't yet](<docs/Getting Started/start_project.md>).

## Getting Started

Click the green arrow to start your Jupyter server. 
<img src="/images/docs/startjupyter.png" alt="Project page in Saturn Cloud UI" class="doc-image">  

Click "Jupyter Lab" when it is ready to use. Open a python Notebook in the Jupyter Lab interface.
<img src="/images/docs/jlab.png" alt="View of Jupyter Server card when running" class="doc-image">

### Check Libraries
You should first make sure your have the `dask-saturn` and `dask.distributed` Python libraries installed inside your instance. (We include these libraries installed in all our environments, and if you’re [building your own images](<docs/Using Saturn Cloud/images.md>), we strongly recommend you include these libraries as well.) In a Jupyter notebook chunk, you can do this by running the following code.

```python
import dask_saturn
from dask.distributed import Client
```

> If it tells you either library is not present, then you will need to shut down your instance and add `dask-saturn` or `dask` to your start script or image. It won't work to just install it on your Jupyter server, because the worker machines you're going to create will not get that instruction.

### Select Parameters
Now you are ready to set up a cluster. No other libraries are required. We need to use the `SaturnCluster` class to create a cluster, and connect to it. This takes several arguments that you'll want to review.

* **n_workers**: how many instances to start with
* **nprocs**: the number of processes per machine (usually the number of cores)
* **nthreads**: the number of threads per process
* **scheduler_size**: the size of machine to use for the Dask scheduler
* **worker_size**: the size of machine to use for each Dask worker
* **worker_is_spot**: whether you wish to use EC2 Spot Instances, which may be less reliable but are cheaper than traditional EC2 Instances

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

### Create Cluster/SaturnCluster Object

After you decide what sort of cluster and how many machines you want, you'll run code something like this in your notebook.

```python
cluster = dask_saturn.SaturnCluster(
    n_workers=10,
    worker_size='8xlarge',
    scheduler_size='2xlarge',
    nthreads=32,
    worker_is_spot=True,
)
```
> If you chose the parameters of your Cluster in the UI, you can run `SaturnCluster` with no arguments at all, because Saturn Cloud already knows what your cluster should look like.
> Eg., `cluster = dask_saturn.SaturnCluster()`

Once you have run this, you'll see a periodic status message that the cluster creation is pending.

```python
#> INFO:dask-saturn:Starting cluster. Status: pending
#> INFO:dask-saturn:Starting cluster. Status: pending
```

When the cluster creation is complete, you'll see a notification.

```python
#> INFO:dask-saturn:Cluster is ready
#> INFO:dask-saturn:Registering default plugins
#> INFO:dask-saturn:{}
```

### Connect to Dask
Now the "cluster" object (you can give it any name you like) exists. You still need to connect it to Dask, however, which requires one more step.

```python
from dask.distributed import Client # You already imported this when testing your libraries, but don't forget!

client = Client(cluster)
client
```
Now you'll be able to see the parameters of the Dask distributed client you've just set up. Your Dask cluster is ready to use!


<img src="/images/docs/client.png" alt="View of Cluster parameters widget inside Jupyter notebook" class="doc-image">
