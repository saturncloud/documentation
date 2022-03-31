# Import Existing Docker Images

If you already have a Docker image in your personal or business repository, you can easily add it to Saturn Cloud.

Select **Images** in the Saturn Cloud toolbar. From here, you'll see the blue **New Image** button at the top right of the screen. Click this, and you'll be taken to a form. 

There, select **Import an external image**, and fill in the correct **Image URI** that leads to your image.

![New image form with Import external image selected](/images/docs/import-external-image.png "doc-image")

Then select the correct **Owner** of the image and whether it is intended for CPU or GPU instances.

## Docker Image Requirements

Docker images must have certain software installed, depending on what type of resource they are being used for.
For examples of Docker image installation processes, see the Saturn Cloud [images GitHub repo](https://github.com/saturncloud/images).

### Docker Image Requirements - Jupyter Servers

Jupyter servers must have Python and JupyterLab installed in a particular way using the steps below.

1. Conda installed in `/opt/conda` with Python ≥ 3.7.
2. (optional) `sshd` installed on the system.
3. Your custom environment installed in `/opt/conda/envs/saturn` -- This can contain anything you wants to use, but it should contain at least `ipykernel`. 
4. `/opt/conda/envs/saturn` registered with Jupyter -- Do this with the following code:

```bash
        ${CONDA_DIR}/envs/saturn/bin/python -m ipykernel install \
                --name python3 \
                --display-name 'saturn (Python 3)' \
                --prefix=${CONDA_DIR}
```

5. A user named `jovyan` with `uid=1000` and their home directory created.

These steps are not required but will make the user experience better:

6. Add the `jovyan` user to sudo-ers.
7. Set the PATH with the following code:

```bash
        PATH= /opt/conda/envs/saturn/bin:/opt/conda/bin:${PATH} 
```

8. Make sure Python environments are owned by the jovyan user.

### Docker Image Requirements - RStudio Servers

RStudio servers must have R, RStudio, and Python all installed.

1. Conda installed in `/opt/conda` with Python ≥ 3.7.
2. `sshd` installed on the system.
4. A user named `jovyan` with `uid=1000` and their home directory created.
5. R installed to either `/usr/lib/R` or `/usr/local/lib/R`. The `jovyan` user must have recursive ownership and read/write access to the folder.
6. RStudio Server Open Source installed, with configuration files configured to the minimums matching those in [saturncloud/saturnbase-rstudio](https://github.com/saturncloud/images/tree/main/saturnbase-rstudio).
7. The Renviron file located at either `/usr/lib/R/etc/Renviron` or `/usr/local/lib/R/etc/Renviron` must be modified to include the line below. This ensures R will correctly read the installed packages when using RStudio. 

```
R_LIBS=/usr/local/lib/R:/usr/local/lib/R/site-library:/usr/lib/R/site-library:/usr/lib/R/library
```

### Docker Image Requirements - Jobs and Deployments

The requirements for job and deployment resources are simpler than the other types of resources since no IDE is required. _All images that are used for Jupyter server and RStudio server resources will also work for jobs and deployments._

1. Conda installed in `/opt/conda` with Python ≥ 3.7.
2. `sshd` installed on the system.
3. A user named `jovyan` with `uid=1000` and their home directory created.
