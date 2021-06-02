# Advanced Enterprise Deployment Options

Enterprise users may need some customizations when you install Saturn Cloud, and we're happy to help you get set up. See below for a few of the most common requests we get - if you need something we haven't covered, 
[please let us know and we'll help](/docs)!


## Authentication and Single Sign On

We can do that! Saturn Cloud Enterprise can be configured to use LDAP or other identity providers. [Reach out to us about your requirements](/docs) and we can determine the best solution for you.

## Custom VPC and Networking Requirements

Our default installation provisions a VPC with a public and private subnet in 2 availability zones.  Saturn itself is served through an external load balancer.

Saturn can be configured to run only on internal load balancers, internal subnets with custom CIDR blocks, and custom routing tables.  If you need any of these, [reach out to us with your requirements](/docs).

## Restricted Access to your AWS Account

Our default installation for Saturn Cloud Enterprise requires companies to give us access to an [IAM role in your AWS account](/docs).  However, for some customers this isn't a viable option for enterprise security reasons. We understand this, and we have two solutions that can allow Saturn Cloud Enterprise installation without delegating us continuous access to the IAM role.

- [Create the IAM role just for installation](/docs), then remove the trust policy once installation is complete. For periodic support and maintenance, we can coordinate with you to arrange a time window for the trust policy to be temporarily added again.
- Create the IAM role without a trust policy delegating us access.  In this case, we will walk you through doing installation, support and maintenance yourself.