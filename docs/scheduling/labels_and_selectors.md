# Labels and Selectors

- **Labels** — key-value pairs attached to objects (Pods, Nodes, ReplicaSets...) to tag/group them (e.g. by app, tier, environment).
- **Selectors** — used to filter/query objects by their labels.
- This is how ReplicaSet/Deployment/Service know *which* Pods belong to them — not by name, but by matching labels via `selector.matchLabels`.
- **Annotations** are similar to labels but not used for filtering — just extra metadata (e.g. build version, contact info).

## Attach labels to a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: App1
    function: Front-end
spec:
  containers:
    - name: nginx
      image: nginx
```

## Query by label (selector)

```bash
kubectl get pods --selector app=App1
kubectl get pods -l app=App1,function=Front-end

kubectl get all --selector env=prod
```

## How a ReplicaSet selects its Pods

The RS `spec.selector.matchLabels` must match `spec.template.metadata.labels` — that's the link between them.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: App1        # <- must match template labels below
  template:
    metadata:
      labels:
        app: App1
        function: Front-end
    spec:
      containers:
        - name: nginx
          image: nginx
```
