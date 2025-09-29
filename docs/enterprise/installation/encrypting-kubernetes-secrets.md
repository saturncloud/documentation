# Encrypting Kubernetes Secrets

Secrets inside Saturn Cloud are encrypted inside the database. When containers are deployed - these secrets are deployed into the Kubernetes cluster.

We currently do not encrypt Kubernetes secrets inside EKS by default because doing so requires a KMS key to be created within your account.

If you would like to enable Kubernetes secrets encryption - please contact support@saturncloud.io with the ARN of the KMS key. We can then enable secrets encryption within EKS.

{{% enterprise_docs_view %}}
