# Create a Dask Cluster in the UI

In Saturn Cloud, Dask clusters are groups of worker machines created for specific purposes, such as to use with an existing Jupyter server or to power a scheduled Prefect job.

To start a Dask cluster, you may either set it up inside the UI, or create it programmatically with code in Jupyter. This tutorial will show you how to do it in the UI, which we recommend.

First, no matter how you are creating your Dask cluster, you must start a project. [You may need to create it if you haven't yet](<docs/Getting Started/start_project.md>).

## Getting Started

Open your project page, and find the card where your Jupyter server is shown. To the right of this card, you'll see the option to "Attach a Dask Cluster".
<img src="/images/docs/make_cluster.png" alt="Project page in Saturn Cloud UI" class="doc-image">  
Click the button, and you'll be taken to a form. 
<img src="/images/docs/make_cluster2.png" alt="Create Dask Cluster form in Saturn Cloud UI" class="doc-image">  

### Select Parameters
The form will ask you to make some choices about the kind of cluster you want.

* **Scheduler Size**: the size of machine to use for the Dask scheduler
* **Worker Size**: the size of machine to use for each Dask worker
* **Number of Workers**: how many instances to start with
* **Number of Worker Processes**: the number of processes per machine (usually 1)
* **Number of Worker Threads**: the number of threads per process (usually the number of cores)
* **Spot Instance checkbox**: whether you wish to use EC2 Spot Instances, which may be less reliable but are cheaper than traditional EC2 Instances. [Learn more about Spot Instances here](<docs/Reference/resources_wont_start.md#spot-instances>)!

### Create Cluster
After you have filled in the form, click "Create" to save your choices. You'll be returned to the project's page, which will now show a card for your Dask cluster.

> The Dask cluster is NOT running yet at this point. You must click the green arrow to start it.

<img src="/images/docs/make_cluster3.png" alt="Cluster card in Project page of Saturn Cloud UI" class="doc-image">  

When the startup completes, your cluster is ready to use! You'll access it from inside your Jupyter server, so your next step is to click the Jupyter Lab button and enter the workspace.

### Create SaturnCluster Object
Now you are ready to connect to your cluster. We need to use the `SaturnCluster` class, and no arguments are required because your cluster is already built. Just run the below code in your notebook.

```python
sc = dask_saturn.SaturnCluster()
```

### Connect to Dask
Now the "sc" object (you can give it any name you like) exists. You still need to connect it to Dask, however, which requires one more step.

```python
client = Client(sc)
client
```
Now you'll be able to see the parameters of the Dask distributed client you've just set up. Your Dask cluster is ready to use!

<img src="/images/docs/client.png" alt="View of Cluster parameters widget inside Jupyter notebook" class="doc-image">

## Spot Instances

[Amazon EC2 Spot Instances](https://aws.amazon.com/ec2/spot/) let you take advantage of unused EC2 capacity in the AWS cloud. Spot Instances are available at up to a 90% discount compared to On-Demand prices. In Saturn, you can use Spot Instances for Dask cluster workers. When creating your Dask cluster, please make sure to check the box field for “Spot Instance” right below the “Worker Size” field as shown below.

<img src="/images/docs/create-dask-workers-spot.png" alt="Screenshot of cluster creation form" class="doc-image">
