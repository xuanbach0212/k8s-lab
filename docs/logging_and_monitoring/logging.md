# Logging

## View logs

```bash
kubectl logs -f event-simulator-pod
```

If there are multiple containers in a Pod, you must specify the container name explicitly:

```bash
kubectl logs -f <pod-name> <container-name>

# example
kubectl logs -f event-simulator-pod event-simulator
```
