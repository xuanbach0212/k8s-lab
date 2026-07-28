# ReplicaSet

- maintain a stable set of replica pods at any given time

- Replication Controller is the older version of ReplicaSet

### sample yaml of ReplicaSet

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
  selector:
    matchlabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
```

### some cmd

kubectl apply -f <https://kubernetes.io/examples/controllers/frontend.yaml>

kubectl get rs

kubectl describe rs/frontend

kubectl scale --replicas=3 rs/foo

kubectl scale --replicas=3 -f foo.yaml
