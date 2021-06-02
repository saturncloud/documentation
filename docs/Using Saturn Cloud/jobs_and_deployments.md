# Create and Use Deployments and Jobs
## Overview

Deployments and Jobs are custom-defined applications that you can run in your Saturn Cloud account. A Deployment can be a model, a dashboard, or another application, and it usually runs continuously. A Job is similar, except that it runs only once or at a designated interval only. Deployments and Jobs use the same libraries, conda environment, and image that your Jupyter servers use, meaning that they can (but don't need to be) developed inside of Jupyter Lab.

{{% alert title="TL;DR" %}}
A _Job_ is a collection of code that completes a task and concludes. Jobs may run either on a scheduled recurring basis, or as needed. A _Deployment_ is a collection of code that is continuously running.
{{% /alert %}}

***

## How to Create...

Jobs and Deployments are tied to a Project. Files within the working directory (default: `/home/jovyan/project`) are included in the same place in a Job or Deployment's container, and that directory is set as the starting directory.

### Jobs
To create a job, first create a project, then on the project page press the "create a job" button:

<img src="/images/docs/jobs_and_deployments_create_a_job.png" alt="Create a job button" class="doc-image">

This will allow you to edit the job in detail.

<img src="/images/docs/jobs_and_deployments_setup_job.png" alt="New job options" class="doc-image">

The options to set up the job are as follows:

* __Name__ - the name of the job (must be unique).
* __Command__ - the command to execute the job within the project. Typically something like `python run.py` where `run.py` is a script in the project.
* __Hardware__ - the type of hardware to use for the job. Normally should the same as the Jupyter running on the project.
* __Size__ - the size of the instance to use for the job. Normally should be the same as the Jupyter running on the project.
* __Image__ - the image to use for the job. Normally should be the same as the Jupyter running on the project.
* __Run this job on a schedule__ - should the job be run once, or on a repeating basis. If on a schedule, more options become available:
  * __Cron Schedule__ - the frequency at which to run the job, specified as a <a href="https://pkg.go.dev/gopkg.in/robfig/cron.v2" target='_blank' rel='noopener'>cron expression</a>. Note that scheduling can be at the minute level at most (not second)
  * __Concurrency Policy__ - should an instance of a job be able to run if the previous one hasn't stopped? Select _Allow_ for concurrent jobs, _Forbid_ if only one job can run at a time (cancelling the new job), and _Replace_ if only one job can run at a time (cancelling the original job).
  * __Backoff limit__ - attempts at starting the task before it should be considered it to have failed.
* __Managed Repository Reference__ - the branch, tag, or commit hash to check out for the managed (built-in) git repository for the associated project.
* __Environment Variables__ (Advanced setting) - any specific environment variable values for the job.
* __Start Script (Bash)__ (Advanced setting) - the script to run at startup of the image before the job is executed.
* __Working Directory__ (Advanced setting) - where within the project the job command should be executed.

Pressing create will add the job to the project. Once you create the job you'll see it listed on the project page. Pressing the green triangle "go" button will make the job execute. If it's on a schedule, it will run at the times you specify.

<img src="/images/docs/jobs_and_deployments_run_job.png" alt="Job run window" class="doc-image">

### Deployments
To create a deployment, go to a project page and click the "Create a deployment button":

<img src="/images/docs/jobs_and_deployments_create_a_deployment.png" alt="Create a deployment button" class="doc-image">

This pops open a screen to choose the particular options for the deployment.

<img src="/images/docs/jobs_and_deployments_setup_deployment.png" alt="New deployment options" class="doc-image">

The options to set up the deployment are as follows:

* __Name__ - the name of the deployment (must be unique).

* __Command__ - the command to execute the deployment within the project. These can vary dramatically depending on what the deployment task is (ex: run a panel dashboard or flask API). Pressing the radio buttons will populate some common defaults.
* __Hardware__ - the type of hardware to use for the deployment. Normally should be the same as the Jupyter running on the project.
* __Size__ - the size of the instance to use for the deployment. Normally should be the same as the Jupyter running on the project.
* __Image__ - the image to use for the deployment. Normally should be the same as the Jupyter running on the project.
* __Managed Repository Reference__ - this is the branch, tag, or commit hash to check out for the managed (built-in) git repository for the associated project.
* __Environment Variables__ (Advanced setting) - any specific environment variable values for the deployment.
* __Start Script (Bash)__ (Advanced setting) - the script to run at startup of the image before the deployment code runs.
* __Working Directory__ (Advanced setting) - where within the project the deployment command should be executed.

After choosing these options and pressing the create deployment button, the deployment will show up attached to the project. Pressing the green arrow next to the deployment name will cause the deployment to begin running.

<img src="/images/docs/jobs_and_deployments_run_deployment.png" alt="Deployment run window" class="doc-image">

***

## Accessing a Deployment
In the deployment's details, a URL is shown. This is the URL for the deployment - clicking it will open the deployment.  Deployments are protected by an internal authentication layer. When accessed, the user's Saturn session is verified. If the user is not signed in, they will be prompted to sign in. Currently, any signed-in Saturn user can access a deployment if they have authorization.

### Dashboards
Anyone who is logged in to Saturn and given appropriate permissions can view your deployed dashboards.  Saturn Cloud can check whether someone is logged in, and it is possible to grant them access to the underlying resource.

### Models
Anyone who is logged in to Saturn and given appropriate permissions can view your deployed models as well, but this is slightly easier because of how we use API tokens. Every Saturn user service (Jupyter/Dask/Deployments) has an API token passed as an environment variable, that can identify who owns a given service.  If you pass that token in http headers, Saturn cloud will allow access to the resource.

```python
import requests
import os

url = "https://hellosaturn.deploy.example.com"
SATURN_TOKEN = os.environ["SATURN_TOKEN"]
response = requests.get(
    url,
    headers={"Authorization": f"token {SATURN_TOKEN}"}
)
```


{{% alert title="Note:" %}}
Be aware that Deployments are constantly running.  This means that they are always available, as well as always using resources and incurring expenses.
{{% /alert %}}


## Troubleshooting Deployments and Jobs
For general troubleshooting, the deployment's logs can be viewed by clicking the "Logs" button in the deployment details.
<!-- TODO: needs screenshot -->

### The Deployment never gets to "Ready" status
The most likely cause of this is that deployment's containers are either crashing, or exiting too quickly. Kubernetes expects deployments' containers to be long-running processes - if the deployment's code is a simple short task, such as something that pulls work from a queue, it may need to be changed to a loop.

If the containers are crashing, errors should be shown in the deployment's logs.

### The Deployment's status is "Ready", but accessing the resource gives a status 502
The most likely cause of this is that nothing is bound to port 8000 within the deployment's containers. 8000 is the only forwarded HTTP port - applications need to bind to it to be accessible. Another possibility is that the server is bound to 127.0.0.1  and not 0.0.0.0  - it needs to listen on all addresses to be accessible from outside of the container.

<!-- TODO: none of this troubleshooting actually gives helpful things to DO, need to add actual advice -->

