# Enterprise Installation

The Saturn Cloud Enterprise plan lets you run Saturn Cloud within an organization's AWS account, allowing you more control over access to the tool and how it runs. This page describes how to install Saturn Cloud into your AWS account, providing two methods: automatically installing via the AWS Marketplace (recommended), or manually installing via Docker. While we recommend using the AWS Marketplace, there may be organizational IT policies or special customizations that require using the manual installation process.

<ul class="nav nav-tabs" id="myTab" role="tablist">
  <li class="nav-item mb-0">
    <a class="nav-link active" id="automatic-tab" data-toggle="tab" href="#automatic" role="tab" aria-controls="automatic" aria-selected="true">Automatic installation (recommended)</a>
  </li>
  <li class="nav-item mb-0">
    <a class="nav-link" id="manual-tab" data-toggle="tab" href="#manual" role="tab" aria-controls="manual" aria-selected="false">Manual installation</a>
  </li>
</ul>
<div class="tab-content" id="myTabContent">
<div class="tab-pane fade show active p-2" id="automatic" role="tabpanel" aria-labelledby="automatic-tab">

## Automatic Installation via AWS Marketplace

These steps install Saturn Cloud through the AWS Marketplace, automating most of the process.

### 1. Sign up for Saturn Cloud

To begin the signup process, visit <a href="https://manager.aws.saturnenterprise.io/register" target='_blank' rel='noopener'>the Saturn Cloud Installation Manager</a>.

> Once installed, Your Saturn Cloud installation will be hosted at "app.{orgname}.saturnenterprise.io", so the company name will be converted into a valid DNS name.

The Manager will direct you to the AWS Marketplace, where you can subscribe to Saturn Cloud.

<img src="/images/docs/aws-marketplace.png" alt="Screenshot of signup in AWS Marketplace for Saturn Cloud" class="doc-image">

### 2. Grant Saturn Cloud access to an IAM user or role
<span id="create-role">
Our installer needs an IAM Role in order to provision resources into your AWS account, and provide customer support and product updates. The installer can create this role for you with credentials--or, if you'd like, you can [create the role yourself](<docs/Enterprise/advanced_enterprise.md>).

Sign in to the AWS Management Console. In the navigation bar on the upper right, choose your account name or number and then choose **My Security Credentials**. In general, it's better to use IAM users, but If you are signed in as the AWS root user,  expand the Access keys (access key ID and secret access key) section and click **Create New Access Key**.

<img src="/images/docs/root-access-key.png" alt="Screenshot of AWS management console Access Keys page" class="doc-image">

If you are signed in as an IAM user, select **AWS IAM credentials** and click **Create Access Key**.

<img src="/images/docs/iam-access-key.png" alt="Screenshot of AWS management console IAM credentials tab" class="doc-image">

When prompted, download the CSV file. This is your only opportunity to save your secret access key. After the installation is complete, these credentials are no longer needed and can be safely deleted.

Go back to the <a href="https://manager.aws.saturnenterprise.io/role/creator/saturn" target='_blank' rel='noopener'>Saturn Cloud installer</a>.

<img src="/images/docs/give-us-keys.png" alt="Screenshot of Saturn Cloud installer page asking for IAM credentials" class="doc-image">

Paste your **Access Key ID** and **Secret Access Key** into the form and click **Next**. We will use the keys to create a role.  For security reasons, we will **not** save the keys.

### 3. Deploy Saturn Cloud

At this stage, click "Deploy" and Saturn Cloud will be installed into your AWS account. Installation typically takes 10-25 minutes. When it is complete, you will receive an email instructing you to reset your password for the `admin` account on your new Saturn deployment and how to log in.

<h5 class="text-primary"><b>At this point the installation is complete.</b></h5>

</div>
<div class="tab-pane fade p-2" id="manual" role="tabpanel" aria-labelledby="manual-tab">

## Manual Installation via a Docker Container

In order to follow these instructions, you will need the following:

1. A bash-compatible shell to run the commands.
2. Console access to your AWS account.
3. Permission in your AWS account to create IAM roles and/or permission to create CloudFormation stacks.
4. A place to run [Docker](https://www.docker.com/) containers from (either an SSH session to a machine with Docker access, or docker running locally) that has a persistent disk. This location must have internet access, as well as either AWS credentials or an IAM role assigned (if running from an EC2 instance).

### 1. Sign up for Saturn Cloud

To begin the signup process, visit <a href="https://manager.aws.saturnenterprise.io/register" target='_blank' rel='noopener'>the Saturn Cloud Installation Manager</a>. The Manager will direct you to the AWS Marketplace, where you will need to subscribe to Saturn Cloud.

<img src="/images/docs/aws-marketplace.png" alt="Screenshot of signup in AWS Marketplace for Saturn Cloud" class="doc-image">

### 2. Create your installation configuration

Contact support@saturncloud.io. We will help you generate your installation configuration. It will look something like this:

```
org_name: ...
region: ...
aws_account_id: ...
private_subnets:
- ...
public_subnets:
- ...
worker_subnets:
- ...
```

Store this in an empty directory, with the filename `config.yaml`

### 3. Create the installation role

You can create the role for the Saturn installation via a CloudFormation stack. If you are installing Saturn yourself - this probably means you will want to modify the cloud formation template to adjust the trust relationship. Please contact us if you need assistance doing so.

### 4. Set up the environment

Set up a few environment variables in your bash shell.


```sh
export INSTALLER_TAG=...
export DATA_DIR=....
export AWS_AUTH_VARS=...
export AWS_DEFAULT_REGION=...
```

We will provide you with the `INSTALLER_TAG`, which will point to the latest version of our Installer. `DATA_DIR` should point to a directory on disk where you've written the `config.yaml` from Step 2. `AWS_AUTH_VARS` is a set of docker cli args that will authenticate the docker container with IAM credentials. `AWS_DEFAULT_REGION` should be the same as the region you want to see your resources on AWS UI (EX: us-east-2).

**AWS_AUTH_VARS**

- If you've signed in via `aws configure` and have an `~/.aws/config` file that contains credentials:

    ```sh
    export AWS_AUTH_VARS="-v ${HOME}/.aws:/root/.aws"
    ```

- If you have `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_DEFAULT_REGION` set:
    ```sh
    export AWS_AUTH_VARS="-e AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY -e AWS_DEFAULT_REGION"
    ```
-  If running from an EC2 instance with an assigned IAM role, then `AWS_AUTH_VARS` isn't necessary. Set it to an empty string.
    ```sh
    export AWS_AUTH_VARS=""
    ```

### 5. Run the installer

```sh
docker run -e AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION --rm -it -v ${DATA_DIR}:/sdata ${AWS_AUTH_VARS} saturncloud/saturn-aws:${INSTALLER_TAG} python saturn_aws/scripts/main.py install
```

This will take some time - typically 15-45 minutes. If you encounter errors, [contact us](/docs/reporting-problems/) and we will help debug. When installation completes successfully, you will receive an email instructing you to reset your password for the `admin` account on your new Saturn deployment.

### 6. Backup the configuration files to S3

```sh
docker run -e AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION --rm -it -v ${DATA_DIR}:/sdata ${AWS_AUTH_VARS} saturncloud/saturn-aws:${INSTALLER_TAG} python saturn_aws/scripts/main.py backup
```

You'll receive an email shortly with instructions for how to log in to Saturn.

<h5 class="text-primary"><b>At this point the installation is complete.</b></h5>

</div>
</div>
