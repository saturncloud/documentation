# PyTorch

## PyTorch using GPUs

Saturn Cloud has a built in GPU image for PyTorch that has all the required libraries to get started using PyTorch on a GPU. When creating a new resource, select the `saturn-pytorch` image. Once the resource starts, you PyTorch code should be ready to run. This will also work well with Dask, and is how the [Saturn Cloud PyTorch examples](<docs/Examples/PyTorch/qs-01-pytorch-gpu.md>) run.

## PyTorch using CPUs

If you want to use PyTorch but on a CPU resource (which may be cheaper depending on which Saturn Cloud plan you are using), you can manually set up PyTorch yourself by creating a resource with the following settings:

* **Hardware:** CPU
* **Image:** saturn
* **Extra Packages (Conda):** Add the following: `pytorch torchvision torchaudio cpuonly -c pytorch`