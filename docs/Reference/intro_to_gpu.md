# Introduction to GPUs

GPU technology has led to massive performance gains for machine learning tasks, as well as enabling us to solve more complex and difficult data science problems.  By applying GPUs to data science problems judiciously and thoughtfully, you can accelerate your work and your productivity substantially - but before this, you'll need to understand how a GPU works and why it makes such a difference.


{{% alert title="GPU Tutorials" %}}
If you already know how GPUs work, and you'd just like to try some examples of GPUs in Saturn Cloud, check out our tutorials:
* [Random Forest with RAPIDS on GPU](<docs/Examples/python/RAPIDS/qs-01-rapids-single-gpu.md>) 
* [PyTorch on GPU](<docs/Examples/python/PyTorch/qs-01-pytorch-gpu.md>) 
{{% /alert %}}


We also have [a short video explaining how GPUs work for machine learning](https://youtu.be/vcILiZMYpJo) - check it out!

## What is a GPU?

CPU and GPU are simply different types of processor hardware that can power computing. A CPU (central processing unit) is general purpose and versatile - likely most of the electronics you use employ this kind of processor. When not otherwise specified, a computing core is likely to be a CPU. On the other hand, a GPU (graphic processing unit) is more complex in its design and more narrow in its functions. This kind of compute hardware was originally developed to quickly and efficiently render video and images on computer displays. Incidentally, doing this required the ability for the chip to run many, many very similar processes and calculations at the same time - an ability that is also very handy for lots of machine learning tasks. Because the GPU is optimized for this kind of work, we can use it to accelerate machine learning.


But we have already used a lot of terminology that might confuse the unfamiliar reader, so it's important to understand these things before proceeding. Note that these definitions may not always be uniform or agreed upon by all authors, but for this documentation we'll use these.

* **processor/microprocessor**: A piece of hardware that manages instructions and calculation tasks, generating computations to power a computer. Might also be referred to as a chip. Contains one or many cores.
* **core**: One of multiple pieces of hardware inside a processor that received instructions and completes calculations.
* **thread**: A unit of work completed by a core. 

A core receives instructions to process a thread, and returns the results of the work after executing. When we talk about the number of cores a machine or computer has, we mean a concrete piece of hardware. When we talk about the threads, we are discussing how many simultaneous units of work we want to try to run.

It's also important to know: A core can really only do one thing at a time. It fakes multitasking by switching back and forth between threads. So when you ask your cluster to have more threads than it has cores, you're asking it to try and multitask in this way.


This might still be confusing, however. Let's look at diagrams to help.

### CPU Diagram
To begin, we can look at the CPU microprocessor here. First, we see the CPU core at the left. This is where actual mathematical calculations get computed in a CPU, and a usual CPU microprocessor has several. A good sized CPU has maybe 64 of these. That means the CPU can efficiently run at least 64 threads of work at the same time - maybe more, if you ask it to multitask or if there is a lot of waiting/downtime in your threads. 

We won't discuss the memory caches too much here, but those give the processes places to store data temporarily that is needed to complete tasks. For example, computing the product of two numbers requires those two numbers to be known, so you'll need them stored somewhere. The closer those numbers are stored to the core, the faster it can retrieve them, multiply them, and return the result. 

![Diagram of a CPU components](/images/docs/cpu.png "doc-image")


### GPU Diagram

Moving on to a GPU, we can look at the essential features of the NVIDIA Ampere GPU microprocessor. You can immediately tell that this is a more complicated piece of hardware! The cores in NVIDIA GPUs are known as "CUDA" cores - this is not the same in Intel GPUs but they have a core type of their own that is similar. 

CUDA cores are grouped in partitions, where as you can see there are multiple types: INT (integer arithmetic) 32 bit, FP (floating point arithmetic) 32 bit, and FP 64 bit. These are all different types of core that are optimized for different types of arithmetic. There is also a Tensor core in each partition, and assorted other elements that all contribute to the GPU being able to complete many calculations simultaneously, using all those cores. We previously noted that a large CPU might have 64 cores - a large GPU might have thousands of CUDA cores inside it.

![Diagram of NVIDIA Ampere GPU components](/images/docs/gpu-ampere.png "doc-image")

Other manufacturers and other versions of GPU microprocessors will have different architecture, so this is only representative of the NVIDIA Ampere, but it should give you a good sense of how complex the hardware is.

## Which to Use?
Now that we have a general understanding of this hardware, we can discuss when and why you'd use this GPU hardware instead of a CPU.

### Computation
In many cases, a CPU will be perfectly fine for your machine learning needs. Unless your ML work is highly parallelizable and involves the kind of computation GPUs are good at, a CPU can easily be faster.

### Libraries/Packages
Many popular libraries and frameworks are not written to work with GPU architecture. As a result, while the tasks could theoretically be done on GPUs, the libraries are not compatible with GPU hardware as written today. As we have learned, a GPU is very different under the hood than a CPU, so adapting existing CPU software to run on a GPU is not a trivial task. 

Despite this, there are a number of good libraries in the Dask and Pydata ecosystem allowing you to take advantage of GPUs for machine learning. 

![table of libraries for GPU machine learning](/images/docs/gpu-ml-libs.png "doc-image")

### Cost
For a variety of reasons, accessing GPU capable resources on cloud computing resources is significantly more expensive than CPU alone. Purchasing actual GPU hardware is even worse, with GPUs going for hundreds or thousands of dollars retail. (This is mainly because people mining for cryptocurrencies use many, many GPUs for this task, and they buy up the supply.)

All this given, some machine learning work is great for GPUs. Deep learning is a notable example - training complex neural networks can be sped up astronomically on GPUs, and even more so on GPU clusters. There are also great opportunities for speeding up grid searches and large ensembles with GPUs.

## Get a GPU on Saturn Cloud!

Saturn Cloud offers access to a wide range of GPU hardware for our customers - even on our free tier. To try one, [start a Jupyter server resource](<docs/quickstart.md>) and select a GPU as the Hardware. This will give you a choice of T4 or V100 class GPUs. A T4 is somewhat less powerful but also less expensive than a V100.

![New Jupyter server](/images/docs/new-jupyter-server-options.jpg "doc-image")

When you select this hardware, you will automatically be given a selection of images with GPU enabled software to choose from. You can then start the Jupyter instance and try out the GPU for yourself! Instead of creating a new Jupyter server from scratch, you can also use one of the resource templates. These resources are prepopulated with the correct libraries and example notebooks:

* [Random Forest with RAPIDS on GPU](<docs/Examples/python/RAPIDS/qs-01-rapids-single-gpu.md>) 
* [PyTorch on GPU](<docs/Examples/python/PyTorch/qs-01-pytorch-gpu.md>) 

## Troubleshooting machine learning on GPUs

Sometimes you may have thought you had set up a GPU correctly for machine learning but you find that the code doesn't run, or it runs but uses the CPU instead of the GPU. For any GPU ML usage--not just on Saturn Cloud--There is a sequence of different reasons a GPU might fail to be used correctly by your code:

* The machine may not have the right _system drivers_ to use the GPU for ML.
* The machine may not be using the _Python libraries_ that utilize the GPU.
* There may be issues in your Python code.

Let's briefly go through these one by one.

### The machine may not have the right _system drivers_ to use the GPU for ML

The standard libraries for machine learning a GPU are the NVIDIA CUDA Toolkit and these are included in Saturn Cloud GPU images so you should have them already. That said, sometimes machine learning libraries require
specific versions of CUDA (say, version 10 vs 11), so it is worth checking that the correct CUDA driver is installed for your needs. To check the version of CUDA installed, from the command line run:

```
/usr/local/cuda/bin/nvcc --version
```

### The machine may not be using the _Python libraries_ that utilize the GPU

Even if you do have a GPU on the system and the correct drivers, you need to ensure that the Python libraries you are using supports GPU. For instance, with earlier versions of TensorFlow (version 1.15 or earlier) to
utilize the gpu you would need to pip install a special `tensorflow-gpu` library. While modern versions of TensorFlow have GPU support built in, it's possible your code might be using an earlier version. Similarly, other
machine learning libraries may require special installation for GPUs, so check to make sure that isn't the case.

### There may be issues in your Python code

Even if you have a GPU system with the right drivers and correct Python libraries, it's possible your code isn't utilizing it. For instance, with PyTorch if you want to use the GPU you have to transfer your model
onto the GPU device:

```python
device = torch.device("cuda:0" if torch.cuda.is_available() else "cpu")
model.to(device)
```

Make sure that your code correctly accesses the GPU to do your machine learning.
