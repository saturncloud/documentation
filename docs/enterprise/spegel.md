# Spegel (Image Pull Acceleration)|
| `saturnComponents.spegel.enabled` | `false` (Nebius: `true`) | Master switch. When false, none of the resources below exist. |
| `values.mirroredRegistries` | `[]` | Registries Spegel writes mirror config for. List the registry hosts your cluster actually pulls from in volume. Leaving it empty has Spegel attempt to mirror every registry, which is rarely what you want. |
| `values.containerdPrep.enabled` | `true` | Whether to deploy the DaemonSet that patches containerd. |
| `values.mirrorResolveRetries` | `3` | Spegel default. Max peer lookups per layer before falling through to upstream. |
| `values.mirrorResolveTimeout` | `20ms` | Spegel default. Per-attempt timeout. Increase only if intra-cluster latency is unusually high. |

Anything under `values:` is passed through to the upstream Spegel chart, so see the [chart's values.yaml](https://github.com/spegel-org/spegel/blob/main/charts/spegel/values.yaml) for the full list. Common overrides:

- `values.serviceMonitor.enabled: true` to expose Prometheus metrics if you run kube-prometheus-stack.
- `values.spegel.registryFilters: ["^registry\\.k8s\\.io/"]` to exclude registries Spegel should not touch.

The Spegel image itself is set centrally in the operator's `images.spegel` field and is automatically rewritten through `imageMirror` for air-gapped deployments. You should not need to override it in the Spegel CR.

## Verifying

After enabling Spegel and waiting for both DaemonSets to roll out:

```bash
kubectl -n spegel get ds
```

You should see `spegel` and (if enabled) `spegel-containerd-prep`, both with the desired number of pods ready.

Check that containerd is reading the mirror config. On a node:

```bash
ls /etc/containerd/certs.d/
```

You should see a directory per mirrored registry, each containing a `hosts.toml` that points the registry's host at `localhost:5000` (Spegel's local port) with the upstream as fallback.

Spegel exposes a metrics endpoint and a debug web page on each pod. Port-forward to see them:

```bash
kubectl -n spegel port-forward ds/spegel 9090:9090
# Then visit http://localhost:9090/metrics
```

Useful metrics:

- `spegel_mirror_requests_total{cache_type="hit"}`: pulls served from a peer
- `spegel_mirror_requests_total{cache_type="miss"}`: pulls that fell through to the upstream
- `spegel_advertised_images`: number of images this node is advertising to peers

A healthy Spegel deployment shows the hit count climbing as nodes pull the same images.

## Disabling

Set `saturnComponents.spegel.enabled: false` in the operator values and re-apply. The operator uninstalls the Helm release, which removes both DaemonSets, the Spegel namespace, and the mirror configuration Spegel had written to `/etc/containerd/certs.d`.

The containerd-prep DaemonSet is also removed, but it does not roll back the `config_path` setting it added to `config.toml`. Leaving `config_path` set is harmless (containerd just finds no mirror entries) and means you can re-enable Spegel later without another containerd restart. If you want to fully undo the change, edit `config.toml` and restart containerd manually.

After Spegel is gone, all image pulls fall through to the upstream registry exactly as they did before Spegel was installed.

## Troubleshooting

**Spegel pods running, but pulls still take the same time as before.**
Check `/etc/containerd/certs.d` on a node. If it is empty or missing, containerd does not have `config_path` set, and Spegel's mirror config is being ignored. Either enable `containerdPrep` or set `config_path` via your node-provisioning mechanism.

**Spegel metrics show miss but never hit.**
Check `discard_unpacked_layers` on the node (`grep discard_unpacked /etc/containerd/config.toml`). If it is `true`, containerd is throwing layers away after unpacking, so peers have nothing to serve.

**ImagePullBackOff on `gcr.io/google-containers/pause`.**
This registry path was retired by Google in March 2025. If you copied the containerd-prep manifest from an older example that hardcodes it, switch to the Saturn-managed component (which uses `saturn-k8s-utils` for the keepalive) or update the manifest to use `registry.k8s.io/pause`.

{{% enterprise_docs_view %}}
