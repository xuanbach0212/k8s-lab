# Backup and Restore Methods

There are 2 things worth backing up in a cluster: the **resource configs** (Deployments, Services, ConfigMaps...) and **etcd** (the actual state store — backing this up covers everything).

## 1. Backup resource configs (imperative, quick & dirty)

```bash
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

- Simple but incomplete — easy to miss objects that aren't covered by `get all` (e.g. ConfigMaps, Secrets, RBAC). Prefer backing up etcd instead.
- Best practice: keep resource definitions as declarative yaml files in git (source of truth) instead of relying on cluster state.

## 2. Backup etcd (recommended — covers the whole cluster state)

etcd's data dir + certs are on the control plane node (paths from the static pod manifest, `/etc/kubernetes/manifests/etcd.yaml`).

```bash
ETCDCTL_API=3 etcdctl snapshot save /opt/snapshot-pre-boot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

Check the snapshot:

```bash
ETCDCTL_API=3 etcdctl snapshot status /opt/snapshot-pre-boot.db
```

## 3. Restore etcd from a snapshot

Restore is a **local, offline** operation — it just reads the `.db` file and writes a new data directory, no live etcd connection needed (so no `--endpoints`/certs here).

Since etcd v3.5, `snapshot restore` moved out of `etcdctl` into a dedicated tool, **`etcdutl`** — `etcdctl snapshot restore` still works today but prints a deprecation warning and will eventually be removed. Prefer `etcdutl` going forward.

```bash
# etcdctl (older / still works, deprecated)
ETCDCTL_API=3 etcdctl snapshot restore /opt/snapshot-pre-boot.db \
  --data-dir /var/lib/etcd-from-backup

# etcdutl (recommended, same flags)
etcdutl snapshot restore /opt/snapshot-pre-boot.db \
  --data-dir /var/lib/etcd-from-backup
```

`snapshot status` also works on both tools the same way:

```bash
etcdutl snapshot status /opt/snapshot-pre-boot.db
```

Either command creates a **new** data directory — it doesn't restore in place. Then point etcd at it:

1. Edit `/etc/kubernetes/manifests/etcd.yaml`: change `hostPath` for the etcd data volume to `/var/lib/etcd-from-backup`.
2. kubelet watches the manifests folder and picks up the change on its own (~20s poll) and recreates the etcd static pod — usually no manual reload needed.
3. If it doesn't pick it up (or you want it to apply immediately), force kubelet to reload:

```bash
systemctl daemon-reload
systemctl restart kubelet
```

```bash
# verify
kubectl get pods -n kube-system | grep etcd
kubectl get nodes
```
