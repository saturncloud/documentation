# Shared Folders

{{% alert title="Team Feature" %}}
This functionality is only available to users on the Hosted Organizations and Enterprise plans.
{{% /alert %}}

In many data science situations there are datasets that need to be shared within a data science team: such as data to train models, results from analysis, and output graphs. While Saturn Cloud can easily [connect to data](<docs/using-saturn-cloud/connect_data.md>) outside of Saturn Cloud, the platform also provides a way to create **shared folders** which stay within Saturn Cloud and can persist between different resources. Shared folders allow you to pass data between resources (including those owned by different users) as easily as saving a file within a resource. This can make it easier for data scientists to work using the same data without having to set up connections to different tools.

Any user or group in Saturn Cloud can own a shared folder:

* In the case of the user, the folder contents will be accessible from any resource owned by the user.
* In the case of a group owned shared folder, not only will resources owned by the group have access to the folder, any resource owned by any member of the group will also have access to the shared folder.
* Each shared folder has the option to change the **visibility** setting. If the visibility is set to organization then anyone in the organization can access the folder, even if they are not the owner.

Notes about the current implementation of shared folders:

1. Currently, all shared folders are automatically attached to all resources. In a future release of Saturn Cloud users will have the option to attach the shared folder only to specific resources.
2. Shared folders are only usable by workspace resources (Jupyter server and R server resources), and their associated Dask clusters. The ability for jobs, deployments, and prefect cloud flows to use shared folders will be added in an upcoming release.
3. Shared folders are mounted in fixed paths across all resources. In a future release you'll be able to specify exactly where the shared folder is stored.

## Creating shared folders

To create a shared folder, go to the **Shared Folders** tab on the Saturn Cloud application. This will list all the active shared folders you have, and give you the option to edit or delete them. You can also create a new shared folder.

![Shared folders list](/images/docs/shared-folders-list.webp "doc-image")

To create a new shared folder, choose from the following options:

* **Owner** - The Saturn Cloud user or group who will own the resource. They have the ability to access the data and edit the settings for it.
* **Disk space** - The size of the disk to use. While Saturn Cloud does not charge you for shared folders, a larger disk will cause more AWS resources to be consumed on your account.
* **Visibility** - If you choose to make the shared folder visibile to the entire organization then any user or group will be able to view and edit the data. This is useful for data shared across an entire company.

![New shared folder](/images/docs/new-shared-folder.webp "doc-image")

## Using shared folders

If a Jupyter server or R server resource has access to a shared folder it will show up in the **Shared Folders** section of the resource. This section will display the connected shared folders and the path they take within the file system of the resource. Currently, each shared folder is mounted in the location `/home/jovyan/shared/{owner}/{folder-name}`.

![Shared folders on a resource](/images/docs/using-shared-folders.webp "doc-image")

Once you start the resource, you'll see the folder in your JupyterLab or R IDE. Any file you save into that folder will be stored in the cloud and other resources will be able to use it.
