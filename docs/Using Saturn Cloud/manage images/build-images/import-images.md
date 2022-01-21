# Import Existing Docker Images

If you already have a Docker image in your personal or business repository, you can easily add it to Saturn Cloud.

Select **Images** in the Saturn Cloud toolbar. From here, you'll see the blue **New Image** button at the top right of the screen. Click this, and you'll be taken to a form. 

There, select **Import an external image**, and fill in the correct **Image URI** that leads to your image.

![New image form with Import external image selected](/images/docs/import-external-image.png "doc-image")

Then select the correct **Owner** of the image and whether it is intended for CPU or GPU instances.

## Docker Image Requirements
In order for the Docker image to work correctly in Saturn Cloud, you must have the following:

1. Conda installed in `/opt/conda` with Python ≥ 3.6
2. `sshd` installed on the system
3. Your custom environment installed in `/opt/conda/envs/saturn` -- This can contain anything you wants to use, but it should contain at least `ipykernel`. 
4. `/opt/conda/envs/saturn` registered with Jupyter -- Do this with the following code:

```bash
        ${CONDA_DIR}/envs/saturn/bin/python -m ipykernel install \
                --name python3 \
                --display-name 'saturn (Python 3)' \
                --prefix=${CONDA_DIR}
```

5. A user named `jovyan` with `uid=1000` 

## Recommended Configurations

For a better experience (though not necessary):

1. Add the `jovyan` user to sudo-ers.
2. Set the PATH with the following code:

```bash
        PATH= /opt/conda/envs/saturn/bin:/opt/conda/bin:${PATH} 
```

3. Make sure Python environments are owned by the jovyan user.