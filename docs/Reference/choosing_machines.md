# Choosing Machines for your Work

In most situations, when we begin a machine learning project, we have little or no choice in the computing resources available to us. How big is your laptop? That's the size of resources you get.

However, when you use Saturn Cloud for your data science, you actually have the freedom to choose from many computing specifications! This creates a challenge - how do you know which machine or set of machines is what you need?

The first thing to remember is *Don't Panic*. When you use Saturn Cloud, changing your machine size is as easy as changing your mind, and takes only a few clicks.


## One Machine, or Many?
Think about the work you have in front of you, and ask yourself a few questions.

1. Is this work going to involve a very large dataset? (For example, multiple GB of data)
2. Is this work going to require massive compute resources? (For example, might it take days or weeks on a laptop?)

These questions, if answered *Yes*, might mean that you want more than one machine. Using Dask makes parallelizing code easy, so taking code that's run on one machine to a cluster is not difficult. Take a look at our documentation [about Dask](<docs/Reference/dask_concepts.md>) and [setting up a cluster](<docs/Using Saturn Cloud/Create Cluster/create_cluster.md>) if you want to go this route.

If you choose multiple machines, your cluster setup will look something like this: 
<img src="/images/docs/dask-cluster.png" alt="Diagram of a Dask Cluster including client, scheduler, and workers" class="doc-image">

If you answered *No*, or *I'm not sure*, then you may want to start with just one machine (the Client), at least for now.

Either way, you still have more choices to make.

## GPU or CPU?
Do you need a GPU for your computation? How would you know?

A GPU is best suited to tasks that can be very parallelized. Meaning, if you need to do a small task in about the same way dozens or hundreds of times, then a GPU might be useful. But if your tasks are complex in other ways that don't necessarily lend themselves to parallelism, a CPU might be better.

GPUs are really good for certain things (hyperparameter tuning, or ensemble models, for example) but are not always the right option. A GPU has less immediate access to memory caches, and it isn't supported by nearly as many programming libraries and frameworks. Many times, a task would be faster and also easier to run on a CPU.

So, think about your task, and decide what kind of machine you need. Remember, you can always change your mind! But changing your mind between CPU and GPU may require some refactoring of your code later.

## Size?

Now you're ready to look at the sizes of machine (cores and RAM) available! Saturn Cloud makes a wide range of machines available to users, and you can find a list just by running this in your Jupyter server:

```python
dask_saturn.describe_sizes()

#> {'medium': 'Medium - 2 cores - 4 GB RAM',
#> 'large': 'Large - 2 cores - 16 GB RAM',
#> 'xlarge': 'XLarge - 4 cores - 32 GB RAM',
#> '2xlarge': '2XLarge - 8 cores - 64 GB RAM',
#> '4xlarge': '4XLarge - 16 cores - 128 GB RAM',
#> '8xlarge': '8XLarge - 32 cores - 256 GB RAM',
#> '12xlarge': '12XLarge - 48 cores - 384 GB RAM',
#> '16xlarge': '16XLarge - 64 cores - 512 GB RAM',
#> 'g4dnxlarge': 'T4-XLarge - 4 cores - 16 GB RAM - 1 GPU',
#> 'g4dn4xlarge': 'T4-4XLarge - 16 cores - 64 GB RAM - 1 GPU',
#> 'g4dn8xlarge': 'T4-8XLarge - 32 cores - 128 GB RAM - 1 GPU',
#> 'p32xlarge': 'V100-2XLarge - 8 cores - 61 GB RAM - 1 GPU',
#> 'p38xlarge': 'V100-8XLarge - 32 cores - 244 GB RAM - 4 GPU',
#> 'p316xlarge': 'V100-16XLarge - 64 cores - 488 GB RAM - 8 GPU'}
```

A short reminder:
* Cores means the number of vCPUs in the machine.
* RAM indicates how much the machine can hold in memory.

You can mix and match these sizes, as well. Your workers in your cluster will all be the same specifications, but your Jupyter server can be larger or smaller if you want. Most of the time, scheduler machines will stay small. If you need GPUs, then select GPU machines for your Jupyter server and your workers, but your scheduler doesn't need to have one.

## Example

Let's look at one example, the `g4dn4xlarge` or T4 XLarge GPU machine. Amazon gives us lots of <a href="https://aws.amazon.com/ec2/instance-types/" target='_blank' rel='noopener'>details about the EC2 instances</a>, so we can research to see what this machine contains.

It...
* is in the G4 family: https://aws.amazon.com/ec2/instance-types/g4/ and features NVIDIA T4 GPUs, hence the shorthand name.
* contains one GPU chip, and 4 "cores" otherwise known as vCPUs (virtual CPUs).
* has 16 GB of RAM
* has 125 GB of instance storage (disk space on the machine)

This gives us some information. First, if you don't need a GPU, this machine is probably not that useful for you. It would cost more than the equivalent XLarge that doesn't have the GPU, and otherwise has the same parameters.

Second, it will hold up to 16 GB of data in memory for you at any given time. Decide if that's big enough for what you're up to- remember that's the absolute most it'll hold.

Third, because it's a T4, it has a slightly older NVIDIA GPU than the cutting edge found in P4 machines (the A100 Tensor Core GPU). This might be fine for you, but be aware that there are faster and more sophisticated GPUs available too.

Beyond that, if you think the machine might do the trick, try it! If it's too slow, you can bump up. If it's too costly, or more resources than you require, you can scale back.

If you find that the choice you've made is either too much, or not enough, you can adjust in a few ways.
* different sized machine/s
* more or fewer machines

As your work evolves, you might find that a cluster makes sense now, when it didn't before. In Saturn Cloud, you can add that cluster very simply, use it when you need it, and continue with your work.
