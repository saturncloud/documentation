# TensorFlow

## TensorFlow using GPUs

Saturn Cloud has a built in GPU image for TensorFlow that has all the required libraries to get started using TensorFlow on a GPU. When creating a new resource, select the `saturn-tensorflow` image. Once the resource starts, you TensorFlow code should be ready to run.

## PyTorch using CPUs

If you want to use TensorFlow but on a CPU resource (which may be cheaper depending on which Saturn Cloud plan you are using), you can manually set up TensorFlow yourself by creating a resource with the following settings:

* **Hardware:** CPU
* **Image:** saturn
* **Extra Packages (Pip):** Add the following: `tensorflow`