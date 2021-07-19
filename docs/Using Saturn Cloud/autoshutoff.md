# Autoshutoff

One of the most useful Saturn Cloud features is Autoshutoff. This can prevent you from wasting money on expensive resources that are left on over the weekend.

## How does it work?

When Jupyter servers are created, you can choose an autoshutoff period. This defaults to 1 hour, but you can also select 6 hours, 3 days, or 7 days. Jupyter records user activity, in this case defined as having a web browser pointed at the Jupyter server. Once you close your browser tab, or laptop, the autoshutoff timer starts. When the idle time exceeds the configured autoshutoff period, your Jupyter instance is shut down.

<img src="/images/docs/autoshutoff.png" alt="Select Autoshutoff" class="doc-image">

On most systems, closing your laptop will also sleep your machine, which will also trigger the autoshutoff timer.

## Limitations

- Because we use the web browser to detect activity, if you are primarily interacting with your instance over SSH, your Jupyter instance may shut down pre-maturely. We recommend choosing a longer autoshutoff period (6 hours or greater) for these use cases.
- There is no built-in autoshutoff for Dask clusters. Dask clusters attached to Jupyter instances automatically shut down when the instances are shut down, so if you're using a Jupyter notebook in conjunction with a Dask cluster you will get good autoshutoff behavior. However if you are accessing the Dask cluster without starting Jupyter (for example [connecting to the dask cluster directly from outside of Saturn](../external-connect/)), you will have to shut down your cluster manually.
