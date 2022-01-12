# Configure Auto-Shutoff in Saturn Cloud

Auto-shutoff is a setting that sets a period of time of inactivity after which a Jupyter server will automatically shut down. This can prevent you from wasting money on expensive resources that are accidentally left on.

{{% alert title="RStudio support" %}}
Currently RStudio resources to not have the ability to auto-shutoff. This will be coming in a future release.
{{% /alert %}}

## How does it work?

When Jupyter servers are created, you can choose an auto-shutoff period. This defaults to 1 hour, but you can also select 6 hours, 3 days, 7 days, or never. Saturn Cloud determines if a Jupyter server resource is running by either (1) a browser window for JupyterLab is open or (2) an SSH connection is open to the resource. If neither of those are true for longer than the configured auto-shutoff period, your Jupyter server is shut down. Note that in the case of a browser window being open, for modern browsers having a laptop go to sleep will disconnect the browser window and thus start the idle timer.

<img src="/images/docs/autoshutoff.png" alt="Select auto-shutoff" class="doc-image">

The auto-shutoff behavior for a Dask cluster connected to a resource follows the resource itself--if the Jupyter server shuts down the associated Dask cluster will shut down as well. In rare situations you may choose to turn on a Dask cluster without turning on the corresponding JupyterLab resource, for example if you're [connecting to the Dask cluster directly](<docs/Using Saturn Cloud/External Connect/sagemaker_external_connect.md>). In these situations the Dask cluster won't ever automatically shut down.
