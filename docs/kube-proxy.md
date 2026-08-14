# kube-proxy

- A network proxy that runs on each node in the cluster.
- Maintains network rules on nodes, allowing network communication to Pods from sessions inside or outside the cluster.

## Get static pod manifest

```bash
sudo cat /etc/kubernetes/manifests/kube-proxy.yaml
```

## Get running process

```bash
ps -aux | grep kube-proxy
```
