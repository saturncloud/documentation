# Create a Resource

> These instructions assume you already have a Saturn Cloud account. If you do not yet have an account, it's fast and easy to [create one](<docs/Getting Started/signing_up.md>).

Navigate to <a href="https://app.community.saturnenterprise.io/auth/login" target='_blank' rel='noopener'>Saturn Cloud Hosted</a>, and log in.

From here, you can choose to create a resource from a pre-built template or create your own from scratch.

## Use a Template Resource

Creating a resource from a template is as easy as clicking a card! As you see here, we have a number of template resources ready for you to try, and you don't need to set any of the custom parameters to begin.

<img src="/images/docs/resources-page-02.png" alt="Screenshot of Saturn Cloud resource page with template resource cards visible" class="doc-image">

When you click one of these cards you'll be taken to your fully set up resource:

<img src="/images/docs/quickstart3b.png" alt="Screenshot of a quick start resource page after creation, called 'pytorch'" class="doc-image">

To start your resource click the green arrow. If you have a Dask cluster attached to the resource you'll need to start that separately.

<img src="/images/docs/quickstart3c.png" alt="Arrows to start Jupyter server" class="doc-image">

Your machines will start up, and when ready the **JupyterLab** button will turn bright blue, letting you know you can select it.
Click that button, and you'll find yourself in a full featured Jupyter Lab workspace, and you're ready to run code!

## Create a Resource from scratch

If you want to create your own resource and use your preferred specifications, that's also easy to do. To create a Jupyter Server resource, click the **Create Jupyter Server** button at the top right of the resource page. You can also [create a job or deployment](<docs/Using Saturn Cloud/jobs_and_deployments.md>) using a similar flow.

<img src="/images/docs/create-resource.png" alt="Create resource page" class="doc-image">

* **Name** - Identify the resource with a name of your choosing, and if you like, provide a description. If you will want to use SSH to access this resource later, click the appropriate button. ([If you need more information, visit our page about SSH connection.](<docs/Using Saturn Cloud/ide_ssh.md>) )
* **Disk Space** - This will choose how much memory your machine will have. If you plan to store very big data files, you may want to increase from the default, but otherwise the default of 10GB is a good place to start.
* **Hardware** - Choose what sort of hardware you will want for your Jupyter server. Unless you plan to use GPU computing, CPU is probably a good choice. A T4 GPU will be less powerful but also less expensive than a V100 GPU. Choose the size of machine that will suit your needs - don't worry about choosing wrong, you can always edit this later. If you want more info, we have [an article about how to think about the machine you need](<docs/Reference/choosing_machines.md>).
* **Image** - An image will describe the libraries and packages you need to run your code.  Make sure that if you choose a GPU based machine, you also choose a GPU image. You can choose from the Saturn Cloud default image selection when you create a resource. This is the same way you will select a custom image, if you choose to create one. However, if you don't know what sort of image you want, or need to set up a custom image, you can [visit our Images documentation to learn more](<docs/Using Saturn Cloud/images.md>).
* **Shutoff After** - Choose an auto-shutoff time interval that works for you. This will help you avoid any unwanted expenses.
* **Repositories (optional)** - If you know the Git repositories you want to use, you can select them now - but if you don't know yet, or haven't added them, don't worry - you can edit all of this later. We have a set of instructions for [how to connect to your Git repos](<docs/Using Saturn Cloud/gitrepo.md>) that might be helpful!
* **Advanced Settings (optional)** - There are some additional customization options you might want to apply, mainly the Start Script and Environment Variables. When you create the resource, if you scroll down to "Advanced Settings (optional)" and open the section, you can customize the Start Script and/or Environment Variables for the client and the workers your resource might contain. These settings are applied every time the Jupyter server (and cluster) start.

Click "Create" to have your new resource built. After this, you'll be taken to the resource page that shows the parameters you've requested.

<img src="/images/docs/quickstart3b.png" alt="Screenshot of a resource page after creation, called 'pytorch'" class="doc-image">

To start your resource click the green arrow. If you have a Dask cluster attached to the resource you'll need to start that separately.

<img src="/images/docs/quickstart3c.png" alt="Arrows to start Jupyter server" class="doc-image">

Your machines will start up, and when ready the **JupyterLab** button will turn bright blue, letting you know you can select it.
Click that button to start coding on Saturn Cloud.