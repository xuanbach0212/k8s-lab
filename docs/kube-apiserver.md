# kube-apiserver

- kube-apiserver acts like a central management component handling request -> validate -> authenticate -> interface with etcd -> coordinating with other system components

- lifecycle of api server request:
  authenticate user -> validate request -> retrieve data -> update etcd -> scheduler -> kubelet

## Get Option

sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml

## Get running process

ps -aux | grep kube-apiserver
