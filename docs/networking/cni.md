# CNI (Container Network Interface)

- A standard for how a container runtime asks a **plugin** to wire a container into the network.
- The runtime creates the network namespace, then calls the plugin with `ADD` (on create) / `DEL` (on delete) / `CHECK`.
- The plugin does the work from [network_namespaces.md](network_namespaces.md): create veth pair, attach to bridge/overlay, assign IP (via IPAM), set routes.
- In k8s, the **container runtime** (containerd / CRI-O) invokes CNI — not kubelet directly (that was dockershim, removed in 1.24).

## Where things live on a node

```bash
ls /opt/cni/bin/          # plugin binaries (bridge, host-local, loopback, flannel, calico...)
ls /etc/cni/net.d/        # plugin config — if multiple files, the first one alphabetically wins
```

Sample config (`/etc/cni/net.d/10-bridge.conf`):

```json
{
  "cniVersion": "1.0.0",
  "name": "mynet",
  "type": "bridge",
  "bridge": "cni0",
  "isGateway": true,
  "ipMasq": true,
  "ipam": {
    "type": "host-local",
    "subnet": "10.244.0.0/16",
    "routes": [{ "dst": "0.0.0.0/0" }]
  }
}
```

## IPAM (IP Address Management)

- Deciding which IP each Pod gets and making sure there are no duplicates — the CNI plugin's job, not k8s's.
- Built-in IPAM plugins: `host-local` (allocates from a per-node range, tracked in a local file), `dhcp`.
- Each node usually gets its own Pod subnet (`node.spec.podCIDR`) carved out of the cluster Pod CIDR.

## Common plugins

| Plugin | Notes |
|--------|-------|
| **Flannel** | Simple overlay (VXLAN). Does **not** enforce NetworkPolicy. |
| **Calico** | BGP or overlay, enforces NetworkPolicy. Very common. |
| **Cilium** | eBPF-based, enforces NetworkPolicy (L3–L7), can replace kube-proxy. |
| **Weave Net** | Seen in older courses — project is no longer maintained. |

## Install a plugin (kubeadm)

```bash
# Pod CIDR must be set at init and match the plugin's config
kubeadm init --pod-network-cidr=10.244.0.0/16

# e.g. Flannel (defaults to 10.244.0.0/16)
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

Until a CNI plugin is installed, nodes stay `NotReady` and CoreDNS Pods stay `Pending`.

## Find which plugin a cluster uses

```bash
ls /etc/cni/net.d/
kubectl get pods -n kube-system        # look for flannel / calico / cilium pods
kubectl get pods -A | grep -iE 'flannel|calico|cilium|weave'
```
