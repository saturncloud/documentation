# Prefect Cloud Flows

Prefect Cloud is a hosted, high-availability, fault-tolerant service that handles all the orchestration responsibilities for running data pipelines.
It gives you complete oversight of your workflows and makes it easy to manage them. It provides cloud convenience with on-prem security.
It follows [hybrid model](https://medium.com/the-prefect-blog/the-prefect-hybrid-model-1b70c7fd296): users will design, test and build workflow in form of code that is orchestrated on the cloud but executed on their private infrastructure. In this case the "private infrastructure" are the resources running on Saturn Cloud. Once the workflow is registered with Prefect Cloud, codeless version of workflow is sent to cloud, which provides fully-managed orchestration service to your workflow on Saturn Cloud.

Saturn Cloud connects to Prefect Cloud via a special **Prefect Cloud Flow** resource type. This resource is started and stopped via Prefect Cloud--unlike other resource types you do not directly interact with it. Instead you manage a Prefect Cloud account which then makes the appropriate Saturn Cloud calls. This allows you to use Prefect Cloud as your centralized data science pipeline management location.

> Note that Prefect Cloud is distinct from _Prefect Core_, which is the open-source Python library that can be run to manage data pipelines. Prefect Core can be run within a single Jupyter Server resource like any other Python library, whereas Prefect Cloud allows you to orchestrate across multiple resources in a cloud hosted pipeline. Generally if you have a complex pipeline that you want to manage it's better to use the Prefect Cloud service than to try and manage a system within a single Saturn Cloud resource.

## Prefect Cloud Components

Below are the different components of Prefect Cloud and how they connect to Saturn Cloud.

### Flows

A [flow](https://docs.prefect.io/core/concepts/flows.html) in Prefect Cloud is a container for multiple tasks which understands the relationship between those tasks.
When a flow changes, a new "flow version" is created. Example changes include:

-   some tasks have been added or removed
-   the dependencies between tasks have changed
-   the flow is using a different execution mechanism (like a Dask cluster instead of a local Python process)

The Prefect Cloud UI keeps track of all these versions, and knows to link all versions of the same flow into one "flow group".

_Each time a flow is executed, a Prefect Cloud Flow resource in Saturn Cloud is spun up to do the computations. When it is complete the resource is shut down._

### Agents

A Prefect Agent is a small always-on service responsible for running flows and reporting their logs and statuses back to Prefect Cloud. Prefect Agents are always "pull-based"--they are configured to periodically make requests to Prefect Cloud to determine if new work is scheduled to run. When the agent receives a request from Prefect Cloud, the agent is responsible for inspecting following details of the flow and then kicking off a flow run.

_When using Saturn Cloud, the Prefect Agents will be always running in the Saturn Cloud account. These can be adjusted in the Prefect Agents tab of the Saturn Cloud app._

## Saturn Cloud + Prefect Cloud Architecture

Using Saturn Cloud and Prefect Cloud together looks like this:

1. Using credentials from your Prefect Cloud account, you create an **Agent** running in Saturn Cloud.
2. You create a Saturn Cloud Jupyter server resource which defines all the dependencies your code needs.
3. In a Jupyter server with all those dependencies set up, you write flow code in Python using the `prefect` library
4. In your Python code, you use the <code><a href="https://github.com/saturncloud/prefect-saturn" target="_blank" rel="noopener">prefect-saturn</a></code> library to "register" your flow with Saturn Cloud, and the <code><a href="https://github.com/PrefectHQ/prefect" target="_blank" rel="noopener">prefect</a></code> library to register it with Prefect Cloud. Your flow will be automatically labeled to match with Prefect agents running in your Saturn cluster.
5. `prefect-saturn` adds the following features to your flow by default:
    - storage: <code><a href="https://docs.prefect.io/orchestration/execution/storage_options.html#webhook" target="_blank" rel="noopener">Webhook</a></code>
    - run config: <code><a href="https://docs.prefect.io/orchestration/flow_config/run_configs.html#kubernetesrun" target="_blank" rel="noopener">KubernetesRun</a></code>
    - executor: <code><a href="https://docs.prefect.io/api/latest/executors.html#localexecutor" target="_blank" rel="noopener">LocalExecutor</a></code>, using a `prefect_saturn.SaturnCluster`
    - labels: `saturn-cloud, webhook-flow-storage, <YOUR_CLUSTER_DOMAIN>`
6. When Prefect Cloud tells your Prefect Agent in Saturn to run the flow, Saturn Cloud creates a kubernetes job to run the flow.

Using this integration, you'll write code with the **`prefect` library** which talks to **Saturn Cloud** and **Prefect Cloud**. Their responsibilities are as follows:

-   **`prefect` library**
    -   describe the work to be done in a flow
    -   tell Prefect Cloud about the flow, including when to run it _(on a schedule? on demand?)_
    -   store that flow somewhere so it can be retrieved and run later
-   **Saturn Cloud**
    -   provide a hosted Jupyter Lab experience where you can author and test flows, and a library for easily deploying them (<code><a href="https://github.com/saturncloud/prefect-saturn" target="_blank" rel="noopener">prefect-saturn</a></code>
    -   run an Agent that checks Prefect Cloud for new work
    -   when Prefect Cloud says "yes run something", retrieve flows from storage and run them
    -   automatically start up a flow execution environment (a single node or a distributed Dask cluster) to run your flow, with the following guarantees:
        -   is the size you asked for
        -   has a GPU your code can take advantage of (if you requested one)
        -   has the exact same environment as the Jupyter notebook where you wrote your code
        -   has all of the code for your project (like other libraries you wrote)
        -   has all of the credentials and secrets you've added (like AWS credentials or SSH keys)
    -   display logs in the Saturn Cloud UI
    -   send logs and task statuses back to Prefect Cloud, so you have all the information you need to react if anything breaks
-   **Prefect Cloud**
    -   keep track of all the flows you've registered
    -   when it's time to run those flows (either on demand or based on a schedule), tell Agents to run them
    -   display a history of all flow runs, including success / failure of individual tasks and logs from all tasks
    -   allow you to kick off a flow on-demand using a CLI, Python library, or clicking buttons in the UI

## Detailed steps for using Prefect Cloud and Saturn Cloud

### Set Up a Prefect Cloud Account <a name="setup"></a>

First, create an account with Prefect Cloud:

1. Sign up at <a href="https://www.prefect.io/cloud/" target="_blank" rel="noopener">https://www.prefect.io/cloud/</a>.
2. Once logged in, <a href="https://docs.prefect.io/orchestration/concepts/projects.html#creating-a-project" target="_blank" rel="noopener">create a project</a>.
3. Following <a href="https://docs.prefect.io/orchestration/concepts/api_keys.html" target="_blank" rel="noopener">the Prefect documentation</a>, create a User API Key and a Service Account API Key. Store these for later.

User API Key: allows a user to register new flows with Prefect Cloud. To generate User API Key, go to Account Settings > API Keys within the Prefect Cloud UI and click "Create an API Key".

<img src="/images/docs/prefect_user_api.png" alt="Prefect user API key" class="doc-image">

Service Account API Key: must be created by an admin. Allows an agent to communicate with Prefect Cloud. To create service accounts and associated API keys, go to Team > Service Accounts.

<img src="/images/docs/prefect_service_api.png" alt="Prefect service API key" class="doc-image">

### Create a Prefect Cloud Agent in Saturn Cloud

Prefect Cloud "agents" are always-on processes that poll Prefect Cloud and ask _"want me to run anything? want me to run anything?"_. In Saturn Cloud, you can create these agents with a few clicks and let Saturn handle the infrastructure.

1. Log in to the Saturn UI as an admin user.
2. Navigate to the "Secrets" page and add a Prefect Cloud Service Account API Key.
    - `Name`: Choose a Unique identifier for this. Name should be only lowercase letters, numbers, and dashes, such as prefect-runner-token.
    - `Value`: the Service Account API Key you created during [setup](#setup)
3. Navigate to the "Prefect Agents" page. Create a new agent.
    - `Name`: Each Prefect Agent must have a unique name.
    - `Prefect Runner Token`: Select from dropdown, the name you used to set a Unique identifier in secrets page.
4. Start Prefect Agent by clicking the play button.

<img src="/images/docs/prefect-agent-create.png" alt="Create a Prefect Agent form page in Saturn Cloud UI" class="doc-image">

After a few seconds, your agent will be ready!
Click on the Agent's status to see the logs for this agent.
<img src="/images/docs/prefect-agent-logs.png" alt="Log viewing page in Saturn Cloud UI showing the logs for a Prefect Agent" class="doc-image py-4">

In the Prefect Cloud UI, you should see a new `KubernetesAgent` up and running!

<img src="/images/docs/prefect-agent-in-cloud-ui.png" alt="View of the Prefect Cloud website showing the equivalent agent running inside the UI" class="doc-image">

### Create and Register a Flow

Now that you've created an account in Prefect Cloud and set up an agent in Saturn Cloud to run the work there, it's time to create a flow!

1. Return to the Saturn UI.
2. Navigate to the "Secrets" page and add a Prefect Cloud User API Key.
    - `Name`: Choose a Unique identifier for this. Name should be only lowercase letters, numbers, and dashes, such as `prefect-user-token`.
    - `Value`: the User API Key you created during [setup](<docs/security.md#setup>).
3. Navigate to the "Resources" page and create a new Jupyter Server with the following specs.
    - `Name`: Name of the resource.
    - `Image:` Choose image as per your requirements in workflow.
    - `Workspace Settings`
        - `Hardware`, `Disk Space`, `Shutoff After`: keep the defaults
    - `Environment Variables`
        ```shell
        PREFECT_CLOUD_PROJECT_NAME='set this to the name of your project, which you created in Prefect Cloud '
        ```
    - `Start script`
        ```shell
        pip install --upgrade prefect-saturn
        ```
4. Once the resource is created, start it by clicking the play button.
5. Once that server is ready, click "JupyterLab" to launch JupyterLab.
6. In JupyterLab, open a new notebook and start working or access your code in git repo folder, if you have [added repository to a resource](https://saturncloud.io/docs/using-saturn-cloud/gitrepo/#add-a-git-repository-to-saturn-cloud).
7. You can see some sample workflows and information on how to register this flow in the [Saturn Cloud examples](<docs/examples/python/prefect/qs-01-prefect-singlenode.md>).

Once you've registered a flow, it will create a new Saturn Cloud resource specifically for running the flow. If you go to the Resources page of Saturn Cloud you should see a new resource created.

### Inspect Flow Runs

Now that your flow has been created and registered with both Saturn Cloud and Prefect Cloud, you can track it's progress in the Prefect Cloud UI.

1. In the Prefect Cloud UI, go to `Flows --> name of your flow`. Click `Schematic` to see the structure of the pipeline.

<img src="/images/docs/prefect-flow-schematic.png" alt="View of the Prefect Cloud page showing a diagram of the scheduled flow's structure" class="doc-image">
<br>

2. Click `Logs` to see logs for this flow run.
    - From this page, you can search the logs, sort them by level, and download them for further analysis.

<img src="/images/docs/prefect-cloud-flow-run-logs.png" alt="View of the Prefect Cloud page showing logs for the run of the scheduled flow" class="doc-image">

3. In the Saturn Cloud UI, navigate to "Prefect" resource associated with this work. This will bring you to a table of the prefect flows. Click on the flow's name in that table. This will take you to the flow's details page, where you can see a list of flow runs. Click the icon under "logs" in the flow run table to view logs from a flow run.

_This view allows you to see some logs that won't be visible in Prefect Cloud, including any output generated by your resource's start script._

<img src="/images/docs/prefect-flow-details-page-saturn-ui.png" alt="View of details page for one Prefect Cloud flow, showing a table of flow runs" class="doc-image">

<img src="/images/docs/prefect-cloud-flow-run-logs-saturn-ui.png" alt="View of the logs from a Prefect Cloud flow run, shown in the Saturn product" class="doc-image">

4. In the Saturn Cloud UI, navigate back to the `Prefect Agents` page. Click the `running` status for the agent you previously set up. You should see new logs messages confirming that the agent has received a flow to run.

<img src="/images/docs/prefect-agent-flow-run-logs.png" alt="Log viewing page in Saturn Cloud UI showing the logs for a Prefect Agent" class="doc-image">

### Clean Up

If you have scheduled your flow to run in set of intervals, and want to clean it up follow the instructions below.

**In Prefect Cloud**

1. navigate to `Flows`. Delete the newly created flow.

**In Saturn Cloud**

2. Logged in as the user who created the flow, navigate to the Prefect resource and delete it as well as the Jupyter server used to create the flows.
3. Logged in as the user you used to create a Prefect agent, navigate to the `Prefect Agents` page. Click the delete button to stop and delete the Prefect agent.

### Learn and Experiment!

To learn more about `prefect-saturn`, see <a href="https://github.com/saturncloud/prefect-saturn" target="_blank" rel="noopener">https://github.com/saturncloud/prefect-saturn</a>.

To see examples of creating a workflow and running on Prefect Cloud check out the <a href="<docs/examples/python/prefect/qs-01-prefect-singlenode.md>" target="_blank" rel="noopener">Saturn Cloud Examples</a>.

Prefect Cloud feature is available for Enterprise users only. If you have any questions about [Saturn Enterprise](https://saturncloud.io/plans/enterprise/) or in general, send us an email at support@saturncloud.io.
