# Install Packages and Software

Saturn Cloud allows nearly all libraries and packages to be installed, provided they can run in a Linux environment. That includes the following:

* Particular packages for Python like XGBoost or boto3
* Linux libraries and programs like `libssl-dev`
* Entirely distinct data science programming languages like [Julia](https://julialang.org/)

Saturn Cloud has three methods for installing packages into a resource, each with different tradeoffs depending on what you want to do:

1. Install packages from within a running resource (only lasts until the resource is shut down).
2. Use the resource settings to install software at resource startup (slows the resource start time)
3. Creating a new image with the required software (the most work)

## 1. Install software from within a running resource

If a Saturn Cloud resource is running, you can install packages and libraries into it. The installed software will only exist until the resource is shut down--if you start
it again it will no longer be there. The temporary nature of this method makes it great for testing out libraries and finding a set you like, at which time you can switch to one of the other
two methods.

To install software from within a running resource, in JupyterLab open up a terminal window (or if you are SSHing into the resource open up a terminal that way). From there you can directly use command line installation methods like `pip`, `conda`, and `apt-get`.

![Open terminal](/images/docs/open-terminal.webp "doc-image")

<a resource-startup></a>

## 2. Use the resource settings to install software at resource startup

Once you know exactly what packages and software you want to install, you can set it to be installed right at the moment when the resource is started. Compared to the first method, you'll only have to set this up once and from that point the packages will be there every time the resource is started. The downside is that the package and library installation process will run each time, so if your packages take a while to install then your resources will take a while to start.

To install at startup time, go to the settings page of your resource. In the **Extra Resources** section you'll see choices for installing via conda, pip, or apt. If what you need to install can be installed via one of those three methods, you can list the packages there. 

![Extra packages](/images/docs/extra-packages.webp "doc-image")

To install anything else, under **Advanced Settings** you'll see **Start Script (Bash)**. In this window you can put any Linux commands you want to run during startup:

![Advanced settings](/images/docs/advanced-settings.webp "doc-image")

In the event that your software fails to install, for instance if you try to install a Python package that doesn't exist, your resource will have an error when it starts. You can view the resource logs by clicking the **Logs** link on the resource.

## 3. Creating a new image with the required software

If you are finding that having the packages and libraries installed at resource startup is taking too long, you can instead have your resource use an image with the software already built in. Click the **Images** button on the sidebar for the tools to do this:

![Images sidebar](/images/docs/images-sidebar.webp "doc-image")

The [images creation docs page](<docs/user-guide/how-to/create-images.md>) has detailed instructions on creating and using an image. This can dramatically speed up the start time of a resource since it won't have to install the packages each time. The downside is that images often take half an hour or more to build, and if you need to debug the image build process, this can take hours to complete. So, try this method only after you've used the first two methods of software installation to determine exactly what you want to install.

## Additional notes when installing packages and software

Python packages should be installed to the `saturn` conda environment. The `base` conda environment is used for managing JupyterLab and should not be accessed directly. By default packages should be installed into `saturn`. If you are having trouble activating conda from the command line you may need to run:

```console
conda init bash
bash
conda activate saturn
```

If you are installing a new programming language and want to use it with JupyterLab, you'll need to register the kernel with Jupyter. The [method for doing so](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels) depends on the programming language you are using.
