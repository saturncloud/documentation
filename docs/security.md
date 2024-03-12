# Security

Security is a must have for any data science platform. This article describes how Saturn Cloud should be used securely. These approaches are things we recommend for the majority of our enterprise customers. We also have examples of what a [high security installation](/docs) looks like.

## Network Security

Network Security is the most important aspect of security for a data science platform. Data science platforms should not be exposed on the public internet. Deploying a data science platform on a private subnet, accessible only through a VPN is a great to guarantee that the platform cannot be attacked or accessed by outside parties. Network Security within your VPN is secondary, but can also be used to guarantee that unauthorized users do not gain access to sensitive data.

At Saturn Cloud, our default configuration is to deploy Saturn Cloud on the public internet since not all of our customers have VPNs. Those who have VPNs are encouraged to subscribe to the Advanced tier, which supports deployment within private subnets of existing VPCs.


## SSO and Authentication

Single sign on ensures that passwords cannot be lost of compromised. Single sign on also guarantees that when users leave your company, they also loose access to your infrastructure and data. We recommend that all access to tools at your company be authenticated through SSO.

At Saturn Cloud, SSO is available to advanced tier subscribers via [Auth0](https://auth0.com/). Auth0 can connect to virtually any identity provider. We have documentation for [Okta](/docs), [Azure AD](/docs), and [Google Sign On](/docs), but almost any identity provider can be supported, just ask!.


## IAM Roles

IAM roles are much better than IAM users because you don't have to worry about security tokens which can be lost or compromised. With IAM roles - you also want the capability of giving different permission sets to different users. We also recommend that you create fine-grained IAM roles so that people only get access to the data and resources that they actually need.

Many data science platforms use a single IAM role for the entire installation, attached to the EC2 instance. We recommend that you always use IAM roles to grant access to data and resources, not IAM users. At Saturn Cloud, we leverage [IAM roles for service accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html). This enables us to map IAM roles to [Saturn Cloud users, groups and containers](/docs).


## Credential Management

Not all data sources can be accessed using IAM roles. Some databases for example will require usernames and passwords for access. It is important to prevent passwords and API tokens from being compromised by accidentally being built into docker images, written down in notebooks, or checked into git.

Saturn Cloud can manage [secrets, database passwords, and api tokens](/docs) for your users. These secrets can be mapped to arbitrary files, or environment variables on your system. This means data scientists should can just load secrets using `os.environ` rather than writing them down in locations where they can be compromised.

{{% security_docs_view %}}
