# Connect to Dask from Azure

Dask is a powerful Python library for running processes in parallel and over distributed systems. To get the full benefits of Dask, it's often necessary to have a set of machines all acting as Dask workers so that the computations can be spread across all of them. But setting up a system of machines that are all correctly connected in a cloud environment like Azure can be tricky—things like permissions, network connectivity, and correctly starting and stopping the instances together can make setting up Dask a complex task. One way you can avoid having to manage a Dask cluster when using Azure is by having Saturn Cloud manage the Dask cluster for you—then with only a few clicks you can have a Dask cluster online and available to use.

Saturn Cloud is a platform for data scientists to easily work—allowing them to start and stop data science resources, use GPUs for machine learning, and run distributed code through Dask. Saturn Cloud is available to connect to through all major cloud providers, and even directly from running Python on a laptop. For Azure, you can use the Azure Machine Learning Workspace to create a notebook that uses Dask through Saturn Cloud. You could also connect from an Azure virtual machine or any other Azure resource that can host Python. Your Python code running in Azure will connect to the Dask cluster to execute the Dask commands, and then the results can be returned to your Python environment.

<img src="/images/docs/azure_external_connect_00.png" alt-text="Azure connection to Saturn Cloud" class="doc-image">

The rest of this article will walk you through the steps of running Dask from Azure through Saturn Cloud using an Azure Machine Learning Workspace.

## Preparing a notebook on Azure

This section is to create a notebook in the Azure Machine Learning Workspace that has the proper Python libraries to connect to Dask on Saturn Cloud.

If you don't already have an Azure Machine Learning Workspace you'll need to make one. In the Azure main portal click "Create a resource" and search for "Machine Learning." Choose the name, region, and other options appropriate for what you're doing. For a deeper guide on creating the workspace, see  <a href="https://docs.microsoft.com/en-us/azure/machine-learning/how-to-manage-workspace?tabs=azure-portal" target='_blank' rel='noopener'>this Azure article</a>.

<img src="/images/docs/azure_external_connect_01.png" alt-text="Azure create resource" class="doc-image">

In the Azure Machine Learning Studio connected to the workspace you just created, select "Start now" under notebooks. When first using a notebook you may be required to choose a compute instance. For the purposes of this article it isn't important (since the Dask work happens on the Dask instances not on the compute one), but depending on the task you want to do you may need a large instance.

<img src="/images/docs/azure_external_connect_02.png" alt-text="Azure create Notebook" class="doc-image">

Once inside, connect to the terminal

<img src="/images/docs/azure_external_connect_03.png" alt-text="Azure connect to terminal" class="doc-image" style="width:400px;">

At this point we need to install the correct Python version (3.8) as well as several packages used for distributed computing. Then we need to connect the Python 3.8 kernel so that the notebook knows it's there:

```bash
# (1) Install Python 3.8 and supplemental software
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt-get -y install python3.8
sudo apt-get -y install python3.8-distutils
sudo apt-get -y install python3.8-apt

# (2) Install pip
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3.8 get-pip.py

# (3) Use pip to install required Python libraries for Dask, AzureML, and Saturn Cloud
python3.8 -m pip install pandas==1.2.3 dask==2.30.0 distributed==2.30.1 saturn-client dask-saturn==0.2.2 ipykernel azureml cryptograph==2.5

# (4) connect Python 3.8 to the Notebook
ipython kernel install --name python3.8 --user
```

In order this (1) installs Python 3.8 as well as supplemental software that is needed for some of the Python packages we're using. (2) installs pip for Python 3.8 (3) installs the Python packages we need (for Dask, Saturn Cloud, and Azure ML), and (4) then registers the Python 3.8 kernel.

At this point you can create a new notebook by right clicking you username in the file directory and choosing create new file. Once you make the new notebook set the kernel as `python3.8` and you will be ready to use Dask from Saturn Cloud.

<img src="/images/docs/azure_external_connect_04.png" alt-text="Create new file" class="doc-image" style="width:400px;">

## Creating a Saturn Cloud project

First, go to [saturncloud.io](https://saturncloud.io) and click "Start For Free" on the upper right corner. It'll ask you to create a login.

<img src="/images/docs/homepage.jpg" alt-text="Saturn Cloud homepage" class="doc-image">

Once you have done so, you'll be brought to the Saturn Cloud projects page. Click "Create Custom Project"

<img src="/images/docs/custom_project.jpg" alt-text="Create custom project" class="doc-image">

Given the project a name (ex: "azure-demo"), but you can leave all other settings as their defaults—we'll be editing the project settings later directly from Azure. Then click "Create"

<img src="/images/docs/azure_external_connect_07.jpg" alt-text="Custom project choices" class="doc-image">

After the project is created you'll be brought to that project's page. At this point you'll need two id strings:

- **project_id** - the id for this particular project. You can get this from the url of the project page. For example: `https://app.community.saturnenterprise.io/dash/projects/a753517c0d4b40b598823cb759a83f50` has the project_id: `a753517c0d4b40b598823cb759a83f50`.
- **user_id** - the id that lets saturn know you're the right user. Go to  <a href="https://app.community.saturnenterprise.io/api/user/token" target='_blank' rel='noopener'>https://app.community.saturnenterprise.io/api/user/token</a> and save the page as `token.json`, then upload that file to the Azure Machine Learning Workspace. Do not share this file with others. To upload the file click the add files button in the Azure Machine Learning Workspace.

<img src="/images/docs/azure_external_connect_08.png" alt-text="Azure add files" class="doc-image" style="width:400px;">

With these steps complete Saturn Cloud is ready for you to start using for Dask.

## Using Python with Dask

In your Python Notebook in the Azure Machine Learning Workspace, run the following chunk to connect to Saturn Cloud. Note that you'll need to replace the `project_id` with your project_id value from the "Creating a project in Saturn Cloud" section:

```bash
from dask_saturn.external import ExternalConnection
from dask_saturn import SaturnCluster
import dask_saturn
from dask.distributed import Client, progress
import json

with open('token.json') as f:
  data = json.load(f)

project_id = "PUT-YOUR-PROJECT-ID-HERE"

conn = ExternalConnection(
    project_id=project_id,
    base_url='https://app.community.saturnenterprise.io',
    saturn_token=data['token']
)
conn
```

Next we need to create the Dask cluster within the Saturn Cloud project. With this chunk we choose the number of workers, the size of the workers, the number of threads, and so on. The specific values you want here will depend on what you are using the Dask cluster for.

```bash
cluster = SaturnCluster(
    external_connection=conn,
    n_workers=4,
    worker_size='8xlarge',
    scheduler_size='2xlarge',
    nthreads=32,
    worker_is_spot=False)
```

The follow code creates the Dask client and waits for the workers to be ready to use:

```bash
client = Client(cluster)
client.wait_for_workers(4)
client
```

And with that you're now ready to use Dask from Azure! You can use Dask commands from your Azure notebook to have the Saturn Cloud cluster do computations for you. You can also monitor the cluster performance and schedule jobs and deployments from the Saturn Cloud app. Check out our [getting started documentation](<docs/Enterprise/installation.md>) for more guides, and consider whether our [Saturn Hosted Free, Saturn Hosted Pro, or Enterprise plan](/docs) is best for you!

You can also connect to Dask from [SageMaker](<docs/Using Saturn Cloud/External Connect/sagemaker_external_connect.md>), [Google Colab](<docs/Using Saturn Cloud/External Connect/colab_external_connect.md>), or [anywhere else outside of Saturn Cloud](<docs/Using Saturn Cloud/External Connect/colab_external_connect.md>).
