# Connect via SSH

## Connect to an external IDE

Now you can set up the connection to your preferred IDE. You'll need to provide the IDE with the following:

-   **Host**: Also called "Hostname" or "Server Hostname". This is the part to the right of the `@` in the URL above.
-   **User**: This is the part to the left size of the `@` in the URL above.
-   **Private key filepath**: This is the location on your computer where the SSH private key is stored. This is the complement to the SSH public key you saved in Saturn Cloud.

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

-   Host: <Host (see above)>
-   Port: 22
-   User name: <User (see above)>
-   Authentication type: Key pair
-   Private key file: <Private key filepath (See above)>
-   Passphrase: The passphrase to your key, if you set one

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

-   [Load Data: Python](<docs/Examples/python/load-data/qs-load-data-local-files.md>)
-   [Load Data: R](<docs/Examples/R/load-data/qs-r-load-data-local-files.md>)
