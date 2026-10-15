# Ingress

- A single L7 (HTTP/HTTPS) entry point that routes traffic to multiple Services by **host** and **path** — instead of one LoadBalancer per Service.
- Also handles TLS termination.
- 2 parts:
  - **Ingress controller** — the actual reverse proxy (nginx, Traefik, HAProxy, ...). **Not** installed by default — without one, Ingress objects do nothing.
  - **Ingress resource** — the routing rules.
- The controller itself is exposed via a NodePort/LoadBalancer Service.
- Ingress API is feature-frozen — new work goes into [Gateway API](gateway_api.md). Note the community `ingress-nginx` controller has been retired (no more releases after March 2026) — prefer another controller or Gateway API for new setups.

## Sample Ingress — split by path

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /wear
            pathType: Prefix
            backend:
              service:
                name: wear-service
                port:
                  number: 80
          - path: /watch
            pathType: Prefix
            backend:
              service:
                name: watch-service
                port:
                  number: 80
```

## Split by host

```yaml
spec:
  ingressClassName: nginx
  rules:
    - host: wear.my-online-store.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: wear-service
                port:
                  number: 80
    - host: watch.my-online-store.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: watch-service
                port:
                  number: 80
```

## Notes

- `pathType`: `Prefix` (matches `/wear`, `/wear/x`), `Exact`, `ImplementationSpecific`.
- `rewrite-target: /` — strips the matched path before forwarding (`/wear/x` → `/x` on the backend). Without it the app receives `/wear/...` and may 404.
- `ingressClassName` picks the controller; check available ones with `kubectl get ingressclass`.
- Ingress must be in the **same namespace** as the Services it routes to.
- Requests that match no rule go to `spec.defaultBackend` (if set).

## Commands

```bash
# imperative
kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"

kubectl get ingress -A
kubectl describe ingress ingress-wear-watch   # shows rules + backends

kubectl get ingressclass
```
