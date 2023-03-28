# Libraires, dependencies and Docker images in Saturn Cloud

Working with your existing workflows is a central tenet of the design of Saturn Cloud. Your existing code will run in Saturn Cloud with minimal effort. Most people will start using the built in Saturn Cloud images, and augment them by manually installing packages within the workspace, or by using the extra pacakges functionality to add packages to the image at startup.

If you already have a conda `environment.yaml` or pip `requirements.txt` file, these can be dropped into the Saturn Cloud image builder, which will build an image and store it in your companies ECR. If you already build Docker images, those can be adapted for Saturn Cloud as long as they meet a few requirements detailed here. Many companies also build images using GitHub CI (or any other CI tool) for ECR, and register them with Saturn Cloud.

We always encourage companies to use Docker images, rather than relying on installing packages manually or at startup. This is because changes to what packges are available in package repositories can break your package install process, or cause them to install slighlty different versions of libraries. A Docker image bakes in all the dependencies and will never change over time.

### Base images

We ship a number of base images that are used by the image builder. These base images contain all of the Saturn Cloud requirements, and include installations of Jupyter or R Studio. The sources for these images are publicly available on GitHub, and can be pulled from public.ecr.aws. You don't need to use our base images in order to build Saturn compatible images, but it makes things easier.



### GPU images

CPU images are fairly straightforward. GPU images can be a bit trickier. This has nothing to do with Saturn Cloud, but is instead because compatibility with different CUDA versions can be a bit finnicky. Our base images are each tied to a specific version of CUDA. This isn't a requirement, but it mirrors how NVIDIA packages their images (and our base GPU images are based on NVIDIA CUDA images). GPU images already tend to be quite large. Packaging only the CUDA version you need is one easy way to keep the image as small as possible.
