# IT Teams and Security

Below highlights a few common reasons why IT Security teams like Saturn Cloud. We also have an overview of [our most popular security features](/docs), as well as examples of our [most secure deployments](/docs)

## Isolation within your AWS account

Security is the primary motivation for teams to move from Saturn Hosted (which runs in our AWS account) to Saturn Enterprise. Saturn enterprise is installed within your AWS account. You can also customize the VPC that Saturn Cloud is deployed in. This enables you to use VPC peering and Transit gateways to connect Saturn Cloud to other parts of your infrastructure.

## Network security

For security concious customers, we always recommend that Saturn Cloud be deployed within private subnets within in your VPC, so that Saturn Cloud is only accessible within your VPN. This ensures that external parties can never access your Saturn Cloud installation.

In addition to limiting egress, you can also limit egress for your data science team using transparent proxies, or by disabling egress to the internet (you will want to have on-premise mirrors of conda, pypi, and CRAN repositories in that case.

## IAM security

Saturn Cloud uses IAM roles to manage your instance. We can work with you to scope those access permissions to the bare minimum. You can also modify the trust relationship so we only have access during authorized updates and support requests. For customers that require it - you can completely cut off access to your Saturn Cloud access, and we can support installation and upgrades via Zoom, Google meet, or Microsoft teams.

Saturn Cloud also enables IAM roles on a per-resource basis. That means that data access can be completely managed by IAM roles, and users can attach those roles on a per-resource basis. Authorization for specific IAM roles is granted on a per-user or per-group basis, and can only be done by those that you desginate as Saturn Cloud admins, providing very fine grained security.

## SSO

Saturn Cloud leverages Auth0 to connect to your existing IDP. Regardless of what IDP you use, Saturn Cloud can integrate with it. If you'd like to setup SSO, Please reach out support@saturncloud.io and we can discuss your specific configuration.

SSO can be setup either to authorize all users in your organization, Or you can elect to use SSO solely for authentication, and rely on explicit user invitations within the Saturn Cloud application to manage access to Saturn Cloud.
{{% security_docs_view %}}
