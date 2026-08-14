# Service

- Enables communication between various components, within and outside of the application.

## Types

- **ClusterIP** — only accessible within the cluster; used for internal pod-to-pod communication.
- **NodePort** — exposes the Service on each node's IP at a static port (default: 30000–32767).
- **LoadBalancer** — provisions a load balancer for the application on supported cloud providers.
- **ExternalName** — maps a Service to an external DNS name (no proxying, just CNAME); useful for integrating external services into the cluster.

## Commands

```bash
kubectl get services   # or: kubectl get svc

kubectl get endpointslices
```
