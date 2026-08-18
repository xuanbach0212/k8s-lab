# Cluster Upgrade

- k8s components have their own version, but must respect a supported skew relative to `kube-apiserver` (the baseline):
  - `kube-apiserver`: vX
  - `controller-manager`, `kube-scheduler`: vX or vX-1
  - `kubelet`, `kube-proxy`: vX, vX-1, or vX-2
  - `kubectl`: vX+1, vX, or vX-1
- **Upgrade one minor version at a time** (e.g. 1.30 → 1.31 → 1.32) — you cannot skip a minor version.
- Recommended order: `kube-apiserver` → `controller-manager` + `kube-scheduler` → `kubelet` (node by node) → `kube-proxy` → `kubectl`.
- `kubeadm`/`kubelet`/`kubectl` are pinned via `apt-mark hold` right after the initial install (so they don't silently auto-upgrade). Must `apt-mark unhold` before installing a new version, then `apt-mark hold` again afterwards.
- The Kubernetes apt repo is **versioned per minor release** (`pkgs.k8s.io/core:/stable:/v1.31/...`) — apt can't find the new package version until the repo entry is updated to point at the target minor version. Do this on **every node** (control plane + workers) before installing.

## 0. Point apt to the new minor-version repo (run on every node)

```bash
# replace v1.31 with your target minor version
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key \
  | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' \
  | tee /etc/apt/sources.list.d/kubernetes.list

apt-get update
```

## With kubeadm

### 1. Check available versions

```bash
kubeadm upgrade plan
```

### 2. Upgrade the control plane node

```bash
apt-mark unhold kubeadm
apt-get update && apt-get install -y kubeadm=1.31.0-1.1
apt-mark hold kubeadm

kubeadm upgrade apply v1.31.0

# drain if it also runs workloads
kubectl drain <master-node> --ignore-daemonsets

apt-mark unhold kubelet kubectl
apt-get install -y kubelet=1.31.0-1.1 kubectl=1.31.0-1.1
apt-mark hold kubelet kubectl

systemctl daemon-reload && systemctl restart kubelet

kubectl uncordon <master-node>
```

### 3. Upgrade each worker node — one at a time

```bash
kubectl drain <node> --ignore-daemonsets

apt-mark unhold kubeadm
apt-get install -y kubeadm=1.31.0-1.1
apt-mark hold kubeadm

kubeadm upgrade node

apt-mark unhold kubelet
apt-get install -y kubelet=1.31.0-1.1
apt-mark hold kubelet

systemctl daemon-reload && systemctl restart kubelet

kubectl uncordon <node>
```

### 4. Verify

```bash
kubectl get nodes
```
