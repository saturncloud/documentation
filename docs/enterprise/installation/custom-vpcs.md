# Custom VPCs

Rather than using our standard installation architecture, Saturn Cloud Enterprise can be installed into an existing VPC. Installing Saturn Cloud into an existing VPC can make it easier to integrate with other services your company is running in that VPC. All we need are the vpc and subnet IDs. We may also need to add a [few tags to your VPC for EKS](https://aws.amazon.com/premiumsupport/knowledge-center/eks-vpc-subnet-discovery/).

Installing into an existing VPC also allows you to keep Saturn Cloud off of the public internet. We can isolate Saturn Cloud to private subnets, provided that you have a VPN solution so that your users can connect to those subnets.

If your organization requires - this VPC can have restricted egress to the internet. The below diagram outlines such a configuration. All Saturn Cloud resources are deployed to private subnets. A transit gateway is used to connect that VPC with on-premise resources. Any external internet is routed through on-premise networks so that corporate firewalls can properly restrict egress traffic.

<img src="/images/docs/saturn-architecture-onpremise.webp" class="doc-image"/>
{{% enterprise_docs_view %}}
