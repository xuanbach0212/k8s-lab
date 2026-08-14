# Pod

- k8s does not run containers directly on worker nodes; containers are encapsulated into a k8s object known as a Pod.
- A Pod is a single instance of an application.
- A Pod is the smallest object you can create in k8s.

## Commands

```bash
kubectl run nginx --image nginx

kubectl get pods

kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml

kubectl get pods -o wide
```
