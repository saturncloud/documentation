# Uninstalling Saturn (Enterprise Only)
## Uninstalling Saturn

If you decide you no longer want to use Saturn, please send us an email at support@saturncloud.io, and we'll tear down the deployment for you.  **Please do not delete the CloudFormation stack which provisioned the customer-admin IAM role, the IAM role itself.**  We need this role in order to tear down resources for you.

{{% alert title="Caution" %}}
The Saturn install process provisions many items in your AWS account.  These include:

- VPC
- NAT Gateway
- ELB
- Autoscaling Groups
- EKS Cluster

Do not delete the `customer-admin` role we use to administer your installation.  Shoot us an email, and we'll take care of it and make sure there is nothing left behind. {{% /alert %}}

### What if I already deleted the role?

You can either [re-create the role](<docs/Enterprise/installation.md>), or give us the [AWS IAM access keys that were used to create the role](/docs).
