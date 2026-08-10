# service

- service enable communication between various components within and outside of the application

## Types

- NodePort: Exposes the Service on each Node's IP at a static port (default: 30000-32767)
- ClusterIP: Only accessible within the cluster - Used for internal pod-to-pod communication
- LoadBalancer: Provisions a LoadBalancer for application in supported cloud providers
- ExternalName: Maps a service to external DNS name (no proxying, just CNAME) - Useful for integrating external services into the cluster

## CMD

- kubectl get services or kubectl get svc
- kubectl get endpointslices
