# Self-Installing (Enterprise Only)

## Overview

This page describes how to perform a manual installation of Saturn Cloud into your own AWS account. This requires an advanced understanding of AWS and an understanding of how to run Docker containers. For a smoother, automated installation, [follow the instructions for a managed installation](/docs/using-saturn-cloud/enterprise/installation/).

If, while following these instructions, you find that your installation requires additional steps or configuration, please [contact us](/docs/reporting-problems/).

The instructions below assume a `bash`-compatible shell is in use.

## Preparation

In order to follow these instructions, you will need the following:

1. Console access to your AWS account.
2. Permission in your AWS account to create IAM roles and/or permission to create CloudFormation stacks.
3. A place to run [Docker](https://www.docker.com/) containers from (either an SSH session to a machine with docker access, or docker running locally) that has a persistent disk (there will be a configuration file generated that **must not be lost**). This location must have internet access, as well as either AWS credentials or an IAM role assigned (if running from an EC2 instance).

In addition, depending on your setup and requirements, step through the following questions:

1. Do you require installation into a pre-existing VPC, or do you want the Saturn installer to create a new VPC?
    * Creating a new VPC is the easiest option.
    * If using a pre-existing VPC, note its ID and CIDR.
2. Will your Saturn Cloud installation be accessible from the public internet?
    * If yes, and using a pre-existing VPC, you will need two private subnets in different AZs and two public subnets in the same two AZs. Note the subnet IDs.
    * If no, you must use a pre-existing VPC. You will need two private subnets in different AZs. Note their IDs.

    For either option above, if using a pre-existing VPC, make sure the AZs for your subnets support `p3` and `g4dn` instance types. Using the AWS CLI:
    
    ```sh
    # Set AWS_DEFAULT_REGION to your target region, if not set
    # export AWS_DEFAULT_REGION=us-east-1

    # Check p3 - Saturn requires p3.2xlarge, p3.8xlarge, and p3.16xlarge to be available
    aws ec2 describe-instance-type-offerings --location-type availability-zone --filters Name=instance-type,Values=p3*
    # Check g4dn - Saturn requires g4dn.xlarge, g4dn.4xlarge, and g4dn.8xlarge to be available
    aws ec2 describe-instance-type-offerings --location-type availability-zone --filters Name=instance-type,Values=g4dn*
    ```
3. Will your Saturn Cloud installation be able to call out to the public internet?
    * If yes, but your installation will not be accessible from the public internet, you must ensure that instances within your VPC can call out to the public internet (via an Internet Gateway + NAT Gateway, or another configuration of your choosing).
    * If no, you will need to ensure that your VPC allows egress to `https://manager.aws.saturnenterprise.io/` (`3.134.99.59`), and must set up:
        1. A security group with an inbound rule that allows HTTPS traffic from the VPC's CIDR.
        2. [Private VPC endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html) for the following AWS services, with the security group (from the previous step) attached:
            * `ecr.dkr` (Interface)
            * `ecr.api` (Interface)
            * `ec2` (Interface)
            * `autoscaling` (Interface)
            * `sts` (Interface)
            * `s3` (Gateway)
        3. Either egress to `docker.io`, `k8s.gcr.io`, and `quay.io`, or an image mirror set up with access to those hosts.
            * Currently, the only image mirror Saturn supports is [Artifactory](https://jfrog.com/artifactory/). If you require support for a different image mirror solution, [please get in touch](/docs/reporting-problems/)

## Installation

### 1. Organization and Role Setup
Step through steps 1 and 2 of the [managed installation instructions](/docs/using-saturn-cloud/enterprise/installation/). **Do not proceed to step 3** (the "Deploy" button). Note the role ARN. On the confirmation screen, note the organization name you chose.

If Saturn will not be permitted to assume the created IAM role, you may remove the trust relationship.

### 2. Receive your Authorization Token
[Contact Saturn](/docs/reporting-problems/) and ask for your authorization token. We will need the organization name you chose in step 1. Saturn will also provide the `external_id` value you will need later.

### 3. Set the INSTALLER_TAG variable
Set the `INSTALLER_TAG` environment variable. Saturn will provide the correct value for this.

```sh
export INSTALLER_TAG=<value>
```

### 4. Set the AWS_AUTH_VARS variable
Set the `AWS_AUTH_VARS` variable according to how you intend to authenticate with AWS. Choose one of the following:

* If you've signed in via `aws configure` and have an `~/.aws/config` file that contains credentials:
    ```sh
    export AWS_AUTH_VARS="-v ${HOME}/.aws:/root/.aws"
    ```
* If you have `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_DEFAULT_REGION` set:
    ```sh
    export AWS_AUTH_VARS="-e AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY -e AWS_DEFAULT_REGION"
    ```
* If running from an EC2 instance with an assigned IAM role:
    ```sh
    export AWS_AUTH_VARS=""
    ```

### 5. Create the Installation Configuration

1. Create a directory. This directory must be persistent (must not be lost after this installation is complete). Set an environment variable, `DATA_DIR`, to the directory's path.

    ```sh
    export DATA_DIR=/path/to/my/directory
    ```

2. Inside the directory, create a file named `config.yaml` and open it.
    * Add:
        ```yaml
        customer_authorization_token: <provided by Saturn in step 2>
        external_id: <provided by Saturn in step 2>
        org_name: <value you chose in step 1>
        region: <value you chose in step 1>
        admin_email: <value you chose in step 1>
        is_enterprise: true
        ```
    * Add your AWS account ID. This can be found in the upper-right while signed into the AWS console. Make sure to use quotes!
        ```yaml
        aws_account_id: "your account ID"
        ```
    * If using a pre-existing VPC, add:
        ```yaml
        skip_vpc: true
        vpc_id: <your-vpc-id>
        private_subnets:
        - <ID of your first private subnet>
        - <ID of your second private subnet>
        worker_subnets:
        - <ID of your first private subnet>
        ```
        * If the installation will be accessible from the public internet, add:
            ```yaml
            public_subnets:
            - <ID of your first public subnet>
            - <ID of your second public subnet>
            ```

            Otherwise, add:
            ```yaml
            public_subnets: []
            private_cluster: true
            internal_lb: true
            ```

            Then, add one of the following:
            * If running the install from an EC2 instance with a security group applied to it, and you wish to grant that security group the ability to connect to the cluster, add:
                ```yaml
                k8s_api_allowed_security_group_ids:
                - <your-security-group-id>
                ```
            * If not using the `k8s_api_allowed_security_group_ids` setting above, add:
                ```yaml
                k8s_api_allowed_cidrs:
                - <cidr-for-your-location>
                ```
                Where `<cidr-for-your-location>` contains the location that you are running the installer from.
        * If the instances inside the VPC will not be allowed egress to the public internet, add:
            ```yaml
            populate_examples: false
            image_metadata_bucket: ""
            ```
        * If using an image mirror, add:
            ```yaml
            image_mirror: <URL of your image mirror>
            ```
    Save your changes.

3. Run the installer.

    ```sh
    docker run --rm -it -v ${DATA_DIR}/sdata ${AWS_AUTH_VARS} saturncloud/saturn-aws:${INSTALLER_TAG} python saturn_aws/scripts/main.py install
    ```

    This will take some time - typically 15-45 minutes. If you encounter errors, [contact us](/docs/reporting-problems/) and we will help debug. When installation completes successfully, you will receive an email instructing you to reset your password for the `admin` account on your new Saturn deployment.

### 6. Set Password and Log In
Follow the instructions in [step 4 of the managed installation instructions](/docs/using-saturn-cloud/enterprise/installation/#step-4-log-in-to-your-saturn-cloud-installation) to begin using your cluster. Note: depending on the configuration values chosen above, you may need to follow additional instructions from your IT department to access your new cluster.
