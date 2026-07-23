# kube-proxy

- kube-proxy is a network proxy that runs on each node in cluster
- kube-proxy maintain network rules on nodes. These network rules allow network communication to your pods from network sessions inside or outside of your cluster

## Get Option

sudo cat /etc/kubernetes/manifests/kube-proxy.yaml

## Get running process

ps -aux | grep kube-proxy
