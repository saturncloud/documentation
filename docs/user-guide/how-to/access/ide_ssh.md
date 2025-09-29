# SSH Connections

## Connect to an external IDE

Now you can set up the connection to your preferred IDE. You'll need to provide the IDE with the following:

-   **Host**: Also called "Hostname" or "Server Hostname". This is the part to the right of the `@` in the URL above.
-   **User**: This is the part to the left side of the `@` in the URL above.
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

### Terminal (Mac/Linux)

For Mac and Linux users, you can access the server directly using the terminal application of your choice via:

```bash
ssh <resource URL>
```

If your key is not added to your SSH agent (or no SSH agent is running), the key can be manually specified:

```bash
ssh -i path/to/key <resource URL>
```

### OpenSSH or WSL (Windows)

Using SSH on a Windows computer is slightly more complicated, but definitely doable. Although there are several different strategies, here we'll focus on setting up and using OpenSSH (if you're interested in other SSH clients, check out [PuTTY](https://www.putty.org/)).

OpenSSH is the open-source version of the SSH tools used by Linux administrators, and has been included in Windows installations since 2018. To use it, you will need at least Windows Server 2019 or Windows 10 (build 1809), PowerShell 5.1 or later, and an account with administrator privileges.

Once you have verified that you meet these requirements, you can check to see if OpenSSH is already installed. To do so, navigate to **Settings > Apps > Optional Features**. If you see **OpenSSH Client** and **OpenSSH Server** in this list, you're good to go. Otherwise, click **Add a feature** at the top of the page. Find and install both **OpenSSH Client** and **OpenSSH Server**. Once setup is complete, return to the **Optional Features** list and verify that both client and server now show up.

Next, open the **Services** app from your start menu. Scroll through the Details pane on the right-hand side of the app until you see **OpenSSH SSH Server**, and double click it. In the startup window that pops up, on the General tab, select **Automatic** from the **Startup type** dropdown menu, then click **Apply**.

You can now access your Saturn Cloud resource by opening PowerShell and running the following command:

```bash
ssh <resource URL>
```

If you'd prefer an alternative to OpenSSH, it's also possible to use SSH from a Linux Terminal on a Windows computer by using Windows Subsystem for Linux (WSL).

> If you need help setting up or using WSL, check out the following links:
>
> -   <a href="https://learn.microsoft.com/en-us/windows/wsl/install" target='_blank' rel='noopener'>https://learn.microsoft.com/en-us/windows/wsl/install</a>
> -   <a href="https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-10#1-overview" target='_blank' rel='noopener'>https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-10#1-overview</a>


## Move files to and from your local machine

To see examples of moving data to and from your local machine using SSH, see our loading data examples:

-   [Load Data: Python](<docs/user-guide/examples/python/load-data/qs-load-data-local-files.md>)
-   [Load Data: R](<docs/user-guide/examples/r/load-data/qs-r-load-data-local-files.md>)

