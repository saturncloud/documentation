# What It Takes to Build an ML Platform on Kubernetes
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: model-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: model-api
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

This is the most straightforward use case on raw Kubernetes, but production still requires:

- **Service + Ingress**: a ClusterIP Service, an Ingress or IngressRoute with TLS, and DNS configuration. Each API endpoint needs its own route
- **Rolling update tuning**: `maxUnavailable: 0` ensures zero-downtime, but you need the readiness probe to accurately reflect when the new pod is ready to serve traffic. If your model takes 30 seconds to load into memory, the `initialDelaySeconds` must account for this
- **Load testing and capacity planning**: you need to benchmark the API under expected load to determine appropriate replica counts and resource requests. Under-provisioned requests cause throttling; over-provisioned requests waste resources
- **Canary deployments** (optional): if you want to gradually shift traffic to a new model version, you need a service mesh (Istio, Linkerd) or Traefik traffic splitting middleware. This adds significant operational complexity

#### With Saturn Cloud

A user creates a Deployment resource with a start command that runs their API server on port 8000. Atlas creates the Kubernetes Deployment with a rolling update strategy, a ClusterIP Service, and an IngressRoute through Traefik. Health check endpoints are configurable per deployment. Multiple replicas can be set for high availability, and Atlas configures the Deployment's rolling update to maintain availability during updates. Code changes propagate through git integration (branch or tag pinning, same as Streamlit deployments) or image rebuilds.

## Further Reading

- [Infrastructure engineering](/docs): how Saturn Cloud fits into your existing cloud environment
