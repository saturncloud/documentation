# Load Data From Your Local File System
## Overview
Flat files are a very common data storage option, and lots of our customers will use them at some time. This tutorial will show you how to load a dataset from a file (for example, a CSV or JSON file) on disk into Saturn Cloud.

Before starting this, you should create a Jupyter server resource. See our [quickstart](https://saturncloud.io/docs/start_in_ten/) if you don't know how to do this yet.

## Process

### Upload Files in the UI
If you want to place a flat file in your Saturn Cloud Jupyter server, there's a simple UI option. 

![Jupyter Lab workspace with arrow pointing towards the upload button](https://saturn-public-assets.s3.us-east-2.amazonaws.com/example-resources/local-file-upload-arrow.png "doc-image")

Simply select the file(s) you want to access and they will be uploaded!

### Upload Files via SSH
If you prefer to upload your local files programmatically, Saturn Cloud makes it easy to copy files from your local machine via SSH.

First, you need to set up SSH connections to your resources. See [our documentation](https://saturncloud.io/docs/user-guide/how-to/ide_ssh/) about how to do this. Take note of the SSH URL in the last step - you will need it to transfer files.

#### Transfer Files via SCP

The [`scp`](https://linux.die.net/man/1/scp) command allows you to copy files over SSH connections. To use it, all you need to do is specify the local path to file you want to transfer, the SSH URL, and the path you want the file transferred to in your Saturn Cloud instance.

From the command line on your local machine, the `scp` command takes the following format:


```python
scp "local-filepath" saturn-cloud-ssh-url:"remote-filepath"
```

So your command might look like:


```python
scp "test_file.py" j-ssh-natha-demo-jupyter-dee34c41f0f54daaa2cb20eee84ec28f@ssh.community.saturnenterprise.io:"project/scripts/"
```

And that's it! If you look in your resource's file system, you will see the file.

You can also use `rsync` to perform a similar function. See [the `rsync` documentation](https://linux.die.net/man/1/rsync) for more information.
