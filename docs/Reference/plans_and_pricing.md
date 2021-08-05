# Plans and Pricing

Saturn Cloud has three separate plans available to use: Saturn Cloud Hosted Free, Saturn Cloud Hosted Pro, and Saturn Cloud Enterprise

## Saturn Cloud Hosted Free

This is our most basic plan, designed for people looking to get started with cloud notebooks, GPUs, and Dask. It uses the Saturn Cloud hosted environment, meaning the instances and data live securely in our AWS environment. Users get a set number of hours of resource each month, specifically:

* 10 hours of a Jupyter server
* 3 hours of a Dask cluster (with up to 3 workers)

These hours can be applied to CPU powered instances or GPU powered instances. When your monthly hours end you'll get an error message in Saturn Cloud letting you know. You can keep track of your monthly hours by looking at the sidebar or clicking it for more details:

<img class="py-3" style="width:400px;" src="/images/docs/billing-sidebar.png" alt="Billing sidebar" class="doc-image">

Besides the hourly limit, Saturn Cloud Hosted Free has other limitations to it:

* You are only allowed to use some AWS instance types (`Large` CPU instances and `T4-XLarge` GPU instances)
* You cannot run scheduled jobs

The hours reset at the beginning of each month. If at any time you want to increase your available resources, you can switch to Saturn Cloud Hosted Pro without losing any of your work.

## Saturn Cloud Hosted Pro

The premium version of Saturn Cloud Hosted gives users some great additions over the free version. Like the free version this also runs in the secure AWS environment provided by Saturn Cloud. Compared to the free version you have:

* Access to nearly all AWS instance types, including V100 GPUs
* The ability to use as many Dask workers as you want
* Scheduled jobs

Pricing is based on [hours of time your instances run](/docs). You can view your usage from within the Saturn Cloud UI. When you first sign up for Saturn Cloud Hosted Pro you will be charged $5 for the initial up front credits. These will show up as -$5.00 in the UI. Once you work through the initial $5 in credits, every time you use $10 worth of resources your credit card will be charged. If you need to change your credit card number please [contact us](mailto:sales@saturncloud.io).

## Saturn Cloud Enterprise

This is our recommended plan for people working in corporate settings. The enterprise version has the same full feature set as Saturn Cloud Hosted Pro, but it is installed in your corporate cloud environment instead of ours. You can use the size and type of cloud instances your team needs with as many users as you want. We also provide strong customer support for using Saturn Cloud as well as Dask and GPUs--if you have issues with shifting previously written code to using Dask and GPUs we are happy to provide assistance.

If you are interested in Saturn Cloud Enterprise please [contact us](mailto:sales@saturncloud.io).
