# Connect via SSH

The SSH feature in Saturn Cloud lets you connect to Jupyter server and RStudio server resources with other IDES besides the built in JupyterLab and RStudio. In these situations you can connect to your local computer via SSH and use IDEs such as VSCode, PyCharm, or directly interact from a terminal.

## Seting up SSH keys

You will need to set up SSH to make the connection between Saturn Cloud and your laptop. Your first step is to add an SSH Public Key Credential to your Saturn Cloud account. *The SSH Private Key will then stay on your laptop.*

Go to the **Credentials** page of Saturn Cloud and create a new SSH Public Key. There you can upload the key to use for the SSH connection. The docs page on [credentials](<docs/Using Saturn Cloud/credentials.md>) has further details on uploading a key.

{{% alert title="Actively running resources" %}}
If you have resources that are running when you upload a new SSH Public Key then the resources will need to be restarted before the key can be used to connect to them.
{{% /alert %}}

> If you need help setting up the SSH keys on your laptop, there are some reference options elsewhere online.
> * <a href="https://www.ssh.com/ssh/keygen/" target='_blank' rel='noopener'>https://www.ssh.com/ssh/keygen/</a>
> * <a href="https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2" target='_blank' rel='noopener'>https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2</a>
> * <a href="https://www.redhat.com/sysadmin/configure-ssh-keygen" target='_blank' rel='noopener'>https://www.redhat.com/sysadmin/configure-ssh-keygen</a>

## Enable SSH for a Jupyter server or RStudio server resource

On the resource page, click the pencil/paper icon to edit the resource details. If it is running, you may want to stop it since the resource will restart itself automatically if it is running when you make this change.

<img src="/images/docs/edit-resource-button.jpg" alt="Edit button within a resource" class="doc-image">

You will be taken to a page that allows you to edit the settings of the server. Check the box that says “Enable SSH URL to connect to PyCharm, VSCode, or any other IDE” and click "Save" at the bottom of the page.

<img src="/images/docs/ssh2.png" alt="Screenshot of Saturn Cloud Edit Jupyter Server form, with red arrow pointing to Enable SSH URL button" class="doc-image">

Start the resource now, and you will see an SSH URL provided on the instance's card. You will need to provide this URL to your IDE later.

<img src="/images/docs/ssh3.png" alt="Screenshot of Jupyter Server card with server running, arrow pointing to SSH URL shown, with box encircling it" class="doc-image">

***

## IDE Specific Instructions

Now, you can set up the connection according to your preferred IDE. We have detailed instructions for a few of these below. You'll need some common information for most, and for other IDEs not listed.

* **Host**: Also called "Hostname" or "Server Hostname". This is the hostname from the resource details (see red box in the screenshot above) - remove `joyvan@`
* **User**: `jovyan`
* **Private key filepath**: This is the location on your computer where the SSH Private Key is stored. This is the complement to the SSH Public Key you saved on the Saturn Cloud credentials page.

### VSCode

Using the `Remote - SSH` plugin, VS Code can connect to the resource instance directly. To set this up, following their instructions: <a href="https://code.visualstudio.com/docs/remote/ssh" target='_blank' rel='noopener'>https://code.visualstudio.com/docs/remote/ssh</a>

To make configuration easy, it is suggested to add an entry to your `~/.ssh/config` file (create it if it doesn’t exist):

```
Host myjupyter
    HostName <Host (see above)>
    User <User (see above)>
    IdentityFile <Private key filepath (See above)>
```

Update the name above to match your server as you see fit. Then, from the command palette (Ctrl+Shift+P or Cmd+Shift+P, depending on OS), select `Remote-SSH: Connect to host…`, then select your config entry (created above) from the list. Once opened, select “Open Folder”, then choose `/home/jovyan`, or whichever subfolder in the server you would like to view. 

### PyCharm

> Note: this functionality is only available in the Professional edition of PyCharm - it is not available in the Community (free) edition.

To set up PyCharm to connect to a running resource, follow JetBrains’ instructions here: <a href="https://www.jetbrains.com/help/pycharm/creating-a-remote-server-configuration.html" target='_blank' rel='noopener'>https://www.jetbrains.com/help/pycharm/creating-a-remote-server-configuration.html</a>. Choose SFTP for the credentials type and create a new SSH configuration. Use the following values:

* Host: <Host (see above)>
* Port: 22
* User name: <User (see above)>
* Authentication type: Key pair
* Private key file: <Private key filepath (See above)>
* Passphrase: The passphrase to your key, if you set one

Once the new server configuration has been added, navigate to Tools > Deployment > Remote Host. From the new panel this opens, select the new server configuration you just created. This will let you browse the contents of the server, and edit the files in place. Your work is located within `/home/jovyan`. From Tools > Start SSH Session… a terminal can be opened to run commands.

### Terminal

Using the terminal application of your choice, the server can be accessed via:

```bash
ssh <server URL>
```
If your key is not added to your ssh agent (or no ssh agent is running), the key can be manually specified:

```bash
ssh -i path/to/key <server URL>
```
