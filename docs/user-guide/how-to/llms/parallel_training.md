# Multi-Node Multi-GPU Parallel Training

### **Example Command**:
```bash
torchrun --nproc_per_node=4 script.py --epochs 10 --batch-size 64
```
This command runs the script on 4 GPUs with user-defined script arguments.

By leveraging `torchrun`, PyTorch users can efficiently scale training workloads while managing distributed resources seamlessly.

## Parallel Training with PyTorch on Saturn Cloud

Single node multi-gpu parallel PyTorch training on Saturn Cloud can be run on Saturn Cloud Python Servers interactively, or non-interactively with Saturn Cloud jobs.
Multi node multi-gpu parallel PyTorch training on Saturn Cloud can be run non-interactively with Saturn Cloud Jobs. Even though you will ultimately need to use Saturn Cloud jobs, we strongly recommend
that you start with single node multi-gpu parallel training interactively on a Python server because troubleshooting jobs has a much longer iteration cycle (you have to wait for machines to spin up, for containers to start, etc), where as on a single machine you can iterate quickly (as fast as you can restart a process)

Recall the previous torchrun command:

```
torchrun --nproc_per_node=<NUM_GPUS> --nnodes=<NUM_NODES> --node_rank=<NODE_RANK> --master_addr=<MASTER_IP> --master_port=<MASTER_PORT> script.py [script_args]
```

- `--nproc_per_node`: NUM_GPUs - you can hardcode this value based on the instance configuration you have selected (usually 1, 2, 4 or 8 GPUs)
- `--nnodes`: NUM_NODES - each Saturn Cloud job has an instance count parameter. This determines the size of your training cluster.
- `--node_rank`: NODE_RANK (Rank of the node (0 for master node). Every instance in a Saturn Cloud job is numbered. You can read the rank from the environment variable `SATURN_JOB_RANK`
- `--master_addr`: IP address of the master node. We populate an environment variable `SATURN_JOB_LEADER` with the DNS address of the 0th node.
- `--master_port`: Port for communication - you can choose any port you want. All ports are open for your job nodes to communicate with each other.

In PyTorch you generally do not need the DNS name of the worker nodes, however if you did need to construct it for the 1st, 2nd, or Nth node, the format is `${SATURN_JOB_RUN}-N.${SATURN_INTERNAL_SUBDOMAIN}.${SATURN_NAMESPACE}.svc.cluster.local`. `SATURN_JOB_RUN`, `SATURN_INTERNAL_DOMAIN`, `SATURN_NAMESPACE` are all populated for you when you use Saturn Cloud Jobs.

## Multi-Node Parallel Training with TensorFlow

TensorFlow's multi-node API is `tf.distribute.MultiWorkerMirroredStrategy`. Unlike PyTorch's `torchrun`, which bootstraps the cluster from the rank-0 address, TensorFlow expects each worker to be told the full cluster topology up front via the `TF_CONFIG` environment variable.

`TF_CONFIG` is a JSON document with two sections:

```json
{
  "cluster": {"worker": ["worker-0:5000", "worker-1:5000", "worker-2:5000"]},
  "task":    {"type": "worker", "index": 0}
}
```

- `cluster.worker` is the same list on every pod — every worker's `host:port`, in rank order.
- `task.index` is the only field that differs per pod: it's this pod's position in the worker list.

On Saturn Cloud, every job pod gets these environment variables for free:

- `SATURN_JOB_SCALE` — total number of pods in the job
- `SATURN_JOB_RANK` — this pod's index (0..scale-1)
- `SATURN_JOB_RUN`, `SATURN_INTERNAL_SUBDOMAIN`, `SATURN_NAMESPACE` — used to build peer DNS names

Peer pods resolve at `${SATURN_JOB_RUN}-N.${SATURN_INTERNAL_SUBDOMAIN}.${SATURN_NAMESPACE}.svc.cluster.local`, for any `N` in `0..SATURN_JOB_SCALE-1`. All ports are open for inter-pod traffic, so you can pick any port — the example below uses 5000.

### Building TF_CONFIG from Saturn environment variables

Put this snippet at the top of your training script, **before** any `tf.distribute` call. TensorFlow reads `TF_CONFIG` when the strategy is constructed, so it has to be set first:

```python
import json
import os

scale = int(os.environ["SATURN_JOB_SCALE"])
rank = int(os.environ["SATURN_JOB_RANK"])
run = os.environ["SATURN_JOB_RUN"]
subdomain = os.environ["SATURN_INTERNAL_SUBDOMAIN"]
namespace = os.environ["SATURN_NAMESPACE"]
port = 5000  # any free port; all pod-to-pod ports are open on Saturn

workers = [
    f"{run}-{i}.{subdomain}.{namespace}.svc.cluster.local:{port}"
    for i in range(scale)
]
os.environ["TF_CONFIG"] = json.dumps({
    "cluster": {"worker": workers},
    "task": {"type": "worker", "index": rank},
})

import tensorflow as tf
strategy = tf.distribute.MultiWorkerMirroredStrategy()
```

### Defining the model

Wrap model creation and compilation inside `strategy.scope()`. TensorFlow uses the scope to mirror variables across workers:

```python
with strategy.scope():
    model = tf.keras.Sequential([...])
    model.compile(optimizer="adam", loss="sparse_categorical_crossentropy")

dataset = tf.data.Dataset.from_tensor_slices(...).batch(32)
model.fit(dataset, epochs=10)
```

`MultiWorkerMirroredStrategy()` blocks until every worker has connected, so it's normal for early-starting pods to sit idle for a few seconds while later pods finish pulling images.

### Saving checkpoints

With `MultiWorkerMirroredStrategy`, all workers see the same model weights, but only one of them should write checkpoints to persistent storage — otherwise every worker races to write to the same path. The chief is worker index 0:

```python
is_chief = rank == 0
checkpoint_dir = "/home/jovyan/shared/checkpoints" if is_chief else f"/tmp/ckpt-{rank}"
model.save_weights(f"{checkpoint_dir}/epoch-{epoch}.h5")
```

Non-chief workers still need to call `save_weights` (the strategy synchronizes on it), but writing to a local `/tmp` path keeps them from clobbering the chief's output. After training, only the chief's checkpoint is the one to keep.

### Sharding input data

`tf.data` will auto-shard datasets across workers by default. If your input is a file-based pipeline (TFRecord, etc.), each worker reads a disjoint slice. For in-memory data, you can opt into manual sharding:

```python
options = tf.data.Options()
options.experimental_distribute.auto_shard_policy = tf.data.experimental.AutoShardPolicy.DATA
dataset = dataset.with_options(options)
```

`AutoShardPolicy.DATA` splits batches across workers; `AutoShardPolicy.FILE` (the default for file-based inputs) splits files across workers. Use `FILE` when you have at least one file per worker.

### What we don't cover

`ParameterServerStrategy` uses a different cluster layout (`chief` + `ps` + `worker` task types) and is mostly used for very large recommender models. If you need it, reach out — the Saturn primitives are sufficient but the `TF_CONFIG` shape is different from what this page shows.
