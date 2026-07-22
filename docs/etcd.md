# ETCD

- etcd is a distributed key-value store that maintains configuration of data, state info, and metadata for kube cluster
- these gonna be stored: nodes, pods, congigs, secrets, accounts, roles, bindings, others.
- typically listens on port 2379 for clients request

## Get Option

sudo cat /etc/kubernetes/manifests/etcd.yaml

## Get running process

ps -aux | grep etcd
