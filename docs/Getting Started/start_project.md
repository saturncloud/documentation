# Create a Project

> These instructions assume you already have a Saturn Cloud account. If you do not yet have an account, it's fast and easy to [create one](<docs/Getting Started/signing_up.md>).

Navigate to <a href="https://app.community.saturnenterprise.io/auth/login" target='_blank' rel='noopener'>Saturn Cloud Hosted</a>, and log in.

From here, you can choose to create a pre-built project, or create your own from scratch.

## Use a Quick-Start Project

Getting a quick-start project is as easy as clicking a card! As you see here, we have a number of project templates ready for you to try, and you don't need to set any of the custom parameters to begin.

<img src="/images/docs/quickstart.png" alt="Screenshot of Saturn Cloud project page with quick-start cards visible" class="doc-image">

### Select Project
When you click one of these cards, you'll be asked to confirm that you want to create it, as shown.

<img src="/images/docs/quickstart2.png" alt="Screenshot of Saturn Cloud page for confirming the creation of a quick-start project" class="doc-image">

Then, you'll be taken to your fully set up project!

<img src="/images/docs/quickstart3.png" alt="Screenshot of a quick start project page after creation, called 'pytorch'" class="doc-image">

### Start Resources
To start your resources and begin working, click the green arrows as shown, on the Jupyter server and the Dask cluster.

<img src="/images/docs/quickstart4.png" alt="Screenshot of Saturn Cloud project's Jupyter server card, with arrow pointing to green start button" class="doc-image" style="width:350px;">
<img src="/images/docs/quickstart5.png" alt="Screenshot of Saturn Cloud project's Dask Cluster card, with arrow pointing to green start button" class="doc-image" style="width:350px;">

Your machines will start up, and when ready the "Jupyter Lab" button will turn bright blue, letting you know you can select it.
Click that button, and you'll find yourself in a full featured Jupyter Lab workspace, and you're ready to run code!

***

## Create a Custom Project
If you want to create your own project, and use your preferred specifications, that's also easy to do. Click the "Create Project" button in the left menu, or from the Project page, click "Create Custom Project" in the top right corner.

<img src="/images/docs/create-custom-project.png" alt="Screenshot of Saturn Cloud project page" class="doc-image">

From here, you'll see a form with the Create Project options.

<img src="/images/docs/image5.png" alt="Screenshot of Saturn Cloud Create Project form" class="doc-image">

### Set Custom Options
#### Name
Identify the project with a name of your choosing, and if you like, provide a description. If you will want to use SSH to access this project later, click the appropriate button. ([If you need more information, visit our page about SSH connection.](<docs/Using Saturn Cloud/External Connect/ide_ssh.md>) )

#### Disk Space
This will choose how much memory your machine will have. If you plan to store very big data files, you may want to increase from the default, but otherwise the default of 10GB is a good place to start.

#### Hardware
Choose what sort of hardware you will want for your Jupyter server. Unless you plan to use GPU computing, CPU is probably a good choice. A T4 GPU will be less powerful but also less expensive than a V100 GPU. Choose the size of machine that will suit your needs - don't worry about choosing wrong, you can always edit this later. If you want more info, we have [an article about how to think about the machine you need](<docs/Reference/choosing_machines.md>).

#### Image
An image will describe the libraries and packages you need to run your code.  Make sure that if you choose a GPU based machine, you also choose a GPU image. You can choose from the Saturn Cloud default image selection when you create a project. This is the same way you will select a custom image, if you choose to create one.

<img src="/images/docs/image5.png" alt="Screenshot of Saturn Cloud Create Project form showing Image selection dropdown" class="doc-image">

However, if you don't know what sort of image you want, or need to set up a custom image, you can [visit our Images documentation to learn more](<docs/Using Saturn Cloud/images.md>).

#### Shutoff After
Choose an auto-shutoff time interval that works for you. This will help you avoid any unwanted expenses.

#### Repositories (optional)
If you know the Git repositories you want to use, you can select them now - but if you don't know yet, or haven't added them, don't worry - you can edit all of this later. We have a set of instructions for [how to connect to your Git repos](<docs/Using Saturn Cloud/gitrepo.md>) that might be helpful!

Click "Create" to have your new project built. After this, you'll be taken to the project page that shows the parameters you've requested.

#### Advanced Settings (optional)

There are some additional customization options you might want to apply, mainly the Start Script and Environment Variables. When you create the project, if you scroll down to "Advanced Settings (optional)" and open the section, you can customize the Start Script and/or Environment Variables for the client and the workers your project might contain. These settings are applied every time the Jupyter server (and cluster) start.

<img src="/images/docs/advsettings.png" alt="Screenshot of Saturn Cloud project creation page Advanced Settings section" class="doc-image">

### Start Resources
To start your Jupyter server and begin working, click the green arrow as shown.

<img src="/images/docs/startjupyter.png" alt="Screenshot of Saturn Cloud project's Jupyter server card, with arrow pointing to green start button" class="doc-image">

Your instance will start up, and when ready the "Jupyter Lab" button will turn bright blue, letting you know you can select it.
Click that button, and you'll find yourself in a full featured Jupyter Lab workspace, and you're ready to run code!



<!-- Jupyter labextensions
If you are relying on specific jupyter labextensions, those need to be added in the postBuild in the image. For more information see Images.
Preserve your workspace
By default, Saturn will recreate your workspace every time you start up Jupyter. To override this behavior and have Saturn preserve your workspace between restarts, set environment variable SATURN__JUPYTER_RESET_WORKSPACE=false -->
