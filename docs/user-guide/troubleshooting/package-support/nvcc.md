# NVCC

Our GPU images by default contain CUDA drivers, but do not contain the CUDA compiler. Most Python packages do not require the CUDA compiler, and including the CUDA compiler makes GPU images much larger, and much slower to start.

To install nvcc in a GPU image, you can get the binaries from the NVIDIA conda channel.

```
mamba install -c nvidia cuda-nvcc
```

You can use `conda` instead of `mamba` but `mamba` tends to be much faster.
