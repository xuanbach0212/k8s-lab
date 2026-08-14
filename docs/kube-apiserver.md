# kube-apiserver

- Acts as the central management component, handling: request -> validate -> authenticate -> interface with etcd -> coordinate with other system components.
- Lifecycle of an API server request: authenticate user -> validate request -> retrieve/update data in etcd -> scheduler -> kubelet.

## Get static pod manifest

```bash
sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

## Get running process

```bash
ps -aux | grep kube-apiserver
```
