# Prefect Cloud Concepts:|
|`DaskExecutor` with a Saturn Dask Cluster                     | Saturn Dask cluster     | yes                                        |
|`DaskExecutor` with a Dask `LocalCluster`                     | local Dask cluster      | no                                         |
|`LocalExecutor`, `ResourceManager` with a Saturn Dask Cluster | local Python process    | yes                                        |
|`LocalExecutor` without any Dask                              | local Python process    | no                                         |

The examples below assume you've already created a simple Prefect flow and stored it in a variable `flow` and that you've already created a Prefect Cloud project called "test-project".

```python
from prefect import Flow, Task

PREFECT_CLOUD_PROJECT_NAME = "test-project"

@task
def multiply(x, y):
    return x * y

@task
def subtract(x, y):
    return x - y

with Flow("test-flow") as flow:
    a = multiply(10, 4.5)
    b = subtract(a, 2.8)
```

> NOTE: When options below talk about the cost of startup time for Dask clusters, those are worst-case expected startup times. You can eliminate this cost by starting Saturn Dask clusters in the UI before running a flow. See [the FAQs](#why-does-it-sometimes-take-a-few-minutes-for-my-flow-to-start-executing) for details.

### `DaskExecutor` with a Saturn Dask Cluster

This is the default option when using `prefect-saturn`. In this setup, all tasks in your flow will be converted to Dask tasks and submitted to the same Saturn Cloud Dask cluster.

**When you should use this**

* if your flow would benefit from parallelization
* if your flow needs to work with large datasets
* if up to 15 minutes minutes of setup time at the beginning of flow execution is acceptable

**How to use this**

```python
from prefect_saturn import PrefectCloudIntegration

# tell Saturn Cloud about the flow
integration = PrefectCloudIntegration(
    prefect_cloud_project_name=PREFECT_CLOUD_PROJECT_NAME
)
flow = integration.register_flow_with_saturn(
    flow=flow,
    dask_cluster_kwargs={
        "n_workers": 3,
        "worker_size": "xlarge"
    }
)

# tell Prefect Cloud about the flow
flow.register()
```

### `DaskExecutor` with a Dask `LocalCluster`

If your flow would benefit from parallelism but doesn't need the full power of a distributed Dask cluster, you might want to use a `LocalCluster` instead. Flows runs using this configuration will start executing sooner than those that require a distributed Dask cluster.

**When you should use this**

* if your flow would benefit from parallelization
* if your flow does not need to work with large datasets
* if up to 5 seconds of setup time at the beginning of flow execution is acceptable

**How to use this**

```python
from distributed import LocalCluster
from prefect.executors import DaskExecutor
from prefect_saturn import PrefectCloudIntegration

# tell Saturn Cloud about the flow
integration = PrefectCloudIntegration(
    prefect_cloud_project_name=PREFECT_CLOUD_PROJECT_NAME
)
flow = integration.register_flow_with_saturn(flow=flow)

# override the executor chosen by prefect-saturn
flow.executor = DaskExecutor(
    cluster_class=LocalCluster,
    cluster_kwargs={
        "n_workers": 3,
        "threads_per_worker": 2
    }
)

# tell Prefect Cloud about the flow
flow.register()
```

### `LocalExecutor`, with a `ResourceManager` creating a Saturn Dask Cluster

If your flow code uses Dask operations directly, such as creating Dask DataFrames or training models with `dask-ml`, you may want to use a prefect `ResourceManager`. In this setup, flow tasks execute in the main process in the flow run job, but tasks are able to submit work to a distributed Dask cluster.

**When you should use this**

* if your flow would benefit from parallelization
* if your flow needs to work with large datasets
* if your flow code uses Dask directly (e.g. you have tasks that manipulate Dask DataFrames)
* if up to 15 minutes of setup time at the beginning of flow execution is acceptable

**How to use this**

Prefect's <a href="https://docs.prefect.io/core/idioms/resource-manager.html" target="_blank" rel="noopener"><code>ResourceManager</code></a> is technically a special type of task within a flow, so to use this option you need to set up the resource manager when creating the flow, as shown below.

```python
import dask.array as da
import prefect

from dask_saturn import SaturnCluster
from prefect import Flow, Task, resource_manager
from prefect.executors import LocalExecutor
from prefect_saturn import PrefectCloudIntegration


PREFECT_CLOUD_PROJECT_NAME = "test-project"


@resource_manager
class _DaskCluster:
    def __init__(self, n_workers):
        self.n_workers = n_workers

    def setup(self):
        cluster = SaturnCluster(n_workers=self.n_workers)
        client = Client(cluster)

    def cleanup(self, x=None):
        pass


@task
def multiply(x, y):
    return x * y


@task
def subtract(x, y):
    return x - y


@task
def do_some_dask_stuff() -> None:
    logger = prefect.context.get("logger")
    X = da.random.random((100, 10))
    logger.info(f"mean of Dask array X is {X.mean().compute()}")


with Flow("test-flow") as flow:
    with _DaskCluster(n_workers=3) as resource:
        a = multiply(10, 4.5)
        b = subtract(a, 2.8)
        do_some_dask_stuff()


# tell Saturn Cloud about the flow
integration = PrefectCloudIntegration(
    prefect_cloud_project_name=PREFECT_CLOUD_PROJECT_NAME
)
flow = integration.register_flow_with_saturn(flow=flow)

# override the executor chosen by prefect-saturn
flow.executor = LocalExecutor()

# tell Prefect Cloud about the flow
flow.register()
```

### `LocalExecutor` without any Dask

If your flow wouldn't benefit much from parallelization, you aren't getting much benefit from using Dask. In this situation, you can reduce the total runtime of flow runs by using a `LocalExecutor`.

**When you should use this**

* if your flow would not benefit from parallelization
* if your flow does not need to work with large datasets
* if flow runs need to start executing within 5 seconds of a flow run being triggered by Prefect Cloud

**How to use this**

```python
from distributed import LocalCluster
from prefect_saturn import PrefectCloudIntegration

# tell Saturn Cloud about the flow
integration = PrefectCloudIntegration(
    prefect_cloud_project_name=PREFECT_CLOUD_PROJECT_NAME
)
flow = integration.register_flow_with_saturn(flow=flow)

# override the executor chosen by prefect-saturn
flow.executor = LocalExecutor()

# tell Prefect Cloud about the flow
flow.register()
```

## Frequently Asked Questions

### Is the Dask cluster shut down after a flow run completes?

If you use the default orchestration setup from `prefect-saturn`, the Saturn Dask cluster that Prefect Cloud flows run on is not stopped after the flow run completes.

Since all flow runs from the same flow run on the same cluster, there is a risk that one flow run might complete and stop a cluster that is still in use by another run of the same flow.

If you are confident that you won't have overlapping runs of the same flow, you can tell `prefect-saturn` to stop the cluster after a flow completes by passing `"autoclose": True` into `prefect_saturn.PrefectCloudIntegration.register_flow_with_saturn()`. See <a href="https://github.com/saturncloud/prefect-saturn#customize-dask" target="_blank" rel="noopener">the prefect-saturn docs</a> for more details.

### Why does it sometimes take a few minutes for my flow to start executing?

If you use an orchestration option that involves a Saturn Dask cluster and that Dask cluster isn't currently running when the flow run starts, Saturn will start the cluster automatically for you. This can take a few minutes.

Startup times vary based on factors like:

* how many workers you've selected and the size of those workers
* the size of the container image used for workers, and whether or not that image is already cached on the node(s) workers get scheduled onto
* time to complete any custom steps you've set up in the Saturn Cloud resource's start script

You can avoid this added setup time by starting the flow's Dask cluster directly from the Saturn UI before any flow runs start. You should also find that by default, the cluster will be left up after a flow run completes. So if you trigger two runs of the same flow in quick succession, the second flow run should be faster and shouldn't have to wait for the Dask cluster to start.
