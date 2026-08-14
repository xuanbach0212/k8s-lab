# kube-controller-manager

- Monitors the state of the cluster, then makes changes to move it toward the desired state.
- Contains multiple sub-controllers, e.g. node, replication, deployment, namespace.

## Get static pod manifest

```bash
sudo cat /etc/kubernetes/manifests/kube-controller-manager.yaml
```

## Get running process

```bash
ps -aux | grep kube-controller-manager
```
