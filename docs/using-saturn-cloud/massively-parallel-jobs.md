# Massively Parallel Jobs for Research Workflows

This section is a continuation of the previous section for [dispatching jobs to Saturn Cloud.](/docs) The previous section went over dispatching a single command to Saturn Cloud. However there are also times when you may want to create hundreds or thousands of runs in Saturn Cloud. Doing so by creating a job per run both makeds the Saturn Cloud UI cluttered and difficult to use, but it also makes it hard to retrieve, and understand which runs have succeeded and which have failed.

In addition, with massively parallel jobs you need to expect failures. When you dispatch a single job, the probability of intermittent failures is generally low - this is usually caused by hardware failures, network issues with data stores, AWS request rate limiting, etc. However once you start dispatching millions of jobs, the probability of having to deal with a handful of failures even under ideal conditions goes up dramatically. Massively parallel jobs in Saturn Cloud are designed around these constraints.

## Specifying runs

Runs can be specified in json or yaml (yaml is much slower if you have thousands of runs)

```yaml
nprocs: 4
remote_output_path: sfs://internal/hugo/intercom-backfill-2024-03-24/
runs:
- cmd: python operations/pipelines/hosted/enrich_user_data.py --start-dt 2023-12-31 --end-dt 2024-01-02 --enrich --first-seen
  remote_output_path: sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-01
- cmd: python operations/pipelines/hosted/enrich_user_data.py --start-dt 2024-01-01 --end-dt 2024-01-03 --enrich --first-seen
  remote_output_path: sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-02
- cmd: python operations/pipelines/hosted/enrich_user_data.py --start-dt 2024-01-02 --end-dt 2024-01-04 --enrich --first-seen
  remote_output_path: sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-03
- cmd: python operations/pipelines/hosted/enrich_user_data.py --start-dt 2024-01-03 --end-dt 2024-01-05 --enrich --first-seen
  remote_output_path: sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-04
- cmd: python operations/pipelines/hosted/enrich_user_data.py --start-dt 2024-01-04 --end-dt 2024-01-06 --enrich --first-seen
  remote_output_path: sfs://internal/hugo/intercom-backfill-2024-03-24/2024-01-05
```

For example the above is a batch for a backfill script that populates intercom (our support chat tool) with information about our users so that we can better assist them with technical issues.

We expect the following parameters:

- **nprocs**: The number of runs to run in parallel per machine. This is determined by the machine you intend to run on, and the resource constraints of your run. For example if I have a run that takes approximately 1 GB of memory, and can consume a single CPU, and I am running on a c5.24xlarge which has 192 GB of memory, I may choose to set nprocs to 96 to maximize CPU utilization. However if my run takes 8 GB of memory, it may be safer to set nprocs to something less than (192 / 8 = 24) so I won't consume all the memory on the machine.
- **remote_output_path**: a networked storage location where all results of runs will be stored.
- **runs**: a list of all runs we want to execute. This example only has 4 runs to keep things concise, but you will have more.

Each run expects the following parameters

- **remote_output_path**: this is the same parameter name as the previous section - this should be a subdirectory of the global `remote_output_path` and will house the results for this specific run
- **cmd**: this is the command you would like to execute.


## Batching Runs

The next step is to group runs together for efficiency. Batches are useful in order to reduce the impact of job overhead. For example if it takes 5 minutes to spin up a new machine, and I only allocate 5 minutes of work, then half the time of the job is wasted. At the extreme end, packing all of the runs into a single batch means I cannot leverage multiple machines to do the work. You can play with this parameter. If the batch size is unset, we default to `3 * nprocs`.

```
$  sc split /tmp/recipe.yaml  /tmp/batch.yaml  ~/commands --sync ~/workspace/saturn-operations
```

The above command takes a job recipe (`/tmp/recipe.yaml`) along with a batch specification (`/tmp/batch.yaml`) and writes the resulting batches into `~/commands`. It also synchronizes my local `~/workspace/saturn-operations` source directory with the rmeote job.

In my example, my `batch.yaml` file has 168 runs. I'm planning on executing 4 commands at a time, and so the default batch size is 12. This results in 13 batches, which all look like the following:


```
{
  "nprocs": 4,
  "remote_output_path": "sfs://internal/hugo/intercom-backfill-2024-03-24/"
  "runs": [
    {
      "remote_output_path": "sfs://internal/hugo/intercom-backfill-2024-03-24/2023-09-01",
      "cmd": "python operations/pipelines/hosted/enrich_user_data.py --start-dt 2023-08-31 --end-dt 2023-09-02 --enrich --first-seen",
      "local_results_dir": "/tmp/819b537dd41140e2a0bbd59b877486f3/"
    },
    ... # runs omitted from the output for brevity
    {
      "remote_output_path": "sfs://internal/hugo/intercom-backfill-2024-03-24/2023-09-12",
      "cmd": "python operations/pipelines/hosted/enrich_user_data.py --start-dt 2023-09-11 --end-dt 2023-09-13 --enrich --first-seen",
      "local_results_dir": "/tmp/21ba080f724d4696b0ee6aff3a12da2e/"
    }
  ],
}
```

The above command modified my original recipe and:
- modified the start script to download `~/workspace/saturn-operations`
- modified the start script to download `~/workspace/commands` (where the batches are written)
- modified the command to execute batches.

Here is the updated command section of the recipe

```yaml
  command:
  - sc batch /home/jovyan/commands3/0.json
  - sc batch /home/jovyan/commands3/1.json
  - sc batch /home/jovyan/commands3/2.json
  - sc batch /home/jovyan/commands3/3.json
  - sc batch /home/jovyan/commands3/4.json
  - sc batch /home/jovyan/commands3/5.json
  - sc batch /home/jovyan/commands3/6.json
  - sc batch /home/jovyan/commands3/7.json
  - sc batch /home/jovyan/commands3/8.json
  - sc batch /home/jovyan/commands3/9.json
  - sc batch /home/jovyan/commands3/10.json
  - sc batch /home/jovyan/commands3/11.json
  - sc batch /home/jovyan/commands3/12.json
  - sc batch /home/jovyan/commands3/13.json
```

`sc batch` is a command that will take the above batch file as written, execute it on the machine and ensure that no more than 4 runs (`nprocs`) is running at any given time, and ensure that stdout, stderr, and any results are saved to the `remote_output_path` for the run.

## Run output

The output for every run is stored in the `remote_output_path` specifically

- **stdout**: the standard output of your command
- **stderr**: the standardd error of your command
- **status_code**: the unix status code of your command. 0 means it completed successfully
- **results**: Optional - any result files your job has written (more on this later)


## Results

The sc batch command will populate an environment variable: `SATURN_RUN_LOCAL_RESULTS_DIR`. Anything your job writes to that directory will be copied to `${remote_output_path}/results`


## Failures

As mentioned earlier - as the number of runs increases, having to deal with failures is almost guaranteed. As a result the batching infrastructure makes it easy to skip completed runs, and re-try failed runs. You could identify completed/failed runs your self by reading all the individual `status_code` files. However the `split` command has a few options to make this easier. By default `sc split` will schedule all work from your `batch.json` or `batch.yaml` file. However if you pass `--skip-completed` it will automatically ignore everything that has completed successfully, and if you pass `--skip-failures` it will automatically skip everything that has failed.
