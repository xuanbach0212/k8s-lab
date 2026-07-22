# kube-controller-manager

- monitor the state of the cluster then modify to meet the desired status
- contain multiple sub controller such as node, replication, deployment, namespace

## Get Option

sudo cat /etc/kubernetes/manifests/kube-controller-manager.yaml

## Get running process

ps -aux | grep kube-controller-manager
