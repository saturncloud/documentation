# Saturn Cloud Architecture Components

This article details the AWS resources that are used by Saturn Cloud. Saturn Cloud has a few different configurations - this article is written towards the most secure configuration.

## IAM Resources

- **The installation role**. This IAM role is used to perform the installation. It is the role that creates the EKS cluster. It must be used if you need to recover access to the EKS cluster. [Most customers create this role via cloud formation](/docs). This role has very expansive permissions, but we can provide tighter policies if necessary.
- **The EKS cluster role**. This role is used by the EKS cluster control plane.
- **The EKS worker role**. This role is used by nodes in the EKS cluster.
- **The Saturn UI role**. This role is used by our application running in the EKS cluster.
- **The efs/fsx role (optional)**. If you want to use Saturn Cloud with NFS - this role can mount network filesystems.

## S3 buckets

Saturn Cloud creates a single S3 bucket to hold terraform state, and other application data. The bucket is encrypted and has public access disabled.

## EKS

Saturn Cloud runs within an EKS cluster in your VPC. In our standard configuration, the EKS cluster control plane is on the public internet with CIDR block restrictions limiting access to our support team, as well as the nodes in the cluster. We can also deploy it with private networking only, however that limits the amount of support we can provide.

## Networking

Saturn Cloud can install into an existing VPC. Saturn Cloud uses 2 load balancers, one for SSH traffic, and a second for https and Dask traffic. All load balanacers can be scoped internally to private subnets. You can optionally restrict egress to the internet. In this configuration, we would modify Saturn Cloud to install PyPI, Conda, and R packages from your on-premise mirrors. All docker images would be consumed from ECR.

{{% enterprise_docs_view %}}
