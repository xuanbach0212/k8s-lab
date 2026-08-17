# etcd

- A distributed key-value store that maintains configuration data, state info, and metadata for the cluster.
- Stores: nodes, pods, configs, secrets, accounts, roles, bindings, and more.
- Typically listens on port `2379` for client requests.

## Get static pod manifest

```bash
sudo cat /etc/kubernetes/manifests/etcd.yaml
```

## Get running process

```bash
ps -aux | grep etcd
```
