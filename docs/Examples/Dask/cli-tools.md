# Using Dask to Execute CLI Calls

Occasionally you might find yourself in situations where you want to use Command Line Interface (CLI) tools and for the particular workload it's helpful to execute many calls in parallel. For instance, you may be working with a CLI tool for your specialized industry, and for an experiment you would like to execute the command many times at once across many machines. Dask is a great tool for scheduling these tasks and logging the results.

To use Dask for executing CLI tools, first ensure the tool of your choice is installed on the Saturn Cloud resource you're using. You can install software to a resource by using bash commands in the **Startup Script** of the resource options. This is stored in **Advanced Settings** and can be edited when you create the resource or by clicking the edit button of an existing resource.

![Startup script options](/images/docs/startup-script.png "doc-image")

Once your resource is created (and you've [created a Dask cluster](<docs/Using Saturn Cloud/create_dask_cluster.md>) for it), you can then use Python code to execute the CLI tool on the Dask cluster.

The `subprocess.run` command in Python lets you execute command line tasks. To execute tasks in parallel you can wrap `subprocess.run()` within a function that you'll use on the Dask cluster. This will allow Dask to execute the command on a worker machine, allowing you to call it many times in parallel. You can do so using the [futures](https://docs.dask.org/en/latest/futures.html) capabilities of Dask, specifically the `Client.map()` function.

The example below uses the very basic `echo` CLI bash command to print a string showing some pairs of exponents. A list of inputs is executed in parallel on the Dask cluster using `Client.map()` and `Client.gather()`. The `echo` command prints the results to the output as a CLI command. In practice, you'll probably want to choose a more interesting CLI command besides `echo`.

```python
# for running CLI tools
import subprocess

# Dask libraries and setup
import dask
from dask_saturn import SaturnCluster
from dask.distributed import Client

cluster = SaturnCluster()
client = Client(cluster)

# the function to run on the Dask cluster
def lazy_exponent(args):
    x, y = args
    result_string = f'"{x}**{y}={x ** y}"'
    subprocess.run(["echo", result_string], check=True)

# Executing the function across an input list
inputs = [[1,2], [3,4], [5,6], [9, 10], [11, 12]]

example_futures = client.map(lazy_exponent, inputs)
futures_gathered = client.gather(example_futures)
```

To see the output, click on the **Logs** button of the resource. From there expand the Dask workers to see the output from code executed on the cluster. Or switch to the **aggregated logs** to see all of the Dask worker output combined together. In the event that one of the CLI calls returns an error, the `check=True` argument to `subprocess.run` will cause the function to have a `CalledProcessError` and thus the Dask task to fail, propagating the error back to you in a helpful way.

![Logs button](/images/docs/logs-link.png "doc-image")

Note that when running CLI commands on a Dask cluster, any results saved to the Dask workers will be deleted once the cluster is offline. Thus if your commands save any results, it's important to then take the results and return them to a permanent place. For instance within your `@dask.delayed` function you could add commands to upload files to a location like AWS S3, or you could have the function read the results into Python and return them to the client machine.
