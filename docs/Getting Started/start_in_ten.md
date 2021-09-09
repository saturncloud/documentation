# Saturn Cloud Quickstart

Saturn Cloud is a platform for data science--helping people quickly do work using whatever technology they need, including large instances, GPU processors, distributed computing, and more. 

This guide is for first time users to
get an environment set up in Saturn Cloud to do data science. In just a few steps, you will be able to run Python code in the cloud, and with that you can expand the environment however you need.

<div class="text-center py-3 row">
<div class="embed-responsive embed-responsive-16by9 col-md-10 offset-md-1 col-lg-8 offset-lg-2">
<iframe width="560" height="315" src="https://www.youtube.com/embed/qE0zhXouDSo" title="YouTube video player"
frameborder="0"
allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
allowfullscreen class="embed-responsive-item"></iframe>
</div>
</div>

## Sign Up for Saturn Cloud

Sign up by visiting [saturncloud.io](https://www.saturncloud.io/s/), clicking "Start for Free", and following the instructions. If you have any struggles, we have [a full tutorial](<docs/Getting Started/signing_up.md>).

![Screenshot of signup page indicating GitHub, Google, or email options](/images/docs/signup2.jpg "doc-image")

## Creating and Starting a Resource

A **resource** is an complete environment for running code. They come in multiple types, but the most commonly used one is a **Jupyter server**, which lets you use JupyterLab (or other IDEs) to run Jupyter notebooks and Python scripts.

Create a Jupyter Server resource in your new account. On the **Resources** page you can either
select one of the pre-created template resources like the **RAPIDS** server, or set up your own by pressing the **New Jupyter Server** button and customizing the resource.

<img src="/images/docs/create-jupyter.png" alt="Screenshot of the resource page" class="doc-image">

Once the server is created, you'll need to turn it on. Press the green triangle on the resource's page to start the server. When the card shows that the resource is "Running", you can open it and begin working.

<img src="/images/docs/start-jupyter.png" alt="Screenshot of card in resource for Jupyter server with green 'start' button" class="doc-image">    

## Using the Resource

With a resource from a template, you can run the example notebooks immediately. The RAPIDS quick start, for example, lets you run GPU-accelerated data science code to process data and train machine learning models. [In this tutorial](<docs/Examples/RAPIDS/qs-01-rapids-gpu.md>), you get all the instructions to train a RAPIDS model on Saturn Cloud from start to finish. If you made a custom resource you can upload your own code (or [connect the resource to a git repository](<docs/Using Saturn Cloud/gitrepo.md>)).

<img src="/images/docs/jupyterlab-01.jpg" alt="Jupyter notebook open in JupyterLab" class="doc-image">    

When you're done using the resource, you can shut it down the same way you turned it on, by clicking the button on the card (it will be a red square when the resource is running). By default, the resource will also automatically shut off after the browser window has been inactive for an hour. 

## Customizing Your Resources

Creating and using a Jupyter server resource is at the core of most of what you can do with Saturn Cloud. There are lots of ways you can expand on it:

* **[Creating Dask cluster for the resource](<docs/Using Saturn Cloud/create_dask_cluster.md>)** - One powerful feature of Saturn Cloud is Dask clusters for distributed computing in Python.
* **[Have the resource use a GPU](<docs/Reference/intro_to_gpu.md>)** - Saturn Cloud resources can use GPUs as easily as CPUs for faster machine learning.
* **[Connect to data](<docs/Using Saturn Cloud/connect_data.md>)** - How to connect to different data locations from a Saturn Cloud resource.
* **[Connect a git repo](<docs/Using Saturn Cloud/gitrepo.md>)** - Make a Saturn Cloud resource link with your git repositories to run your code.
* **[Use other IDEs (like PyCharm or VSCode)](<docs/Using Saturn Cloud/ide_ssh.md>)** - Connect to a Jupyter server resource from your local IDE, using an SSH connection.
* **[Schedule jobs and run deployments](<docs/Using Saturn Cloud/jobs_and_deployments.md>)** - Jobs and deployments are two other types of resources. These let you schedule scripts to run or set up continuously running resources (like APIs and dashboards).
