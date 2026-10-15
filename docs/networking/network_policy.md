# Network Policy

- By default, **all Pods can talk to all Pods** (allow-all).
- A NetworkPolicy selects Pods by label and restricts which traffic is allowed **into** (`Ingress`) and/or **out of** (`Egress`) them.
- Once a Pod is selected by any policy for a direction, everything in that direction not explicitly allowed is **denied**.
- Policies are additive (union of allows) — there are no "deny" rules.
- Rules are **stateful**: allowing ingress on a port also allows the reply traffic; no egress rule needed for responses.
- Enforced by the CNI plugin — **Flannel does not support it** (policies are accepted but silently ignored). Calico, Cilium do.

## Sample — allow only the API Pod to reach the DB on 3306

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db                 # which Pods this policy applies to
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              name: api-pod
      ports:
        - protocol: TCP
          port: 3306
```

## Selector types in `from` / `to`

- `podSelector` — Pods in the **same namespace** as the policy.
- `namespaceSelector` — all Pods in matching namespaces.
- `ipBlock` — CIDR ranges (e.g. an external backup server).

### AND vs OR — watch the dashes

```yaml
# AND — one item: Pods labelled api-pod AND in the prod namespace
- from:
    - podSelector:
        matchLabels:
          name: api-pod
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: prod

# OR — two items: api-pod Pods in this namespace, OR any Pod in prod
- from:
    - podSelector:
        matchLabels:
          name: api-pod
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: prod
```

## Egress example — DB can push backups to an external server

```yaml
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 192.168.5.10/32
      ports:
        - protocol: TCP
          port: 80
```

If you restrict egress, remember to also allow DNS (UDP/TCP `53` to `kube-dns`) — otherwise name resolution breaks.

## Default deny all in a namespace

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}        # all Pods in the namespace
  policyTypes:
    - Ingress
    - Egress
```

## Commands

```bash
kubectl get netpol
kubectl describe netpol db-policy

# test connectivity
kubectl exec -it <pod> -- nc -zv -w 2 db-service 3306
```
