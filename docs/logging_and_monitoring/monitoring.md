# Monitoring

## How to monitor a node or pod

- Metrics Server (built-in, powers `kubectl top`)
- Prometheus
- Elastic Stack
- Datadog
- Dynatrace

## Install metrics-server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

## Commands

```bash
kubectl top node
kubectl top pod
```
