# Resources in Saturn Cloud

The resource is the fundamental unit of work for Saturn Cloud. There are 4 types

- Workspace: [Python Server](/docs). This is a development environment for Python code. This machine runs a Jupyter Lab server, however with the SSH integration you can connect PyCharm, VS code, or any other IDE.
- Workspace: [R Server](/docs). This is a development environment for R code. This machine runs an R studio server, however with the SSH integration you can connect PyCharm, VS code or any other IDE.
- [Deployment](/docs). This is a non-interactive server, typically used to server models, APIs and dashboards. It is designed to run 24/7
- [Job](/docs). This is a non-interactive server, typically used to write data pipelines. It is designed to run once and then complete - often on a schedule.

[Dask clusters](/docs) can be attached to any resource.

Saturn Cloud resources are very simple, they are made up of

- a hardware configuration
- a docker image
- some initialization scripts, often used to install packages
- your git repositories
- secrets used to access data

The simplicity is intentional - resources are meant to be pretty transparent. Anything you can run in Saturn Cloud you should be able to run outside of it, and vice versa. If you ever decide to leave Saturn Cloud, you just need to find a place to run your docker images with your git repositories, and it should be just the same as when you were a Saturn Cloud user. Code that runs in Saturn Cloud is regular Python or R code (or Julia or Java, and anything else). It does not have to be written in a special notebook, or even in a notebook at all.

### Workspaces

Python and R workspace host Jupyter and R studio respectively. In addition, you can add SSH access, which allows you to use virtually any IDE, including PyCharm and VS Code. These workspaces are running in docker images. Every workspace gets a persistent EBS volume attached, so that anything users write to the home directory is preserved across restarts. We have invested significantly in making these docker images feel like desktops.

Docker is supported, even though you are running in a docker container, we can mount the docker socket inside the container so that your code can call out to the Docker API. There is also an option to expose port 8000, so that if you are developing a dashboard, you can access it from your browser.

The Git integration is setup to automatically ensure your repositories are cloned to a specific location. Using the integration is not technically necessary, since users can clone git repositories manually using the command line - however the Git integration ensures similar patterns between interactive (workspaces) and non-interactive resources (deployments and jobs)


### Deployments and Jobs

Deployments are typically used to deploy models, APIs and dashboards. Jobs are typically used for data pipelines, or model re-trainings. Deployments and Jobs have almost all of the same configuration options as workspaces. This is intentional so that anything you develop within a workspace can be run as a deployment or a Job.

Deployments are extremely generic. The user has to specify what command line invocation will start their deployment, and the deployment is expected to serve HTTP traffic on port 8000. Your choice of framework (Bokeh, Plotly Dash, Panel, Streamlit, Shiny) is not important. If you write code that does this, Saturn Cloud can host it, and our authentication proxy (the same thing that protects your Python and R workspace) will allow only authorized users access to your deployment.

Jobs are also extremely generic. The user has to specify what command line invocation will start the job. Your choice of framework (Prefect, Dagster) does not matter. As long as you can express a command line that will run your job, you can run it in Saturn Cloud. Jobs can be scheduled (via a cron like schedule expression) or run manually. They can also be dispatched via our API.

The Git integration is the primary mechanism for how your code ends up running in the Deployment or Job. These are usually comprised of a docker image that has all the necessary software dependencies, as well as 1 or more Git repositories that contain the source code for your Job or Deployment. Because we are using the Saturn Cloud git integration, you can also make use of branches or tags to ensure that the source code powering a Job or a Deployment does not change over time.

### Attachments

In Saturn Cloud, there are a number of objects that can be attached to a resource.

- **Images** - this is the only mandatory attachment. This determines the base software configuration for the image. It can be changed at any time.
- **Git repositories** - ensures that a Git repository is cloned on resource startup. For interactive resources like workspaces, this is less important because users can always clone git repositories manually in the terminal, however for non-interactive resources like deployments and jobs using attaching a Git repository is how user code gets to the resource.
- **Secrets** - automatic mounting access credentials as environment variables or files.
- **IAM roles** - associating IAM roles with a resource. Only one IAM role can be attached to a resource at a time.
- **Dask clusters** - scalable compute clusters
- **Shared folders** - NFS mounts for resources

Workspaces, deployments and jobs all support the same attachments. This means you can easily convert workflows from one resource type to another.
