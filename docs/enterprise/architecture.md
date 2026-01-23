# Architecture|
| **Kubernetes Version** | 1.24+ |
| **Container Runtime** | containerd, CRI-O, Docker (deprecated) |
| **Operator Pattern** | Helm-based (Operator SDK) |
| **Database** | PostgreSQL |
| **Logging** | Elasticsearch, Fluent Bit |
| **Monitoring** | Prometheus, Kube-State-Metrics |
| **Ingress** | Traefik |
| **TLS** | Cert-Manager with Let's Encrypt |
| **GPU Support** | NVIDIA (multiple CUDA versions), AMD ROCm |
| **Storage** | EBS, Azure Disk, GCE PD, OCI Block Volume, NFS/FSx |
| **Authentication** | JWT (RS256), SSO (SAML, OIDC, OAuth 2.0) |
| **Registry** | public.ecr.aws/saturncloud |

## Additional Resources

- **Helm Charts**: [github.com/saturncloud/saturn-k8s](https://github.com/saturncloud/saturn-k8s)
- **Terraform Examples**: [github.com/saturncloud/saturncloud-reference-terraform](https://github.com/saturncloud/saturncloud-reference-terraform)
- **Operations Guide**: [Operations](<docs/enterprise/operations.md>)
- **Customization Guide**: [Customization](<docs/enterprise/customization.md>)
- **Support**: support@saturncloud.io
