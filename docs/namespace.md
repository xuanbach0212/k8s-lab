# namespace

- provide isolate group of resources within cluster
- Each namespace can have its own policies and resource quotas.
- Divide cluster resources among different users, teams, or projects.
- other service and communication with other namespace via <service_name>.<namespace>.svc.cluster.local

## CMD

kubectl config set-context $(kubectl config current-context) --namespace=dev

### Resource quotas

<https://kubernetes.io/docs/concepts/policy/resource-quotas/>
