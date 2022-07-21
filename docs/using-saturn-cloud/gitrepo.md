# Git Repositories

This article describes how use existing git repositories with your Saturn Cloud resources. Saturn Cloud resources have the ability to connect to git repositories which can load the code at resource startup. This document first covers setting up SSH credentials then goes into adding git repositories to Saturn Cloud and finally adding those to resources. When the document refers to _git hosts_ it means services like GitHub, Bitbucket, and GitLab which host git repositories for you.

{{% alert title="Manually configuring git" %}}
This article discusses the Saturn Cloud built-in git functionality. If you so choose you could alternatively set up git yourself by having a [resource startup script](<{{ ref "install-packages.md"}}>) clone a git repo, or by manually using the terminal within an Jupyter or RStudio server to do so. The Saturn Cloud built-in functionality has a number of conveniences and no drawbacks compared to manually setting up git yourself, so we recommend you try it.
{{% /alert %}}

## Set up git SSH Keys

Saturn Cloud supports using SSH keys as a protocol to connect to git repositories, which requires a public key and private key pair shared with Saturn Cloud and your git host.
So the first step is sharing the right keys. Thankfully, Saturn Cloud will automatically generate a public/private key pair for you by default, but you can change it as needed.

Go to the **Git Repositories** tab in Saturn Cloud, at the bottom you'll see an **SSH Keys** area, which will first request you create a key with a new Saturn Cloud account. You can generate a public/private key pair within Saturn Cloud or upload your own:

![Git SSH key generation](/images/docs/git-ssh-key-generating.jpg "doc-image")

After you have a key pair, take the SSH public key and add it to your git Host to create the secure connection. Refer to your git host for how to do this (for example, [here are the directions for GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)).

![Git SSH key UI](/images/docs/git-ssh-key.jpg "doc-image")

There are several adjustments you can make for the SSH key:

* If you'd like to change the key (either upload a new one or regenerate a key pair in Saturn Cloud), press the **Replace Key** button.
* Normally Saturn Cloud will use a single private key for all your repositories. If you have different keys for different repositories, like if you're using multiple git hosts, you can
Set the private keys on a per-repo basis by sliding the **Allow Multiple Keys** toggle.

## Add a Git Repository to Saturn Cloud

Once you've set up your SSH keys you can add git repositories to your resources. On the resource page, school to the **git repositories** section and either add a new repository or choose an existing one. Once the repository is added, you can adjust several properties of it.

![Resource git repositories](/images/docs/git-ui.png "doc-image")

* **Remote URL:** The URL for the repository in the git host (this is the link you'd use when running `git clone` at terminal)
* **Location:** the folder that will store the git repository. This will be a sub folder of `/home/jovyan/git-repos/`
* **Restart behavior:** When a resource restarts, what should happen to the repository? Either have it stay in its current state (good for tasks like exploratory analysis), or have it reset to the default reference (good for systems like deployments where you want to use the latest version). For job and deployment resources this must be recloned on restart.
* **Reference:** What branch, commit, or tag to clone when the repository is recloned.

Now, when you log in to your Jupyter server, at the top level of your file system  the folder `git-repos` will contain all the repositories attached to this resource.

_If a git repository is removed from a resource via the UI then you will need to manually delete the folder within a Jupyter or RStudio server._

## Using git in within your Jupyter Server

To do git commands within Jupyter Server like pushing, pulling, and committing, you can use one of two methods: either from the command line or by using the GUI provided by a JupyterLab plugin:

You can open a terminal window from within the Jupyter server and use git from the command line. First open the launcher by pressing the "+" button:

![JupyterLab launcher button](/images/docs/terminal-01.png "doc-image")

Then select "terminal" to go to the command line of the resource:

![New terminal button](/images/docs/terminal-02.png "doc-image")

Alternatively, you can also use the git functionality built into JupyterLab with a GUI, using the plugin on the left of the screen:

![Git plugin button](/images/docs/git-plugin.png "doc-image")

## Using git within RStudio server

When using RStudio you can use the RStudio built in git functionality, which defaults to a tab in the upper
right hand corner of the IDE. You can also use the terminal to directly run git commands, using the terminal tab in the lower left hand corder of the screen.

![RStudio git](/images/docs/rstudio-git.png "doc-image")