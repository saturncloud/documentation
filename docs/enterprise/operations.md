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

### Recovering After a Long Shutdown

The operator holds a credential for the Saturn Cloud manager (`manager.saturnenterprise.io`). The credential is valid for 7 days and the operator renews it about once a day, so it never expires while the operator is running and can reach the manager.

It does expire if the operator cannot renew it for about 6 to 7 days in a row. That happens when:

- the cluster, or its system nodes, are stopped or scaled to zero for a week or more;
- the operator deployment is scaled to zero for that long;
- the cluster cannot reach the manager for that long, for example after a firewall or egress change.

**What you see**: while the operator is running, it emits Kubernetes `Warning` events on the `saturn-auth-token` secret: `TokenRefreshFailed` for each failed renewal, and `RefreshTokenExpiringSoon` once less than 3 days are left. The operator also exposes the metrics `saturn_operator_refresh_token_expiry_timestamp_seconds`, `saturn_operator_token_refresh_last_success_timestamp_seconds` and `saturn_operator_token_refresh_failures_total` on its metrics port, if you want to alert on them. After the credential has expired, the next time the operator pod starts it goes into `CrashLoopBackOff`, its log says it has no usable manager credentials, and it emits a `BootstrapTokenRequired` event. Older operator versions show only the log message `stored refresh token expired at ...; needs a new bootstrap token`.

```bash
kubectl -n saturn-system get events --field-selector involvedObject.name=saturn-auth-token
```

Running workloads keep running, but the operator stops reconciling until you give it a new bootstrap token.

**To recover**:

1. Get a new bootstrap token. It is emailed to you, and is valid for 4 hours. (Installs on AWS use `https://manager.aws.saturnenterprise.io` instead; the operator's error message shows the right address.)

   ```bash
   curl -X POST https://manager.saturnenterprise.io/v2/resend-setup \
       -H "Content-Type: application/json" \
       -d '{
         "name": "your-organization-name",
         "email": "your-email@example.com"
       }'
   ```

2. Pass it to the operator.

   - **Helm installs**: find the release name and the installed chart version, then upgrade with the same version and only the token changed. Leaving out `--version` would also upgrade the chart.

     ```bash
     helm list -n saturn-system
     helm upgrade <release> saturncloud/saturn-helm-operator \
         --version <installed chart version> \
         --namespace saturn-system \
         --reuse-values \
         --set bootstrapToken=NEW_BOOTSTRAP_TOKEN
     ```

     If you installed the Nebius marketplace chart, the value is `saturn-helm-operator.bootstrapToken` instead of `bootstrapToken`.

   - **Terraform installs**: set `saturn_bootstrap_token` to the new token (in `terraform.tfvars`, or `TF_VAR_saturn_bootstrap_token` where the reference Terraform asks for that) and run `terraform apply`.

3. The operator pod restarts, exchanges the bootstrap token for a new 7-day credential, and resumes reconciling. Check it with `kubectl -n saturn-system get pods`.

If you plan to stop a cluster for more than about 6 days, expect to repeat these steps when you start it again.

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
