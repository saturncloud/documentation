# Quick Git Setup

# Quick Git Setup for Saturn Cloud

This guide helps you set up Git integration quickly so you can attach repositories to your resources during creation.

## 1. Set Up SSH Keys (One-Time Setup)

1. Go to **User Profile** → **Access Keys** in the sidebar

<img src="/images/sidebar-click-git-quickstart-for-ssh.png" alt="Git SSH Key setup in sidebar" class="doc-image">

2. Find the **Git SSH Key** section and click the **New Private Key** button.

<img src="/images/git-ssh-key-section.png" alt="Git SSH Key section" class="doc-image">

3. In the key creation window, choose your preference for **Value**:
   - **Saturn Cloud Generated Key**: Let Saturn Cloud automatically generate a secure public/private key pair.
   - **Upload Your Own Key**: Paste your own existing private key into the text area.
4. Click **Submit** to save the key.

<img src="/images/create-ssh-key.png" alt="Create SSH Key" class="doc-image">

5. Copy the displayed public key and add it to your Git host (e.g., GitHub, GitLab, Bitbucket) to authorize access.

<img src="/images/copy-public-ssh-key.png" alt="Copy Public SSH Key" class="doc-image">

## 2. Add Your Repository to Saturn Cloud

1. Go to **Git Repositories** in the sidebar

<img src="/images/sidebar-git-repositories.png" alt="Git Repositories in sidebar" class="doc-image">

2. Click **New Git Repository** button

<img src="/images/new-git-repository.png" alt="New Git Repository" class="doc-image">

3. Enter your repository URL (the same URL you'd use for `git clone`)
4. Click **Submit**

<img src="/images/add-git-repositories.png" alt="Add Git Repository" class="doc-image">

## 3. Attach to Your Resource

When creating a Jupyter Server or R Server from Workspaces, Deployments, or Jobs:

<img src="/images/sidebar-resources.png" alt="Resources in sidebar" class="doc-image">

1. In the creation form, find the **Details** section after completing **Instance Size** and **Resource Type** sections.
2. Scroll down to the **Git repositories** section and select your repository from the dropdown
3. Click **Create Resource**

<img src="/images/git-repository-resource-creation.png" alt="Git Repository Resource Creation" class="doc-image">

Your code will be automatically cloned when the resource starts!

## Need Help?

- **[Full Git documentation](/docs)** - Complete guide with all options
- **[GitHub SSH setup](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)** - GitHub-specific instructions
- **[GitLab SSH setup](https://docs.gitlab.com/ee/user/ssh.html)** - GitLab-specific instructions
