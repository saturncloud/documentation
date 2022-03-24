# Connect via SSH

The SSH feature in Saturn Cloud lets you connect to Jupyter server and RStudio server resources directly from your local machine. This allows you to use IDEs such as VSCode, PyCharm, or to interact with the resources from a terminal. You can also use SSH to move files to and from your local machine.

To use SSH in Saturn Cloud, you must do the following:
1) Set up an SSH key pair.
2) Add the public key to Saturn Cloud.
3) Enable SSH access to your resource.

## Set up an SSH key pair

You need to set up SSH to make the connection between Saturn Cloud and your local machine. First, create an SSH key pair. 

> If you need help setting up the SSH keys on your local machine, check out any of these online resources:
> * <a href="https://www.ssh.com/ssh/keygen/" target='_blank' rel='noopener'>https://www.ssh.com/ssh/keygen/</a>
> * <a href="https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2" target='_blank' rel='noopener'>https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2</a>
> * <a href="https://www.redhat.com/sysadmin/configure-ssh-keygen" target='_blank' rel='noopener'>https://www.redhat.com/sysadmin/configure-ssh-keygen</a>

Once you have your key pair set up, add your SSH public key to your Saturn Cloud account. ***The SSH private key will stay on your local machine.***

## Add your SSH public key to Saturn Cloud

Sign in to your Saturn Cloud account, click on your username, and select **User Settings** from the menu on the left. There, add your SSH credential information. *This is a secure storage location, and it will not be available to the public or other users without your consent.*

<img src="/images/docs/settings-page.png" style="width:200px;" alt="Saturn Cloud left menu with arrow pointing to Settings tab" class="doc-image">

Under "Access Keys", find a section called "Resource Connection SSH Keys." Click on **New**, and you will be taken to the SSH key creation form. 

![User settings page with arrow to new SSH key page](/images/docs/user-settings-page.png "doc-image")

Select the appropriate owner, give the key a name, and copy the entire contents of your SSH public key file into the "Value" section.

![Add SSH Key page](/images/docs/ssh-key-form.png "doc-image")

With this complete, your SSH credentials will be accessible by Saturn Cloud resources!

{{% alert title="Actively running resources" %}}
If you have resources that are running when you upload a new SSH public key, the resources will need to be restarted before the key can be used to connect.
{{% /alert %}}

## Enable SSH for a Jupyter server or RStudio server resource

On the resource page, click **Edit** to edit the resource details. If it is running, you may want to stop it since the resource will need to be restarted when you make this change.

<img src="/images/docs/edit-resource-button.jpg" alt="Edit button within a resource" class="doc-image">

You will be taken to a page that allows you to edit the settings of the server. Check the box that says "Allow SSH Connections," and click **Save** at the bottom of the page.

<img src="/images/docs/ssh2.png" alt="Screenshot of Saturn Cloud Edit Jupyter Server form, with red arrow pointing to Enable SSH URL button" class="doc-image">

Start the resource, and you will see an SSH URL provided on the instance's card. Press the **clipboard icon** to copy this string.

<img src="/images/docs/ssh3.png" alt="Screenshot of Jupyter Server card with server running, arrow pointing to SSH URL shown, with box encircling it" class="doc-image">

***

## Connect to an external IDE

Now you can set up the connection to your preferred IDE. You'll need to provide the IDE with the following:

* **Host**: Also called "Hostname" or "Server Hostname". This is the part to the right of the `@` in the URL above.
* **User**: This is the part to the left size of the `@` in the URL above.
* **Private key filepath**: This is the location on your computer where the SSH private key is stored. This is the complement to the SSH public key you saved on the Saturn Cloud credentials page.

### VSCode

Using the `Remote - SSH` plugin, VS Code can connect to the running resource directly. To set this up, follow these instructions:

<a href="https://code.visualstudio.com/docs/remote/ssh" target='_blank' rel='noopener'>https://code.visualstudio.com/docs/remote/ssh</a>

From the command palette (Ctrl+Shift+P or Cmd+Shift+P, depending on OS), select **Remote-SSH: Connect to host…**, then select **+ Add New SSH Host...**. Paste the string copied from the resource card when prompted to enter the SSH command. VSCode will automatically parse the host and user values for you.

Add the entry to your `~/.ssh/config` file. Open the file, and you will see an entry in the form:

```
Host <Host>
    HostName <Host (see above)>
    User <User (see above)>
```

Update the name (to the right of "Host") above to match your server as you see fit. Then, from the command palette, select **Remote-SSH: Connect to host…**, then select your config entry name from the list.

### PyCharm

> Note: This functionality is only available in the Professional edition of PyCharm. It is not available in the Community (free) edition.

To set up PyCharm to connect to a running resource, follow JetBrains’ instructions here:

<a href="https://www.jetbrains.com/help/pycharm/creating-a-remote-server-configuration.html" target='_blank' rel='noopener'>https://www.jetbrains.com/help/pycharm/creating-a-remote-server-configuration.html</a>.

Choose SFTP for the credentials type, and create a new SSH configuration. Use the following values:

* Host: <Host (see above)>
* Port: 22
* User name: <User (see above)>
* Authentication type: Key pair
* Private key file: <Private key filepath (See above)>
* Passphrase: The passphrase to your key, if you set one

Once the new server configuration has been added, navigate to **Tools > Deployment > Remote Host**. From the new panel that this opens, select the new server configuration you just created.

To open a terminal to run commands, navigate to **Tools > Start SSH Session…**

### Terminal

Using the terminal application of your choice, the server can be accessed via:

```bash
ssh <resource URL>
```
If your key is not added to your SSH agent (or no SSH agent is running), the key can be manually specified:

```bash
ssh -i path/to/key <resource URL>
```
## Move files to and from your local machine

To see examples of moving data to and from your local machine using SSH, see our loading data examples:

* [Load Data: Python](<docs/Examples/python/load-data/qs-load-data-local-files.md>)
* [Load Data: R](<docs/Examples/R/load-data/qs-r-load-data-local-files.md>)