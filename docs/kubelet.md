# kubelet

- an agent that runs on each node of the cluster. It makes sure that container are running in a Pod
- kubeadm do not deploy kubelet

## Get Option

sudo cat /etc/kubernetes/manifests/kubelet.yaml

## Get running process

ps -aux | grep kubelet
