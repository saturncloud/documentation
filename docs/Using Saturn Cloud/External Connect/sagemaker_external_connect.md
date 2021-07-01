# Connect to Dask from SageMaker

## Connect to a Saturn Cloud project

If you have not yet created a Saturn Cloud account, go to [saturncloud.io](https://saturncloud.io) and click "Start For Free" on the upper right corner. It'll ask you to create a login.

<img src="/images/docs/homepage.jpg" alt="Saturn homepage" class="doc-image">

Once you have done so, you'll be brought to the Saturn Cloud projects page. Click "Create Custom Project".

<img src="/images/docs/sagemaker-u01.png" alt="Saturn creating a new project" class="doc-image">

Give the project a name (ex: "sagemaker-demo"), but you can leave all other settings as their defaults. Then click "Create".

After the project is created you'll be brought to that project's page. At this point you'll need to retrieve two ID values:

- **`project_id`** - the id for this particular project. You can get this from the URL of the project page. For example: `https://app.community.saturnenterprise.io/dash/projects/a753517c0d4b40b598823cb759a83f50` has the project_id: `a753517c0d4b40b598823cb759a83f50`.
- **`user_id`** - the ID that identifies you as a valid user in Saturn Cloud. Go to <a href="https://app.community.saturnenterprise.io/api/user/token" target='_blank' rel='noopener'>https://app.community.saturnenterprise.io/api/user/token</a> and save the page as `token.json`, then upload that file to the Sagemaker Studio workspace. Do not share this file with others.

> Protect your user token, as it allows access to your account!

You can now load the token inside Sagemaker Studio in a notebook, as shown.

```python
# Load token
import json

with open('../config.json') as f:
  data = json.load(f)
```

## Connect to your Project

Now you are ready to connect your Sagemaker Studio workspace to your Saturn Cloud project, allowing you to interact with it from this notebook. Your `user_id` is required (here shown as `data['token']`), as well as the `project_id` discussed earlier.

```python
from dask_saturn.external import ExternalConnection
from dask_saturn import SaturnCluster
import dask_saturn
from dask.distributed import Client, progress

conn = ExternalConnection(
    project_id=project_id,
    base_url='https://app.community.saturnenterprise.io',
    saturn_token=data['token']
)
conn

#> dask_saturn.external.ExternalConnection at 0x7f04d067e0d0>
```

## Set Up Cluster

Finally, you are ready to set up a cluster in this project! You'll see info messages logging here until the cluster is started and ready to use.

If you have a cluster already created on the project, here you can just start it up without creating a new one, using this same code. You can also ask it to change size using `cluster.scale()`. For more details, we have [documentation about managing clusters](<docs/Using Saturn Cloud/Create Cluster/create_cluster.md>).

```python
cluster = SaturnCluster(
    external_connection=conn,
    n_workers=4,
    worker_size='8xlarge',
    scheduler_size='2xlarge',
    nthreads=32,
    worker_is_spot=False)
```

## Create Client Object

This lets us connect from our Sagemaker environment to this new cluster, and when we call the object, it gives us a link to the Dask Dashboard for that cluster. We can watch at this link to see how the cluster is behaving.

```python
client = Client(cluster)
client.wait_for_workers(4)
client
```

<img src="/images/docs/sagemaker-090556.png" alt="Created Dask client" class="doc-image">

## Analysis!

At this point, you are able to do load data and complete whatever analysis you want. You can monitor the performance of your cluster at the link described earlier, or you can log in to Saturn Cloud and see the Dask dashboard, logs for the cluster workers, and other useful information.

You can also connect to Dask from [Google Colab](<docs/Using Saturn Cloud/External Connect/colab_external_connect.md>), [Azure](<docs/Using Saturn Cloud/External Connect/azure_external_connect.md>), or [anywhere else outside of Saturn Cloud](<docs/Using Saturn Cloud/External Connect/external_connect.md>).
