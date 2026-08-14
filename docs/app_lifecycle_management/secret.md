# Secret

- Used to store sensitive data such as passwords, keys, and tokens.

## Enable encryption at rest

<https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/>

## Use a secret manager

Prefer a CSI driver + external secret manager over plain Secrets: <https://www.youtube.com/watch?v=MTnQW9MxnRI>

## Commands

```bash
kubectl create secret generic app-secret --from-literal=DB_Host=mysql --from-literal=DB_User=root --from-literal=DB_Password=paswrd

kubectl create secret generic app-secret --from-file=app_secret.properties
```

## Ways to inject into Pods

Load all keys as env vars:

```yaml
envFrom:
  - secretRef:
      name: app-secret
```

Single env var:

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: APP_COLOR
```

As a volume:

```yaml
volumes:
  - name: app-secret-volume
    secret:
      secretName: app-secret
```
