# Connect to Dask from Outside Saturn Cloud

What if you'd like to just connect directly from your laptop to a Dask cluster, instead of using a Jupyter server at all? Saturn Cloud lets you do this too!

## Create the Environment

In order to use this feature, you'll need several Dask and Saturn Cloud related Python packages running the same versions as the Dask cluster in Saturn Cloud.
To keep the versioning simple, we recommend you create a conda environment with the right specifications, as shown below. (Run this set of commands in your terminal.)

```bash
conda create -n dask-saturn dask=2.30.0 distributed=2.30.1 python=3.7
conda activate dask-saturn
pip install dask-saturn==0.2.2
```

Now you have the environment all set to go!

> Sometimes we find that people's local environments also have old versions of `pandas`, and this can be an issue. Check that your version of pandas is the one you want.

## Creating a Saturn Cloud project

{{% alert %}}
(If you have already created a Saturn Cloud project you can [skip this section](#connect-to-saturn-remotely).)
{{% /alert %}}

If you don't have a Saturn Cloud account, go to [saturncloud.io](https://saturncloud.io) and click "Start For Free" on the upper right corner. It'll ask you to create a login. Otherwise, log into Saturn Cloud.

<img src="/images/docs/azure_external_connect_05.png" alt-text="Saturn Cloud homepage" class="doc-image">

Once you have done so, you'll be brought to the Saturn Cloud projects page. Click "Create Custom Project"

<img src="/images/docs/custom_project.jpg" alt-text="Create custom project" class="doc-image">

Given the project a name (ex: "external-connect-demo"), but you can leave all other settings as their defaults. In the future you may want to set a specific image or instance size which you can do from the Project UI. Then click "Create"

<img src="/images/docs/azure_external_connect_07.png" alt-text="Custom project choices" class="doc-image">

After the project is created you'll be brought to that project's page. At this point you'll need three values:

- **project_id** - the id for this particular project. You can get this from the url of the project page. For example: `https://app.community.saturnenterprise.io/dash/projects/a753517c0d4b40b598823cb759a83f50` has the project_id: `a753517c0d4b40b598823cb759a83f50`.
- **user_id** - the id that lets saturn know you're the right user. Go to  <a href="https://app.community.saturnenterprise.io/api/user/token" target='_blank' rel='noopener'>https://app.community.saturnenterprise.io/api/user/token</a> and save the page as `token.json`
- **url** - the URL of the Saturn Cloud version you're using. For example for Saturn Hosted, it’s `https://app.community.saturnenterprise.io/`. If you're using an enterprise Saturn Cloud installation, it'll be different.

> Warning: Don't store your user_id token in an public place, or in plain text on places like github. The token needs to be stored securely so that access to your account is safe!

## Connect to Saturn Remotely

Within your Python code you need to establish your remote connection to the service, this will require some parameters to be set to your external code can securely connect to the location of the Dask cluster.

* **url**: The url is the URL of the Saturn deployment. 
* **api_token**: api_token is an API token that tells Saturn who you are. If you are logged in to Saturn, you can retrieve it in JSON form at the following URL endpoint: `/api/user/token` For example, on Saturn Hosted, the URL is `https://app.community.saturnenterprise.io/api/user/token` and it returns something like:
`{"token": "8108a9955a4747559df869a5ce3e1a5f"}`

## Add Connection Code to Script

Now, we can put it all together! Take your `project_id` and your `url`, and fill them in to the chunk below. Make sure the `token.json` file you downloaded is in the same folder.

```python
from dask_saturn.external import ExternalConnection
from dask_saturn import SaturnCluster
from dask.distributed import Client
import json

with open('token.json') as f:
  data = json.load(f)

conn = ExternalConnection(
    project_id="[PROJECT_ID]",
    base_url="[URL]",
    saturn_token=data["token"]
)

cluster = SaturnCluster(
    external_connection=conn,
    n_workers=3,
    worker_size='8xlarge',
    scheduler_size='2xlarge',
    nthreads=32,
    worker_is_spot=False)

client = Client(cluster)
```

Run the chunk, and soon you'll see lines like this:

```python
#> INFO:dask-saturn:Starting cluster. Status: pending
```

This tells you that your cluster is starting up! Eventually you'll see something like:  

```python
#> INFO:dask-saturn:{'tcp://10.0.23.16:43141': {'status': 'OK'}}
```

Which is informing you that your cluster is up and ready to use. Now you can interact with it just the same way you would from Jupyter. If you need help with that, please check out some of our tutorials, such as [Training a Model with Scikit-learn and Dask](<docs/Examples/MachineLearning/sklearn-training.md>), or the <a href="https://github.com/saturncloud/dask-saturn" target='_blank' rel='noopener'>dask-saturn API</a>. 

## Places to connect to Saturn Cloud

Not only can you connect to Saturn Cloud from your laptop or local machine, but you can connect from other cloud-based notebooks. Check out instructions for connecting from [Google Colab](<docs/Using Saturn Cloud/External Connect/colab_external_connect.md>), [SageMaker](<docs/Using Saturn Cloud/External Connect/sagemaker_external_connect.md>), and [Azure](<docs/Using Saturn Cloud/External Connect/azure_external_connect.md>).