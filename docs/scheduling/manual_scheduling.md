# Manual Scheduling

- Every Pod has a `spec.nodeName` field — normally the **kube-scheduler** sets it after picking the best node.
- If `nodeName` is empty, the Pod stays `Pending` forever (no scheduler running / no node assigned).
- You can bypass the scheduler and assign a node yourself, in 2 ways depending on whether the Pod already exists.

## 1. New Pod — set `nodeName` directly

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
  nodeName: node02
```

```bash
kubectl apply -f nginx-pod.yaml
kubectl get pod nginx -o wide   # STATUS should be Running, NODE = node02
```

## 2. Existing Pod (already `Pending`) — `nodeName` is immutable, use a Binding object instead

A Pod's `nodeName` can't be edited after creation. To assign a node the same way the scheduler does, create a `Binding` object and POST it to the Pod's `binding` subresource.

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```

```bash
curl --header "Content-Type:application/json" --request POST --data \
'{"apiVersion":"v1","kind":"Binding","metadata":{"name":"nginx"},"target":{"apiVersion":"v1","kind":"Node","name":"node02"}}' \
http://$SERVER/api/v1/namespaces/default/pods/nginx/binding
```
