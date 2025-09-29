# RAPIDS (Python)

Saturn Cloud has a built in GPU image for RAPIDS that has all the required libraries to get started using RAPIDS. When creating a new resource, select the `saturn-rapids` image. Once the resource starts, your RAPIDS code should be ready to run. This will also work well with Dask, and is how the [Saturn Cloud RAPIDS examples](/docs) run.

If you want to [create your own image](<docs/user-guide/how-to/advanced/create-images.md>), you will need to install the correct version of RAPIDS based on your CUDA version. You can get this from the `rapidsai` conda channel.

```yml
channels:
  - rapidsai
  - defaults
dependencies:
  - rapids=0.14.1=cuda10.1_py37_0
```
