# kubelet

- An agent that runs on each node of the cluster. It makes sure that containers are running in a Pod.
- kubeadm does not deploy kubelet.

## Get static pod manifest

```bash
sudo cat /etc/kubernetes/manifests/kubelet.yaml
```

## Get running process

```bash
ps -aux | grep kubelet
```
