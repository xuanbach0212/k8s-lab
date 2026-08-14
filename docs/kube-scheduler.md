# kube-scheduler

- Determines which nodes are valid for placement, binds the Pod to a node, then the kubelet creates it.

## Get static pod manifest

```bash
sudo cat /etc/kubernetes/manifests/kube-scheduler.yaml
```

## Get running process

```bash
ps -aux | grep kube-scheduler
```
