# Service Networking

- Pods are ephemeral and their IPs change — a **Service** gives a stable virtual IP (ClusterIP) in front of a set of Pods (see [service.md](../core_concepts/service.md)).
- A ClusterIP is **virtual** — no interface, no namespace, nothing listens on it. It only exists as forwarding rules on each node.
- **kube-proxy** (a DaemonSet, one per node) watches Services/EndpointSlices via kube-apiserver and programs those rules.

## kube-proxy modes

| Mode | How |
|------|-----|
| `iptables` (default) | DNAT rules: `ClusterIP:port` → one of the Pod IPs, picked randomly |
| `ipvs` | Kernel load balancer, more algorithms, scales better with many Services |
| `nftables` | Newer replacement for iptables mode (GA since 1.33) |

```bash
# which mode is in use
kubectl logs -n kube-system <kube-proxy-pod> | grep -i proxier
kubectl get configmap kube-proxy -n kube-system -o yaml | grep mode   # empty = iptables
```

## See the rules for a Service

```bash
kubectl get svc my-svc                 # e.g. ClusterIP 10.103.132.104
kubectl get endpointslices -l kubernetes.io/service-name=my-svc

# on a node (iptables mode)
iptables -t nat -L KUBE-SERVICES -n | grep my-svc
iptables -t nat -L -n | grep 10.103.132.104
```

Rule chain: `KUBE-SERVICES` → `KUBE-SVC-xxx` (per Service, load-balances) → `KUBE-SEP-xxx` (per endpoint, DNATs to the Pod IP).

## Service IP range

Set on kube-apiserver — Service IPs are allocated from here, and it must not overlap the Pod CIDR (see [cluster_networking.md](cluster_networking.md)).

```bash
grep service-cluster-ip-range /etc/kubernetes/manifests/kube-apiserver.yaml
```

## NodePort

Same rules as ClusterIP, plus a rule on **every** node for the NodePort — so `<any-node-ip>:<nodePort>` works even if the Pod runs on another node.
