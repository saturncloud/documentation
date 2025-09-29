# Jupyter Servers

A Jupyter server resource is a resource that provides access to the JupyterLab IDE for interactive development. It is best used for day to day data science tasks.

## Before You Start: Set Up Git (Recommended)

If you plan to work with code from a Git repository, we recommend setting up Git integration before creating your resource. This ensures your code is automatically available when the resource starts.

**Quick Git Setup:**
1. **[Set up SSH keys](/docs)** (one-time setup)
2. **[Add your repository](/docs)** to Saturn Cloud
3. **Attach during creation** - see the Git Repositories section in the form below

**Need a faster setup?** See our **[Quick Git Setup guide](/docs)** for step-by-step instructions.

{{% alert title="Why set up Git first?" %}}
Setting up Git before creating your resource means your code will be automatically cloned and ready to use when the resource starts. If you skip this step, you'll need to manually clone repositories or restart your resource after adding Git integration.
{{% /alert %}}

## Create Your Jupyter Server

<img src="/images/jupyter-creation-form.png" alt="Jupyter server creation form showing all available options" class="doc-image">

<hr>

In the above form, you'll supply the following details:

* **Name**: Identify the resource with a name of your choosing, and if you like, provide a description. If you  want to [use SSH to access this resource](/docs), check that box. 
* **Disk Space**: The default of 10GB is a good place to start, but if you plan to store very big data files, you may want to increase this.
* **Hardware**: Whether the resource uses only a CPU or also a GPU.
* **Size**: How powerful the machine will be. This adjusts the CPU and memory, and in the case of GPU hardware the size and number of GPUs.
* **Image**: An image is a Docker image that describes the libraries and packages you need to run your code.  Make sure that if you choose a GPU based machine, you also choose a GPU image. If you don't know what sort of image you want, or need to set up a custom image, [consult our Images documentation](/docs). Note that if there is a Dask cluster associated with the resource, it will use the same image.
* **Working Directory**: This is your working directory at resource startup. Most times, you can leave this as the default.
* **Extra Packages**: You can use Conda, Pip, or Apt to install packages at resource startup. Simply click on the appropriate package manager(s), then list the package(s) in the textbox.
* **Git Repositories**: Attach your code repositories to automatically clone them when the resource starts. This is the easiest way to get your code into Saturn Cloud. If you haven't set up Git yet, see the "Before You Start" section above.
* **Shutoff After**: Choose an [auto-shutoff](/docs) time interval that works for you. This will help you avoid any unwanted expenses. *(Hosted Pro and Enterprise accounts only)*
* **Advanced Settings (optional)**: You can customize the Start Script and/or Environment Variables for the client and the workers your resource might contain. These settings are applied every time the Jupyter server starts.

<hr>

Click **Create** to have your new resource built. After this, you'll be taken to the resource page that shows the parameters you've requested. To attach a [Dask cluster](/docs), click **New Dask Cluster**.

## Next Steps: Customize Your Resource

After creating your Jupyter server, you can customize it for your specific needs:

- **[Connect Git repositories](/docs)**: Clone your code directly into the resource
- **[Install packages](/docs)**: Add software and libraries your code needs
- **[Add secrets](/docs)**: Store API keys and credentials securely
- **[Set up SSH access](/docs)**: Connect from PyCharm, VSCode, or other IDEs
- **[Create a Dask cluster](/docs)**: Scale to distributed computing for large datasets

<img src="/images/docs/quickstart3b.webp" alt="Screenshot of a resource page after creation, called 'pytorch'" class="doc-image">
