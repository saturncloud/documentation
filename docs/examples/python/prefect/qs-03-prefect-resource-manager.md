# Run a Prefect Cloud Data Pipeline with access to a Dask Cluster



## Overview
[Prefect Cloud](https://www.prefect.io/cloud/) is a hosted, high-availability, fault-tolerant service that handles all the orchestration responsibilities for running data pipelines. It gives you complete oversight of your workflows and makes it easy to manage them.

This example has the tasks run on a single machine, but allows the tasks to be able to run Dask commands themselves. This is appropriate for situations where each task can be parallelized within itself. If you want to do everything on a single machine without Dask, see [the single machine example](<docs/examples/python/prefect/qs-01-prefect-singlenode.md>). If you want to have the set of tasks parallelized on the Dask cluster, see [prefect-daskclusters example](<docs/examples/python/prefect/qs-02-prefect-daskclusters.md>).

![Prefect Execution](https://saturn-public-assets.s3.us-east-2.amazonaws.com/example-resources/prefect_execute.png "doc-image")


### Model Details
For this example we will be using famous NYC taxi dataset. This dataset contains information about taxi trips in New York City. For the purposes of this example, we will be looking at the yellow taxi data from January 2019. We are distributing a single task amongst dask clusters. The task includes reading the datafrom nyc taxi dataset, filtering out the rows where price is missing and then calculating amount of variation in passenger count.  

## Modelling Process

### Prerequisites
* created a Prefect Cloud account
* set up the appropriate credentials in Saturn
* set up a Prefect Cloud agent in Saturn Cloud

Details on these prerequisites can be found in [Using Prefect Cloud with Saturn Cloud](https://saturncloud.io/docs/using-saturn-cloud/prefect_cloud/).

### Environment Setup

The code in this example uses prefect for orchestration (figuring out what to do, and in what order) and Dask Cluster for execution (doing the things).

It relies on the following additional non-builtin libraries:

* [`dask-saturn`](https://github.com/saturncloud/dask-saturn): create and interact with Saturn Cloud `Dask` clusters
* [`prefect-saturn`](https://github.com/saturncloud/prefect-saturn): register Prefect flows with both Prefect Cloud and  Saturn Cloud.



```python
import os

import dask.dataframe as dd
from dask.distributed import Client
from dask_saturn import SaturnCluster
from prefect.executors import LocalExecutor
from prefect_saturn import PrefectCloudIntegration

import prefect
from prefect import Flow, resource_manager, task

PREFECT_CLOUD_PROJECT_NAME = os.environ["PREFECT_CLOUD_PROJECT_NAME"]
SATURN_USERNAME = os.environ["SATURN_USERNAME"]
```

Authenticate with Prefect Cloud.


```python
!prefect auth login --key ${PREFECT_USER_TOKEN}
```

## Create a Prefect Cloud Project

Prefect Cloud organizes flows within workspaces called "projects". Before you can register a flow with Prefect Cloud, it's necessary to create a project, if you don't have one yet.

The code below will create a new project in whatever Prefect Cloud tenant you're authenticated with. If that project already exists, this code does not have any side effects.


```python
client = prefect.Client()
client.create_project(project_name=PREFECT_CLOUD_PROJECT_NAME)
```

## Using a ResourceManager to setup a temporary Dask cluster

Resource Managers in Prefect are like context managers. When some task needs an exclusive resource, we can use Resource Manager for setting up this temporary resource. This can be later cleaned when task is done.
Here we are creating a resource manager `_DaskCluster` by using `resource_manager` decorator. The resource manager object has three tasks : `init`, `setup` and `cleanup`. 


```python
@resource_manager
class _DaskCluster:
    def __init__(self, n_workers):
        self.n_workers = n_workers

    def setup(self):
        cluster = SaturnCluster(n_workers=self.n_workers)
        client = Client(cluster)  # noqa: F841

    def cleanup(self, x=None):
        pass
```

<hr>

## Define Tasks

Prefect refers to a workload as a "flow", which comprises multiple individual things to do called "tasks". From [the Prefect docs](https://docs.prefect.io/core/concepts/tasks.html):

> A task is like a function: it optionally takes inputs, performs an action, and produces an optional result.




```python
@task
def read():
    taxi = dd.read_csv(
        "https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2019-01.csv",
        parse_dates=["tpep_pickup_datetime", "tpep_dropoff_datetime"],
    )
    df2 = taxi[taxi.passenger_count > 1]
    df3 = df2.groupby("VendorID").passenger_count.std()
    return df3
```

<hr>

## Construct a Flow

Now that all of the task logic has been defined, the next step is to compose those tasks into a "flow".

Inside our flow we have used Resource Manager `_DaskCluster`. Since task `read` needs Dask directly, we will keep the task inside `_DaskCluster`. 


```python
with Flow("prefect-resource-manager") as flow:
    with _DaskCluster(n_workers=3) as client:  # noqa: F841
        a = read()
```

## Register with Prefect Cloud


Now that the business logic of the flow is complete, we can add information that Saturn Cloud will need to know to run it.


```python
integration = PrefectCloudIntegration(prefect_cloud_project_name=PREFECT_CLOUD_PROJECT_NAME)
flow = integration.register_flow_with_saturn(flow=flow)
# override the executor chosen by prefect-saturn
flow.executor = LocalExecutor()
# tell Prefect Cloud about the flow
flow.register(project_name=PREFECT_CLOUD_PROJECT_NAME)
```

<hr>

## Run the flow

If you have scheduled your flow, it will be run once every 24 hours. You can confirm this by doing all of the following:

* If you are an admin, go to Prefect Cloud Agent page of Saturn Cloud which is at the side bar and check logs for your agent.
* Go to Prefect Cloud. If you navigate to this flow and click "Runs", you should see task statuses and and logs for this flow.

If you have not scheduled your flow or want to run the flow immediately, navigate to the flow in the Prefect Cloud UI and click "Quick Run".

```shell
prefect auth login --key ${PREFECT_USER_TOKEN}
prefect run flow \
    --name ${SATURN_USERNAME}-ticket-model-evaluation \
    --project ${PREFECT_CLOUD_PROJECT_NAME}
```



## Conclusion

In this example, you learned how to create a prefect flow and distribute a task across Dask clusters. Register this flow with Prefect Cloud.

Try changing the code above and re-running the flow eg you can train a model across multiple dask clusters

If you have existing prefect flows, try running one of them on Saturn using this notebook as a template.


