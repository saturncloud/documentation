# Add Secrets

{{% alert title="Credentials are now secrets" %}}
Whereas the previous version of Saturn Cloud featured a _credentials_ component, the latest 2022.04.01 version now has a new _secrets_ component.

Secrets have a similar function as credentials, but they are more secure because you explicitly attach them to each Saturn Cloud resource.

All your previous credentials have been migrated to be secrets.
{{% /alert %}}

Secrets are pieces of information that you want to securely store (e.g., API keys, passwords, configuration files). Saturn Cloud manages secrets for you with a secrets manager. You can access the content of a secret only if you have explicitly attached it to a resource and if that resource is running.

To use secrets in Saturn Cloud, you will add them to your Saturn Cloud secrets manager and then attach them to your resources. You can connect multiple secrets to a resource, and multiple resources can attach to the same secret.

![bipartite graph showing the relationship between secrets and resources](/images/docs/bipartite-graph.png "doc-image-medium")

## Add a secret to Saturn Cloud

To add a secret, open the **Secrets** page in Saturn Cloud.

<img src="/images/docs/secrets_sidebar.png" alt="Screenshot of side menu of Saturn Cloud product with Secrets selected" style="width:150px;" class="doc-image">

This is where you will store your secret information. _This is a secure storage location, and will not be available to the public or other users._

At the top right corner of this page, you will find the "New" button. Click here, and you'll be taken to the Add Secret form. This will give you some options, starting with a name for the secret.

<img src="/images/docs/add_secrets_page.png" alt="Screenshot of Saturn Cloud Create Credentials form" class="doc-image">

Complete the form one time for each secret you wish to add. When completed, you should see the secrets in the Secrets page, like this.

<img src="/images/docs/added_secret.png" alt="Screenshot of Secrets list in Saturn Cloud product" class="doc-image">

## Attach a secret to a resource

Once you have specified your secret in the Secrets page, you need to attach it to your resource to use it. Secrets may be attached to multiple resources.

First, navigate to your resource page and select **Secrets** from the top horizontal menu.

![Resource page opened up on the secrets tab](/images/docs/resource_secret_page.png "doc-image")

You can either attach a secret as an environment variable or as a file.

To add a secret as an environment variable, select **Attach Secret Environment Variable**. In the form, select the secret you wish to attach, the environment variable name to set the value to, and a short description.

![Form to attach a secret environment variable](/images/docs/attach_secret_env_variable.png "doc-image-medium")

Once you click **Create**, your secret will show under the attached secrets list on the resource and be attached to your workspace as an environment variable.

{{% alert %}}
You will need to restart your resource for the secret to be accessible in your workspace.
{{% /alert %}}

![Secret list with environment variable attached](/images/docs/attached_secret.png "doc-image")

To remove the secret from the resource or to edit the environment variable name or description, click on the icons under Actions.

To attach a secret as a file, click on **Attach Secret File** and follow the above steps substituting your preferred file path for the environment variable name.
