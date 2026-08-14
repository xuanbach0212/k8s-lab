# Multi-container Pods

- **Regular init containers** — run to check or set up something, then stop before the main app starts.
- **Sidecar containers** — an init container (e.g. a log agent) that starts before the main app, then keeps running alongside it.
- **Co-located containers** — multiple containers sharing a single Pod.

## Sidecar

`spec.initContainers` with `restartPolicy: Always` is what differs a Kubernetes-native sidecar from a classic init container — it ensures the container is always kept up.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
  initContainers:
    - name: log-shipper
      image: busybox
      command: ["setup-log-shipper.sh"]
      restartPolicy: Always
```
