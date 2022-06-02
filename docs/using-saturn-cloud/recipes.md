# Resource Recipes

There are many situations where you may want to store information about the configuration of a Saturn Cloud resource, such as:

-   Storing how an analysis was run so that you can rerun it later
-   Sharing details of a resource with a colleague so they can clone it
-   Saving a configuration so you can programmatically create a new resource with that configuration via an API

The way you store information about a resource is through a **Saturn Cloud resource recipe**. A resource recipe is a JSON file that
stores all the details about a resource, such as the extra libraries installed, the number of workers of a
connected Dask cluster, the git repos used, and more. Here is a simple example of what a recipe might look like:

```json
{
    "name": "example-dask",
    "image_uri": "public.ecr.aws/saturncloud/saturn:2022.01.06",
    "description": "Use distributed computing with Dask",
    "extra_packages": {
        "pip": "lightgbm"
    },
    "working_directory": "/home/jovyan/examples/examples/dask",
    "git_repositories": [
        {
            "url": "https://github.com/saturncloud/examples"
        }
    ],
    "jupyter_server": {
        "disk_space": "10Gi",
        "instance_type": "large"
    },
    "dask_cluster": {
        "num_workers": 3,
        "worker": { "instance_type": "large" },
        "scheduler": { "instance_type": "large" }
    },
    "version": "2022.01.06"
}
```

You can see that this resource has the Python package `lightgbm` installed via git, it uses the `saturncloud/examples` GitHub repository,
and more. Importantly, at the bottom of the object is the field `version` which indicates which schema is being used for the recipe. The
resource recipe schema is constantly evolving as Saturn Cloud gains new features, so it is important to keep track of what schema is being used
at the time. **The full JSON schema can be found in the [saturncloud/recipes](https://github.com/saturncloud/recipes) GitHub repo.**

You can download the recipe for an existing resource by going to the resource details page and clicking the **Download Recipe** button. This
will let you save the JSON file with all the details of the resource. Note that the file will include optional fields, such as the owner of the resource,
which are not required for recreating it.

![Recipe download button](/images/docs/recipe-download-button.png "doc-image")

You can also create a resource file from scratch by using the JSON schema found in the [saturncloud/recipes](https://github.com/saturncloud/recipes) GitHub repo.

To create a new resource using a recipe, click the appropriate create new resource button on the resource list page.
At the top you'll see a **Use a Recipe** button. This will let you paste in a recipe for a resource. You will also be able to choose
the owner and name of the new recipe to make.

![Recipe create button](/images/docs/recipe-create-button.png "doc-image")

## Embedding create from recipe buttons

If you have a particular configuration of a Saturn Cloud resource that you want other people to be able to use, you can create a link or button that when a user
clicks it they will create the resource in their Saturn Cloud account. The tool below will let you generate such links from a URL for a recipe file.

{{<recipeButton>}}
