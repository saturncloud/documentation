# RStudio Servers

An RStudio resource is a resource that provides access to the RStudio IDE for interactive development. It is best used when your primarily programming in R.

To create a RStudio Server resource, click the **Create RStudio Server** button at the top right of the **Resources** page. You will be presented with the following form:

<img src="/images/docs/create-resource.png" alt="Create resource page" class="doc-image">

In the above form, you'll supply the following details:

* **Name**: Identify the resource with a name of your choosing, and if you like, provide a description. If you  want to [use SSH to access this resource](<docs/Using Saturn Cloud/ide_ssh.md>), check that box. 
* **Disk Space**: The default of 10GB is a good place to start, but if you plan to store very big data files, you may want to increase this.
* **Hardware**: [Choose the size of machine that will suit your needs](<docs/Reference/choosing_machines.md>). Don't worry about choosing wrong; you can always edit this later. Unless you plan to use GPU computing, CPU is probably a good choice. If you decide to use a GPU, a T4 GPU will be less powerful but also less expensive than a V100 GPU.
* **Image**: An image is a Docker image that describes the libraries and packages you need to run your code.  Make sure that if you choose a GPU based machine, you also choose a GPU image. If you don't know what sort of image you want, or need to set up a custom image, [consult our Images documentation](<docs/Using Saturn Cloud/images.md>). Note that if there is a Dask cluster associated with the resource, it will use the same image.
* **Working Directory**: This is your working directory at resource startup. Most times, you can leave this as the default.
* **Shutoff After**: *[RStudio server resources do not currently support the auto-shutoff feature]*
* **Advanced Settings (optional)**: You can customize the Start Script and/or Environment Variables for the client and the workers your resource might contain. These settings are applied every time the RStudio server starts.

Click **Create** to have your new resource built. After this, you'll be taken to the resource page that shows the parameters you've requested. 