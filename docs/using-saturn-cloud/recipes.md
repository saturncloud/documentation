# Resource Recipes

There are many situations where you may want to store information about the configuration of a Saturn Cloud resource, such as:

-   Storing how an analysis was run so that you can rerun it later
-   Sharing details of a resource with a colleague so they can clone it
-   Saving a configuration so you can programmatically create a new resource with that configuration via an API

The way you store information about a resource is through a **Saturn Cloud resource recipe**. A resource recipe is a YAML file that
stores all the details about a resource, such as the extra libraries installed, the number of workers of a connected Dask cluster, the git repos used, and more. Here is a simple example of what a recipe might look like:

```yaml
type: workspace
spec:
  name: example-dask
  image: saturncloud/saturn-python:2023.09.01
  description: "Use distributed computing with Dask"
  extra_packages:
    pip: lightgbm
  working_directory: /home/jovyan/examples/examples/dask
  git_repositories:
    - url: git@github.com:saturncloud/examples.git
      path: /home/jovyan/workspace/examples
      public: false
      on_restart: preserve changes
      reference: null
      reference_type: branch
  disk_space: "10Gi"
  instance_type: "large"
  dask_cluster:
    num_workers: 3
    scheduler:
      instance_type: medium
    worker:
      instance_type: large
      num_processes: 1
      num_threads: 2
      use_spot_instances: false
state:
  id: 30ea539fdef64274a9a777ada32535ae
  status: running
```

You can see that this resource has the Python package `lightgbm` installed via git, it uses the `saturncloud/examples` GitHub repository, and more. Importantly, at the bottom of the object is the field `version` which indicates which schema is being used for the recipe. The resource recipe schema is constantly evolving as Saturn Cloud gains new features, so it is important to keep track of what schema is being used at the time. **The full JSON schema can be found in the [saturncloud/recipes](https://github.com/saturncloud/recipes) GitHub repo.**

You can download the recipe for an existing resource by going to the resource page and navigating to the **Manage** tab, and then clicking the **Download Recipe** button. This will let you save the JSON file with all the details of the resource. Note that the file will include optional fields, such as the owner of the resource, which are not required for recreating it.

![Recipe download button](/images/docs/recipe-download-button-new.webp "doc-image")

You can also create a resource file from scratch by using the JSON schema found in the [saturncloud/recipes](https://github.com/saturncloud/recipes) GitHub repo.

To create a new resource using a recipe, click the appropriate create new resource button on the resource list page.
At the top you'll see a **Use a Recipe** button. This will let you paste in a recipe for a resource. You will also be able to choose
the owner and name of the new recipe.

![Arrow pointing at recipe create button](/images/docs/recipe-create-button-arrow.webp "doc-image")

## Embedding create from recipe buttons

If you have a particular configuration of a Saturn Cloud resource that you want other people to be able to use, you can create a link or button that when a user
clicks it they will create the resource in their Saturn Cloud account. The tool below will let you generate such links from a URL for a recipe file.

{{<recipeButton>}}

## Recipes for cloning

The recipe encapsulates the state of the resource as well. In the above the state includes the ID of the resource, and it's status

```
state:
  id: 30ea539fdef64274a9a777ada32535ae
  status: running
```

This is useful if you want to use infrastructure as code patterns and you want to update a resource using recipes. However if your goal is to store the specification of the resource, so that it can be cloned, this field (along with other fields like "owner" and "subdomain" are removed since they should not be used when you are cloning a resource (When cloning a resource, you wouldn't clone the run time state since the new resource that has been cloned will always be "stopped" regardless of the state of the recipe that you are cloning. Similarly, the owner field is removed when you clone a recipe, since you often want to clone a resource to a different owner.
