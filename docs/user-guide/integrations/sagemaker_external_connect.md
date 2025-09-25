# Connect to Dask from SageMaker

## Creating a Saturn Cloud resource

If you don't have a Saturn Cloud account, go to [saturncloud.io](https://saturncloud.io) and click "Start For Free" on the upper right corner. It'll ask you to create a login. Otherwise, log into Saturn Cloud. Once you have done so, you'll be brought to the Saturn Cloud resources page. Click "New Jupyter Server"

![New Jupyter server button](/images/docs/new-jupyter-server-button.webp "doc-image")

Given the resource a name (ex: "external-connect-demo"), but you can leave all other settings as their defaults. In the future you may want to set a specific image or instance size which you can do from the resource page. Then click "Create"

![New Jupyter server options](/images/docs/new-jupyter-server-options.webp "doc-image")

After the resource is created you'll be brought the page for it. Next, we need to add a Dask cluster to this resource. Press the **New Dask Cluster** button, which will pop up a dialog for setting the Dask cluster. Choose the size each worker, the number of workers, and other options for the Dask cluster (see [Create a Dask Cluster](<docs/user-guide/how-to/create_dask_cluster.md>) for details on those), then click **Create**.

![New Dask cluster options](/images/docs/new-dask-cluster-options.webp "doc-image")

Once the Dask cluster is created you'll see it has a **Connect Externally** button, which provides instructions for making the external connection.

![Connect externally button](/images/docs/connect-externally-button.webp "doc-image")

First, ensure that the client connecting to the Dask cluster has the appropriate libraries, in particular the version of `dask-saturn` shown by the UI. You'll also want to include `dask` and `distributed`, ideally with the same version as that in the cluster.

Next, set the `SATURN_BASE_URL` and `SATURN_TOKEN` environmental variables in the client machine to the values show in the dialog which let the system know which particular Saturn Cloud Dask cluster to connect to. For guidance on how to set environment variables, see our [environment variable documentation](<docs/user-guide/how-to/environment-variables.md>.

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

Which is informing you that your cluster is up and ready to use. Now you can interact with it just the same way you would from a Saturn Cloud Jupyter server. If you need help with that, please check out some of our tutorials, such as [Training a Model with Scikit-learn and Dask](<docs/user-guide/examples/python/dask/machine-learning/qs-machine-learning-model-training.md>), or the <a href="https://github.com/saturncloud/dask-saturn" target='_blank' rel='noopener'>dask-saturn API</a>. 

## Analysis!

At this point, you are able to do load data and complete whatever analysis you want. You can monitor the performance of your cluster at the link described earlier, or you can log in to Saturn Cloud and see the Dask dashboard, logs for the cluster workers, and other useful information.

You can also connect to Dask from [Google Colab](<docs/user-guide/integrations/colab_external_connect.md>), [Azure](<docs/user-guide/integrations/azure_external_connect.md>), or [anywhere else outside of Saturn Cloud](<docs/user-guide/integrations/sagemaker_external_connect.md>).
