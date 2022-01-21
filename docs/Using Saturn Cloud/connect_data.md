# Connect to Data

There are two distinct ways to use data with Saturn Cloud:

* **Upload the data to the resource.** - Jupyter and RStudio server resources have connected drive to them. The home directory of the drive persists even after you shut off the resource. For these types of resources you can upload data directly to that drive using the built in upload functionality of [JupyterLab](<docs/Examples/python/load-data/qs-load-data-local-files.md>) or RStudio, or by using an [SSH connection](<{{ ref "ide_ssh.md"}}>) to place data on the drive.
* **Connect the Saturn Cloud resource to an external dataset.** - If the data lives on an external cloud location such as an AWS S3 bucket, a Snowflake database, or a Kaggle dataset, you can connect to those locations using the tooling specific to your programming language. For examples of how to make these connections in Python, see our [loading data examples](<docs/Examples/python/load-data/qs-load-data-snowflake.md>).