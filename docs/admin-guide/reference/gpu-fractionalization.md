# GPU Fractionalization

## Introduction

GPU fractionalization is a powerful technique that allows multiple workloads to share a single GPU, maximizing hardware utilization and reducing costs. In many scenarios, a single workload may not fully utilize a GPU's compute capacity, leading to expensive idle resources. By fractionalizing GPUs, organizations can run multiple smaller workloads on the same physical GPU, improving efficiency and enabling more developers and data scientists to access GPU resources simultaneously. This is particularly valuable for inference workloads, development environments, and training smaller models that don't require an entire GPU's resources.

## How Saturn Cloud Manages GPU Fractionalization

Saturn Cloud implements GPU fractionalization through [NOS](https://github.com/nebuly-ai/nos), an open-source Kubernetes module that enables efficient GPU sharing through dynamic GPU partitioning. NOS works by intelligently partitioning GPUs based on workload demands, similar to how Kubernetes Cluster Autoscaler manages nodes, but at the GPU resource level.

### Key Features of NOS in Saturn Cloud

- **Dynamic Partitioning**: NOS automatically adjusts GPU partitioning based on pending workloads, maximizing the number of pods that can be scheduled
- **Multi-Instance GPU (MIG)**: Saturn Cloud uses NOS with MIG configuration, providing hardware-level isolation between GPU partitions for maximum security and performance isolation
- **Safety First**: Never disrupts running workloads - only creates new GPU partitions when resources are available
- **Intelligent Scheduling**: Uses an internal scheduler to simulate different partitioning configurations and selects the optimal one

This architecture ensures that Saturn Cloud users can efficiently share GPU resources across multiple workloads while maintaining proper isolation and resource guarantees through MIG's hardware-level partitioning, dramatically improving GPU utilization and reducing costs.

## How GPU Fractionalization Works with NOS

### Understanding Multi-Instance GPU (MIG)

Multi-Instance GPU (MIG) is NVIDIA's hardware-level GPU partitioning technology that allows a single physical GPU to be divided into multiple smaller, isolated GPU instances. Each MIG instance has dedicated memory, cache, and streaming multiprocessors, providing true hardware isolation between workloads.

#### Benefits of MIG

- **Hardware Isolation**: Each MIG instance is fully isolated at the hardware level, preventing interference between workloads
- **Predictable Performance**: Dedicated resources ensure consistent performance without noisy neighbor effects
- **Quality of Service**: Guaranteed memory bandwidth and compute resources for each partition
- **Error Isolation**: Failures in one MIG instance don't affect others running on the same GPU
- **Security**: Hardware-level separation provides strong security boundaries between different workloads

#### Supported GPU Models

MIG is available on select NVIDIA data center GPUs designed specifically for multi-tenant environments:
- **A100**: Up to 7 MIG instances per GPU (Ampere architecture)
- **A30**: Up to 4 MIG instances per GPU (Ampere architecture)
- **H100**: Up to 7 MIG instances per GPU with enhanced capabilities (Hopper architecture)
- **H200**: Up to 7 MIG instances with 141GB HBM3e memory (Hopper architecture)
- **B200**: Up to 7 MIG instances with 180GB HBM3e memory (Blackwell architecture)

#### MIG Profiles and Configurations

MIG instances are created using predefined profiles that specify the compute and memory resources. Common profiles include:

- **1g.10gb**: 1/7 of GPU compute, 10GB memory - ideal for inference workloads
- **2g.20gb**: 2/7 of GPU compute, 20GB memory - suitable for medium-sized models
- **3g.40gb**: 3/7 of GPU compute, 40GB memory - for larger training jobs
- **4g.40gb**: 4/7 of GPU compute, 40GB memory - balanced compute/memory ratio
- **7g.80gb**: Full GPU (A100 80GB) - when you need maximum resources

The exact profiles available depend on your GPU model and memory configuration. Saturn Cloud and NOS work together to automatically select the optimal MIG configuration based on your workload requirements.

#### MIG Mode and GPU Reset Requirements

Understanding when GPU resets are required is important for planning maintenance windows:

- **Enabling MIG Mode**: Requires a GPU reset on Ampere GPUs (A100, A30). Hopper and newer GPUs (H100, H200, B200) enable MIG mode without requiring a reset.
- **Disabling MIG Mode**: Always requires a system reboot regardless of GPU generation
- **Reconfiguring MIG Profiles**: Once MIG mode is enabled, creating, destroying, or reconfiguring MIG instances is dynamic and does not require any GPU reset or system reboot

This dynamic reconfiguration capability allows Saturn Cloud to adjust GPU partitioning throughout the day based on workload demands - for example, using many small instances for inference during business hours and consolidating to larger instances for training overnight.

### The NOS Architecture

NOS operates as a Kubernetes-native solution with two main components that work together to manage GPU fractionalization dynamically.

#### GPU Partitioner - The Brain

The GPU Partitioner is the central controller that makes intelligent decisions about how to partition GPUs across the cluster. It runs as a Kubernetes controller and performs several key functions:

- **Monitors Pending Workloads**: Continuously watches for pods that are pending due to insufficient GPU resources
- **Batch Processing**: Groups pending pods together for efficient processing, configurable via batch window timeouts
- **Simulation Engine**: Uses the Kubernetes scheduler framework to simulate different MIG configurations and predict their outcomes
- **Optimization**: Selects the partitioning plan that maximizes the number of pods that can be scheduled
- **State Management**: Maintains the desired state of GPU partitioning across the cluster

The GPU Partitioner never directly modifies GPUs - instead, it determines the optimal configuration and communicates this to the MIG Agents.

#### MIG Agent - The Executor

The MIG Agent runs as a DaemonSet on every GPU node in the cluster. Each agent is responsible for:

- **MIG Profile Management**: Creates and deletes MIG profiles on the physical GPUs based on instructions from the GPU Partitioner
- **Safety Enforcement**: Ensures that MIG instances currently in use by running pods are never deleted
- **Status Reporting**: Updates node annotations to reflect the current GPU configuration and availability
- **Health Monitoring**: Tracks the health and status of MIG instances on the node

The MIG Agent only runs on nodes labeled with `nos.nebuly.com/gpu-partitioning: mig`, allowing fine-grained control over which nodes participate in GPU fractionalization.

#### Communication Through Kubernetes

NOS leverages Kubernetes' native mechanisms for coordination:

- **Node Annotations**: Used to communicate GPU status and availability
  - Example: `nos.nebuly.com/status-gpu-0-1g.10gb-free: 3` indicates 3 free 1g.10gb instances on GPU 0
  - Example: `nos.nebuly.com/status-gpu-0-1g.10gb-used: 2` indicates 2 used 1g.10gb instances
- **Custom Resources**: MIG partitioning plans are stored as Kubernetes custom resources
- **Label Selectors**: Nodes opt into GPU fractionalization through specific labels
- **Event-Driven**: Changes in pod scheduling or GPU status trigger immediate re-evaluation

This architecture ensures that NOS integrates seamlessly with Kubernetes while maintaining reliability and scalability across large GPU clusters.

#### NOS and Kubernetes Scheduling

NOS integrates with the standard Kubernetes scheduler rather than replacing it:

- **Simulation Mode**: When the GPU Partitioner needs to test different MIG configurations, it uses the Kubernetes scheduler framework to simulate pod placement
- **Standard Scheduling**: Once MIG instances are created, pods are scheduled using the standard Kubernetes scheduler with the newly available MIG resources
- **No Custom Scheduler Required**: Workloads continue to use the default scheduler - NOS simply ensures the right GPU resources are available

This design keeps the system simple and compatible with existing Kubernetes deployments.

### How GPU Fractionalization Works in Practice

To understand how NOS manages GPU fractionalization, let's follow the journey of a typical workload from submission to execution.

#### Detecting GPU Demand

When you submit a workload requesting a fraction of a GPU (such as `nvidia.com/mig-1g.10gb`), the GPU Partitioner detects this request. Rather than processing each request individually, NOS batches pending workloads over a configurable time window. This batching approach allows for better GPU partitioning decisions by considering multiple workloads together.

#### The Simulation Phase

Once NOS has collected a batch of pending workloads, it enters the planning phase. The GPU Partitioner takes a snapshot of your cluster's current state - which nodes have GPUs, how they're currently partitioned, and what workloads are already running.

Using the Kubernetes scheduler framework, NOS runs multiple simulations to determine what MIG configuration would allow the most pending pods to be scheduled. It tests various scenarios - splitting one A100 into seven small instances for inference workloads, or keeping another as a single large instance for a training job. The optimization algorithm evaluates each potential configuration based on scheduling success, resource utilization, and minimal disruption to existing workloads.

#### Dynamic Reconfiguration

With a plan selected, NOS executes the GPU reconfiguration. The GPU Partitioner communicates the plan to the MIG Agents running on each affected node. These agents then create new MIG profiles as specified. On modern GPUs like the H100 and H200, this reconfiguration happens without any GPU reset - workloads continue running on unaffected MIG instances while new ones are being created.

Once the MIG Agents update the node annotations to reflect the new GPU capacity, the Kubernetes scheduler sees these resources and begins placing the pending pods. The process from detection to execution typically completes in seconds.

### Working with Fractionalized GPUs

Now that we understand how NOS creates GPU fractions, let's explore how to actually use them in your workloads.

#### Requesting the Right GPU Fraction

When deploying a workload on Saturn Cloud, you specify your GPU requirements using standard Kubernetes resource requests. For example, if you're running an inference service that needs just a small GPU slice:

```yaml
resources:
  limits:
    nvidia.com/mig-1g.10gb: 1  # Requests one 1g.10gb MIG instance
```

The naming convention follows a predictable pattern: `nvidia.com/mig-<compute>g.<memory>gb`. This makes it easy to understand exactly what resources you're getting - `1g.10gb` means 1/7 of the GPU's compute power and 10GB of memory.

#### What Happens Behind the Scenes

When your pod requests a specific MIG instance, the Kubernetes scheduler becomes very particular about placement. It will only consider nodes that have exactly the MIG profile you requested available. This exact matching ensures predictable performance - a pod requesting `1g.10gb` won't accidentally end up on a larger `2g.20gb` instance where it might not utilize all the resources efficiently.

This strict matching might seem limiting, but it's actually a feature. It ensures that your workload gets exactly the resources it needs for optimal performance, and helps Saturn Cloud maintain efficient cluster utilization by preventing resource waste.

### Ensuring Reliability and Safety

NOS manages GPU fractionalization without disrupting running workloads through several reliability mechanisms.

#### Workload Protection

NOS follows a simple rule: never disrupt a running workload. When the MIG Agent receives instructions to reconfigure a GPU, it first checks if any MIG instances are currently in use. If they are, those instances remain untouched. Your production inference service continues running while NOS creates new MIG instances on the same GPU for other workloads.

This protection extends to failure scenarios. If creating a new MIG profile fails on one GPU, NOS continues with other GPUs and retries the failed operation later. This partial failure recovery means temporary issues don't prevent the rest of your cluster from adapting to workload demands.

#### Handling Failures

GPUs can fail, nodes can restart, or configuration can drift from the desired state. NOS handles these scenarios through continuous reconciliation. The system monitors the actual state of GPU partitioning and compares it to the desired state. When discrepancies are detected - such as a MIG instance disappearing after a node restart - NOS automatically recreates it.

The system includes cleanup mechanisms with grace periods. When a MIG instance is no longer needed, NOS waits before deletion, giving any pods that might be scheduling time to land. This prevents resources from being deleted just as a pod is about to use them.

### Monitoring Your Fractionalized GPUs

NOS provides several ways to observe and troubleshoot GPU fractionalization.

#### Checking GPU Status

The most direct way to see your GPU partitioning is through node annotations. NOS continuously updates these to reflect the current state. For instance, you might see:

```
nos.nebuly.com/status-gpu-0-1g.10gb-free: "3"
nos.nebuly.com/status-gpu-0-1g.10gb-used: "2"
```

This tells you that GPU 0 has been partitioned into 1g.10gb instances, with 3 available and 2 in use. It's a real-time view that helps you understand resource availability at a glance.

For deeper investigation, you can use standard NVIDIA tools. Running `nvidia-smi mig -lgip` on a node shows the exact MIG configuration, including which instances are occupied and their specific resource allocations.

#### Understanding Scheduling Decisions

When a pod isn't scheduling as expected, Kubernetes events provide insights. The command `kubectl describe pod <pod-name>` shows the scheduler's decision-making process, including why a pod might be pending. You might see messages indicating that no nodes have the requested MIG profile available, or that all matching instances are already in use.

NOS logs its decision-making process in detail. The GPU Partitioner logs show why it chose specific partitioning configurations, what simulations it ran, and how it optimized for pod scheduling. These logs help you understand how your cluster's GPU partitioning evolved.

## Installing NOS with Helm

For self-managed Saturn Cloud installations, you can install NOS using Helm. Here are the basic steps:

### Prerequisites

- NVIDIA GPU Operator installed
- GPUs that support MIG (A100, A30, H100, H200, B200)

### Installation Steps

1. Install NOS directly from the OCI registry:
```bash
helm install oci://ghcr.io/nebuly-ai/helm-charts/nos \
  --version 0.1.2 \
  --namespace nebuly-nos \
  --generate-name \
  --create-namespace
  --set gpuPartitioner.devicePlugin.config.namespace=nvidia-gpu-operator
```

2. Label GPU nodes to enable MIG partitioning:

```bash
kubectl label node <gpu-node-name> nos.nebuly.com/gpu-partitioning=mig
```

3. Ensure that your Terraform for your k8s cluster applies that node label for newly create nodes.

### Managed Saturn Cloud Installations

Most Saturn Cloud deployments are fully managed. If you have a managed installation and would like to enable GPU fractionalization, contact support@saturncloud.io and we can configure it for your installation. Our team will handle the setup and optimization based on your specific workload requirements.