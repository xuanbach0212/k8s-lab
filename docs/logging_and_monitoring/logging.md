## how to monitor the node or pod

Metric Server (top - built-in)
Prometheus
Elastic Stack
Datadog
Dynatrace

### install metrics-server

kubectl apply -f <https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml>

### CMD

kubectl top node
kubectl top pod
