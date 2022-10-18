# Using the targets Package


## Overview
[Targets](https://cran.r-project.org/web/packages/targets/index.html) is a pipeline toolkit for R. It allows for reproducible workflows without unnecessarily repeating calculations. It also can use parallel backends ([future](https://cran.r-project.org/web/packages/future/index.html) or [clustermq](https://github.com/mschubert/clustermq)). 

To illustrate this package, we use the same data, model, and functions that were used in the [furrr](<docs/examples/r/future/qs-r-furrr.md>) example. See that example for a thorough explanation. All these functions are contained in the "functions.R" file in this repository.

We will be using the future backend for parallel processing. Learn more about how the future package works [here](<docs/examples/r/future/qs-r-future.md>).

## Modeling Process
### Imports
The only library we need to import now is the targets library itself. All other libraries are imported during the workflow process.

```{r imports}
library(targets)
```

### Use a "_targets.R" file
First, we need a file named "_targets.R". This file will contain all the information for targets to create a workflow.

The "_targets.R" file does the following:

* Imports the appropriate libraries
* Imports the functions from "functions.R"
* Sets global options for targets, including the packages that each target node needs
* Creates an execution environment for the future package
* Creates a list of target nodes

Target nodes are defined in a list using the function `tar_target()`. Each target node is a single step in a workflow. A node runs an R command and returns a value. 

[`tar_target`](https://docs.ropensci.org/targets/reference/tar_target.html) has several inputs, but the important ones here are:

* **name**: The name of the target node -- Targets are referenced by name, so downstream nodes can reference upstream nodes by name.
* **command**: The R function to run
* **format**: A storage format for the return value -- This can have considerable positive effects to run time if you are moving large files around.
* **deployment**: This dictates where the function is to be run. It can either be "main" or "worker." Many functions in this graph will not be improved by parallelization, so they are run on the "main" process.

Below is an example target node. Take a look at the "_targets.R" file for more information.

```{r tar target, eval = FALSE}
tar_target(
    preprocessed_data,
    preprocess_data(data),
    format = "qs",
    deployment = "main"
)
```

### Show the Target Graph
Once the target workflow is defined by the "_targets.R" file, we can take a look at the resulting directed acyclic graph (DAG). Running `tar_visnetwork()` will output a DAG that shows the relationship between target nodes. This can be very useful when you are first setting up a workflow.

```{r tar_visnetwork}
tar_visnetwork()
```

Below is an example of the DAG for a workflow. As you can see, each node is connected and has a status.

![Example DAG](https://saturn-public-assets.s3.us-east-2.amazonaws.com/example-resources/targets-DAG-legend.png "doc-image")


### Watch the Progress
If you want to see a live view of the graph and various statuses, you can run the `tar_watch()` command. This command opens a shiny app that displays a variety of information. It is updated, by default, every 10 seconds.

```{r tar_watch}
tar_watch()
```

You can see an example for a pipeline below:

![Example tar_watch](https://saturn-public-assets.s3.us-east-2.amazonaws.com/example-resources/targets-watch.png "doc-image")

### Run the Targets Workflow

Finally, let's actually run the workflow. In this case, we are using `tar_make_future()` because we want to use the future parallel backend. Because we defined our plan as multisession in the "_targets.R" file, the code we marked to send to workers will be sent to appropriate future processes.

```{r tar_run}
tar_make_future(workers = 8)
```

That's it! This particular workflow takes approximately five minutes to compute all the hyperparameter options. You can find the complete results in the "report.html" document.

If, for instance, you stop the computation in the middle to change a parameter, targets will remember what has already been computed and skip running those steps in the next run. Pretty neat!

## Conclusion
The targets package is fantastic for writing pipeline code. It has the added benefit of running on parallel backends like future.  

Thanks to [Deep Learning With Keras To Predict Customer Churn](https://blogs.rstudio.com/ai/posts/2018-01-11-keras-customer-churn/) and the [Targets R Package Keras Model Example](https://github.com/wlandau/targets-keras) for the inspiration for this article.
