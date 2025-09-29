# Secrets

{{% alert title="Credentials are now secrets" %}}
Whereas the previous version of Saturn Cloud featured a _credentials_ component, the latest 2022.04.01 version now has a new _secrets_ component.

Secrets have a similar function as credentials, but they are more secure because you explicitly attach them to each Saturn Cloud resource.

All your previous credentials have been migrated to be secrets.
{{% /alert %}}

Secrets are pieces of information that you want to securely store (e.g., API keys, passwords, configuration files). Saturn Cloud manages secrets for you with a secrets manager. You can access the content of a secret only if you have explicitly attached it to a resource and if that resource is running.

To use secrets in Saturn Cloud, you will add them to your Saturn Cloud secrets manager and then attach them to your resources. You can connect multiple secrets to a resource, and multiple resources can attach to the same secret.

![bipartite graph showing the relationship between secrets and resources](/images/docs/bipartite-graph.webp "doc-image-medium")

## Add a secret to Saturn Cloud

To add a secret, open the **Secrets** page in Saturn Cloud.

<img src="/images/docs/secrets_sidebar.webp" alt="Screenshot of side menu of Saturn Cloud product with Secrets selected" style="width:150px;" class="doc-image">

This is where you will store your secret information. _This is a secure storage location, and will not be available to the public or other users._

At the top right corner of this page, you will find the "New" button. Click here, and you'll be taken to the Add Secret form. This will give you some options, starting with a name for the secret.

<img src="/images/docs/add_secrets_page.webp" alt="Screenshot of Saturn Cloud Create Credentials form" class="doc-image">

Complete the form one time for each secret you wish to add. When completed, you should see the secrets in the Secrets page, like this.

<img src="/images/docs/added_secret.webp" alt="Screenshot of Secrets list in Saturn Cloud product" class="doc-image">

## Attach a secret to a resource

Once you have specified your secret in the Secrets page, you need to attach it to your resource to use it. Secrets may be attached to multiple resources.

First, navigate to your resource page and select **Secrets** from the top horizontal menu.

![Resource page opened up on the secrets tab](/images/docs/resource_secret_page.webp "doc-image")

You can either attach a secret as an environment variable or as a file. You can also modify the name that the environment variable will be on the resource, or the file path for the file with the secret.
You can also remove the secret on the page by pressing the appropriate button in the **Actions** column.

![Secret list with environment variable attached](/images/docs/attached_secret.webp "doc-image")

{{% alert %}}
Note: If the resource is already running when you attached a new secret, you will need to restart it before the secret is on it.
{{% /alert %}}

Once the secret has been attached (and the resource restarted if needed), the secrets stored as files will show up in your file browser. For secrets that are environment variables, see our
[environment variables documentation](<docs/user-guide/how-to/connect/environment-variables.md>) for how to access them.