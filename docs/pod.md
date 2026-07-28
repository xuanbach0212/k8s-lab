# pod

- k8s does not containers directly on worker nodes, the containers are encapsulated into k8s object known as pod
- pod is a single instance of application
- pod is the smallest object that you can create in k8s

### some cmd

kubtectl run nginx --image nginx
kubectl get pods

kubectl apply -f <https://k8s.io/examples/pods/simple-pod.yaml>

kubectl get pods -o wide
