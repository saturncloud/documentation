# Advanced Enterprise Deployment Options

Enterprise users may need some customizations when you install Saturn Cloud, and we're happy to help you get set up. See below for a few of the most common requests we get - if you need something we haven't covered,
[please let us know and we'll help](<docs/getting_help.md>).

## Authentication and Single Sign On

Saturn Cloud Enterprise can be configured to use LDAP, Google Workspace credentials, or other identity providers. [Reach out to us about your requirements](<docs/getting_help.md>) and we can determine the best solution for you.

## Custom VPC and Networking Requirements

Our default installation provisions a VPC with a public and private subnet in 2 availability zones.  Saturn itself is served through an external load balancer.

Saturn can be configured to run only on internal load balancers, internal subnets with custom CIDR blocks, and custom routing tables.  If you need any of these, [reach out to us with your requirements](<docs/getting_help.md>).

## Manually creating the Saturn Cloud Enterprise installation IAM role

The role created by our cloud formation stack will create a trust relationship to our account. If you are running the installer yourself or if you have security concerns then you can modify the trust relationship to point to your account.

In AWS Console, navigate to <a href="https://console.aws.amazon.com/cloudformation" target='_blank' rel='noopener'>the CloudFormation section</a>. Click the "Create Stack" button. From the dropdown, choose the standard option ("With new resources").

<img src="/images/docs/cf-stack.png" alt="Screenshot of AWS Console showing CloudFormation panel, with Create Stack button centered" class="doc-image">

On the next screen, choose "Template is ready" option in the "Prepare template" section. For the template, use the URL: `https://s3.us-east-2.amazonaws.com/saturn-cf-templates/iam-role.cft`

Click Next.

<img src="/images/docs/cf-stack2.png" alt="Screenshot of AWS Console showing Create Stack form" class="doc-image">

On the next screen, give the stack a name of your choice (for example, "Saturn Cloud Access"). The external ID can be found on the installer page. Click "Next"

<img src="/images/docs/cf-stack3.png" alt="Screenshot of AWS Console showing Create Stack form, with Stack Name and Parameters shown" class="doc-image">

On the next screen, all fields are optional. Proceed to the next page when you have configured these items as desired.

<img src="/images/docs/cf-stack4.png" alt="Screenshot of AWS Console showing Configure Stack Options" class="doc-image">

Review all values on the next page - step back and making corrections if needed. When you're ready to create the stack, check the checkbox next to "I acknowledge that AWS CloudFormation might create IAM resources with custom names." Then, click on the "Create stack" button.

<img src="/images/docs/cf-stack5.png" alt="Screenshot of AWS Console showing warning displayed before Create Stack can be selected" class="doc-image">

Once stack creation is complete, you need to provide the ARN for the created role to the installer in order to continue with the deployment. You may find this info on the "Outputs" tab in the AWS console. The ARN is the string that starts with "arn:aws:iam" in the "Value" column, as shown below.

## Installing to environments without internet access

Saturn can be installed in an AWS account with no access to the internet. In order to do this - you'll need a VPC that meets a few requirements, or a Docker image mirror.

- You need to ensure that your VPC allows egress to `https://manager.aws.saturnenterprise.io/` (`3.134.99.59`)
- A security group with an inbound rule that allows HTTPS traffic from the VPC’s CIDR
- [Private VPC endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html) for the following AWS services, with the security group (from the previous step) attached:
  - `ecr.dkr` (Interface)
  - `ecr.api` (Interface)
  - `ec2` (Interface)
  - `autoscaling` (Interface)
  - `sts` (Interface)
  - `s3` (Gateway)
- Either egress to `docker.io`, `k8s.gcr.io`, and `quay.io`, or an image mirror set up with access to those hosts. Currently, the only image mirror Saturn supports is [Artifactory](https://jfrog.com/artifactory/). If you require support for a different image mirror solution, [please get in touch](https://deploy-preview-345--saturn-cloud.netlify.app/docs/reporting-problems/)

With these fulfilled, follow the [manual Saturn Cloud installation steps](/docs).
