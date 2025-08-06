# CLI Calls
Occasionally you might find yourself wanting to use Command Line Interface (CLI) tools in parallel. For instance, you may be working with a CLI tool for your specialized industry, and for an experiment you would like to execute the command many times at once across many machines. Dask is a great tool for scheduling these tasks and logging the results. To use Dask for executing CLI tools, first ensure the tool of your choice is installed on the Saturn Cloud resource you're using. You can install software to a resource by using bash commands in the **Startup Script** of the resource options. This is stored in **Advanced Settings** and can be edited when you create the resource or by clicking the edit button of an existing resource. Once your resource is created (and you've [created a Dask cluster](https://saturncloud.io/docs/user-guide/using-saturn-cloud/create_dask_cluster/) for it), you can then use Python code to execute the CLI tool on the Dask cluster.

The `subprocess.run` command in Python lets you execute command line tasks. To execute tasks in parallel you can wrap `subprocess.run()` within a function that you'll use on the Dask cluster. This will allow Dask to execute the command on a worker machine, allowing you to call it many times in parallel. You can do so using the [delayed](https://docs.dask.org/en/latest/delayed.html) capabilities of Dask.

The example below uses the very basic `echo` CLI bash command to print a string showing some pairs of exponents. A list of inputs is executed in parallel on the Dask cluster using `dask.compute()`. The `echo` command prints the results to the output as a CLI command. In practice, you'll probably want to choose a more interesting CLI command besides `echo`.

First, start the Dask cluster associated with your Saturn Cloud resource.


```python
from dask_saturn import SaturnCluster
from dask.distributed import Client

client = Client(SaturnCluster())
```

After running the above command, it's recommended that you check on the Saturn Cloud resource page that the Dask cluster as fully online before continuing. Alternatively, you can use the command `client.wait_for_workers(3)` to halt the notebook execution until all three of the workers are ready.

This is the function to run on the Dask cluster:


```python
import subprocess
import dask


@dask.delayed
def lazy_exponent(args):
    x, y = args
    result_string = f'"{x}**{y}={x**y}"'
    subprocess.run(["echo", result_string], check=True)
```

Next, execute the function on the input list. This is going to cause the Dask cluster to run the CLI command on the workers:


```python
inputs = [[1, 2], [3, 4], [5, 6], [9, 10], [11, 12]]

results = dask.compute([lazy_exponent(x) for x in inputs])
```

To see the output, click on the **Logs** button of the resource. From there expand the Dask workers to see the output from code executed on the cluster. Or switch to the **aggregated logs** to see all of the Dask worker output combined together. In the event that one of the CLI calls returns an error, the `check=True` argument to `subprocess.run` will cause the function to have a `CalledProcessError` and thus the Dask task to fail, propagating the error back to you in a helpful way.



Note that when running CLI commands on a Dask cluster, any results saved to the Dask workers will be deleted once the cluster is offline. Thus if your commands save any results, it's important to then take the results and return them to a permanent place. For instance within your `@dask.delayed` function you could add commands to upload files to a location like AWS S3, or you could have the function read the results into Python and return them to the client machine.
