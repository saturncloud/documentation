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

## Multi-Node Parallel Training with DeepSpeed

DeepSpeed is most often launched two ways: with its native `deepspeed` CLI (which SSHes from a leader pod into workers using a hostfile), or on top of `torchrun`, with DeepSpeed initialized from inside the training script. **On Saturn Cloud — and on Kubernetes generally — the `torchrun` path is the right one.** Each pod runs its own launcher, no cross-pod SSH is required, no hostfile to maintain, and it reuses the same primitives as the PyTorch section above. HuggingFace Accelerate and Trainer drive DeepSpeed this way under the hood, so most modern DeepSpeed code already assumes this model.

### Launch command

The launch command on every pod is identical to the PyTorch `torchrun` example — DeepSpeed doesn't change how the cluster is bootstrapped:

```bash
torchrun \
  --nnodes=$SATURN_JOB_SCALE \
  --node_rank=$SATURN_JOB_RANK \
  --nproc_per_node=$(nvidia-smi -L | wc -l) \
  --master_addr=$SATURN_JOB_LEADER \
  --master_port=29500 \
  train.py --deepspeed --deepspeed_config ds_config.json
```

`--nproc_per_node` should match the number of GPUs on the instance — `nvidia-smi -L | wc -l` works, or you can hardcode it (1, 2, 4, 8) based on the instance size you picked.

### Initializing DeepSpeed in the script

Inside `train.py`, initialize the distributed process group through DeepSpeed and then wrap your model. DeepSpeed reads `RANK`, `WORLD_SIZE`, `LOCAL_RANK`, `MASTER_ADDR`, and `MASTER_PORT` from the environment that `torchrun` sets — you don't need to plumb anything through yourself:

```python
import deepspeed
import torch

deepspeed.init_distributed(dist_backend="nccl")

model = MyModel()
model_engine, optimizer, _, _ = deepspeed.initialize(
    model=model,
    model_parameters=model.parameters(),
    config="ds_config.json",
)

for batch in dataloader:
    loss = model_engine(batch)
    model_engine.backward(loss)
    model_engine.step()
```

`deepspeed.initialize` returns a wrapped model engine that handles the optimizer step, gradient accumulation, and ZeRO partitioning according to your config.

### A minimal `ds_config.json`

A typical ZeRO-3 config for multi-node training looks like this:

```json
{
  "train_batch_size": 256,
  "gradient_accumulation_steps": 1,
  "optimizer": {
    "type": "AdamW",
    "params": {"lr": 3e-5}
  },
  "bf16": {"enabled": true},
  "zero_optimization": {
    "stage": 3,
    "overlap_comm": true,
    "contiguous_gradients": true
  }
}
```

`train_batch_size` is the **global** batch size across all ranks — DeepSpeed divides by `world_size * gradient_accumulation_steps` to get the per-GPU micro-batch. If that division isn't an integer, DeepSpeed will refuse to start.

### Checkpoints

Use DeepSpeed's own checkpoint API rather than `torch.save`. It coordinates across ranks so each writes its shard of the optimizer state under ZeRO:

```python
model_engine.save_checkpoint("/home/jovyan/shared/checkpoints", tag=f"step-{step}")
```

Point this at a shared volume (e.g. an attached Saturn Cloud disk mounted on every pod) so all ranks can write to the same directory. To resume:

```python
_, client_state = model_engine.load_checkpoint("/home/jovyan/shared/checkpoints")
```

### When you would use the native `deepspeed` launcher instead

If you have an existing setup that runs `deepspeed --hostfile=...` and you want to keep it, you can — but it requires sshd in your image, passwordless SSH between pods, and a hostfile written at pod start. None of that is hard, but it's strictly more moving parts than the `torchrun` path above, with no upside on Kubernetes. Reach out if you have a specific reason to need it and we'll help you wire it up.

## Multi-Node Parallel Training with HuggingFace Accelerate

`accelerate launch` is the launcher most HuggingFace training code (Trainer, SFTTrainer, DPOTrainer, custom loops using `Accelerator`) expects. Under the hood it shells out to `torchrun`, so it uses the same Saturn primitives as the PyTorch and DeepSpeed sections above — but the flag names are different and worth showing explicitly because users will reach for `accelerate launch` by name.

### Launch command

Run this on every pod:

```bash
accelerate launch \
  --num_machines=$SATURN_JOB_SCALE \
  --machine_rank=$SATURN_JOB_RANK \
  --num_processes=$(( SATURN_JOB_SCALE * $(nvidia-smi -L | wc -l) )) \
  --main_process_ip=$SATURN_JOB_LEADER \
  --main_process_port=29500 \
  train.py
```

- `--num_machines` — total pods (`SATURN_JOB_SCALE`).
- `--machine_rank` — this pod's index (`SATURN_JOB_RANK`).
- `--num_processes` — **global** number of processes across all pods. Accelerate wants the total, not per-node, so multiply by GPUs-per-node.
- `--main_process_ip` / `--main_process_port` — rendezvous address, same role as torchrun's `--master_addr` / `--master_port`.

### Using Accelerate with DeepSpeed or FSDP

Accelerate can drive DeepSpeed or FSDP for you instead of you wiring them in directly. Add the flag to the launch line:

```bash
accelerate launch --use_deepspeed --deepspeed_config_file ds_config.json ...
# or
accelerate launch --use_fsdp --fsdp_sharding_strategy=FULL_SHARD ...
```

When you do this, **do not** also call `deepspeed.initialize()` or wrap your model in `FullyShardedDataParallel` in the script — Accelerate handles it. Just use the `Accelerator` API:

```python
from accelerate import Accelerator

accelerator = Accelerator()
model, optimizer, dataloader = accelerator.prepare(model, optimizer, dataloader)

for batch in dataloader:
    loss = model(batch)
    accelerator.backward(loss)
    optimizer.step()
```

### Config files

You can replace most of the launch flags with an `accelerate_config.yaml` generated by `accelerate config`, then run `accelerate launch --config_file accelerate_config.yaml train.py`. The Saturn env vars (`SATURN_JOB_SCALE`, `SATURN_JOB_RANK`, `SATURN_JOB_LEADER`) still need to be passed in — either via the CLI flags above, which override the config file, or by templating them into the YAML at pod start.

## Multi-Node Parallel Training with PyTorch FSDP

FSDP (Fully Sharded Data Parallel) is PyTorch's native answer to DeepSpeed ZeRO — it shards model parameters, gradients, and optimizer state across ranks. The launch model is identical to plain `torchrun`; only the in-script wrapping changes.

### Launch command

Exactly the same as the PyTorch `torchrun` example — FSDP doesn't introduce any new launcher concerns:

```bash
torchrun \
  --nnodes=$SATURN_JOB_SCALE \
  --node_rank=$SATURN_JOB_RANK \
  --nproc_per_node=$(nvidia-smi -L | wc -l) \
  --master_addr=$SATURN_JOB_LEADER \
  --master_port=29500 \
  train.py
```

### Wrapping the model

Inside `train.py`, initialize the process group, then wrap the model with `FullyShardedDataParallel`:

```python
import torch
import torch.distributed as dist
from torch.distributed.fsdp import FullyShardedDataParallel as FSDP
from torch.distributed.fsdp import ShardingStrategy
from torch.distributed.fsdp.wrap import transformer_auto_wrap_policy

dist.init_process_group(backend="nccl")
local_rank = int(os.environ["LOCAL_RANK"])
torch.cuda.set_device(local_rank)

model = MyModel().cuda()
model = FSDP(
    model,
    sharding_strategy=ShardingStrategy.FULL_SHARD,
    device_id=local_rank,
    auto_wrap_policy=transformer_auto_wrap_policy,
)

optimizer = torch.optim.AdamW(model.parameters(), lr=3e-5)

for batch in dataloader:
    loss = model(batch)
    loss.backward()
    optimizer.step()
    optimizer.zero_grad()
```

`ShardingStrategy.FULL_SHARD` is the ZeRO-3 equivalent (params + grads + optimizer state sharded). `SHARD_GRAD_OP` is ZeRO-2 (only grads + optimizer state), and `NO_SHARD` is plain DDP. For transformer models, `transformer_auto_wrap_policy` is the right default; for other architectures, use `size_based_auto_wrap_policy`.

### Checkpoints

FSDP checkpoints need special handling because each rank only holds a shard of the weights. The simplest pattern is to gather the full state dict to rank 0 and save once:

```python
from torch.distributed.fsdp import FullStateDictConfig, StateDictType

save_policy = FullStateDictConfig(offload_to_cpu=True, rank0_only=True)
with FSDP.state_dict_type(model, StateDictType.FULL_STATE_DICT, save_policy):
    state = model.state_dict()
    if dist.get_rank() == 0:
        torch.save(state, "/home/jovyan/shared/checkpoints/model.pt")
```

For very large models where gathering the full state dict on one rank is infeasible, use `StateDictType.SHARDED_STATE_DICT` and the `torch.distributed.checkpoint` API, which writes per-shard files in parallel. This is the recommended path for models that don't fit in a single rank's CPU memory.

## Multi-Node Parallel Training with JAX

JAX uses a different distributed model than PyTorch: instead of a per-process launcher, every process calls `jax.distributed.initialize()` and JAX handles the rendezvous itself. The Saturn env vars map cleanly onto what JAX expects.

### Launch command

JAX has no `torchrun` equivalent — you just run your Python script directly on every pod:

```bash
python train.py
```

That's the whole launch line. All the distributed setup happens inside the script.

### Initializing the JAX distributed runtime

Call `jax.distributed.initialize()` at the very top of your script, before any other JAX calls:

```python
import os
import jax

jax.distributed.initialize(
    coordinator_address=f"{os.environ['SATURN_JOB_LEADER']}:29500",
    num_processes=int(os.environ["SATURN_JOB_SCALE"]),
    process_id=int(os.environ["SATURN_JOB_RANK"]),
)

print(f"rank {jax.process_index()} of {jax.process_count()}, "
      f"local devices: {jax.local_device_count()}, "
      f"global devices: {jax.device_count()}")
```

- `coordinator_address` — host:port of the rank-0 pod. `SATURN_JOB_LEADER` resolves to that pod's DNS name; pick any free port.
- `num_processes` — total pods.
- `process_id` — this pod's rank.

After `initialize` returns, `jax.devices()` lists every GPU across every pod, and `jax.local_devices()` lists just this pod's GPUs.

### Sharding computation

JAX's modern distributed API is `jax.sharding`. Build a mesh that spans all devices and shard your arrays across it:

```python
import jax
import jax.numpy as jnp
from jax.sharding import Mesh, PartitionSpec as P, NamedSharding

mesh = Mesh(jax.devices(), axis_names=("data",))
sharding = NamedSharding(mesh, P("data"))

batch = jnp.ones((global_batch_size, features))
batch = jax.device_put(batch, sharding)

@jax.jit
def train_step(params, batch):
    ...

params = train_step(params, batch)
```

For model parallelism, use a 2D mesh (`Mesh(devices.reshape(num_pods, gpus_per_pod), ("data", "model"))`) and shard parameters along the `model` axis. For Flax / NNX users, `nnx.spmd` and `flax.linen.with_partitioning` build on top of `jax.sharding` the same way.

### Checkpoints

Use [Orbax](https://orbax.readthedocs.io/) for distributed JAX checkpointing — it handles sharded saves and restores correctly across processes. The naive `pickle` / `jnp.save` approach doesn't work once arrays are sharded across pods.

```python
import orbax.checkpoint as ocp

checkpointer = ocp.StandardCheckpointer()
checkpointer.save("/home/jovyan/shared/checkpoints/step-1000", params)
```

Point the path at a shared volume so every process can write its shard.
