# Installation

The Saturn Cloud Enterprise plan lets you run Saturn Cloud within an organization's AWS account, allowing you more control over access to the tool and how it runs. This page describes how to install Saturn Cloud into your AWS account, providing three methods:

1. Automatically installing via the AWS Marketplace (recommended)
2. Manually installing via Docker.
3. Manually installing via Docker and using IAM roles instead of IAM users

While we recommend using the AWS Marketplace, there may be organizational IT policies or special customizations that require using the manual installation process.

<ul class="nav nav-tabs nav-justified my-3 flex-column flex-sm-row" id="myTab" role="tablist">
  <li class="nav-item mb-0">
    <a class="nav-link active" id="automatic-tab" data-toggle="tab" href="#automatic" role="tab" aria-controls="automatic" aria-selected="true">Automatic (recommended)</a>
  </li>
  <li class="nav-item mb-0">
    <a class="nav-link" id="manual-tab" data-toggle="tab" href="#manual" role="tab" aria-controls="manual" aria-selected="false">Manual</a>
  </li>
  <li class="nav-item mb-0">
    <a class="nav-link" id="manual-no-iam-tab" data-toggle="tab" href="#manual-no-iam" role="tab" aria-controls="manual-no-iam" aria-selected="false">Manual (no IAM users)</a>
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

<img src="/images/docs/aws-marketplace.png" alt="Screenshot of signup in AWS Marketplace for Saturn Cloud" class="doc-image"/>

### 2. Grant Saturn Cloud access to an IAM user or role
<span id="create-role"></span>
Our installer needs an IAM Role in order to provision resources into your AWS account, and provide customer support and product updates. The installer can create this role for you with credentials, or, if you'd like, you can [create the role yourself.](<docs/Enterprise/installation/advanced_enterprise.md>)

Sign in to the AWS Management Console. In the navigation bar on the upper right, choose your account name or number and then choose **My Security Credentials**. In general, it's better to use IAM users, but If you are signed in as the AWS root user,  expand the Access keys (access key ID and secret access key) section and click **Create New Access Key**.

<img src="/images/docs/root-access-key.png" alt="Screenshot of AWS management console Access Keys page" class="doc-image"/>

If you are signed in as an IAM user, select **AWS IAM credentials** and click **Create Access Key**.

<img src="/images/docs/iam-access-key.png" alt="Screenshot of AWS management console IAM credentials tab" class="doc-image"/>

When prompted, download the CSV file. This is your only opportunity to save your secret access key. After the installation is complete, these credentials are no longer needed and can be safely deleted.

Go back to the <a href="https://manager.aws.saturnenterprise.io/role/creator/saturn" target='_blank' rel='noopener'>Saturn Cloud installer</a>.

<img src="/images/docs/give-us-keys.png" alt="Screenshot of Saturn Cloud installer page asking for IAM credentials" class="doc-image"/>

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

<img src="/images/docs/aws-marketplace.png" alt="Screenshot of signup in AWS Marketplace for Saturn Cloud" class="doc-image"/>

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
<div class="tab-pane fade p-2" id="manual-no-iam" role="tabpanel" aria-labelledby="manual-no-iam-tab">

## Manual Installation via a Docker Container (without IAM roles)

The Saturn Cloud Enterprise plan lets you run Saturn Cloud within an organization's AWS account, allowing you more control over access to the tool and how it runs. Our standard installation process creates an IAM user which the Saturn UI uses to interact with AWS. The approach documented here creates an OIDC provider which can associate IAM roles with EKS pods and replaces all IAM users with IAM roles.

### 1. Sign up for Saturn Cloud

To begin the signup process, visit <a href="https://manager.aws.saturnenterprise.io/register" target='_blank' rel='noopener'>the Saturn Cloud Installation Manager</a>. The Manager will direct you to the AWS Marketplace, where you will need to subscribe to Saturn Cloud.

<img src="/images/docs/aws-marketplace.png" alt="Screenshot of signup in AWS Marketplace for Saturn Cloud" class="doc-image"/>

<span id="create-role"></span>

### 2. Create the IAM roles for installing and running Saturn Cloud

In this step you'll create a role to run the Saturn Cloud installation from, and the roles
you'll want the application to use when it's running.

#### 2a. Creating the role for installing Saturn Cloud

You can create the role for the Saturn installation [via a CloudFormation stack](<docs/Enterprise/installation/advanced_enterprise.md>#create-role). Our standard cloud formation template grants access to our AWS account to perform the installation. If you are doing the install yourself, you will want to use this template instead: `https://s3.us-east-2.amazonaws.com/saturn-cf-templates/iam-role-self-managed.cft`

#### 2b. Creating the roles for running Saturn Cloud

Saturn Cloud requires IAM resources to operate. This includes:

- an IAM role and IAM policies for the EKS cluster
- an IAM role, an instance profile and IAM policies for the EKS worker nodes
- an IAM role for the saturn application

Most customers that are on this path have very specific requirements for provisioning IAM resources. We've provided [a sample terraform](/static/enterprise-installation-no-iam-terraform.tf) to use as a starting
point to customize. _You can also create equivalent IAM resources in the console, or however else you manage your AWS resources._

The sample terraform creates an IAM role for the Saturn Cloud application. The assume-role policy for that role has some placeholder values, which we will alter once the OIDC provider has been created. If you do use this terraform, you must replace `{orgname}` in the following with your `orgname`. 

{{% alert title="Where to run the Terraform command" %}}

There is no requirement for where to run the Terraform script from. If you need a machine to use, you can wait to create the IAM roles to run Saturn Cloud until after you've completed step 3 and made an EC2 instance to use.
{{% /alert %}}

### 3. Create your installation configuration

The Saturn Cloud installation need a configuration specific to your organization. The easiest way
to create it is to contact [support@saturncloud.io](mailto:support@saturncloud.io). We will help you generate your installation configuration. It will look something like this:

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
enable_irsa: true
```

Save this file as `config.yaml`--we'll use it in the next step.

### 4. Spin up an EC2 instance to run the installation

These steps will set up an EC2 instance to run the Saturn Cloud installation.

#### 4a. Install Docker
We recommend performing the installation from an EC2 instance. Spin up the EC2 instance with the Ubuntu 20.04 AMI. Attach the IAM role you created in the previous step to this instance. Once the instance is created, you'll need to install Docker using the following commands:

```shell
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io
```

#### 4b. Copy the installation configuration

Next, copy the `config.yaml` file from step (3) onto the machine. You do not need to save it
in any particular location, but keep track of the path you saved it to.

#### 4c. Set the environment variables

From there, you'll need to set environment variables for the installation:

```shell
export INSTALLER_TAG=...
export DATA_DIR=....
export AWS_DEFAULT_REGION=...
```

We will provide you with the `INSTALLER_TAG`, which will point to the latest version of our Installer. `DATA_DIR` should point to a directory on disk where you've written the `config.yaml` from Step (4b). `AWS_DEFAULT_REGION` should be the same as the region you want to see your resources on AWS UI (EX: us-east-2).

### 5. Run the installer to setup AWS resources

At this point, you can run the installation Docker container on the EC2 instance set up in step (4):

```shell
docker run -e AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION --rm -it -v ${DATA_DIR}:/sdata saturncloud/saturn-aws:${INSTALLER_TAG} python saturn_aws/scripts/main.py install --skip-k8s
```

This will take some time - typically 15-45 minutes. If you encounter errors, [contact us](/docs/reporting-problems/) and we will help debug. The last step - associating the OIDC provider can take up to 30 minutes.

### 6. Modify the assume-role policy for the saturn UI IAM role

The sample terraform script created an IAM role with placeholder values following assume role policy:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/OIDC_PROVIDER"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "OIDC_PROVIDER:sub": "system:serviceaccount:saturn:saturnadmin"
        }
      }
    }
  ]
}
```

You should update the role so to replace `ACCOUNT_ID` with your AWS ACCOUNT ID, and replace `OIDC_PROVIDER with the output of:

```
aws eks describe-cluster --name saturn-cluster-{orgname} --query "cluster.identity.oidc.issuer" --output text | sed -e "s/^https:\/\///"
```

### 7. Run the installer to setup k8s resources

Finally, we need to set up the Kubernetes cluster for Saturn Cloud. Run the following command from
the installation EC2 instance

```shell
docker run -e AWS_DEFAULT_REGION=$AWS_DEFAULT_REGION --rm -it -v ${DATA_DIR}:/sdata saturncloud/saturn-aws:${INSTALLER_TAG} python saturn_aws/scripts/main.py install-k8s
```

You may stop your installation EC2 instance, but do not terminate it. You can use this instance in the future for updates. You'll receive an email shortly with instructions for how to log in to Saturn. Please backup a copy of the `config.yaml`.

<h5 class="text-primary"><b>At this point the installation is complete.</b></h5>


</div>
</div>
