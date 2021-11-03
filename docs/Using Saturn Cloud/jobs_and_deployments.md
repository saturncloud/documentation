# Deployments and Jobs in Saturn Cloud
## Overview

Deployments and jobs are custom-defined applications that you can run in your Saturn Cloud account. A deployment can be a model, a dashboard, or another application, and it usually runs continuously. A job is similar, except that it runs only once or at a designated interval only. Deployments and jobs are both types of _resources_, which are distinct computing elements of Saturn Cloud.

{{% alert title="TL;DR" %}}
A _job_ is a collection of code that completes a task and concludes. Jobs may run either on a scheduled recurring basis, or as needed. A _deployment_ is a collection of code that is continuously running.
{{% /alert %}}

***

## How to Create Deployments and Jobs

Deployments and jobs can be created in a very similar manner to Jupyter server resources.

### Deployments

To create a deployment, go to the **Resources** tab of Saturn Cloud and press **New Deployment**

![New deployment button](/images/docs/new-deployment-button.jpg "doc-image")

This pops open a screen to choose the particular options for the deployment.

![New deployment options](/images/docs/new-deployment-options.jpg "doc-image")

The options to set up the deployment are as follows:

* __Name__ - the name of the deployment (must be unique).
* __Command__ - the command to execute the deployment within the resource. These can vary dramatically depending on what the deployment task is (ex: run a panel dashboard or flask API). **The command must listen to port 8000 for requests, no other ports will be open.** The server must also be bound to `0.0.0.0` and not `127.0.0.1`. Pressing the radio buttons will populate some common defaults.
* __Hardware__ - the type of hardware to use for the deployment, either a CPU or GPU.
* __Size__ - the size of the instance to use for the deployment.
* __Image__ - the image to use for the deployment.
* __Environment Variables__ (Advanced setting) - any specific environment variable values for the deployment.
* __Start Script (Bash)__ (Advanced setting) - the script to run at startup of the image before the deployment code runs.
* __Working Directory__ (Advanced setting) - where within the file system the deployment command should be executed.

After choosing these options and pressing the create deployment button, you go be sent to the resource page for that deployment. Pressing the green arrow next to the deployment name will cause the deployment to begin running.

![Start deployment button](/images/docs/start-deployment.jpg "doc-image")

{{% alert title="Note:" %}}
Be aware that Deployments are constantly running.  This means that they are always available, as well as always using resources and incurring expenses.
{{% /alert %}}

### Jobs

To create a job, go to the **Resources** tab of Saturn Cloud and press **New Job**

![New job button](/images/docs/new-job-button.jpg "doc-image")

This will allow you to edit the job in detail.

![New job options](/images/docs/new-job-options.jpg "doc-image")

The options to set up the job are as follows:

* __Name__ - the name of the job (must be unique).
* __Command__ - the command to execute the job within the resource. Typically something like `python run.py` where `run.py` is a script in the resource.
* __Hardware__ - the type of hardware to use for the job, either a CPU or GPU.
* __Size__ - the size of the instance to use for the job.
* __Image__ - the image to use for the job.
* __Run this job on a schedule__ - should the job be run once, or on a repeating basis. If on a schedule, more options become available:
  * __Cron Schedule__ - the frequency at which to run the job, specified as a <a href="https://pkg.go.dev/gopkg.in/robfig/cron.v2" target='_blank' rel='noopener'>cron expression</a>. Note that scheduling can be at the minute level at most (not second)
  * __Concurrency Policy__ - should an instance of a job be able to run if the previous one hasn't stopped? Select _Allow_ for concurrent jobs, _Forbid_ if only one job can run at a time (cancelling the new job), and _Replace_ if only one job can run at a time (cancelling the original job).
  * __Backoff limit__ - attempts at starting the task before it should be considered it to have failed.
* __Environment Variables__ (Advanced setting) - any specific environment variable values for the job.
* __Start Script (Bash)__ (Advanced setting) - the script to run at startup of the image before the job is executed.
* __Working Directory__ (Advanced setting) - where within the file directory the job command should be executed.

Pressing **Create** will add the job to the resource list and you will be taken to the resource page for the new job. Pressing the green triangle "go" button will make the job execute. If it's on a schedule, it will run at the times you specify.

![Start job button](/images/docs/start-job.jpg "doc-image")

***

## Accessing a Deployment

In the deployment's details, a URL is shown. This is the URL for the deployment - clicking it will open the deployment.  Deployments are protected by an internal authentication layer. When accessed, the user's Saturn Cloud session is verified. If the user is not signed in, they will be prompted to sign in. Currently, any signed-in Saturn Cloud user can access a deployment if they have authorization.

### Dashboards

Anyone who is logged in to Saturn Cloud and given appropriate permissions can view your deployed dashboards. Saturn Cloud can check whether someone is logged in, and it is possible to grant them access to the underlying resource.

### APIs

To access a deployed API programmatically, you'll need to ensure the machine making the request is authorized to access it. The HTTP request must include an authorization token in the header. If the machine making the request is a different Saturn Cloud resource (like a Jupyter Server), then the token is stored in an environmental variable name `SATURN_TOKEN`. If the request is being made from a machine outside of Saturn Cloud, then a user can get the token from first logging into Saturn Cloud and then going to [https://app.community.saturnenterprise.io/api/user/token](https://app.community.saturnenterprise.io/api/user/token) (the token is the hexadecimal value in quotes).

Once you have the correct token, you can make the HTTP request by adding a header key of `Authorization` with a value of `token [YOUR TOKEN]`, for example:

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

## Troubleshooting Deployments and Jobs

For general troubleshooting, the logs for a deployment or job can be viewed by clicking the **Status** button in the resource details.

![Job status link](/images/docs/job-status.jpg "doc-image")

### The Deployment Never Gets to "Ready" Status

The most likely cause of this is that deployment's containers are either crashing, or exiting too quickly. Kubernetes expects deployments' containers to be long-running processes - if the deployment's code is a simple short task, such as something that pulls work from a queue, it may need to be changed to a loop.

If the containers are crashing, errors should be shown in the deployment's logs.

### The Deployment's status is "Ready", but accessing the resource gives a status 502

The most likely cause of this is that nothing is bound to port 8000 within the deployment's containers. 8000 is the only forwarded HTTP port - applications need to bind to it to be accessible. Another possibility is that the server is bound to 127.0.0.1 and not 0.0.0.0  - it needs to listen on all addresses to be accessible from outside of the container.

<!-- TODO: none of this troubleshooting actually gives helpful things to DO, need to add actual advice -->
