# Managing Images

When you start a project in Saturn Cloud, you may require special libraries or customizations for your work. The way to ensure that these all get installed when the machine is spun up is by using images. An image will contain the instructions that explain all the libraries and tools you want installed before you start work.

When starting out with Saturn Cloud, most people will use one of our standard images, which provide most data science packages you will need. However, if (for example) your company has a designated Docker image, you can use that instead!

***

## Select a Default Image

You can choose from the Saturn Cloud default image selection when you create a custom project. This is the same way you will select a custom image, if you choose to create one below.

To create a custom project, select "Create Project" from the "Projects" section of the menu. 

<img src = "/images/docs/create_proj.png" style="width:200px;" alt="Screenshot of side menu of Saturn Cloud product with Create Project selected" class="doc-image">

In this form, you will see a section called "Workspace Settings". One of the selectors in this section is "Images". Click that selector, and you'll be given a list of all the images available to you.

<img src="/images/docs/image5-create-project.jpg" alt="Screenshot of Saturn Cloud Create Project form" class="doc-image">

Choose your image, and continue creating the project as usual.

If you don't see the image you want, please let us know at hello@saturncloud.io and we'll help you!

***

## Build a Custom Image

To build your own image, select "Images" from the "Tools" section of the menu. 

<img src = "/images/docs/docker1.png" style="width:200px;" alt="Screenshot of side menu of Saturn Cloud product with Images selected" class="doc-image">

From here, you'll see the blue "New Image" button at the top right of the screen. Click this, and you'll be taken to a form. 

<img src="/images/docs/docker2.png" alt="Screenshot of Saturn Cloud Add Image form" class="doc-image">

In the first half of the form, you will choose the source of your Docker image. If your image is hosted somewhere else, and you have a link to it, select "External" and the form's contents will change to look like this. Otherwise, leave this box as "Saturn", even if you are going to be filling in your own image specifications.

### External: Add an Image Path

If you're adding an image that already exists in your personal or business repository, just fill in the correct Image URI that leads to your image. If you don't know what this is, let us know or consult with your image hosting system.

<img src="/images/docs/docker4.png" alt="Screenshot of Saturn Cloud Add Image form with 'External' toggle selected" class="doc-image">

### Internal: List Specifications

If your image is not externally created, you can just fill in the parameters you want for the image, including the names of the libraries and packages. 

<img src="/images/docs/image_internal.png" alt="Screenshot of Saturn Cloud Add Image form with 'Saturn' toggle selected, showing Conda Environment pane" class="doc-image">

#### Options

* **Source**: A “Saturn” image is one that Saturn builds for you via configuration files specified below. External is any existing image you want to add to the system. Saturn is configured to pull from the ECR (Elastic Container Registry) associated with your AWS account, and can also pull from any public repository. External images must conform to a certain specification to fully function in Saturn. 
* **Share with**: Admin users can elect to share the image with any user in Saturn (including your whole company)
* **Image URI**: the name of the image you wish to load (if External), or build.

The below options are only available for Internal ("Saturn") image building.

* **Copy configuration from**: If you’ve already built an image, this drop down lets you load that configuration, so that you can modify it, and do a new build.
* **Build Data**: Saturn knows how to build images using a few configuration file types. More information is available from [repo2docker](https://repo2docker.readthedocs.io/en/latest/config_files.html).
  * **Conda environment**: Things you would add to an environment.yml file
  * **pip environment**: Things you would add to a requirements.txt
  * **apt get**: a list of apt get packages
  * **postBuild**: a bash script that is executed in your Docker container after everything else has been run

#### Replicate a Conda Environment
One powerful configuration option is to use a custom conda environment. To do this, in the `Build Data` section, paste in the YAML describing an environment. If you have an existing conda environment you use locally, you can run `conda env export –no-builds` to generate this YAML file. 

When you build an image from scratch like this, it generates a brand new python environment. These are not modifications of Saturn's default images, so you must include *all* the packages you’re going to want to use.

<img src="/images/docs/image-build-conda-env.png" alt="Screenshot of Saturn Cloud Add Image form with some items included to create an image from a conda environment" class="doc-image">

### Dask Distributed Library
Then, select the version of the Dask `distributed` library you want your image to contain. If you're not sure, or just want the most current version, you can visit the [PyPi page](https://pypi.org/project/distributed/) or [the official changelog](https://distributed.dask.org/en/latest/changelog.html) to see what the current version is.  

### CPU or GPU
Finally, you need to indicate whether your image is customized for CPU or GPU. If you have not requested libraries specially designed for GPU computation, then CPU is usually the right choice.

Click "Add" when you have entered all the information, and your image will be built. *Note: This may take some time, as all your packages and dependencies need to be built to ensure it will work!* 

After the image build is complete, your image is ready to use in creating a new project.

### See Your Image

If you find that later you want to refer back to the specifications of an image you built in Saturn Cloud, you can visit the images page and click the names of each. Custom images don't currently show their full contents.

<img src="/images/docs/images_ui.png" alt="Screenshot of Saturn Cloud Images list" class="doc-image">

***

## Advanced: GPU Images

Saturn supports instances with T4 GPUs (g4dn), as well as instances with V100 GPUs (including p3.16xlarge which has 8 V100 GPUs). These instances must be paired with GPU images, which means the images contain libraries that run on GPU hardware. Otherwise there will be no CUDA drivers available, and you won’t be able to use the GPU card. Saturn offers a default `saturn-gpu` image, which includes common GPU utilities for machine learning including RAPIDS, PyTorch and Tensorflow/Keras.

If you are building your own custom image, you'll want to take this into account. 

### Tensorflow
Tensorflow has GPU specific versions. In `pip`, call for `tensorflow-gpu`. In conda, look through the list to find a GPU build.

```bash
$ conda search tensorflow
...
#> tensorflow                     2.2.0 eigen_py36h84d285f_0  pkgs/main
#> tensorflow                     2.2.0 eigen_py37h1b16bb3_0  pkgs/main
#> tensorflow                     2.2.0 gpu_py37h1a511ff_0  pkgs/main
#> tensorflow                     2.2.0 gpu_py38hb782248_0  pkgs/main
#> tensorflow                     2.2.0 mkl_py36h5a57954_0  pkgs/main
```

Then in the image specifications you can ask for:

```yml
dependencies:
  - tensorflow=2.2.0=gpu_py37h1a511ff_0
```

### PyTorch
Similarly, PyTorch has GPU versions. Conda is the easiest way to access these. Look for "cuda" in the name, as this indicates GPU support.

```bash
$ conda search pytorch
...
#> pytorch                        1.5.1 py3.7_cpu_0                      pytorch
#> pytorch                        1.5.1 py3.7_cuda10.1.243_cudnn7.6.3_0  pytorch
#> pytorch                        1.5.1 py3.7_cuda10.2.89_cudnn7.6.5_0   pytorch
#> pytorch                        1.5.1 py3.7_cuda9.2.148_cudnn7.6.3_0   pytorch
#> pytorch                        1.5.1 py3.8_cpu_0                      pytorch
#> pytorch                        1.5.1 py3.8_cuda10.1.243_cudnn7.6.3_0  pytorch
#> pytorch                        1.5.1 py3.8_cuda10.2.89_cudnn7.6.5_0   pytorch
```

You have to consider the CUDA version and the Python version here. This example shows selecting CUDA 10.1.

```yml
channels:
  - pytorch
  - defaults
dependencies:
  - pytorch=1.5.1=py3.7_cuda10.1.243_cudnn7.6.3_0
```

### RAPIDS
Like PyTorch, for RAPIDS you need to specify the CUDA version. You can get this from the `rapidsai` conda channel.

```yml
channels:
  - rapidsai
  - defaults
dependencies:
  - rapids=0.14.1=cuda10.1_py37_0
```