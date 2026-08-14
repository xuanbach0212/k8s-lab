# Deployment

- Manages a set of Pods to run an application workload.
- Provides declarative updates for Pods and ReplicaSets.

## Use cases

- Create a Deployment to roll out a ReplicaSet.
- Declare a new state for the Pods.
- Rollback to an earlier Deployment revision.
- Scale up the Deployment.
- Pause the rollout.
- Clean up older ReplicaSets.
- Check the status of a Deployment.

## Sample Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

## Commands

```bash
kubectl create deployment --image=nginx nginx

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml

kubectl create -f nginx-deployment.yaml

kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```
