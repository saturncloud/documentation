# Train a Model with XGBoost and Dask - Tutorial

<div class="row">
  <div class="column">
    <img src="/images/docs/dask-logo-horizontal.svg" style="width:300px;" class="doc-image">
  </div>
  <div class="column">
    <img src="/images/docs/xgboost-logo-transparent.png" style="width:275px;" class="doc-image">
  </div>
</div>

## Overview

`XGBoost` is a popular algorithm for supervised learning with tabular data. It has been used in many winning solutions in data science competitions, and <a href="https://www.capitalone.com/tech/machine-learning/how-to-control-your-xgboost-model/" target="_blank" rel="noopener">in real-world solutions at large enterprises like Capital One</a>.

The `xgboost` Python package allows for efficient single-machine training using multithreading. However, the amount of training data you can use is limited by the size of that one machine. To solve this problem, XGBoost supports distributed training using several different interfaces. This tutorial explores distributed XGBoost training using <a href="https://dask.org/" target="_blank" rel="noopener">Dask</a>.

In this tutorial, you'll learn:

* how distributed training works for XGBoost, and what types of performance changes you can expect when you add more data or machines
* how to use Dask to accelerate XGBoost model training
* how to diagnose performance problems and speed up training

This tutorial does NOT cover how to improve the statistical performance of XGBoost models. There is already excellent coverage of that topic on the internet.

If you just want to test out some XGBoost + Dask code without a lot of other discussion, create the `Data Science Pipeline` project in your Saturn account and run the notebook <a href="https://github.com/saturncloud/examples/blob/main/examples/data-science-pipeline/xgboost-dask.ipynb" target="_blank" rel="noopener">nyc-taxi/xgboost-dask.ipynb</a>.

<hr>

## Part 1: How Distributed Training for XGBoost Works

### How XGBoost Training Can Be Parallelized

This tutorial assumes that you have some familiarity with Gradient Boosted Decision Trees (GBDTs) generally and XGBoost specifically. If you do not, please refer to <a href="https://xgboost.readthedocs.io/en/latest/tutorials/model.html" target="_blank" rel="noopener">"Introduction to Boosted Trees"</a> in the XGBoost documentation, then return to this tutorial.

In XGBoost, like in other gradient-boosted decision tree algorithms, trees are built one node at a time. When building a tree, a set of possible "split points" (tuples of `(feature, threshold)`) is generated, and then each split point is evaluated. The one that leads to the best reduction in loss is chosen and added to the tree. Nodes are added to the tree this way until tree-specific stopping conditions are met. Trees are added to the model this way until model-wide stopping conditions are met. To make a prediction with the model, you pass new data through all of the trees and add the output of the trees together.

Because trees need to be built sequentially like this, GBDTs can't benefit from building trees in parallel the way that random forests can. However, they can still be sped up by adding more memory and CPU. This is because the work that needs to be done to choose a new tree node is parallelizable.

Consider this very-oversimplified pseudocode for how one new node is added to a tree in XGBoost.

```python
for split_point in split_points:
    loss = 0
    new_model = global_model.copy()
    new_model.current_tree.add(split_point)
    for observation in data:
        predicted_value = new_model.predict(observation)
        loss += loss_function(predicted_value, observation.actual_value)
```

When you see a `for` loop or something being summed, there's opportunity to parallelize! That is exactly where XGBoost is able to apply parallelism to speed up training. The main training process can assign subsets of the data to multiple threads, and tell each of them "evaluate all the split points against your data and tell me how it goes". Adding more CPUs or GPUs allows you to efficiently use more threads, which means more subsets can be evaluated at the same time, which reduces the total time it takes for each of these "add a node to the tree" steps.

To get access to more CPUs or GPUs, you have these options:

* get a bigger machine
* get more machines

When XGBoost and other machine learning libraries talk about "distributed training", they're talking about the choice "get more machines".

### How XGBoost Distributed Training Works

In the single-machine case, parallelization is really efficient because those threads created by the main program don't need to move any data around. They all have shared access to the dataset in memory, and each access different pieces of it.

In a distributed setting, with multiple machines, this is more complicated. Subsets of the training data need to be physically located on the machines that are contributing to training. These machines are called "workers".

XGBoost distributed training uses a project called Reliable AllReduce and Broadcast Interface (<a href="https://github.com/dmlc/xgboost/tree/master/rabit" target="_blank" rel="noopener"><code>rabit</code></a>) to manage the training process. The `rabit` "tracker" process is responsible for understanding which workers exist, and which partitions of the data they have. Each worker runs a `rabit` "worker" process, which is responsible for communication with the tracker.

Each time a node is added to a tree, XGBoost has to first has to decide on the set of split points that should be considered. Each worker generates a list of these based on its own subset of the training data. Because generating a list of all possible split points is often not feasible, workers generate histograms for each feature, and only the bin boundaries in those histograms are considered as possible split points. Workers also sum up the contribution of rows in each histogram range to the overall loss. They then perform AllReduce, sending these "gradient histograms" to each other until all workers have identical copies of the global merged gradient histogram.

During these "sync-ups", where workers are all passing messages to each other to create the global merged view of the model, the workers cannot start on adding another node to the tree. If the number and size of workers that you have available is fixed, then one of the ways you can reduce training time is to reduce how much total time is spent in those sync-ups.

The number of times this sync-up has to happen is determined by the total number of tree nodes that will be created across all trees, which depends on:

* number of nodes per tree: `tree_method`, `max_depth`, `min_child_weight`, `max_leaves` + `grow_policy`
* number of trees: `num_round`, `early_stopping_rounds`

The amount of time it takes for each of these sync-ups is determined by the amount of data that has to be communicated, which depends on:

* the number of features (columns in the training data)
* the number of bins in the gradient histograms (default 256, `max_bin` if using `tree_method="hist"`)

See <a href="https://discuss.xgboost.ai/t/dask-is-distributed-training-globally-data-parallel-and-locally-feature-parallel/1929/4" target="_blank" rel="noopener">this fantastic description from Hyunsu Cho on the XGBoost discussion boards</a> for more details.

<hr>

## Part 2: XGBoost Training with Dask

In Part 1, you learned some of the intuition behind distributed XGBoost training. In this section, you'll get hands-on exposure to distributed XGBoost training with Dask.

### Set Up Environment

In Saturn, go to the "Projects" page and create a new project called `xgboost-tutorial`. Choose the following settings:

- **image**: saturncloud/saturn:2020.10.23
- **disk space**: 10 Gi
- **size**: Medium - 2 cores - 4 GB RAM
- **start script**:
    - ```shell
      pip install 'xgboost>=1.3.0'
      ```

Start up that project's Jupyter, and go into Jupyter Lab.

Import the libraries used in this example.

```python
import dask.array as da
import dask.dataframe as dd
import xgboost as xgb

from dask.distributed import Client, wait
from dask_ml.metrics import mean_absolute_error
from dask_saturn import SaturnCluster
```

### Set Up Dask Cluster

Next, set up your Dask cluster. Saturn Cloud allows you to do this programmatically via <a href="https://github.com/saturncloud/dask-saturn" target="_blank" rel="noopener"><code>dask-saturn</code></a>.

```python
n_workers = 3
cluster = SaturnCluster(
    n_workers=n_workers,
    scheduler_size='medium',
    worker_size='4xlarge',
    nprocs=1
)
client = Client(cluster)
client.wait_for_workers(n_workers)
cluster
```

The code above creates a Dask cluster with 3 `4xlarge` workers. Each of these has 16 cores and 128 GB RAM.

{{% alert title="Instance Sizes in Saturn" %}}
To see a list of all available instance sizes in Saturn, run <code style="color: red;">dask_saturn.describe_sizes()</code>.
{{% /alert %}}

Notice that that code does not set `nthreads` in the cluster. That is important. When `nthreads` isn't set, Dask's default behavior is to use all available cores on each worker. This is the ideal setting for running workloads like machine learning training.

### Load Data

`xgboost` allows you to train on Dask collections, like <a href="https://docs.dask.org/en/latest/dataframe.html" target="_blank" rel="noopener">Dask DataFrames</a> and <a href="https://examples.dask.org/array.html" target="_blank" rel="noopener">Dask Arrays</a>. This is really powerful because it means that you never have to have a single machine that's big enough for all of your training data.

These collections are *lazy*, which means that even though they look and feel like a single matrix, they're actually more like a list of function calls that each return a chunk of a matrix. With these collections, you don't need to (and shouldn't!) create the entire dataset on one machine and then distribute it to all of the workers. Instead, Dask will tell each of the workers "hey, read these couple chunks into your memory", and each worker does that on their own and keeps track of their own piece of the total dataset.

There is a lot more information on this in <a href="https://examples.dask.org/dataframe.html" target="_blank" rel="noopener">the Dask documentation</a>.

For this tutorial, create some random numeric data using `dask.array`.

```python
num_rows = 1e6
num_features = 100
num_partitions = 10
rows_per_chunk = num_rows / num_partitions

data = da.random.random(
    size=(num_rows, num_features),
    chunks=(rows_per_chunk, num_features)
)

labels = da.random.random(
    size=(num_rows, 1),
    chunks=(rows_per_chunk, 1)
)
```

At this point, `data` and `labels` are lazy collections. They won't be read into the workers' memory until some other computation asks for them. If we start training a model right now, the training will block until those arrays have been materialized. Dask's garbage collection might also remove them from memory when training ends, which means they'll have to be recreated if we want to train another model.

To avoid this situation and to make iterating on the model faster, use `persist()`. This says "hey Dask, go materialize all the pieces of this matrix and then keep them in memory until I ask you to delete them".

```python
data = data.persist()
labels = labels.persist()

_ = wait(data)
_ = wait(labels)
```

Training data for `xgboost.dask` needs to be prepared in a special object called `DaskDMatrix`. This is like the XGBoost `DMatrix` that you might be familiar with, but is backed by Dask's distributed collections (Dask DataFrame and Dask Array).

```python
dtrain = xgb.dask.DaskDMatrix(
    client=client,
    data=taxi_train[features],
    label=taxi_train[y_col]
)
```

### Train

Now that the data have been set up, it's time to train a model!

```python
xgb_params = {
    "verbosity": 1,
    "max_depth": 5,
    "random_state": 708,
    "objective": "reg:squarederror",
    "learning_rate": 0.1,
    "tree_method": "hist"
}

n_rounds = 50

result = xgb.dask.train(
    client=client,
    params=xgb_params,
    dtrain=dtrain,
    num_boost_round=n_rounds
)
```

The `train()` function from `xgboost` accepts any parameters that can be passed to `xgboost.train()`, with one exception: `nthread`. Any value for `nthread` that you pass will be ignored by `xgboost`, since that library resets `nthread` to the number of logical cores on each Dask worker.

For details on the supported parameters in `xgboost`, see <a href="https://xgboost.readthedocs.io/en/latest/parameter.html" target="_blank" rel="noopener">the XGBoost parameter docs</a>.

### Evaluate

`xgb.dask.train()` produces a regular `xgb.core.Booster` object, the same model object produced by single-machine training.

```python
booster = result["booster"]
type(booster)

# xgboost.core.Booster
```

`xgboost.dask.predict()` can be used to create predictions on a Dask collection using an XGBoost model object. Because the model object here is just a regular XGBoost model, using `xgboost.dask` for batch scoring doesn't require that you also perform training on Dask.

Run the code below to create a validation set and test how well the model we trained in previous steps performs.

```python
holdout_data = da.random.random(
    size=(1000, num_features),
    chunks=(50, num_features)
)

holdout_labels = da.random.random(
    size=(num_rows, 1),
    chunks=(rows_per_chunk, 1)
)

preds = xgb.dask.predict(
    client=client,
    model=bst,
    data=holdout_data
)
```

Given these predictions, you can use the built-in metrics from `dask-ml` to evaluate how well the model performed. These metrics functions are similar to those from `scikit-learn`, but take advantage of the fact that their inputs are Dask collections. For example, `dask_ml.metrics.mean_absolute_error()`, will split up responsibility for computing the MAE into many smaller tasks that work on subsets of the validation data. This runs faster than doing the predictions on the client, and allows you to perform evaluations on very large datasets without needing to move those datasets around or pull them back to the client.

```python
from dask_ml.metrics import mean_absolute_error

mae = mean_absolute_error(
    y_true=holdout_labels,
    y_pred=preds,
    compute=true
)

print(f"Mean Absolute Error: {mae}")
```

### Deploy

Like the previous section mentioned, the model object produced by `xgb.dask.train()` isn't a special Dask-y model object. It's a normal `xgboost.core.Booster`. You can save this model using any of the methods described in <a href="https://xgboost.readthedocs.io/en/latest/tutorials/saving_model.html" target="_blank" rel="noopener">the "Model IO" section of the XGBoost docs</a>.

<hr>

## Part 3: Speeding Up Training

In the previous sections, you learned how to train and evaluate an XGBoost model using Dask. This sections describes how to speed up model training.

### Change Dask Settings

Before getting into the specifics of LightGBM, you might be able to speed up training by taking one of the following actions that are generally useful in machine learning training on Dask.

* [restart the workers](<docs/Reference/troubleshooting_dask.md>)
* [increase the worker size](<docs/Reference/troubleshooting_dask.md>)
* [add more workers](<docs/Reference/troubleshooting_dask.md>)
* [use all available cores on each Dask worker](<docs/Reference/troubleshooting_dask.md>)
* [repartition your training data](<docs/Reference/troubleshooting_dask.md>)

### Make Your Model Smaller

In ["How XGBoost Distributed Training Works"](<docs/Examples/LoadData/qs-snowflake-dask.md#how-xgboost-distributed-training-works>), you learned how different parameters of characteristics of the input data impact training time. This section describes how to adjust those parameters to improve training time.

Note that some of these changes might result in worse statistical performance.

#### Grow Smaller Trees

To grow shallower trees, you can change the following XGBoost parameters. Growing shallower trees might also act as a form of regularization that improves the statistical performance of your model.

**Decrease `max_depth`**

This parameter is an integer that controls the maximum distance between the root node of each tree and a leaf node. Decrease `max_depth` to reduce training time.

**Decrease `max_leaves` if `grow_policy="lossguide"`**

By default, XGBoost grows trees level-wise. This means that it will add splits one at a time until all leaf nodes at a given depth have been split, and then will move on to the next level down. For a given value of `max_depth`, this might produce a larger tree than depth-first growth, where new splits are added based on their impact on the loss function.

You can speed up training by switching to depth-first tree growth. To do this in XGBoost, set the `grow_policy` parameter to `"lossguide"`. To simplify the choice for managing complexity with `grow_policy="lossguide"`, set `max_depth` to 0 (no depth limit) and just use `max_leaves` to control the complexity of the trees instead.

```python
xgb_params = {
    "objective": "reg:squarederror",
    "tree_method": "approx",
    "grow_policy": "lossguide",
    "max_leaves": 31,
    "max_depth": 0
}
```

When choosing values for these parameters, consider how different values impact the maximum number of leaves per tree. This is a "maximum" because trees might never reach the full `max_leaves` or `max_depth` if they fail to meet other tree-level stopping conditions like restrictions on the minimum number of data points that must fall in each leaf nodes.

In a tree that is grown level-wise, the maximum number of nodes in the tree is 2<sup>max_depth</sup>-1. That means changing from `{"grow_policy": "depthwise", "max_depth": 5}` to `{"grow_policy": "lossguide", "max_depth": 0, "max_leaves": 31}` will not change the maximum possible number of nodes per tree.

For more information, see <a href="https://lightgbm.readthedocs.io/en/latest/Features.html#leaf-wise-best-first-tree-growth" target="_blank" rel="noopener">"Leaf-wise (Best-first) Tree Growth"</a> in the LightGBM docs.

**Increase `min_child_weight`**

This parameter is a number (default: 1.0) that prevents the creation of a new node if it is "too pure". This is the sum of the second derivative of the loss function over all points. For some regression objectives, this is just the minimum number of records that have to fall into each node. For classification objectives, it represents a sum over a distribution of probabilities. See <a href="https://stats.stackexchange.com/a/323459" target="_blank" rel="noopener">this Stack Overflow answer</a> for a more detailed description.

#### Grow Fewer Trees

XGBoost allows two methods to control the number of total trees in the model.

**Decrease ``num_round``**

The ``num_round`` parameter controls the number of boosting rounds that will be performed. Since it uses decision trees as the learners, this can also be thought of as "number of trees".

Choosing the right value of `num_round` is highly dependent on the data and objective, so this parameter is often chosen from a set of possible values through hyperparameter tuning.

**Use early stopping**

When you ask XGBoost to train a model with ``num_round = 100``, it will perform 100 boosting rounds. If the difference in training fit between, say, round 80 and round 100 is very small, then you could argue that waiting for those final 20 iterations to complete wasn't worth the time.

This is where early stopping comes in. If early stopping is enabled, after each boosting round the model's performance is evaluated against a validation set that contains data not available to the training process. That performance is then compared to the performance as of the previous boosting round. If the model's performance fails to improve for some number of consecutive rounds, XGBoost decides "ok, this model is not going to get any better" and stops the training process.

That "number of consecutive rounds" is controlled by the parameter `early_stopping_rounds`. For example, `early_stopping_rounds=1` says *"the first time performance on the validation set does not improve, stop training"*. In practice, it's usually better to choose a value higher than 1 since there can be some randomness in the change in validation performance.

To enable early stopping, use the keyword argument `early_stopping_rounds` and provide a validation set in keyword argument `evals`.

<details><summary>code sample (click me)</summary>

```python
import dask.array as da
import xgboost as xgb

num_rows = 1e6
num_features = 100
num_partitions = 10
rows_per_chunk = num_rows / num_partitions

data = da.random.random(
    size=(num_rows, num_features),
    chunks=(rows_per_chunk, num_features)
)

labels = da.random.random(
    size=(num_rows, 1),
    chunks=(rows_per_chunk, 1)
)

X_eval = da.random.random(
    size=(num_rows, num_features),
    chunks=(rows_per_chunk, num_features)
)

y_eval = da.random.random(
    size=(num_rows, 1),
    chunks=(rows_per_chunk, 1)
)

dtrain = xgb.dask.DaskDMatrix(
    client=client,
    data=data,
    label=labels
)

dvalid = xgb.dask.DaskDMatrix(
    client=client,
    data=X_eval,
    label=y_eval
)

result = xgb.dask.train(
    client=client,
    params={
        "objective": "reg:squarederror",
    },
    dtrain=dtrain,
    num_boost_round=10,
    evals=[(dvalid, "valid1")],
    early_stopping_rounds=3
)
bst = result["booster"]

# check value of eval metric(s) at each iteration
print(result["history"])

# check which iteration was best
print(f"best iteration: {bst.best_iteration}")
```
</details>

#### Consider Fewer Splits

In any tree-based supervised learning model, the process of growing each tree involves evaluating "split points". A split point is a combination of a feature and a threshold on that feature's values.

<img src="/images/docs/decision-tree.png" style="width:500px;" alt="Tree diagram showing a decision tree with thresholds and classification" class="doc-image">

*image credit:* <a href="https://pubs.rsc.org/en/content/articlelanding/2009/MB/b907946g#!divAbstract" target="_blank" rel="noopener">Geurts et al (2009), Royal Society of Chemistry</a>

You can substantially reduce the training time by decreasing the number of these "split points" that must be considered to create a new node in the tree.

Per <a href="https://discuss.xgboost.ai/t/xgboost-distribute-training-is-slow/966/2" target="_blank" rel="noopener">the XGBoost maintainers</a>:

> The amount of AllReduce communication linearly increases with respect to the number of features.

> [you should worry about] the bottleneck in AllReduce, as we have to communicate histograms of size M * K * T. It can be quite big, if we have high-dimensional data (large M) or if we grow deep trees (large T). ...the number of bins in the histograms is M * K * T, where M is the number of features, K is the number of quantiles (by default 256), and T is the number of tree nodes added so far in the tree.

**Perform Feature Selection**

To reduce the number of features, consider adding a preprocessing step that trains a model on a smaller dataset and performs feature selection.

**Decrease `max_bins`**

It isn't practical for XGBoost to consider every possible split of every feature, so it approximates that process by binning continuous features into histograms. XGBoost parameter `max_bins` controls how many bins are created, and defaults to 256. Decreasing this value will result in fewer splits being considered each time a node is added to a tree, but might hurt statistical performance.

## Conclusion

In this tutorial, you learned how distributed training can speed up XGBoost workflows and / or allow you to train models using more input data. You also saw how to implement this using Python and Dask.

To learn more about these topics, see the links below.

**Dask**

* <a href="https://distributed.dask.org/en/latest/manage-computation.html" target="_blank" rel="noopener">Managing Computation (docs.dask.org)</a>
* <a href="https://distributed.dask.org/en/latest/memory.html" target="_blank" rel="noopener">Managing Memory (docs.dask.org)</a>
* <a href="https://docs.dask.org/en/latest/best-practices.html#processes-and-threads" target="_blank" rel="noopener">Processes and Threads (docs.dask.org)</a>

**GBDTs**

* <a href="https://datascience.stackexchange.com/a/26950" target="_blank" rel="noopener">Depth-First vs. Breadth-First Tree Growth (Stack Overflow)</a>
* <a href="https://xgboost.readthedocs.io/en/latest/tutorials/model.html" target="_blank" rel="noopener">Introduction to Boosted Trees (XGBoost docs)</a>

**XGBoost**

* <a href="https://discuss.xgboost.ai/t/dask-is-distributed-training-globally-data-parallel-and-locally-feature-parallel/1929" target="_blank" rel="noopener">Bottlenecks in XGBoost distributed training (XGBoost discussion board)</a>
* <a href="https://stats.stackexchange.com/a/323459" target="_blank" rel="noopener">Explanation of <code>min_child_weight</code> for XGBoost (Stack Overflow)</a>
* <a href="https://stackoverflow.com/questions/34151051/how-does-xgboost-do-parallel-computation#:~:text=1%20Answer&text=Xgboost%20doesn't%20run%20multiple,and%20run%20with%20n_rounds%3D1" target="_blank" rel="noopener">How does XGBoost do parallel computation? (Stack Overflow)</a>
* <a href="https://www.capitalone.com/tech/machine-learning/how-to-control-your-xgboost-model/" target="_blank" rel="noopener">How to control your XGBoost model (Capital One)</a>
* <a href="https://xgboost.readthedocs.io/en/latest/tutorials/dask.html#threads" target="_blank" rel="noopener">How <code>xgboost.dask</code> handles multithreading (XGBoost docs)</a>
* <a href="https://www.kdd.org/kdd2016/papers/files/rfp0697-chenAemb.pdf" target="_blank" rel="noopener">XGBoost: A Scalable Tree Boosting System (KDD)</a>
* <a href="https://discuss.xgboost.ai/t/xgboost-distribute-training-is-slow/966/2" target="_blank" rel="noopener">"XGBoost distributed training is slow" (XGBoost discussion board)</a>
