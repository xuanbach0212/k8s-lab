# DNS in Kubernetes

- k8s runs a cluster DNS server — **CoreDNS** (Deployment in `kube-system`, 2 replicas by default with kubeadm).
- Exposed via a Service named **`kube-dns`** (name kept for compatibility), usually ClusterIP `10.96.0.10`.
- kubelet writes that IP into every Pod's `/etc/resolv.conf` (kubelet config `clusterDNS`, in `/var/lib/kubelet/config.yaml`).

## Record format

| Object | FQDN |
|--------|------|
| Service | `<service>.<namespace>.svc.cluster.local` |
| Pod | `<ip-with-dashes>.<namespace>.pod.cluster.local` (e.g. `10-244-2-5.default.pod.cluster.local`) |

- Same namespace: `curl http://web-service` works.
- Other namespace: `curl http://web-service.apps` (or the full FQDN).
- Pod records are not created by default in many setups — kubeadm's Corefile enables them with `pods insecure`.

## Inside a Pod

```bash
kubectl exec -it <pod> -- cat /etc/resolv.conf
# nameserver 10.96.0.10
# search default.svc.cluster.local svc.cluster.local cluster.local
```

The `search` list is why short names work — `web-service` is tried as `web-service.default.svc.cluster.local` first. Pod names are **not** in the search list, so Pods must be resolved by their full FQDN.

## CoreDNS config

Stored in the `coredns` ConfigMap (mounted as `/etc/coredns/Corefile`):

```bash
kubectl get configmap coredns -n kube-system -o yaml
```

```
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {   # serves cluster.local records
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {                       # anything else -> node's upstream DNS
       max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}
```

## Debug DNS

```bash
kubectl run test --image=busybox:1.28 --rm -it --restart=Never -- nslookup web-service

kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl get svc kube-dns -n kube-system
kubectl logs -n kube-system -l k8s-app=kube-dns
```

`busybox:1.28` is pinned on purpose — `nslookup` in newer busybox images is known to misbehave against cluster DNS.
