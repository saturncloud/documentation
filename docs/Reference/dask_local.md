# Dask Dashboard with Local Clusters

[Dask clusters](<docs/Using Saturn Cloud/Create Cluster/create_cluster_ui.md>) on Saturn Cloud enable you to scale your workloads across
a cluster of machines. There are scenarios when it is advantageous to run Dask on a single node, utilizing the CPUs as 
workers for the "cluster". This is called a `LocalCluster`. If you have run Dask on your laptop and initialized
a `Client()`, then you have run a `LocalCluster`!

Saturn Cloud has Jupyter instances with up to 4TB of RAM, so if that is enough for your workload, you can consider choosing
one of those machines and running a `LocalCluster`. Another common workload is using multiple
GPUs on the same instance with a `LocalCUDACluster`. In these cases, the Dask section of the project page will not
display your link to the Dask dashboard. This article explains how to access the Dask dashboard for local
clusters.

## Local Clusters

You can create a local cluster on a Jupyter server by leaving the arguments to the Dask `Client` blank:

```python
from dask.distributed import Client
client = Client()
```

Dask is able to take advantage of multi-GPU acceleration using packages like [RAPIDS](https://rapids.ai) and [XGBoost](https://xgboost.readthedocs.io/en/latest/tutorials/dask.html). 
When using multiple GPUs across multiple instances, you would initialize a [SaturnCluster](http://localhost:1313/docs/using-saturn-cloud/create-cluster/create_cluster_ui/). 
If you are using a single Jupyter server with multiple GPU cards on the same instance (such as a V100-8XLarge), you would initialize a `LocalCUDACluster` 
from [dask-cuda](https://docs.rapids.ai/api/dask-cuda/nightly/quickstart.html):

```python
from dask_cuda import LocalCUDACluster
from dask.distributed import Client

cluster = LocalCUDACluster()
client = Client(cluster)
```

## Accessing Dask Dashboard

Previewing the `client` object displays information about the Dask client along with a link to the Dask Dashboard:
```python
client
```

<table style="border: 2px solid white;">
<tr>
<td style="vertical-align: top; border: 0px solid white">
<h3 style="text-align: left;">Client</h3>
<ul style="text-align: left; list-style: none; margin: 0; padding: 0;">
  <li><b>Scheduler: </b>tcp://127.0.0.1:44603</li>
  <li><b>Dashboard: </b>http://127.0.0.1:8787/status</li>
</ul>
</td>
<td style="vertical-align: top; border: 0px solid white">
<h3 style="text-align: left;">Cluster</h3>
<ul style="text-align: left; list-style:none; margin: 0; padding: 0;">
  <li><b>Workers: </b>2</li>
  <li><b>Cores: </b>2</li>
  <li><b>Memory: </b>15.50 GB</li>
</ul>
</td>
</tr>
</table>

The "Dashboard" URL links to the localhost, but since this is running inside of Saturn Cloud, that link will not work. Saturn utilizes a Jupyter proxy to be able to access interfaces hosted on the server. You can copy the URL of the Jupyter window from your browser and replace `/lab/*` with `/proxy/8787/status`. For example, your Jupyter URL might be:

> https://j-aaron-proj.community.saturnenterprise.io/user/aaron/examples-cpu/lab/workspaces/examples-cpu

Then your dashboard URL would be: 

> https://j-aaron-proj.community.saturnenterprise.io/user/aaron/examples-cpu/proxy/8787/status

If you find yourself doing this often, or if you want multiple local clusters in one Jupyter Server, you can utilize the following function to display a clickable link to the dashboard:

```python
import os
from IPython.display import display, HTML

def local_dashboard_link():
    link = client.dashboard_link.replace(
        'http://127.0.0.1:', 
        "https://{}/user/{}/{}/proxy/".format(
            os.environ['SATURN_JUPYTER_BASE_DOMAIN'],
            os.environ['SATURN_USERNAME'],
            os.environ['SATURN_PROJECT_NAME'],
        )
    )

    display(HTML(
        f'<a href="{link}" target="_blank" rel="noopener">{link}</a>'
    ))

local_dashboard_link()
```

If you find that you need to scale your workloads up more, then you will want to consider a full 
[Dask cluster](<docs/Using Saturn Cloud/Create Cluster/create_cluster_ui.md>)!