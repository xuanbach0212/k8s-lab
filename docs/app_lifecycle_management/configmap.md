# ConfigMap

- Used to store non-confidential data as key-value pairs.

## The imperative way

```bash
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod

# alternative: load from a file
kubectl create configmap app-config --from-file=app_config.properties
```

## The declarative way

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

## Inject into a Pod (all keys as env vars)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
            name: app-config
```

## Inject a single env var

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

## Inject as a volume

```yaml
volumes:
  - name: app-config-volume
    configMap:
      name: app-config
```
