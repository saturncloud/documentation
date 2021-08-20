# Using Git Repositories in Saturn Cloud

## Alternative: Using a Start Script

Most git repositories should be able to use the method above for adding to resources. For unusual setups that are not supported by Saturn's git repo integration, you can set up automatic repo cloning using a feature called the "start script". This is a small shell script which runs every time Saturn creates a Jupyter server, Deployment, Dask worker, or Prefect Cloud flow run.

In your Saturn resource, expand the "Advanced Settings" section.

![Advanced settings](/images/docs/advanced-settings.jpg "doc-image")

Paste the following into the "Start Script" section, with the appropriate variables for your particular repo. This script tells Saturn to use the SSH private key credential you added, and includes code to make sure that the repository you cloned isn't overwritten if you restart a resource. If you set up multiple SSH keys, you can replace `default_git_ssh_key` with the name of the SSH key you want to use (otherwise leave it as it is). Replace `eigen` and `git@gitlab.com:libeigen/eigen.git` with the name of the repo you want to clone and the SSH URL of the repo, respectively.

```shell
REPO_HOST=gitlab.com
PRIVATE_KEY_LOCATION=/home/jovyan/.ssh/default_git_ssh_key
REPO_NAME=eigen
REPO_SSH_URL=git@gitlab.com:libeigen/eigen.git

# tell git where to look for an SSH private key
if ! grep ${REPO_HOST} /home/jovan/.ssh/config; then
    echo "Host ${REPO_HOST}" >> /home/jovyan/.ssh/config
    echo " HostName ${REPO_HOST}" >> /home/jovyan/.ssh/config
    echo " User git" >> /home/jovyan/.ssh/config
    echo " IdentityFile ${PRIVATE_KEY_LOCATION}" >> /home/jovyan/.ssh/config
    echo " StrictHostKeyChecking no" >> /home/jovyan/.ssh/config
fi

# only clone if the repo is not already there
mkdir -p /home/jovyan/git-repos
if [ ! -d /home/jovyan/git-repos/${REPO_NAME}/.git ]; then
    git clone ${REPO_SSH_URL} /home/jovyan/git-repos/${REPO_NAME}
fi
```
