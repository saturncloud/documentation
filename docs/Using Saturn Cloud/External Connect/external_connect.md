# Connect to Dask from Outside Saturn Cloud

What if you'd like to just connect directly from your laptop to a Dask cluster, instead of using a Jupyter server at all? Saturn Cloud lets you do this too!

## Create the Environment

In order to use this feature, you'll need several Dask and Saturn Cloud related Python packages running the same versions as the Dask cluster in Saturn Cloud.
To keep the versioning simple, we recommend you create a conda environment with the right specifications, as shown below. (Run this set of commands in your terminal.)

```bash
conda create -n dask-saturn dask=2.30.0 distributed=2.30.1 python=3.7
conda activate dask-saturn
pip install dask-saturn==0.3.0
```

Now you have the environment all set to go!

> Sometimes we find that people's local environments also have old versions of `pandas`, and this can be an issue. Check that your version of pandas is the one you want.

## Creating a Saturn Cloud resource

If you don't have a Saturn Cloud account, go to [saturncloud.io](https://saturncloud.io) and click "Start For Free" on the upper right corner. It'll ask you to create a login. Otherwise, log into Saturn Cloud. Once you have done so, you'll be brought to the Saturn Cloud resources page. Click "New Jupyter Server"

![New Jupyter server button](/images/docs/new-jupyter-server-button.jpg "doc-image")

Given the resource a name (ex: "external-connect-demo"), but you can leave all other settings as their defaults. In the future you may want to set a specific image or instance size which you can do from the resource page. Then click "Create"

![New Jupyter server options](/images/docs/new-jupyter-server-options.jpg "doc-image")

After the resource is created you'll be brought the page for it. Next, we need to add a Dask cluster to this resource. Press the **New Dask Cluster** button, which will pop up a dialog for setting the Dask cluster. Choose the size each worker, the number of workers, and other options for the Dask cluster (see [Create a Dask Cluster](<docs/Using Saturn Cloud/create_dask_cluster.md>) for details on those), then click **Create**.

![New Dask cluster options](/images/docs/new-dask-cluster-options.jpg "doc-image")

Once the Dask cluster is created you'll see it has a **Connect Externally** button, which provides instructions for making the external connection.

![Connect externally button](/images/docs/connect-externally-button.jpg "doc-image")

First, ensure that the client connecting to the Dask cluster has the appropriate libraries, in particular the version of `dask-saturn` shown by the UI. You'll also want to include `dask` and `distributed`, ideally with the same version as that in the cluster.

Next, set the `SATURN_BASE_URL` and `SATURN_TOKEN` environmental variables in the client machine to the values show in the dialog. Those let saturn know which particular Dask cluster to connect to.

Finally, from within the client machine you can then connect to the Dask cluster from Python:

```python
from dask_saturn import SaturnCluster
from dask.distributed import Client

cluster = SaturnCluster()
client = Client(cluster)
client
```

Run the chunk, and soon you'll see lines like this:

```python
#> INFO:dask-saturn:Starting cluster. Status: pending
```

This tells you that your cluster is starting up! Eventually you'll see something like:  

```python
#> INFO:dask-saturn:{'tcp://10.0.23.16:43141': {'status': 'OK'}}
```

Which is informing you that your cluster is up and ready to use. Now you can interact with it just the same way you would from a Saturn Cloud Jupyter server.

## Places to connect to Saturn Cloud

Not only can you connect to Saturn Cloud from your laptop or local machine, but you can connect from other cloud-based notebooks. Check out instructions for connecting from [Google Colab](<docs/Using Saturn Cloud/External Connect/colab_external_connect.md>), [SageMaker](<docs/Using Saturn Cloud/External Connect/sagemaker_external_connect.md>), and [Azure](<docs/Using Saturn Cloud/External Connect/azure_external_connect.md>).