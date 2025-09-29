# IT Teams and Security

Below are some common reasons why IT Security teams choose Saturn Cloud. We also provide an overview of [our most popular security features](/docs) and examples of our [high-security deployments](/docs).

## Isolation within your AWS account

Security is the primary motivation for teams to move from Saturn Hosted (which runs in our AWS account) to Saturn Enterprise. Saturn Enterprise is installed within your AWS account, and you can customize the VPC where Saturn Cloud is deployed. This allows the use of VPC peering and Transit Gateways to connect Saturn Cloud to other parts of your infrastructure.

## Network security

For security-conscious customers, we recommend deploying Saturn Cloud within private subnets of your VPC so it is accessible only through your VPN. This ensures that external parties cannot access your Saturn Cloud installation.

In addition to limiting ingress, you can also limit egress for your data science team using transparent proxies or by disabling internet access entirely. In that case, you will need on-premise mirrors of `conda`, `pypi`, and `CRAN` repositories.

For additional Enterprise security topics: [No‑egress deployments](/docs/enterprise/installation/no-internet/), [Hardening guidance](/docs/enterprise/installation/high-security/).

## IAM security

Saturn Cloud uses IAM roles to manage your instances. We can work with you to scope access permissions to the bare minimum. You can also modify the trust relationship so we only have access during authorized updates and support requests. For customers that require it, you can cut off Saturn Cloud’s access entirely and handle installation and upgrades via Zoom, Google Meet, or Microsoft Teams.

See also:

- [User/role mapping and permissions](/docs/enterprise/installation/saturn-users-and-iam-roles/)
- [Access models and controls](/docs/enterprise/access/)

Saturn Cloud also supports IAM roles on a per-resource basis. Data access can be managed by IAM roles, and users can attach those roles to individual resources. Authorization for specific IAM roles is granted on a per-user or per-group basis and can only be done by Saturn Cloud admins you designate, providing fine-grained security.

## SSO

Saturn Cloud uses Auth0 to connect to your existing identity provider (IDP). Regardless of which IDP you use, Saturn Cloud can integrate with it. If you'd like to set up SSO, please reach out to support@saturncloud.io to discuss your configuration.

See also:

- [Identity setup overview](/docs/enterprise/installation/identity/)
- [Azure Entra ID configuration](/docs/enterprise/installation/identity/azure/)
- [Google Workspace configuration](/docs/enterprise/installation/identity/google/)
- [Okta configuration](/docs/enterprise/installation/identity/okta/)

SSO can be configured to authorize all users in your organization or used solely for authentication, with explicit user invitations managing access within the Saturn Cloud application.

{{% security_docs_view %}}
