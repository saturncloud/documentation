# Images in Saturn Cloud

When you start a resource in Saturn Cloud, you may require special libraries or customizations for your work. The way to ensure that these all get installed when the machine is spun up is to use images.

A **Saturn Cloud image** is a Docker image that describes the libraries and packages you need to run your code. It will contain the instructions that explain all the libraries and tools you want installed before you start work. In this sense, it's analogous to the operating system and software installed on your local computer. Saturn Cloud images have specific requirements for what needs to be installed on them in order for them to run on the platform.

## Getting Started with Saturn Cloud Images

When starting out with Saturn Cloud, most people [use one of our default images](<docs/images/default-images/saturn-r.md>), which provide most data science packages that you will need. It is also possible to [create or upload your own image](<docs/using-saturn-cloud/manage-images/build-images/import-images.md>).

### Choosing the Right Image for the Job

Saturn Cloud offers standard CPU images in addition to GPU images, each with different sets of libraries.

The image you choose to use on a resource depends on the following criteria:

-   Which program language you primarily use (e.g., Python, R)
-   Whether you want to do your work on CPU only or with GPU acceleration
-   Which machine learning frameworks you want to use (e.g., RAPIDS, PyTorch, TensorFlow)

For instance, if you use Python and want to do analysis on a CPU, you'll want to try the image called `saturncloud/saturn-python`.

However, if you prefer to use R and want to use GPU-accelerated TensorFlow, then you'll want to try the image called `saturncloud/saturn-r-tensorflow`.

### Attaching a Saturn Cloud Image to a Resource

When you [create a resource in Saturn Cloud](/docs), you will choose a specific image from a dropdown menu.

Saturn Cloud's standard images start with `saturncloud/`.

![Image selector for new resource](/images/docs/new-resource-image-selector.webp "doc-image")

When selecting an image from the dropdown menu, you will only be presented with images that correspond to the hardware type associated with the resource. That is, GPU resources can only use GPU images, and CPU resources can only use CPU images.

If you need packages that are not included among Saturn Cloud's standard images, you can install them at start-up using various package managers (e.g., apt, conda, pip, CRAN). When you add packages in this way, they will be reinstalled every time the resource starts. See the article on [installing software and packages](<docs/using-saturn-cloud/install-packages.md>) for more detail.

> **Note**: If there is a Dask cluster associated with the resource, the Dask scheduler and workers will all use the same image as the Jupyter server.

### Using Custom Images

Once you have established which packages you want to use your environment, you can [create a custom image](<docs/using-saturn-cloud/manage-images/build-images/import-images.md>) that packages them all together. Using a custom image can significantly decrease start-up time because packages won't need to be reinstalled every time the resource starts.
