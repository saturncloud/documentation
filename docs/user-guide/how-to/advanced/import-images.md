# Import Existing Docker Images |
| `SATURN_SYSTEM_PYTHON` | The `python` executable that has JupyterLab installed. Used to launch the IDE. |
| `SATURN_USER_PYTHON` | The `python` executable for your user environment, with the libraries you work in. |

Set each to the absolute path of a `python` executable. If your image has a single Python that holds both JupyterLab and your libraries, point both variables at it:

```bash
SATURN_USER_PYTHON=/usr/bin/python
SATURN_SYSTEM_PYTHON=/usr/bin/python
```

This is also how you run images that were never built for Saturn Cloud at all. For example, a service image that doesn't ship JupyterLab or a conda environment will still run as a job or deployment once these variables tell Saturn Cloud where its Python lives.

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

Use [Saturn Cloud recipes](https://saturncloud.io/docs/user-guide/how-to/recipes) to add images via the API.

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
