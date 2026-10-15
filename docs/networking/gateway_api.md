# Gateway API

- The successor to [Ingress](ingress.md) — more expressive and role-oriented.
- Splits responsibility across separate resources so infra team, cluster operators, and app developers each own their part.
- Supports HTTP, gRPC, TLS, TCP/UDP routing, header matching, traffic splitting, etc. **in the spec** (Ingress needed controller-specific annotations for most of this).
- Not built into k8s — install the CRDs, then a controller that implements them (Envoy Gateway, Istio, Cilium, NGINX Gateway Fabric, Traefik, ...).

## Resources

| Resource | Owned by | Purpose |
|----------|----------|---------|
| `GatewayClass` | infra provider | Which controller implements Gateways (like `IngressClass`) |
| `Gateway` | cluster operator | A listener: port, protocol, hostname, TLS |
| `HTTPRoute` (`GRPCRoute`, `TLSRoute`...) | app developer | Routing rules from a Gateway to Services |

## Install the CRDs

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml

kubectl get crd | grep gateway
```

## Sample

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: example-class
spec:
  controllerName: example.com/gateway-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example-gateway
spec:
  gatewayClassName: example-class
  listeners:
    - name: http
      protocol: HTTP
      port: 80
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-route
spec:
  parentRefs:
    - name: example-gateway
  hostnames:
    - "www.example.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /wear
      backendRefs:
        - name: wear-service
          port: 80
```

## Traffic splitting (canary)

```yaml
rules:
  - backendRefs:
      - name: app-v1
        port: 80
        weight: 90
      - name: app-v2
        port: 80
        weight: 10
```

## TLS on the Gateway

```yaml
listeners:
  - name: https
    protocol: HTTPS
    port: 443
    hostname: "www.example.com"
    tls:
      mode: Terminate
      certificateRefs:
        - kind: Secret
          name: example-tls
```

## Commands

```bash
kubectl get gatewayclass
kubectl get gateway
kubectl get httproute
kubectl describe gateway example-gateway    # check listener status / attached routes
```
