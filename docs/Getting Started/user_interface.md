# Introduction to the Saturn Cloud Product

When starting to use the Saturn Cloud product, there are a few concepts that are helpful to know. This document will give a brief tour of the areas of the Saturn Cloud product, to prepare you to be productive quickly.

## Resources

A _resource_ is where all the work done in Saturn Cloud resides. Each resource is independent from the rest, making them great ways to split our different types of activities you're doing. You can have a resource for each project you're working on, analysis you're doing, and so on.

A resource is one of the following:

* Jupyter Server - the most popular resources, these allow you to interactively run code via JupyterLab or by connecting via SSH from an IDE like PyCharm or VSCode.
* Job - a task that runs on command or on a schedule
* Deployment - a continuously running activity like hosting a dashboard or API.
* Prefect Cloud flow - a special type of resource specific for running Prefect jobs. These are [created differently](/docs) than the other types of resources.

Each user can have multiple resources, and resources can be cloned (including by other users with sufficient permissions). Each resource may optionally have a Dask cluster associated with it, allowing the resource to run computations across multiple machines.

The resources page is where you can go to view all of your resources and create new ones.

<img src="/images/docs/resources-page-02.png" alt="Screenshot of Saturn Cloud Resources page" class="doc-image">

## Images

An _image_ is a Docker image that contains a Python environment to be attached to various resources. A resource is set to use one image, and if there is a Dask cluster associated with that resource it'll use the same image.

Saturn Cloud includes pre-built images for users to get up and running quickly. Users can build custom images by navigating to the "Images" tab from the Saturn Cloud UI.

To learn to build or use an image, [visit our Setup documentation](<docs/Using Saturn Cloud/images.md>).

<img src="/images/docs/images_ui.png" alt="Screenshot of Saturn Cloud Images page" class="doc-image">

## Credentials

You may need to have secret credentials in your working environment in order to access tools or data. The _Credentials_ section of the Tools menu is where all this information can be safely stored.

To learn more about adding and using your own credentials, visit our [Setup documentation on the topic](<docs/Using Saturn Cloud/credentials.md>).

<img src="/images/docs/creds_ui.png" alt="Screenshot of Saturn Cloud Credentials page" class="doc-image">

## Git Repositories

We make it easy for you to set up a persistent connection to your git repositories, and you can find all the steps to do that on our [git repos documentation](<docs/Using Saturn Cloud/gitrepo.md>).

<img src="/images/docs/repos4.png" alt="Screenshot of Saturn Cloud Git Repositories page" class="doc-image">
