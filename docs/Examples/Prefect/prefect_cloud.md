# Using Prefect Cloud with Saturn Cloud

## Overview

This tutorial explains how to use Prefect Cloud and Saturn Cloud together.

The tutorial ["Develop a Scheduled Data Pipeline"](<docs/Examples/Prefect/prefect_cloud.md>) introduces how to build data pipelines using `prefect`, and how to speed them up by executing them on a Saturn Dask Cluster. If you are not familiar with `prefect` yet, consider reading that article first and then coming back to this one.

For this tutorial, we'll create a flow that mimics the process of getting a batch of records, using a machine learning model to score on them, and capturing metrics.

## Set Up a Prefect Cloud Account <a name="setup"></a>

To begin this tutorial, you'll need an existing Prefect Cloud account. Prefect Cloud's free tier allows you to run a limited number of flows, so you can run this tutorial without spending any money on Prefect Cloud!

1. Sign up at <a href="https://www.prefect.io/cloud/" target="_blank" rel="noopener">https://www.prefect.io/cloud/</a>
1. Once logged in, <a href="https://docs.prefect.io/orchestration/concepts/projects.html#creating-a-project" target="_blank" rel="noopener">create a project</a>. For the purpose of this tutorial, call it `dask-prefect-tutorial`.
1. Following <a href="https://docs.prefect.io/orchestration/concepts/api_keys.html" target="_blank" rel="noopener">the Prefect documentation</a>, create a User API Key  and a Service Account API Key. Store these for later.
    - Service Account API Key: must be created by an admin. Allows an agent to communicate with Prefect Cloud.
    - User API Key: allows a user to register new flows with Prefect Cloud

## Create a Prefect Cloud Agent in Saturn Cloud

Prefect Cloud "agents" are always-on processes that poll Prefect Cloud and ask *"want me to run anything? want me to run anything?"*. In Saturn Cloud, you can create these agents with a few clicks and let Saturn handle the infrastructure.

1. Log in to the Saturn UI as an admin user.
1. Navigate to the "Credentials" page and add a Prefect Cloud Service Account API Key
    - `Type`: Environment Variable
    - `Shared With`: your user only
    - `Name`: `prefect-runner-token`
    - `Variable Name`: `PREFECT_RUNNER_TOKEN`
    - `Value`: the Service Account API Key you created during [setup](#setup)
1. Navigate to the "Prefect Agents" page. Create a new agent.
    * `Name`: `test-prefect-agent`
    * `Prefect Runner Token`: select `prefect-runner-token`
1. Start that Prefect Agent by clicking the play button.

<img src="/images/docs/prefect-agent-create.png" alt="Create a Prefect Agent form page in Saturn Cloud UI" class="doc-image">

After a few seconds, your agent will be ready!
Click on the Agent's status to see the logs for this agent.
<br>

<img src="/images/docs/prefect-agent-logs.png" alt="Log viewing page in Saturn Cloud UI showing the logs for a Prefect Agent" class="doc-image">
<br>

In the Prefect Cloud UI, you should see a new `KubernetesAgent` up and running!

<img src="/images/docs/prefect-agent-in-cloud-ui.png" alt="View of the Prefect Cloud website showing the equivalent agent running inside the UI" class="doc-image">

## Create and Register a Flow

Now that you've created an account in Prefect Cloud and set up an agent in Saturn to run the work there, it's time to create a flow!

1. Return to the Saturn UI.
1. Navigate to the "Credentials" page and add a Prefect Cloud User API Key.
    - `Type`: Environment Variable
    - `Name`: `prefect-user-token`
    - `Variable Name`: `PREFECT_USER_TOKEN`
    - `Value`: the User API Key you created during [setup](<docs/getting_help.md#setup>)
1. Navigate to the "Resources" page and create a new Jupyter Server with the following specs.
    * `Name`: test-prefect-jupyter-server
    * `Image:` any of the available non-gpu `saturncloud/saturn` images you want
    * `Workspace Settings`
        * `Hardware`, `Disk Space`, `Shutoff After`: keep the defaults
    * `Environment Variables`
        ```shell
        PREFECT_CLOUD_PROJECT_NAME=dask-prefect-tutorial
        ```
    * `Start script`
        ```shell
        pip install --upgrade dask-saturn prefect-saturn
        ```
1. Once the Resource is created, start it by clicking the play button.
1. Once that server is ready, click "JupyterLab" to launch JupyterLab.
1. In JupyterLab, open a terminal and run the code below to fetch the example notebook that accompanies this tutorial.
    ```shell
    cd /home/jovyan/project/
    EXAMPLE_REPO_URL=https://raw.githubusercontent.com/saturncloud/examples/main/examples/prefect/

    wget ${EXAMPLE_REPO_URL}/02-prefect-cloud.ipynb
    ```
1. In the file browser in the left-hand navigation, double-click that notebook to open it. Follow the instructions in it and run the cells in order. Return to this article when you're done.

Once you've registered a flow, it will create a new Saturn Cloud resource specifically for running the flow. If you go to the Resources page of Saturn Cloud you should see a new resource created.

## Inspect Flow Runs

Now that your flow has been created and registered with both Saturn Cloud and Prefect Cloud, you can track it's progress in the Prefect Cloud UI.

1. In the Prefect Cloud UI, go to `Flows --> ticket-model-evaluation`. Click `Schematic` to see the structure of the pipeline.

<img src="/images/docs/prefect-flow-schematic.png" alt="View of the Prefect Cloud page showing a diagram of the scheduled flow's structure" class="doc-image">
<br>

2. Click `Logs` to see logs for this flow run.
    - from this page, you can search the logs, sort them by level, and download them for further analysis

<img src="/images/docs/prefect-cloud-flow-run-logs.png" alt="View of the Prefect Cloud page showing logs for the run of the scheduled flow" class="doc-image">

3. In the Saturn Cloud UI, navigate to "Prefect" resource associated with this work. This will bring you to a table of the prefect flows. Click on the flow's name in that table. This will take you to the flow's details page, where you can see a list of flow runs. Click the icon under "logs" in the flow run table to view logs from a flow run.

_This view allows you to see some logs that won't be visible in Prefect Cloud, including any output generated by your resource's start script._

<img src="/images/docs/prefect-flow-details-page-saturn-ui.png" alt="View of details page for one Prefect Cloud flow, showing a table of flow runs" class="doc-image">

<img src="/images/docs/prefect-cloud-flow-run-logs-saturn-ui.png" alt="View of the logs from a Prefect Cloud flow run, shown in the Saturn product" class="doc-image">

5. Return to the flow's resource page in the Saturn UI. Click the "Dask Cluster" button, which will take you to the details page for the Dask cluster used to execute the flow. Click the "Dashboard" link on this page to view the Dask diagnostic dashboard.

<img src="/images/docs/prefect-cloud-flow-attached-dask-cluster.png" alt="View of dask Dask cluster details page in the Saturn product, including a link to the Dask dashboard" class="doc-image">

<img src="/images/docs/dask-dashboard-metrics.png" alt="View of dask Dashboards inside Saturn Cloud UI showing CPU, memory, and bandwidth" class="doc-image">

6. In the Saturn Cloud UI, navigate back to the `Prefect Agents` page. Click the `running` status for the `test-prefect-agent` agent you previously set up. You should see new logs messages confirming that the agent has received a flow to run.

<img src="/images/docs/prefect-agent-flow-run-logs.png" alt="Log viewing page in Saturn Cloud UI showing the logs for a Prefect Agent" class="doc-image">

## Clean Up

The flow created in this tutorial is set to run every 24 hours. Once you're done with this tutorial, be sure to tear everything down!

**In Prefect Cloud**

1. navigate to `Flows`. Delete the `ticket-model-evaluation` flow.

**In Saturn Cloud**

1. Logged in as the user who created the flow, navigate to the Prefect resource and delete it as well as the Jupyter server used to create the flows.
1. Logged in as the user you used to create a Prefect agent, navigate to the `Prefect Agents` page. Click the delete button to stop and delete the Prefect agent.
1. Navigate to the `Credentials` page. Remove the credentials `prefect-runner-token` and `prefect-user-token`.

## Learn and Experiment!

To learn more about `prefect-saturn`, see <a href="https://github.com/saturncloud/prefect-saturn" target="_blank" rel="noopener">https://github.com/saturncloud/prefect-saturn</a>.

To learn more about Prefect Cloud, see <a href="https://docs.prefect.io/orchestration/" target="_blank" rel="noopener">https://docs.prefect.io/orchestration/</a>.

To learn more about Saturn Cloud's integration with Prefect Cloud, see ["Prefect Cloud Concepts"](<docs/Examples/Prefect/prefect_cloud_concepts.md>).

If you have any other questions or concerns, send us an email at support@saturncloud.io.
