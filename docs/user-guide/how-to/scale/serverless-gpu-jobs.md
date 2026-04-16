# Serverless GPU Jobs

A serverless GPU job is a unit of work that runs on a GPU when triggered and costs nothing when idle. Saturn Cloud handles this with the Job resource: you define the job once, trigger it via HTTP, and Saturn Cloud allocates a GPU machine, runs your code, and tears the machine down when it finishes.

This page covers how to set up and operate jobs in this mode.

## The model

A serverless GPU job in Saturn Cloud has three states:

1. **Defined.** The job exists as a recipe: image, instance type, command, git repositories, secrets, environment variables. No machine is allocated. No cost is incurred.
2. **Running.** A trigger (HTTP POST, cron, or manual) causes Saturn Cloud to allocate a GPU instance, pull the image, run the start script, clone git, and execute the command.
3. **Torn down.** The command exits, the machine is released, logs are archived. Back to idle.

Each trigger produces a new pod on a new machine. Concurrent triggers produce concurrent pods, up to the capacity available in your cluster. There is no warm pool and no shared state between invocations.

## Defining and triggering a job

Jobs are created, updated, and started through the Saturn Cloud REST API. The `saturn-client` CLI (`sc`) and the Saturn Cloud UI are clients for that API, so any workflow shown here can be driven from a terminal, a script, a CI pipeline, or another service interchangeably.

A minimal recipe for a GPU job:

```yaml
schema_version: 2025.10.01
type: job
spec:
  name: gpu-inference
  image: saturncloud/saturn:2025.06.01
  instance_type: nebius-gpu-h200-sxm-1gpu-16vcpu-200gb
  command: python run_inference.py
  working_directory: /home/jovyan/workspace
  git_repositories:
    - url: git@github.com:your-org/your-repo.git
      reference: main
  scale: 1
  use_spot_instance: false
```

Apply the recipe and start an invocation:

```bash
$ sc apply recipe.yaml --start
```

The same operation over HTTP:

```python
import requests

headers = {"Authorization": f"token {SATURN_TOKEN}"}

# Create or update the job (PUT /api/recipes is the apply endpoint)
requests.put(f"{SATURN_BASE_URL}/api/recipes", headers=headers, json=recipe).raise_for_status()

# Start an invocation
requests.post(f"{SATURN_BASE_URL}/api/jobs/{job_id}/start", headers=headers).raise_for_status()
```

The start call returns once the pod is scheduled. It does not wait for the command to finish. Concurrent starts produce concurrent pods, up to the GPU capacity available in your cluster. Cron schedules and the "start" button in the Saturn Cloud UI call the same start endpoint.

To poll for completion:

```bash
$ sc pods job gpu-inference
$ sc logs job gpu-inference <pod-name>
```

See [Dispatching Jobs](<docs/user-guide/how-to/scale/dispatching-jobs.md>) for the full recipe reference and CLI usage.

## Passing inputs

Jobs receive inputs through three channels:

- **Command-line arguments** in the recipe's `command` field.
- **Environment variables** set in `environment_variables`.
- **Files in networked storage** (S3, [shared folders](/docs), SaturnFS) that the command reads at runtime.

For invocations that need different inputs per call, the common patterns are:

1. Have the command read a parameter (e.g., an S3 key, a run ID) from an environment variable, and update the recipe with the new value before starting the invocation.
2. Write the input payload to a known S3 location keyed by run ID, and pass the run ID as an argument.
3. Create a separate job per parameter combination. For hundreds or thousands of parameter combinations, use [Massively Parallel Jobs](<docs/user-guide/how-to/scale/massively-parallel-jobs.md>) which packs many runs onto shared machines.

## Handling output

The command writes its results to networked storage before exiting. Saturn Cloud captures:

- Job logs (live and archived)
- The Unix exit code
- Pod status and timing

It does not capture stdout as a return value, files written to the local disk, or in-memory state. The trigger is asynchronous, so the caller polls job status or reads the output location rather than receiving a response payload.

A typical command looks like:

```python
# run_inference.py
import os, json, boto3

run_id = os.environ["RUN_ID"]
s3 = boto3.client("s3")

# Read input
payload = json.loads(s3.get_object(Bucket="my-bucket", Key=f"in/{run_id}.json")["Body"].read())

# Do GPU work
result = run_model(payload)

# Write output
s3.put_object(Bucket="my-bucket", Key=f"out/{run_id}.json", Body=json.dumps(result))
```

## Cold starts

Each invocation starts on a fresh machine. The overhead before your command runs is:

1. Machine allocation from the cluster
2. Docker image pull
3. Start script execution
4. Git clone

Total overhead is typically a few minutes depending on image size and cluster state. Ways to keep it manageable:

- Build a custom image with your dependencies baked in, so the start script does not have to install packages. See [Build Docker images](/docs#build-docker-images).
- Keep git repositories shallow, or pin a tag so you only clone what you need.
- Use `scale` and `nprocs` to run multiple invocations per machine when you have many small units of work. See [Massively Parallel Jobs](<docs/user-guide/how-to/scale/massively-parallel-jobs.md>).

If your workload is small enough that startup dominates, consider whether a [Deployment](/docs) fits better, it keeps a GPU warm and serves requests directly.

## Concurrency

Concurrent POSTs produce concurrent pods. There is no built-in queue or rate limit at the job level; scheduling is bounded by the GPU capacity available in your cluster. If you need coordinated multi-node runs (for example, distributed training with torchrun or DeepSpeed), use `scale` to request multiple machines for a single invocation. Saturn Cloud injects the environment variables needed for the processes to discover each other.

## Cost model

- No charge while the job is idle.
- Billed per-second for the instance type from pod start to pod teardown.
- Spot instances are supported via `use_spot_instance: true` for workloads that tolerate interruption.

Usage is attributed to the job's owner (user or group), which surfaces in Saturn Cloud's usage tracking for per-user and per-project cost allocation.

## Related

- [Dispatching Jobs](<docs/user-guide/how-to/scale/dispatching-jobs.md>): Full reference for recipes, the CLI, and the HTTP trigger API.
- [Massively Parallel Jobs](<docs/user-guide/how-to/scale/massively-parallel-jobs.md>): Batch hundreds or thousands of invocations onto shared machines.
- [Deploying Jobs](/docs): Development workflow for building jobs from interactive workspaces.
- [Deployments](/docs): Use when you need a warm, always-on GPU service.
