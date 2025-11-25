# Customization

Saturn Cloud Enterprise is designed to integrate with your existing infrastructure and tooling. This guide covers customization options.

## Docker Images

Saturn Cloud runs your Docker containers with code from your Git repositories. You have two options:

**Use Saturn Cloud images**: Pre-built images with NVIDIA libraries (CUDA, cuDNN, NCCL, NeMo, RAPIDS). Updated regularly with security patches and new library versions.

**Build your own images**: Use any Docker image. Common patterns:
- Start from Saturn Cloud base images and add your dependencies
- Start from NVIDIA base images (nvcr.io) and add Saturn Cloud requirements
- Use completely custom images from your own registry

See the [Creating Images documentation](/docs) for details.

## Node Pool Customization

The reference Terraform includes standard node pool configurations (CPU and GPU sizes). You can customize:

- Node pool sizes (different vCPU/memory combinations)
- GPU types (add additional GPU models beyond the reference configuration)
- Autoscaling limits (min/max node counts)
- Boot disk sizes and types
- Availability zone placement

**Important**: The Saturn Cloud operator configuration must match your node pool setup. Saturn Cloud can only provision workloads on node pools that exist in your Terraform. When customizing node pools, update the `instanceConfig` in the operator values to reflect your infrastructure.

## Deploying Additional Services

You can deploy additional services into the same Kubernetes cluster as Saturn Cloud. Common patterns:

### Workflow Orchestrators

Deploy Prefect, Flyte, Dagster, or Airflow alongside Saturn Cloud. Saturn Cloud workspaces and jobs can trigger workflows in these systems or be triggered by them.

Example use case: Prefect orchestrates your ML pipeline, with individual steps running as Saturn Cloud jobs.

### Databases

Deploy ClickHouse, PostgreSQL, TimescaleDB, or other databases in the cluster. Saturn Cloud workloads connect to these databases using standard connection strings.

Example use case: ClickHouse for experiment tracking and analytics, accessed from Saturn Cloud workspaces.

### Security and Monitoring

Deploy security agents and monitoring tools via DaemonSets:

- **Security**: Crowdstrike Falcon, SentinelOne, Orca Security
- **Monitoring**: Datadog agents, your own Prometheus/Grafana stack
- **Logging**: Additional log collectors or forwarders

Saturn Cloud workloads run under your cluster's security and monitoring policies automatically.

### Custom Ingress Controllers

Saturn Cloud uses Traefik as the ingress controller for routing traffic to the Saturn Cloud UI, workspaces, jobs, and deployments.

You can deploy your own ingress controllers (Nginx, Istio, etc.) for services you manage separately. Both ingress controllers coexist in the same cluster.

## Network Configuration

Saturn Cloud integrates with your VPC and subnet configuration. Customization options:

- **Private load balancers**: Deploy Saturn Cloud behind a private load balancer (contact support@saturncloud.io for configuration)
- **Network policies**: Add additional network policies for workload isolation beyond Saturn Cloud's defaults
- **VPN integration**: Deploy Saturn Cloud within your VPN for private access

## Storage

Saturn Cloud uses the cloud provider's native storage:

- **Persistent volumes**: For workspace home directories
- **Shared folders**: For team collaboration (uses cloud provider's shared filesystem infrastructure)

You can provision additional storage (object storage, NFS, etc.) and mount it in Saturn Cloud workloads.

## Portability and Migration

Saturn Cloud workloads are standard Kubernetes pods running your containers and code from Git repositories.

**Exporting configurations**: Resource configurations export as YAML recipes via the Saturn Cloud CLI or API.

**Data ownership**: All data (workspace home directories, shared folders, databases) stays in your cloud account. If you stop using Saturn Cloud, your data remains accessible.

**Migration off Saturn Cloud**: Redeploying your workloads on standard Kubernetes is straightforward:
- Your containers run without modification
- Git repositories remain the source of truth for code
- No proprietary runtime or data formats to convert

## Integration Examples

Customers integrate Saturn Cloud with a wide range of tools. Common examples include:

- **Feature stores**: Feast, Tecton
- **Experiment tracking**: Weights & Biases, MLflow (self-hosted or SaaS), Neptune, Comet
- **Data platforms**: Snowflake, Databricks, BigQuery
- **CI/CD**: GitHub Actions, GitLab CI, Jenkins
- **Secret management**: HashiCorp Vault, your cloud provider's secret manager

This list is not comprehensive. If you need to integrate with a specific tool, contact support@saturncloud.io to discuss your requirements.

See the [Integrations documentation](/docs) for specific examples.

{{% enterprise_docs_view %}}
