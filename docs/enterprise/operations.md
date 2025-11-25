# Operations

This guide covers operational aspects of running Saturn Cloud Enterprise across all cloud providers.

## Operator and Platform Components

Saturn Cloud runs via the saturn-helm-operator, which manages platform components as Kubernetes custom resources. The operator reconciles Helm releases every 2 minutes and handles:

- Helm chart updates for Saturn Cloud components
- Image registry credentials (12-hour validity for pulling Saturn Cloud images)
- SSL certificate management (3-month validity)
- Email service authentication

### Operator Failure Modes

If the saturn-helm-operator stops reconciling:

- **Existing workloads continue running**: User workspaces, jobs, and deployments are unaffected
- **New workloads affected after 12 hours**: Once image registry credentials expire, new resources cannot pull Saturn Cloud images
- **Platform access affected after 3 months**: SSL certificate expiration makes the platform inaccessible
- **Email notifications stop**: Email service authentication expires

**Notification**: Saturn Cloud support receives automatic alerts when the operator stops reconciling and will reach out to help resolve.

## Upgrades

Saturn Cloud applies platform upgrades via Terraform updates and Helm chart releases.

**Upgrade cadence**: Typically every 6 months, scheduled with your team. More frequent updates available if needed.

**Downtime**: Upgrades cause 1-2 minutes of downtime for the Saturn Cloud UI and API. User workloads (workspaces, jobs, deployments) continue running and are unaffected.

**Backwards compatibility**: Platform upgrades do not affect running workloads. Your Docker containers remain unchanged until you redeploy them with new configurations. Existing workspace and job definitions continue working after upgrades.

## Debugging

### Workspace, Job, and Deployment Failures

Logs for all Saturn Cloud resources are accessible via:
- **Saturn Cloud UI**: Click on the resource, view logs in the Status tab
- **kubectl**: `kubectl logs <pod-name>` for direct pod log access

Most issues are escalated directly to Saturn Cloud support, who resolves them with users. Platform engineers typically do not need to debug individual workspace failures.

### Platform Component Failures

**Atlas (API server and database) failure**: If Atlas goes down:
- Platform UI and API become unavailable
- Running workspaces, jobs, and deployments continue executing
- Authentication stops working after existing sessions expire (typically a few hours)
- Production deployments can be configured with authentication disabled (safe when deployed within your VPN, behind your own auth layer, or as public APIs)

**Other component failures**: Individual component failures (Traefik, auth-server, ssh-proxy) affect specific functionality but do not impact running user workloads. Contact Saturn Cloud support for component-level issues.

## Performance

Saturn Cloud executes your Docker containers directly in Kubernetes. There is no proprietary runtime or performance overhead beyond standard Kubernetes pod scheduling.

Saturn Cloud infrastructure (logging, metrics collection, usage tracking) runs alongside your workloads, not in the execution path. Performance characteristics are determined by:
- Your container and code
- Kubernetes scheduling overhead (minimal)
- Underlying cloud provider compute performance

## Monitoring and Alerts

Saturn Cloud includes:
- **Logging**: Fluent Bit log aggregation, accessible via UI
- **Metrics**: Prometheus for cluster and resource metrics
- **Operator monitoring**: Automatic alerts to Saturn Cloud support for operator failures

You can deploy additional monitoring tools (Datadog, your own Prometheus stack, etc.) into the same Kubernetes cluster.

{{% enterprise_docs_view %}}
