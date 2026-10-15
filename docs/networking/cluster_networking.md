# Cluster Networking

- Every node needs at least one interface with an IP on a network shared with the other nodes, a unique hostname, and a unique MAC address.
- Required ports must be open between nodes (firewall / security groups).

## Required ports

Control plane:

| Component | Port |
|-----------|------|
| kube-apiserver | `6443` |
| etcd | `2379` (clients), `2380` (peers, only for multi-master) |
| kubelet | `10250` |
| kube-scheduler | `10259` |
| kube-controller-manager | `10257` |

Worker nodes:

| Component | Port |
|-----------|------|
| kubelet | `10250` |
| kube-proxy (healthz) | `10256` |
| NodePort Services | `30000–32767` |

Reference: <https://kubernetes.io/docs/reference/networking/ports-and-protocols/>

## Inspect a node

```bash
ip addr                         # node IP + interfaces (cni0, flannel.1, cali*, ...)
ip addr show type bridge        # bridges on the node
ip route                        # default gateway + routes to other nodes' Pod subnets
ip link show eth0               # MAC address + state

ss -tlnp | grep -E 'kube|etcd'  # which component listens on which port
```

## The 3 IP ranges in a cluster

They must **not overlap** with each other.

| Range | Set by | How to find it |
|-------|--------|----------------|
| Node IPs | infra (Vagrant / cloud / DHCP) | `kubectl get nodes -o wide` |
| Pod CIDR | `kubeadm init --pod-network-cidr` + CNI config | `kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'`, or the CNI config / logs |
| Service CIDR | kube-apiserver `--service-cluster-ip-range` (kubeadm default `10.96.0.0/12`) | `grep service-cluster-ip-range /etc/kubernetes/manifests/kube-apiserver.yaml` |
