# TensorFlow (Python)

## TensorFlow using GPUs

Saturn Cloud has a built in GPU image for TensorFlow that has all the required libraries to get started using TensorFlow on a GPU. When creating a new resource, select the `saturn-tensorflow` image. Once the resource starts, you TensorFlow code should be ready to run.

If you want to [create your own image](<docs/Using Saturn Cloud/manage-images/build-images/create-images.md>), you will need to install the GPU version of Tensorflow. In `pip`, the library is call `tensorflow-gpu`. In conda, look through the list to find a GPU build.

```bash
$ conda search tensorflow
...
#> tensorflow                     2.2.0 eigen_py36h84d285f_0  pkgs/main
#> tensorflow                     2.2.0 eigen_py37h1b16bb3_0  pkgs/main
#> tensorflow                     2.2.0 gpu_py37h1a511ff_0  pkgs/main
#> tensorflow                     2.2.0 gpu_py38hb782248_0  pkgs/main
#> tensorflow                     2.2.0 mkl_py36h5a57954_0  pkgs/main
```

Then in the image specifications you can ask for:

```yml
dependencies:
  - tensorflow=2.2.0=gpu_py37h1a511ff_0
```

## TensorFlow using CPUs

If you want to use TensorFlow but on a CPU resource (which may be cheaper depending on which Saturn Cloud plan you are using), you can manually set up TensorFlow yourself by creating a resource with the following settings:

* **Hardware:** CPU
* **Image:** saturn
* **Extra Packages (Pip):** Add the following: `tensorflow`