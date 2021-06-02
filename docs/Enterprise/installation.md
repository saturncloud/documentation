# Installing (Enterprise Only)

## Overview

Welcome! If you are an Enterprise user who would like to install Saturn Cloud in your existing AWS account, this page is for you.

This page describes how to perform a managed installation through Saturn's tooling. If you prefer (or have requirements that mean that Saturn cannot assume an IAM role inside your AWS account), you can instead [manually perform the installation yourself](/docs/using-saturn-cloud/enterprise/self_installation/).

To install Saturn Cloud Enterprise, there are a few important steps to complete. Full instructions for each of these steps follow below.

1. Sign up in the Saturn Manager tool, which is where you'll do configurations of Saturn Enterprise. This step only needs to be completed once, and is only necessary for Enterprise accounts. This process will direct you to visit the AWS website to complete the steps to connect Saturn Cloud to your account.

2. Set up the AWS Identity and Access Management procedures to allow Saturn Cloud to install into your AWS account.

3. Deploy Saturn Cloud.

4. Log in to your new Saturn Cloud account and add users and set permissions.

## Step 1. Sign up for Saturn Cloud

To begin the signup process, visit <a href="https://manager.aws.saturnenterprise.io/register" target='_blank' rel='noopener'>the Saturn Manager</a>. 

> Once installed, Your Saturn Cloud installation will be hosted at "app.{orgname}.saturnenterprise.io", so the company name will be converted into a valid DNS name.

The Manager will direct you to the AWS Marketplace, where you can subscribe to Saturn Cloud.

<img src="/images/docs/aws-marketplace.png" alt="Screenshot of signup in AWS Marketplace for Saturn Cloud" class="doc-image">

## Step 2. Manage IAM Roles in AWS
If you're familiar with AWS Shared Responsibility Model, you may be familiar with IAM (Identity and Access Management) Roles. If not, [visit the AWS site to learn more before proceeding](https://aws.amazon.com/iam/). The next steps here assume you understand the essential concepts of these roles.

After you subscribe to Saturn Cloud Enterprise in the AWS marketplace (see the prior section), you will be redirected to the Saturn Cloud installer. The installer needs an IAM Role in order to provision resources into your AWS account, and provide customer support and product updates. The installer can create this role for you with credentials - or, if you'd like, you can create the CloudFormation stack yourself. 

>If you already have the necessary IAM role and access keys, [you can skip this section entirely](/docs).

### Option A: Saturn Cloud Creates IAM Role

Sign in to the AWS Management Console. In the navigation bar on the upper right, choose your account name or number and then choose **My Security Credentials**. In general, it's better to use IAM users, but If you are signed in as the AWS root user,  expand the Access keys (access key ID and secret access key) section and click **Create New Access Key**.

<img src="/images/docs/root-access-key.png" alt="Screenshot of AWS management console Access Keys page" class="doc-image">

If you are signed in as an IAM user, select **AWS IAM credentials** and click **Create Access Key**.

<img src="/images/docs/iam-access-key.png" alt="Screenshot of AWS management console IAM credentials tab" class="doc-image">


When prompted, download the csv file. This is your only opportunity to save your secret access key. After the installation is complete we won't need these credentials anymore, and can delete them.

Go back to the <a href="https://manager.aws.saturnenterprise.io/role/creator/saturn" target='_blank' rel='noopener'>Saturn Cloud installer</a>.

<img src="/images/docs/give-us-keys.png" alt="Screenshot of Saturn Cloud installer page asking for IAM credentials" class="doc-image">

Paste your **Access Key ID** and **Secret Access Key** into the form and click **Next**. We will use the keys to create a role.  For security reasons, we will **not** save the keys.

***

### Option B: Create the IAM Role Yourself

> This approach requires more work on your part, but may be preferable if you wish to customize or more precisely set preferences. If you're not sure what to do, please contact our support team and we'll help you!

In AWS Console, navigate to <a href="https://console.aws.amazon.com/cloudformation" target='_blank' rel='noopener'>the CloudFormation section</a>. Click the "Create Stack" button. From the dropdown, choose the standard option ("With new resources").
 
<img src="/images/docs/cf-stack.png" alt="Screenshot of AWS Console showing CloudFormation panel, with Create Stack button centered" class="doc-image">

On the next screen, choose "Template is ready" option in the "Prepare template" section. For the template, use the following URL:

`https://s3.us-east-2.amazonaws.com/saturn-cf-templates/iam-role.cft`

Click Next.

<img src="/images/docs/cf-stack2.png" alt="Screenshot of AWS Console showing Create Stack form" class="doc-image">

On the next screen, give the stack a name of your choice (for example, "Saturn Cloud Access"). The external ID can be found on the installer page. Click "Next"
 
<img src="/images/docs/cf-stack3.png" alt="Screenshot of AWS Console showing Create Stack form, with Stack Name and Parameters shown" class="doc-image">

On the next screen, all fields are optional. Proceed to the next page when you have configured these items as desired.

<img src="/images/docs/cf-stack4.png" alt="Screenshot of AWS Console showing Configure Stack Options" class="doc-image">

Review all values on the next page - step back and making corrections if needed. When you're ready to create the stack, check the checkbox next to "I acknowledge that AWS CloudFormation might create IAM resources with custom names." Then, click on the "Create stack" button.
 
<img src="/images/docs/cf-stack5.png" alt="Screenshot of AWS Console showing warning displayed before Create Stack can be selected" class="doc-image">

Once stack creation is complete, you need to provide the ARN for the created role to the installer in order to continue with the deployment. You may find this info on the "Outputs" tab in the AWS console. The ARN is the string that starts with "arn:aws:iam" in the "Value" column, as shown below.

 
<img src="/images/docs/cf-stack6.jpg" alt="Screenshot of AWS Console showing list of Stacks, with SaturnCustomer/AdminRole highlighted in list" class="doc-image">
 
#### Configure and Install Saturn

After you create the role and record the ARN, you're ready to configure your deployment. Choose an "Organization Name", your "Work Email", your desired AWS region, and the ARN for the IAM role from the previous step to continue.

<img src="/images/docs/configure-saturn-install.png" alt="Screenshot of Saturn Cloud Enterprise account details page" class="doc-image">

***

## Step 3. Deploy Saturn

At this stage, click "Deploy" and Saturn Cloud will be installed into your AWS account. Installation typically takes 10-25 minutes. When it is complete, you will receive an email instructing you to reset your password for the `admin` account on your new Saturn deployment.

## Step 4. Log in to your Saturn Cloud installation

After you receive the email, follow the instructions provided to set the `admin` user's password. If that link has expired, you can generate a new one at the URL below. Make sure that you replace {org-name} with the name of the Organization you setup when you subscribed to Saturn.

```
https://app.{org-name}.saturnenterprise.io/auth/resetemail
```

Again: Make sure that you replace {org-name}  with the name of the Organization
you setup when you subscribed to Saturn.

<img src="/images/docs/saturn-password-reset.png" alt="Screenshot of Saturn Cloud Enterprise password reset page" class="doc-image">

Type in `admin` as the username. You should get a new password reset link via email in a few minutes.
