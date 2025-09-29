# Import Existing Docker Images

If you already have a Docker image in your personal or business repository, you can easily add it to Saturn Cloud.

## Docker Image Requirements

Docker images must have certain software installed, depending on what type of resource they are being used for.
For examples of Docker image installation processes, see the Saturn Cloud [images GitHub repo](https://github.com/saturncloud/images). Currently Saturn Cloud scopes IAM permissions to restrict access to ECR images named `saturn*`. If you want to use images with Saturn Cloud, please name them accordingly.

### Docker Image Requirements - Jupyter Servers

Jupyter servers must have Python and JupyterLab installed in a particular way using the steps below.

1. Conda installed in `/opt/saturncloud` with Python ≥ 3.7.
2. (optional) `sshd` installed on the system.
3. Your custom environment installed in `/opt/saturncloud/envs/saturn` -- This can contain anything you wants to use, but it should contain at least `ipykernel`.
4. `/opt/saturncloud/envs/saturn` registered with Jupyter -- Do this with the following code:

```bash
        ${CONDA_DIR}/envs/saturn/bin/python -m ipykernel install \
                --name python3 \
                --display-name 'saturn (Python 3)' \
                --prefix=${CONDA_DIR}
```

5. A user named `jovyan` with `uid=1000` with home set to `/home/jovyan`. For workspaces, EBS volumes are mounted on top of `/home/jovyan`, so you should not persist configuration files there.

These steps are not required but will make the user experience better:

6. Add the `jovyan` user to sudo-ers.
7. Set the PATH with the following code:

```bash
        PATH= /opt/saturncloud/envs/saturn/bin:/opt/saturncloud/bin:${PATH}
```

8. Make sure Python environments are owned by the jovyan user.

### Docker Image Requirements - R Servers

R servers must have R, RStudio, and Python all installed.

1. Conda installed in `/opt/saturncloud` with Python ≥ 3.7.
2. `sshd` installed on the system.
3. A user named `jovyan` with `uid=1000` with home set to `/home/jovyan`. For workspaces, EBS volumes are mounted on top of `/home/jovyan`, so you should not persist configuration files there.
4. R installed to either `/usr/lib/R` or `/usr/local/lib/R`. The `jovyan` user must have recursive ownership and read/write access to the folder.
5. RStudio Server Open Source installed, with configuration files configured to the minimums matching those in [saturncloud/saturnbase-r](https://github.com/saturncloud/images/tree/main/saturnbase-r).
6. The Renviron file located at either `/usr/lib/R/etc/Renviron` or `/usr/local/lib/R/etc/Renviron` must be modified to include the line below. This ensures R will correctly read the installed packages when using RStudio.

```
R_LIBS=/usr/local/lib/R:/usr/local/lib/R/site-library:/usr/lib/R/site-library:/usr/lib/R/library
```

### Docker Image Requirements - Jobs and Deployments

The requirements for job and deployment resources are simpler than the other types of resources since no IDE is required. _All images that are used for Jupyter server and R server resources will also work for jobs and deployments._

1. Conda installed in `/opt/saturncloud` with Python ≥ 3.7.
2. `sshd` installed on the system.
3. A user named `jovyan` with `uid=1000` and their home directory created.

## Adding the Image to Saturn Cloud

External images can be added to Saturn Cloud using two methods: the UI or the API. The UI is useful for one-off image additions, while the API can be used as part of a CI/CD pipeline for adding new images.

### Adding an image using the UI

Select **Images** from the left-hand menu in Saturn Cloud. From here, you'll see the blue **New** button at the top right of the screen. Click this, and you'll be taken to a form.

![Arrow pointing at button to create new image](/images/docs/create-image-arrow.webp "doc-image")

There, you will be presented with the following form. For descriptions of image options, see our [page on creating a new image](/docs#create-an-image-within-saturn-cloud).

![New image options](/images/docs/new-image-form-2.webp "doc-image")

Click **Add** to create the new image. To finish importing your image, you will next need to create a valid version. From the image's page, select **Add External** from the dropdown menu next to **New Version** in the top right.

<img src="/images/docs/add-external-image.webp" style="width:200px;" alt="New Version dropdown showing Add External image option" class="doc-image">

There, fill in the name, optional description, and correct **Image URI** that leads to your image. Then, click **Add** to finish importing your image.

![New external image form](/images/docs/import-external-image2.webp "doc-image")

### Adding an image using the API

Use [Saturn Cloud recipes](https://saturncloud.io/docs/docs/user-guide/recipes/) to add images via the API.

> **Note** - these instructions will only work for Saturn Cloud versions that are greater than or equal to 2022.04.01. Please contact support for instructions that will work on prior Saturn Cloud versions.

First, define the image recipe in json format. Below you can see an example of the images recipe. This particular example adds two images: one titled "saturn" with a version called "2021.11.10" and another titled "saturn-r" with versions "2022.06.01" and "2022.09.01". You can add a single version of an image or multiple versions in a similar manner. See [the full example and schema](https://github.com/saturncloud/recipes/tree/main/images) for complete image recipe specifications.

```json
{
    "images": [
        {
            "name": "saturn",
            "hardware_type": "cpu",
            "versions": [
                {
                    "name": "2021.11.10",
                    "image_uri": "public.ecr.aws/saturncloud/saturn:2021.11.10"
                }
            ],
            "schema_version": "2022.04.01"
        },
        {
            "name": "saturn-r",
            "supports": ["dask", "rstudio-opensource"],
            "hardware_type": "cpu",
            "versions": [
                {
                    "name": "2021.11.10",
                    "image_uri": "public.ecr.aws/saturncloud/saturn-r:2022.06.01"
                },
                {
                    "name": "2021.12.01",
                    "image_uri": "public.ecr.aws/saturncloud/saturn-r:2022.09.01"
                }
            ],
            "schema_version": "2022.04.01"
        }
    ]
}
```

Additionally, you will need a header containing the User Token of an administrator account in the below format. You can find your User Token under **Username > User Settings > User Token**.

```
"Authorization: Bearer <User Token>"
```

Once you have the json recipe and header, POST it to: `<Your Saturn Cloud Base URL>/api/recipes/apply`

For example: `https://app.community.saturnenterprise.io/api/recipes/apply`

The image will then be available to use in your Saturn Cloud resources under your default organization.

> **Note** - If you want to get a list of the images currently available in your Saturn Cloud organization, run a GET command to `<Your Saturn Cloud Base URL>/api/recipes` using your authorization header.

{{% images_docs_view %}}
