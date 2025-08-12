# Installation (Nebius)
{{% alert %}}
Please contact support@saturncloud.io in order to get Saturn Cloud installed in your Nebius account.
{{% /alert %}}

Saturn Cloud Enterprise is most often fully managed. The installation steps are as follows:
1. Create a Nebius account
2. [Create a service account within Nebius in the `editors` group](https://docs.nebius.com/iam/service-accounts/manage)
3. Saturn Cloud support will generate a private/public key pair. We will send you our public key
4. [Create an authorized key using our public key](https://docs.nebius.com/iam/service-accounts/authorized-keys)
5. Send us your Nebius `project-id`, `serviceaccount-id`, and `publickey-id`.

We will use the service account to create a [Nebius managed Kubernetes cluster](https://nebius.com/services/managed-kubernetes) with a number of tailored node groups, and install the Saturn Cloud application into the cluster.

Saturn Cloud Enterprise is fully managed so you won't have to worry about updates or patches, they will be rolled out automatically.

{{% enterprise_docs_view %}}


