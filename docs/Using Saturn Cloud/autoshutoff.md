# Configure Auto-Shutoff in Saturn Cloud

One of the most useful Saturn Cloud features is auto-shutoff. Auto-shutoff is a setting for a period of time of inactivity after which a Jupyter server resource will automatically shut down. This can prevent you from wasting money on expensive resources that are left on over the weekend.

## How does it work?

When Jupyter servers are created, you can choose an auto-shutoff period. This defaults to 1 hour, but you can also select 6 hours, 3 days, 7 days, or never. JupyterLab records user activity, in this case defined as having a web browser pointed at the Jupyter server. Once you close your browser tab, or laptop, the auto-shutoff timer starts. When the idle time exceeds the configured auto-shutoff period, your Jupyter server is shut down.

<img src="/images/docs/autoshutoff.png" alt="Select auto-shutoff" class="doc-image">

On most systems, closing your laptop will also sleep your machine, which will also trigger the auto-shutoff timer.

## Limitations

- Because we use the web browser to detect activity, if you are primarily interacting with your resource over SSH, your Jupyter server may shut down pre-maturely. We recommend choosing a longer auto-shutoff period (6 hours or greater) for these use cases.
- There is no built-in auto-shutoff for Dask clusters. Dask clusters attached to Jupyter servers automatically shut down when the resource they are attached to is shut down, so if you're using a Jupyter notebook in conjunction with a Dask cluster you will get good auto-shutoff behavior. However if you are accessing the Dask cluster without starting Jupyter (for example [connecting to the dask cluster directly from outside of Saturn](<docs/Using Saturn Cloud/External Connect/colab_external_connect.md>), you will have to shut down your cluster manually.
