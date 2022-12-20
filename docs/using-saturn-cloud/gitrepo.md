# Git Repositories

This article describes how use existing git repositories with your Saturn Cloud resources. Saturn Cloud resources have the ability to connect to git repositories which can load the code at resource startup. This document first covers setting up SSH credentials then goes into adding git repositories to Saturn Cloud and finally adding those to resources. When the document refers to _git hosts_ it means services like GitHub, Bitbucket, and GitLab which host git repositories for you.

{{% alert title="Manually configuring git" %}}
This article discusses the Saturn Cloud built-in git functionality. If you so choose you could alternatively set up git yourself by having a [resource startup script](<docs/using-saturn-cloud/install-packages.md>) clone a git repo, or by manually using the terminal within an Jupyter or R server to do so. The Saturn Cloud built-in functionality has a number of conveniences and no drawbacks compared to manually setting up git yourself, so we recommend you try it.
{{% /alert %}}

## Set up git SSH Keys

Saturn Cloud supports using SSH keys as a protocol to connect to git repositories, which requires a public key and private key pair shared with Saturn Cloud and your git host. So the first step is setting up the right keys. Thankfully, Saturn Cloud will automatically generate a public/private key pair for you by default, but you can change it as needed.

From the dropdown **User** menu at the top left, select **Manage [username]**. _This is a secure storage location, and it will not be available to the public or other users without your consent._

<img src="/images/docs/manage-user-settings-arrow.png" style="width:200px;" alt="Saturn Cloud left menu with arrow pointing to manage user tab" class="doc-image">

Under **Access Keys**, find the section called **Git SSH Key.** Click on **Create an SSH Key**, and you will be taken to the SSH key creation form.

![Arrow pointing to Create an SSH key button](/images/docs/create-git-ssh-key-arrow.png "doc-image")

Here, you can generate a public/private key pair within Saturn Cloud or upload your own:

![Git SSH key generation](/images/docs/add-git-ssh-key.png "doc-image")

After you have a key pair, copy the SSH public key and add it to your git Host to create the secure connection. Refer to your git host for how to do this (for example, [here are the directions for GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)).

<img src="/images/docs/git-ssh-key-new.png" style="width:400px;" alt="Git SSH key UI" class="doc-image">

There are several adjustments you can make for the SSH key:

* If you'd like to change the key (either upload a new one or regenerate a key pair in Saturn Cloud), press the **Replace Key** button.
* Normally Saturn Cloud will use a single private key for all your repositories. If you have different keys for different repositories, like if you're using multiple git hosts, you can
Set the private keys on a per-repo basis by sliding the **Allow Multiple Keys** toggle.

## Add a Git Repository to Saturn Cloud

Once you've set up your SSH keys, you can add git repositories to your resources. From the left-hand menu, select the **Git Repositories** tab.

<img src="/images/docs/left-menu-git-repositories-arrow.png" style="width:200px;" alt="Saturn Cloud left menu with arrow pointing to git repos tab" class="doc-image">

From the git repositories page, select the **New** button at the top right corner. Here, you can add a repository via remote URL (this is the link you'd use when running `git clone` in the terminal).

![Screenshot of the page for adding a new git repository](/images/docs/add-git-repository.png "doc-image")

To access your git repository from a particular resource, navigate to the **Git Repos** tab in that resource's page and add your repository from the **Add a git repository** dropdown menu. Note that you can also add a repository directly from this page by clicking **New Git Repository**.

![Screenshot of resource git repository tab](/images/docs/add-git-repo-to-resource.png "doc-image")

Once the repository is added, you can adjust several properties from the edit menu under **Actions**.

<img src="/images/docs/edit-repository-attachment.png" style="width:400px;" alt="Screenshot of edit repository attachment menu" class="doc-image">

* **Path:** the folder that will store the git repository. This will be a sub folder of `/home/jovyan/git-repos/`
* **Reference:** What branch, commit, or tag to clone when the repository is recloned.
* **Restart behavior:** When a resource restarts, what should happen to the repository? Either have it stay in its current state (good for tasks like exploratory analysis), or have it reset to the default reference (good for systems like deployments where you want to use the latest version). For job and deployment resources this must be recloned on restart.

Now, when you log in to your Jupyter server, at the top level of your file system  the folder `git-repos` will contain all the repositories attached to this resource.

_If a git repository is removed from a resource via the UI then you will need to manually delete the folder within a Jupyter or R server._

## Using git in within your Jupyter Server

To do git commands within Jupyter Server like pushing, pulling, and committing, you can use one of two methods: either from the command line or by using the GUI provided by a JupyterLab plugin:

You can open a terminal window from within the Jupyter server and use git from the command line. First open the launcher by pressing the "+" button:

![JupyterLab launcher button](/images/docs/terminal-01.png "doc-image")

Then select "terminal" to go to the command line of the resource:

![New terminal button](/images/docs/terminal-02.png "doc-image")

Alternatively, you can also use the git functionality built into JupyterLab with a GUI, using the plugin on the left of the screen:

![Git plugin button](/images/docs/git-plugin.png "doc-image")

## Using git within R server

When using R you can use the R IDEs built in git functionality, which defaults to a tab in the upper
right hand corner of the IDE. You can also use the terminal to directly run git commands, using the terminal tab in the lower left hand corder of the screen.

![R server git](/images/docs/rstudio-git.png "doc-image")
