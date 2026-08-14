# Namespace

- Provides an isolated group of resources within a cluster.
- Each namespace can have its own policies and resource quotas.
- Divides cluster resources among different users, teams, or projects.
- Services communicate across namespaces via `<service_name>.<namespace>.svc.cluster.local`.

## Commands

```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

## Resource quotas

<https://kubernetes.io/docs/concepts/policy/resource-quotas/>
