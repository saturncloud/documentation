# Massively Parallel Jobs for Research Workflows
completed    0              sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-01    python operations/pipelines/hosted/enrich_user_data.py --start-dt 2023-12-31 --end-dt 2024-01-02 --enrich --first-seen
completed    0              sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-02    python operations/pipelines/hosted/enrich_user_data.py --start-dt 2024-01-01 --end-dt 2024-01-03 --enrich --first-seen
completed    0              sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-03    python operations/pipelines/hosted/enrich_user_data.py --start-dt 2024-01-02 --end-dt 2024-01-04 --enrich --first-seen
completed    0              sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-04    python operations/pipelines/hosted/enrich_user_data.py --start-dt 2024-01-03 --end-dt 2024-01-05 --enrich --first-seen
completed    0              sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-05    python operations/pipelines/hosted/enrich_user_data.py --start-dt 2024-01-04 --end-dt 2024-01-06 --enrich --first-seen
completed    0              sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-06    python operations/pipelines/hosted/enrich_user_data.py --start-dt 2024-01-05 --end-dt 2024-01-07 --enrich --first-seen
completed    0              sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-07    python operations/pipelines/hosted/enrich_user_data.py --start-dt 2024-01-06 --end-dt 2024-01-08 --enrich --first-seen
completed    0              sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-08    python operations/pipelines/hosted/enrich_user_data.py --start-dt 2024-01-07 --end-dt 2024-01-09 --enrich --first-seen
completed    0              sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-09    python operations/pipelines/hosted/enrich_user_data.py --start-dt 2024-01-08 --end-dt 2024-01-10 --enrich --first-seen
```
## Run output

The output for every run is stored in the `remote_output_path` specifically

- **stdout**: the standard output of your command
- **stderr**: the standard error of your command
- **status_code**: the unix status code of your command. 0 means it completed successfully
- **results**: Optional - any result files your job has written (more on this later)


## Results

The sc batch command will populate an environment variable: `SATURN_RUN_LOCAL_RESULTS_DIR`. Anything your job writes to that directory will be copied to `${remote_output_path}/results`

This output makes it easy to grab any runs that failed, and re-run them locally.

## Hardware Selections

This section of the recipe determines the hardware used for runs:

```yaml
spec:
  instance_type: r6axlarge
  scale: 2
```

This means that we will consume at most 2 r6axlarge until all workloads are complete. The instance type and scale can also be passed in as command line options to `sc split`, using the `--instance-type` and `--scale` options.

```
$ sc options
```

will list all available instance sizes.


## Failures

As mentioned earlier - as the number of runs increases, having to deal with failures is almost guaranteed. As a result the batching infrastructure makes it easy to skip completed runs, and re-try failed runs. You could identify completed/failed runs your self by reading all the individual `status_code` files. However the `split` command has a few options to make this easier. By default `sc split` will schedule all work from your `batch.json` or `batch.yaml` file. However if you pass `--skip-completed` it will automatically ignore everything that has completed successfully, and if you pass `--skip-failures` it will automatically skip everything that has failed.

## GPUs

This batching framework will execute for example, 40 runs, 8 at a time on an 8 GPU machine. In doing so, you would want each process to be allocated to a single GPU. The `sc batch` framework will set the environment variables `SATURN_RUN_LOCAL_RANK`. Your code can use that to select the GPU, or you could just set

```
os.environ['CUDA_VISIBLE_DEVICES'] = os.environ['SATURN_RUN_LOCAL_RANK']
```
