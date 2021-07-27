# Using Git Repositories in Saturn Cloud

This article describes how to work with git repositories in Saturn Cloud. As of this writing, only GitHub repos have built-in support.

If you need to work with a repo on another hosted source control management (SCM) service such as AWS CodeCommit, BitBucket, GitLab, or any other SCM service that supports cloning repos over SSH, skip to ["Using a Start Script"](<docs/Reference/resources_wont_start.md#using-a-start-script>).

## Set up SSH Keys

To add Repositories in Saturn Cloud, you'll need to use SSH credentials.

You will need to set up SSH to make the connection between Saturn Cloud and Github. Your first step is to add an SSH **Private** Key Credential to your Saturn Cloud account. *The SSH **Public** Key will then go in Github.*

If you need help setting up a Credential, please [visit our reference page about credentials in Saturn Cloud](<docs/Using Saturn Cloud/credentials.md>).

{{% alert title="Note" %}}
Right now, you need to use a very specific form of SSH creation to connect your Github with Saturn Cloud. We plan to loosen this strictness soon.

In the meantime, create your SSH key in your laptop's terminal using this code:
```ssh-keygen -t rsa -m pem```
{{% /alert %}}

> If you need help setting up your SSH keys on your laptop, there are some reference options elsewhere online.
> * <a href="https://www.ssh.com/ssh/keygen/" target='_blank' rel='noopener'>https://www.ssh.com/ssh/keygen/</a>
> * <a href="https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2" target='_blank' rel='noopener'>https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2</a>
> * <a href="https://www.redhat.com/sysadmin/configure-ssh-keygen" target='_blank' rel='noopener'>https://www.redhat.com/sysadmin/configure-ssh-keygen</a>

If you need help adding the SSH Public Key to Github, they have great reference materials: <a href="https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh" target='_blank' rel='noopener'>https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh</a>

***

## Add Repository
*Note: you are always welcome to manually clone a repo inside Jupyter when working in Saturn Cloud. This instruction will show you how to make a persistent connection to that repo that can be easily reused. Persistent connections are available on your Dask clusters, deployments, jobs and anything else associated with the project.*

Now that your credentials are added, click "Tools" in the menu and choose "Git Repositories". Click the "Create" button in the top right corner.

<img src = "/images/docs/repos1.png" style="width:200px;" alt="Screenshot of side menu of Saturn Cloud product with Git Repositories selected" class="doc-image">

This will take you to a form, where you can set up the repository you wish to use.

In this form, you'll fill in your repository details.
* Remote URL: in Github, this is the link you'd use when running `git clone` at terminal. It must be the SSH link, not an HTTPS link.
* Filepath: You can choose whatever you like here, but it will be made a filepath inside your projects.
* Remote Visibility: This is to let Saturn Cloud know if the repo you're adding requires credentials.
* Branch: You can choose to check out a non-main branch by default if you would like, and you'd make that selection here.
* Expected Usage: select whether you will be only using this repo for reading from, and would like it to be refreshed to the remote state every time you start the project, or if you'd like it to be read/write, so your edits will be persisted and not overwritten.

<img src="/images/docs/repos2.jpg" alt="Screenshot of Saturn Cloud Add Repository form" class="doc-image">

## Attach Credentials
After you've filled out this form, you'll be asked to specify the credential you want to use to access the repo, if it is not public. Use the dropdown to choose the SSH Private Key that you added above.

<img src="/images/docs/repo_creds.png" alt="Screenshot of Saturn Cloud new repository named benchmarks, where Credential select dropdown is shown" class="doc-image">

## Link Repository to Project

At this point, you should add your Repository to a project in order to use it. If you have already created a project, edit it, and add repository as shown below.  If you have not created a project, [follow our instructions for doing that](<docs/Getting Started/start_project.md>), including the step where you can add a Repository.

<img src="/images/docs/image6.png" alt="Screenshot of Saturn Cloud project setup page, where Repositories section is centered" class="doc-image">

Now, when you log in to your Jupyter Server, at the top level of your file system, you'll see the folder `git-repos` which contains all the repositories you have attached to this project. You can also clone others inside the terminal in your Jupyter server if you need to.

## Using a Start Script

For remote hosts that are not supported by Saturn's git repo integration, you can set up automatic repo cloning using a feature called the "start script". This is a small shell script which runs every time Saturn creates a Jupyter server, Deployment, Dask worker, or Prefect Cloud flow run.

1. Create an SSH keypair. Put the public key on your source code management system, and add the private key as a Saturn credential with `location` set to `/home/jovyan/.ssh/id_rsa_example`. See ["Set up SSH Keys"](<docs/Reference/resources_wont_start.md#set-up-ssh-keys>) for instructions.
2. In your Saturn project, expand the "Advanced Settings" section.

    <img src="/images/docs/advsettings.png" alt="Screenshot of Saturn Cloud project creation page Advanced Settings section" class="doc-image">

3. Paste something like the following into the "Start Script" section.

    This script tells Saturn to use the SSH private key credential you added, and includes code to make sure that the repository you cloned isn't overwritten if you restart a resource.

    ```shell
    REPO_HOST=gitlab.com
    PRIVATE_KEY_LOCATION=/home/jovyan/.ssh/id_rsa_example

    # tell git where to look for an SSH private key to use with gitlab.com #
    if ! grep gitlab /home/jovan/.ssh/config; then
        echo "Host ${REPO_HOST}" >> /home/jovyan/.ssh/config
        echo " HostName ${REPO_HOST}" >> /home/jovyan/.ssh/config
        echo " User git" >> /home/jovyan/.ssh/config
        echo " IdentityFile ${PRIVATE_KEY_LOCATION}" >> /home/jovyan/.ssh/config
        echo " StrictHostKeyChecking no" >> /home/jovyan/.ssh/config
    fi

    # only clone if the repo is not already there
    mkdir -p /home/jovyan/git-repos

    if [ ! -d /home/jovyan/git-repos/eigen/.git ]; then
        echo "cloning Eigen from GitLab"
        git clone git@gitlab.com:libeigen/eigen.git /home/jovyan/git-repos/eigen
    fi
    ```
