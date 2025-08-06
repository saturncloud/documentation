# No Internet Access

Saturn can be installed in an AWS account with no access to the internet. In order to do this - you'll need a VPC that meets a few requirements, or a Docker image mirror.

-   You need to ensure that your VPC allows egress to `https://manager.aws.saturnenterprise.io/` (`3.134.99.59`)
-   A security group with an inbound rule that allows HTTPS traffic from the VPC’s CIDR
-   [Private VPC endpoints](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html) for the following AWS services, with the security group (from the previous step) attached:
    -   `ecr.dkr` (Interface)
    -   `ecr.api` (Interface)
    -   `ec2` (Interface)
    -   `autoscaling` (Interface)
    -   `sts` (Interface)
    -   `s3` (Gateway)
-   Either egress to `docker.io`, `k8s.gcr.io`, and `quay.io`, or an image mirror set up with access to those hosts. Currently, the only image mirror Saturn supports is [Artifactory](https://jfrog.com/artifactory/). If you require support for a different image mirror solution, [please get in touch](https://deploy-preview-345--saturn-cloud.netlify.app/docs/reporting-problems/)

With these fulfilled, follow the [manual Saturn Cloud installation steps](/docs).
{{% enterprise_docs_view %}}
