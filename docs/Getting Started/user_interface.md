# Introduction to the Saturn Cloud Product

When starting to use the Saturn Cloud product, there are a few concepts that are helpful to know. This document will give a brief tour of the areas of the Saturn Cloud product, to prepare you to be productive quickly.

## Projects

A "Project" is where all the work done in Saturn Cloud resides. Each user can have multiple projects, and these projects can be shared between users. The services associated with each project are called "Resources" and they are organized in the following manner:

```
└── Project
    ├── Jupyter Server (*)
    │   └── Dask Cluster
    ├── Deployment
    │   └── Dask Cluster

(*) Every Project has a Jupyter Server, while Dask Clusters and Deployments are optional.
```
<img src="/images/docs/project_ui.png" alt="Screenshot of Saturn Cloud Projects page" class="doc-image">

## Images

An "Image" is a Docker image that contains a Python environment to be attached to various Resources. A Project is set to use one Image, and all Resources in that Project will utilize the same Image.

Saturn Cloud includes pre-built images for users to get up and running quickly. Users can build custom images by navigating to the "Images" tab from the Saturn Cloud UI.

To learn to build or use an image, [visit our Setup documentation](<docs/Using Saturn Cloud/images.md>).


<img src="/images/docs/images_ui.png" alt="Screenshot of Saturn Cloud Images page" class="doc-image">


## Credentials

You may need to have secret credentials in your working environment in order to access tools or data. The "Credentials" section of the Tools menu is where all this information can be safely stored.

To learn more about adding and using your own credentials, visit our [Setup documentation on the topic](<docs/Using Saturn Cloud/credentials.md>).


<img src="/images/docs/creds_ui.png" alt="Screenshot of Saturn Cloud Credentials page" class="doc-image">

## Git Repositories

We make it easy for you to set up a persistent connection to your Github repositories, and you can find all the steps to do that on our [Github repos documentation](<docs/Using Saturn Cloud/gitrepo.md>).

<img src="/images/docs/repos3.jpg" alt="Screenshot of Saturn Cloud Repositories page" class="doc-image">


## Resources

Resources is a section that gives you easy, immediate viewing of all the servers and machine instances you have available. This includes Jupyter servers, Dask Clusters, deployments, and anything else you build. This is very helpful to prevent anything running unintentionally and creating unexpected costs.

<img src="/images/docs/resources_ui.png" alt="Screenshot of Saturn Cloud Resources page" class="doc-image">

## Get Help

If our documentation doesn't answer your questions, we are always eager to be of help. You can join the Saturn Cloud Community slack team, where we have staff to assist, as well as other users of Saturn Cloud who have lots of experience. You can also contact us privately via Intercom.
