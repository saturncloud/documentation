# Saturn Cloud Quickstart

Saturn Cloud is a data science platform that helps people quickly do work using whatever technology they need, including high-memory computing, GPU processors, and Dask clusters. 

This guide will help first-time users set up an environment in Saturn Cloud to do data science. In just a few steps, you will be able to run code in the cloud and expand the environment however you need.

<div class="text-center py-3 row">
<div class="embed-responsive embed-responsive-16by9 col-md-10 offset-md-1 col-lg-8 offset-lg-2">
<iframe width="560" height="315" src="https://www.youtube.com/embed/qE0zhXouDSo" title="YouTube video player"
frameborder="0"
allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
allowfullscreen class="embed-responsive-item"></iframe>
</div>
</div>

## Sign Up for Saturn Cloud

Sign up by visiting [saturncloud.io](https://www.saturncloud.io/s/) and clicking on **Start for Free**. Follow the prompts and you'll soon be logged in to your new account

![Saturn Cloud homepage with arrows pointing to "Start for Free](/images/docs/homepage_signup_arrows.jpeg "doc-image")

If you are an Enterprise customer, you will need to use the custom url provided to you.
## Create a Resource

A resource is a complete environment for running code. Each resource is independent, so you can split out the different types of activities you’re doing. You can have a resource for each project you’re working on, each analysis you’re doing, and so on.

A resource is one of the following:

* **Jupyter Server**: The most popular resources, these allow you to interactively run code via JupyterLab or by [connecting via SSH from an IDE like PyCharm or VSCode](<docs/Using Saturn Cloud/ide_ssh.md>).
* **Job**: A task that runs on command or on a schedule *(Hosted Pro and Enterprise accounts only)*
* **Deployment**: A continuously running activity like a hosted dashboard or API *(Hosted Pro and Enterprise accounts only)*
* **Prefect Cloud flow**: A special type of resource specific for running Prefect jobs; these are [created differently](/docs) than the other types of resources. *(Enterprise accounts only)*

Each user can have multiple resources, and resources can be cloned (including by other users with sufficient permissions). Each resource may optionally have a Dask cluster associated with it, allowing the resource to run computations across multiple machines.
<hr>

On the **Resources** page you can do either of the following:
- [Set up your own custom resource](<docs/Using Saturn Cloud/custom_resource.md>) with your specifications by pressing the **New Jupyter Server** button.
- Select one of the pre-created template resources like the **RAPIDS** server.


![Screenshot of the resource page](/images/docs/create-jupyter.png "doc-image")

For the purpose of this quickstart, let's click on the **RAPIDS** template resource. This will set up the enviornment to use GPUs, a Dask cluster, and all the correct settings to start a GPU-accelerated analysis!

## Start a Resource

Once the server is created, you'll need to turn it on. Press the blue **Start** button on the resource's page to start the server. If you have a [Dask cluster](<docs/Using Saturn Cloud/create_dask_cluster.md>) attached to the resource, you’ll need to start that separately. 

![Screenshot of card in resource for Jupyter server with a rectangle around the start button](/images/docs/start_resource_button_rectangles.jpeg "doc-image")

As your machine starts up, the card will display *pending*, and you will see a progress bar showing the steps and overall progress toward starting the server.

When the machine is ready, the card will show *running*, and the JupyterLab button will turn bright blue, letting you know that you can select it. Click that button to start coding on Saturn Cloud!

## Use the Resource

With a templated resource, you can run the example notebooks immediately. The RAPIDS quick start, for example, lets you run GPU-accelerated data science code to process data and train machine learning models. [In this tutorial](<docs/Examples/RAPIDS/qs-01-rapids-single-gpu.md>), you get all the instructions to train a [RAPIDS](https://rapids.ai/) model on Saturn Cloud from start to finish.

![RAPIDS example Jupyter notebook open in JupyterLab](/images/docs/jupyterlab-01.jpg "doc-image")

If you made a custom resource, you can upload your own code (or [connect the resource to a git repository](<docs/Using Saturn Cloud/gitrepo.md>)).

## Customize Your Resource

Creating and using a Jupyter server resource is central to the work you can do on Saturn Cloud. There are many ways you can expand on this:

* **[Create a Dask cluster for the resource](<docs/Using Saturn Cloud/create_dask_cluster.md>).** One powerful feature of Saturn Cloud is the ability to leverage Dask clusters for distributed computing.
* **[Have the resource use a GPU](<docs/Reference/intro_to_gpu.md>).** Saturn Cloud resources can use GPUs as easily as CPUs for faster machine learning.
* **[Connect a git repo](<docs/Using Saturn Cloud/gitrepo.md>).** Connect a Saturn Cloud resource to your git repositories to run your code.
* **[Add credentials to your resouces](<docs/Using Saturn Cloud/credentials.md>).** You may need to have secret credentials in your working environment to access tools or data. The **Credentials** section of the tools menu is where this information can be safely stored.
* **[Use other IDEs (e.g., PyCharm, VSCode)](<docs/Using Saturn Cloud/ide_ssh.md>).** Connect to a resource from your local IDE, using an SSH connection.
* **[Create a custom image](<docs/Using Saturn Cloud/images.md>).** Resources are built upon images with base software and packages. Many people use our standard images, which provide access to many data science packages. However, if, for example, your company has a designed Docker image, you can use that instead.
* **[Schedule jobs and run deployments](<docs/Using Saturn Cloud/jobs_and_deployments.md>).** Jobs and deployments are two other resource types. They let you schedule scripts to run or set up continuously running resources (e.g., APIs, dashboards). *(Hosted Pro and Enterprise accounts only)*

## Stop the Resource
When you're done using the resource, shut it down by clicking the red **Stop** button on the card. By default, the resource will automatically shut off after the browser window has been inactive for an hour. 

## Conclusion
* Use Saturn Cloud as a data science platform so you can quickly do work.
* Do your work in Saturn Cloud resources, where all work done in Saturn Cloud resides.
* Customize your resources however you like to suit your workflow.

Take a look at our [examples pages](/docs) for more inspiration!