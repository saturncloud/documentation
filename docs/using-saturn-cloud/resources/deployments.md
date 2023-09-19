# Deployments

Deployments are custom-defined applications that you can run in your Saturn Cloud account. A deployment can be a model deployed as an API, a dashboard, or another application that runs continuously.

### Create a deployment

To create a deployment, go to the **Resources** tab of Saturn Cloud and press **New Deployment**

![New deployment button](/images/docs/new-deployment-button.webp "doc-image")

This pops open a screen to choose the particular options for the deployment.

![New deployment options](/images/docs/new-deployment-options.webp "doc-image")

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

![Start deployment button](/images/docs/start-deployment.webp "doc-image")

{{% alert title="Note:" %}}
Be aware that Deployments are constantly running.  This means that they are always available, as well as always using resources and incurring expenses.
{{% /alert %}}

## Accessing a Deployment

In the deployment's details, a URL is shown. This is the URL for the deployment - clicking it will open the deployment.  Deployments are protected by an internal authentication layer. When accessed, the user's Saturn Cloud session is verified. If the user is not signed in, they will be prompted to sign in. Currently, any signed-in Saturn Cloud user can access a deployment if they have authorization.

### Dashboards

Anyone who is logged in to Saturn Cloud and given appropriate permissions can view your deployed dashboards. Saturn Cloud can check whether someone is logged in, and it is possible to grant them access to the underlying resource.

### APIs

To access a deployed API programmatically, you'll need to ensure the machine making the request is authorized to access it. The HTTP request must include an authorization token in the header. You can find your user token for the request by going to the **Settings** page of Saturn Cloud. Then you can make the HTTP request by adding a header key of `Authorization` with a value of `token {USER_TOKEN}` where `{USER_TOKEN}` is the value from the setting page, for example:

```python
import requests
import os

url = "https://hellosaturn-deploy.community.saturnenterprise.io"
USER_TOKEN = # your user token
response = requests.get(
    url,
    headers={"Authorization": f"token {USER_TOKEN}"}
)
```

## Troubleshooting Deployments

For general troubleshooting, the logs for a deployment can be viewed by clicking the **Status** button in the resource details.

![Job status link](/images/docs/job-status.webp "doc-image")

### The Deployment Never Gets to "Ready" Status

The most likely cause of this is that deployment's containers are either crashing, or exiting too quickly. Kubernetes expects deployments' containers to be long-running processes - if the deployment's code is a simple short task, such as something that pulls work from a queue, it may need to be changed to a loop.

If the containers are crashing, errors should be shown in the deployment's logs.

### The Deployment's status is "Ready", but accessing the resource gives a status 502

The most likely cause of this is that nothing is bound to port 8000 within the deployment's containers. 8000 is the only forwarded HTTP port - applications need to bind to it to be accessible. Another possibility is that the server is bound to 127.0.0.1 and not 0.0.0.0  - it needs to listen on all addresses to be accessible from outside of the container.
