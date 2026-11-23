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

**What you see**: while the operator is running, it emits Kubernetes `Warning` events on the `saturn-auth-token` secret: `TokenRefreshFailed` for each failed renewal, and `RefreshTokenExpiringSoon` once less than 3 days are left, or once the manager has rejected the stored credential (for example because it was revoked). A rejection that lasts is reported as `BootstrapTokenRequired` instead, as described below. After the credential has expired, the next time the operator pod starts it goes into `CrashLoopBackOff`, its log says it has no usable manager credentials, and it emits a `BootstrapTokenRequired` event. When the operator has working credentials again after a failure, it emits a `Normal` event, `TokenCredentialsRestored`. The `RefreshTokenExpiringSoon` and `BootstrapTokenRequired` messages, and the operator's log, include the recovery steps below with your manager's address filled in. Older operator versions show only the log message `stored refresh token expired at ...; needs a new bootstrap token`.

**`BootstrapTokenRequired` while the operator is running: it cannot recover by itself.** The manager may reject the operator's stored credential at least twice over at least 2 hours, with no successful renewal in between. From then on, the operator emits a `Warning` event `BootstrapTokenRequired` on the `saturn-auth-token` secret instead of `RefreshTokenExpiringSoon`, and repeats it on every check, hourly by default. This is the same reason the operator emits when it cannot start, so one alert covers both cases. The message starts with `Bootstrap token required:`, says that the operator pod is still running, and says when the manager first rejected the credential and how many times. The manager does not say why it rejected the credential. The usual cause is that it revoked the credential, for example because an old credential was presented again after the operator lost its newer one. Revoking the credential also revokes the operator's current access token, so the operator's calls to the manager (Docker credentials, DNS, ingress) are probably already failing. If many installs report this at the same time, the cause is more likely on the manager side. Waiting will not fix this: follow the recovery steps below, which the event message also contains. The event stops once the operator has working credentials again, and the operator then emits `TokenCredentialsRestored`. If you alert on operator events, alert on `BootstrapTokenRequired`. A single `TokenRefreshFailed` can also come from a brief network or manager problem.

```bash
kubectl events -n saturn-system --for secret/saturn-auth-token
```

If your `kubectl` has no `events` command, use `kubectl -n saturn-system get events --field-selector involvedObject.name=saturn-auth-token` instead.

The operator also exposes these metrics on its metrics port, if you want to alert on them:

- `saturn_operator_refresh_token_expiry_timestamp_seconds`: the Unix time at which the stored credential expires, or the time the manager rejected it. After this the operator needs a new bootstrap token.
- `saturn_operator_token_refresh_last_success_timestamp_seconds`: the Unix time of the last successful exchange of a refresh or bootstrap token with the manager.
- `saturn_operator_token_refresh_failures_total`: the number of failed attempts to renew the credential.

Running workloads keep running, but the operator stops reconciling until it has working credentials again.

**If the manager is unreachable, check connectivity first**: when the operator cannot reach the manager (a network error, or an error reply from the manager or a proxy in between), the stored credential may still be good, and a new bootstrap token cannot be exchanged with a manager the cluster cannot reach either. Check that the cluster can reach the manager, for example with `curl` from a pod in `saturn-system`. The operator retries on its next restart. You need a new bootstrap token only if the stored credential has expired or the manager rejects it; the operator's error message says which case applies and, when the manager is unreachable, when the stored credential expires.

**To recover**:

1. Get a new bootstrap token. Send your organization name and the admin email you registered with. If they match your organization, the manager emails you a new bootstrap token, valid for 4 hours. The reply is the same whether or not they match, so check your email rather than the reply. (Installs on AWS use `https://manager.aws.saturnenterprise.io` instead; the operator's error message shows the right address.)

   ```bash
   curl -X POST https://manager.saturnenterprise.io/v2/resend-setup \
       -H 'Content-Type: application/json' \
       -d '{"name":"<organization name>","email":"<registered admin email>"}'
   ```

   If the account has never been activated, the email contains an activation link instead of a bootstrap token. The link is valid for 1 hour. Open it to activate the account, then run the same request again to get the bootstrap token.

2. Pass it to the operator within 4 hours.

   - **Helm installs**: find the release name and the installed chart version, then upgrade with the same version and only the token changed. Leaving out `--version` would also upgrade the chart.

     ```bash
     helm list -n saturn-system
     helm upgrade <release> saturncloud/saturn-helm-operator \
         --version <installed chart version> \
         --namespace saturn-system \
         --reuse-values \
         --set bootstrapToken=<token>
     ```

     If you installed the Nebius marketplace chart, the value is `saturn-helm-operator.bootstrapToken` instead of `bootstrapToken`.

   - **Terraform installs**: if Terraform manages the operator release, use Terraform, not `helm upgrade`. Set the new token and apply:

     ```bash
     export TF_VAR_saturn_bootstrap_token=<token>
     terraform apply
     ```

     A value in `terraform.tfvars` takes precedence over the environment variable. If your `terraform.tfvars` sets `saturn_bootstrap_token` (the Nebius reference Terraform does), put the new token there instead, then run `terraform apply`.

   The chart stores the token in the Secret `saturn-bootstrap-token` (key `token`) in the operator's namespace, and the operator reads it from there through a `secretKeyRef` as `SATURN_BOOTSTRAP_TOKEN`. A new token value rolls the operator pod. The operator runs as one replica.

3. Setting the value is enough. When the operator starts and finds that its stored credentials have expired or were rejected by the manager, it exchanges the unexpired bootstrap token for a new 7-day credential on its own, emits `TokenCredentialsRestored`, and resumes reconciling. Check it with `kubectl -n saturn-system get pods` and `kubectl events -n saturn-system --for secret/saturn-auth-token`.

The `saturn-bootstrap-token` Secret, the `TokenCredentialsRestored` event, the `BootstrapTokenRequired` event while the operator is running, and the startup exchange after the manager rejects the stored credential are new in the current operator release. Older operator versions pass the token as a plain environment variable in the operator Deployment, and exchange it at startup only once the stored access token has expired as well.

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
