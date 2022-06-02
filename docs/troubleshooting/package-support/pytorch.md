# PyTorch (Python)

Directions for setting up PyTorch with Python depend on if you're using a CPU or GPU

## PyTorch using GPUs

Saturn Cloud has a built in GPU image for PyTorch that has all the required libraries to get started using PyTorch on a GPU. When creating a new resource, select the `saturn-pytorch` image. Once the resource starts, your PyTorch code should be ready to run. This will also work well with Dask, and is how the [Saturn Cloud PyTorch examples](<docs/Examples/python/PyTorch/qs-03-pytorch-gpu-dask-single-model.md>) run.

If you want to [create your own image](<docs/using-saturn-cloud/manage-images/build-images/create-images.md>), you will need to install the GPU version of PyTorch. Conda is the easiest way to access these. Look for "cuda" in the name, as this indicates GPU support.

```bash
$ conda search pytorch
...
#> pytorch                        1.5.1 py3.7_cpu_0                      pytorch
#> pytorch                        1.5.1 py3.7_cuda10.1.243_cudnn7.6.3_0  pytorch
#> pytorch                        1.5.1 py3.7_cuda10.2.89_cudnn7.6.5_0   pytorch
#> pytorch                        1.5.1 py3.7_cuda9.2.148_cudnn7.6.3_0   pytorch
#> pytorch                        1.5.1 py3.8_cpu_0                      pytorch
#> pytorch                        1.5.1 py3.8_cuda10.1.243_cudnn7.6.3_0  pytorch
#> pytorch                        1.5.1 py3.8_cuda10.2.89_cudnn7.6.5_0   pytorch
```

You have to consider the CUDA version and the Python version here. This example shows selecting CUDA 10.1.

```yml
channels:
  - pytorch
  - defaults
dependencies:
  - pytorch=1.5.1=py3.7_cuda10.1.243_cudnn7.6.3_0
```

## PyTorch using CPUs

If you want to use PyTorch but on a CPU resource (which may be cheaper depending on which Saturn Cloud plan you are using), you can manually set up PyTorch yourself by creating a resource with the following settings:

* **Hardware:** CPU
* **Image:** saturn
* **Extra Packages (Conda):** Add the following: `pytorch torchvision torchaudio cpuonly -c pytorch`