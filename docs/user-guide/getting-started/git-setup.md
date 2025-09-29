# Quick Git Setup

# Quick Git Setup for Saturn Cloud

This guide helps you set up Git integration quickly so you can attach repositories to your resources during creation.

## 1. Set Up SSH Keys (One-Time Setup)

1. Go to **User Profile** → **Access Keys** in the sidebar
2. Find the **Git SSH Key** section
3. Click **Submit**
4. Copy the public key and add it to your Git host (GitHub, GitLab, Bitbucket)

<img src="/images/sidebar-click-git-quickstart-for-ssh.png" alt="Git SSH Key setup in sidebar" class="doc-image">

<img src="/images/create-ssh-key.png" alt="Create SSH Key" class="doc-image">

## 2. Add Your Repository to Saturn Cloud

1. Go to **Git Repositories** in the sidebar
2. Click **New** button
3. Enter your repository URL (the same URL you'd use for `git clone`)
4. Click **Submit**

<img src="/images/sidebar-git-repositories.png" alt="Git Repositories in sidebar" class="doc-image">

<img src="/images/add-git-repositories.png" alt="Add Git Repository" class="doc-image">

## 3. Attach to Your Resource

When creating a Jupyter Server or R Server:
1. In the creation form, find the **Git Repositories** section
2. Select your repository from the dropdown
3. Choose the branch/commit you want to use
4. Click **Create**

<img src="/images/git-repository-resource-creation.png" alt="Git Repository Resource Creation" class="doc-image">

Your code will be automatically cloned when the resource starts!

## Need Help?

- **[Full Git documentation](/docs)** - Complete guide with all options
- **[GitHub SSH setup](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)** - GitHub-specific instructions
- **[GitLab SSH setup](https://docs.gitlab.com/ee/user/ssh.html)** - GitLab-specific instructions
