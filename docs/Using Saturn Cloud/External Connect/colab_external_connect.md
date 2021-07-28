# Connect to Dask from Google Colab

Dask is a powerful Python library for running processes in parallel and over distributed systems. To get the full benefits of Dask, it's often necessary to have a set of machines all acting as Dask workers so that the computations can be spread across all of them. But setting up a system of machines that are all correctly connected in a cloud environment can be tricky—things like permissions, network connectivity, and correctly starting and stopping the instances together can make setting up Dask a complex task. One way you can avoid having to manage a Dask cluster is by having Saturn Cloud manage the Dask cluster for you—then with only a few clicks you can have a Dask cluster online and available to use.

Saturn Cloud is a platform for data scientists to easily work—allowing them to start and stop data science resources, use GPUs for machine learning, and run distributed code through Dask. Saturn Cloud is available to connect to through all major cloud providers, and even directly from running Python on a laptop. For Google Colab, you can create a notebook that uses Dask through Saturn Cloud. You could also connect from any GCP service that can host Python.

The rest of this article will walk you through the steps of running Dask from Google Colab through Saturn Cloud.

<img src="/images/docs/colab_00.jpg" alt-text="Using Saturn cloud from Google Colab" class="doc-image">

# Installing Packages

At this point we need to install a few packages. Ultimately, your goal is to get a Colab python environment that matches your Dask python environment as much as possible.  For this example, we will execute this code in the notebook

```bash
!pip install dask-saturn dask==2.30.0 distributed==2.30.0 tornado==6.1 numpy==1.20
```

## Creating a Saturn Cloud project

First, go to [saturncloud.io](https://saturncloud.io) and click "Start For Free" on the upper right corner. It'll ask you to create a login.

<img src="/images/docs/homepage.jpg" alt-text="Saturn Cloud homepage" class="doc-image">

Once you have done so, you'll be brought to the Saturn Cloud projects page. Click "Create Custom Project"

<img src="/images/docs/custom_project.jpg" alt-text="Create Saturn Cloud project" class="doc-image">

Given the project a name (ex: "dask"), but you can leave all other settings as their defaults—we'll be editing the project settings later directly from Azure. Then click "Create"

After the project is created you'll be brought to that project's page. At this point you'll need two id strings:

- **project_id** - the id for this particular project. You can get this from the url of the project page. For example: `https://app.community.saturnenterprise.io/dash/projects/1a9b1f26702c40bdb9570653ffbf882a` has the project_id: `1a9b1f26702c40bdb9570653ffbf882a`.
- **user_token** - the token that authenticates you with Saturn. Go to  <a href="https://app.community.saturnenterprise.io/api/user/token" target='_blank' rel='noopener'>https://app.community.saturnenterprise.io/api/user/token</a> and save token.  For example when I visit that URL, I see something like this `{"token": "d1d2e576c79a4d17a72f1df1feba5806"}` In this case, I want to cut and paste `d1d2e576c79a4d17a72f1df1feba5806` as the token

With these steps complete Saturn Cloud is ready for you to start using for Dask.

## Using Python with Dask

In your Colab Notebook , run the following chunk to connect to Saturn Cloud. Note that you'll need to replace the `your-project-id` with your project_id value from the "Creating a project in Saturn Cloud" section, and the `your-saturn-token` with the `user_token` from the previous section:

```bash

from dask_saturn.external import ExternalConnection
from dask_saturn import SaturnCluster
from dask.distributed import Client

conn = ExternalConnection(
    project_id="your-project-id",
    base_url="https://app.community.saturnenterprise.io/",
    saturn_token="your-saturn-token"
)

cluster = SaturnCluster(
    external_connection=conn
)
c = Client(cluster)

```

And with that you're now ready to use Dask from Colab! You can use Dask commands from your Colab notebook to have the Saturn Cloud cluster do computations for you. You can also monitor the cluster performance and schedule jobs and deployments from the Saturn Cloud app. Check out our [getting started documentation](<docs/getting_help.md>) for more guides, and consider whether our [Saturn Hosted Free, Saturn Hosted Pro, or Enterprise plan](/docs) is best for you!

You can also connect to Dask from [SageMaker](<docs/Using Saturn Cloud/External Connect/sagemaker_external_connect.md>), [Azure](<docs/Using Saturn Cloud/External Connect/azure_external_connect.md>), or [anywhere else outside of Saturn Cloud](<docs/Using Saturn Cloud/External Connect/external_connect.md>).
