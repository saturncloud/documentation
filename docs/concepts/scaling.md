# Scalability

Saturn Cloud makes it easy to scale your computations, either on a single machine or on a cluster of machines.

### Single Machine Scaling

CPU instances can scale up to 4TB of RAM. GPU instances can scale up to 8 T4, A10, or V100 GPUs. Enterprise customers can also request different additional types. Though scaling on a single machine is certainly less exciting, it is smpler and easier to do for most users.

Saturn Cloud users on single machines typically scale workloads with

- Python multiprocessing
- Local Dask clusters
- Local Ray clusters

In addition, GPU machines can have up to 8 GPUs. PyTorch and Tensorflow both have toolkits for leveraging multiple GPUs during training.

### Multi-node scaling

Multi-node scaling is useful for several reasons
- it allows you to consume more RAM and CPU than is available on a single machine
- it allows you to consume more GPUs than is available on a single machine
- it allows you to scale up and down as your computational needs change.

Again - single node scaling is much easier than multi-node scaling. You should always test/debug your code with single node scaling before proceding to multi-node scaling.

#### Dask

Dask is a Python framework for executing graphs of function calls over a Cluster. Saturn Cloud has built in support for multi-node Dask clusters that can be attached to Saturn Cloud workspaces, deployments, and jobs. Saturn Cloud dask clusters inherit the exact same software environment that they are attached to. This includes

- The same docker image
- The same secrets
- The same additional pacakges and start up scripts

Dask is often used by data scientists both to scale up to large datasets, as well as to run experiments

##### Dask Collections

This article will not go deep into Dask collections, but there are numerous tutorials for working with these, some by Saturn Cloud.

As mentioned before, Dask is a framework for executing graphs of function calls. Dask collections leverage those graphs, to implement parallel data structures. There are 3 main types

- `dask.array`. This is sort of like a a parallel version of a NumPy array.
- `dask.bag`. This is sort of like a parallel version of a Python list.
- `dask.dataframe`. This is sort of like a parallel version of a Pandas DataFrame.

##### Dask Delayed

Whenever possible, we encourage users to leverage the dask function call directly using `dask.delayed`. `dask.delayed` is a decorator that can be used to turn Python function calls into functions that can be executed in Parallel over a dask cluster. Dask is particularly good for this because the Dask dashboard makes it easy to understand what is happening on your cluster, and Dask propagates remote errors back to the client which makes it much easier to troubleshoot parallel jobs.

#### Saturn Run

[Saturn Run](https://github.com/saturncloud/saturn-run) is a library developed on top of other parallel computation frameworks (Dask included) which lets you leverage clusters for scheduling command line jobs. Jobs are configured via 2 yaml files. The first specifies the execution environment:

```
executor:
  class_spec: DaskExecutor
  cluster_class: SaturnCluster
results:
  class_spec: S3Results
  s3_url: s3://saturn-internal-s3-test/saturn-run-2022.12.13/{name}/
file_syncs:
  - src: /home/jovyan/workspace/julia-example/

```

The second configures the specific tasks.

```
tasks:
  - command: julia /home/jovyan/workspace/julia-example/fibonacci.jl 12
  - command: julia /home/jovyan/workspace/julia-example/fibonacci.jl 30
  - command: julia /home/jovyan/workspace/julia-example/fibonacci.jl 5
  - command: julia /home/jovyan/workspace/julia-example/fibonacci.jl 13
```

Since the interface is a the command line, this can be used with Julia and R (in addition to Python)
