# Rolling Updates and Rollbacks

## Strategy

- `Recreate` — kill all Pods, then create new ones.
- `RollingUpdate` (default) — create a new ReplicaSet and gradually scale it up while scaling the old ReplicaSet down.

## Rollout commands

```bash
# get status
kubectl rollout status deployment/nginx

# get history
kubectl rollout history deployment/nginx

# rollback
kubectl rollout undo deployment/nginx
```
