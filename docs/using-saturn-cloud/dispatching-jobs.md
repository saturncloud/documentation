# Dispatching Jobs for Research Workflows
id-hugo-hello-world-a387e542a27a4e689f16a0fac48901de-nz-0-hplcq    pending      live          2024-04-01T15:29:36+00:00
id-hugo-hello-world-a387e542a27a4e689f16a0fac48901de-wc-0-4ptfv    completed    historical    2024-04-01T01:29:46+00:00    2024-04-01T01:32:31+00:00
id-hugo-hello-world-a387e542a27a4e689f16a0fac48901de-hd-0-w5qx4    completed    historical    2024-04-01T01:15:48+00:00    2024-04-01T01:18:34+00:00

```

You can then request the logs for each pod. Note, Saturn Cloud captures live and historical logs. Live logs are stored on the machine where the job is running. These disappear when the machine is torn down. Historical logs are an archive of the live logs, but there may be a few minute delay before logs end up in the historical log store. As a result, the CLI lets you specify which source you would like to choose for logs. If you omit the source, the client attempts to figure out the best source.

```
$ sc logs job hello-world id-hugo-hello-world-a387e542a27a4e689f16a0fac48901de-nz-0-hplcq
$ sc logs job hello-world id-hugo-hello-world-a387e542a27a4e689f16a0fac48901de-nz-0-hplcq --source live
$ sc logs job hello-world id-hugo-hello-world-a387e542a27a4e689f16a0fac48901de-nz-0-hplcq --source historical
```

## Example: Parameter Scans

Let's suppose you have a job recipe similar to the one you had before:

```yaml
type: job
spec:
  name: param1
  description: ''
  image: community/saturncloud/saturn-python:2023.09.01
  instance_type: large
  environment_variables: {}
  working_directory: /home/jovyan/workspace
  start_script: ''
  git_repositories: []
  secrets: []
  shared_folders: []
  start_dind: false
  command: python train.py --parameter 1
  scale: 1
  use_spot_instance: false
  schedule: null
  owner: orgname/username
```

If you wanted to sweep parameters 1-10, you could generate multiple yaml files:

recipe1.yaml:

```
type: job
spec:
  name: param1
  ...
  command: python train.py --parameter 1
```

recipe2.yaml:
```
type: job
spec:
  name: param2
  ...
  command: python train.py --parameter 2
```

recipe3.yaml:
```
type: job
spec:
  name: param3
  ...
  command: python train.py --parameter 3
```

And then dispatch them with the Saturn Cloud CLI:

```
$ sc apply recipe1.yaml --start
$ sc apply recipe2.yaml --start
$ sc apply recipe3.yaml --start
```

Or with the following Python code:

```
from ruamel.yaml import YAML
conn = SaturnConnection()
for path in ['recipe1.yaml, 'recipe2.yaml', 'recipe3.yaml']:
    with open(path) as f:
        recipe = YAML().load(f)
        result = conn.apply(recipe)
        resource_id = result["state"]["id"]
        conn.start("job", resource_id)
```

This is a reasonable approach for running 20 or fewer jobs. If you want to run more jobs, I would recommend
the section on [Massively Parallel Jobs](/docs)
