To view the logs

kubectl logs -f event-simulator-pod

If there are multiple containers in a pod then you must specify the name of the container explicitly in the command.

kubectl logs -f <pod-name> <container-name>
kubectl logs -f even-simulator-pod event-simulator
