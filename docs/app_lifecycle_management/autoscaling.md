# Autoscaling

- Automatically adjusts the number of running Pods (or their resource requests) based on observed load.
- Requires **Metrics Server** to be installed — HPA reads CPU/memory metrics from it (see [monitoring.md](../logging_and_monitoring/monitoring.md)).

## Types

- **HPA (Horizontal Pod Autoscaler)** — scales the number of Pod replicas up/down based on CPU, memory, or custom metrics.
- **VPA (Vertical Pod Autoscaler)** — adjusts CPU/memory requests and limits of existing Pods instead of changing replica count.
- **Cluster Autoscaler** — scales the number of nodes in the cluster up/down based on pending/unschedulable Pods.

## Sample HPA YAML

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

## Commands

```bash
# imperative way
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=2 --max=10

# declarative way
kubectl apply -f nginx-hpa.yaml

kubectl get hpa

kubectl describe hpa nginx-hpa
```

## Installing VPA

Unlike HPA, VPA is **not built into kube-controller-manager** — it must be installed separately from the `kubernetes/autoscaler` repo.

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler

./hack/vpa-up.sh
```

`vpa-up.sh` checks out the pinned release tag, then calls `vpa-process-yamls.sh apply`, which applies everything under `deploy/` **in order** — no separate manual step needed for CRD/RBAC:

1. **CRD** — `vpa-v1-crd-gen.yaml` (registers the `VerticalPodAutoscaler` resource)
2. **RBAC** — `vpa-rbac.yaml` (ServiceAccounts, ClusterRoles, ClusterRoleBindings for each component)
3. **Deployments** — `updater-deployment`, `recommender-deployment`, `admission-controller-deployment`
4. **Service** — `admission-controller-service`

The admission-controller step also runs `gencerts.sh` right before applying its Deployment, to generate the TLS certs the webhook needs.

Verify the components are running:

```bash
kubectl get pods -n kube-system | grep vpa
```

Uninstall:

```bash
./hack/vpa-down.sh
```

**Prerequisites:**

- `openssl` on the machine running the script (used to generate the admission-controller's TLS certs).
- **Metrics Server** running in the cluster (the recommender component reads resource usage from it).

## Sample VPA YAML

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: nginx-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  updatePolicy:
    updateMode: "Auto"
```

`updatePolicy.updateMode` options:

- `Off` — only produce recommendations, don't apply them automatically.
- `Initial` — apply recommendations only when a Pod is created.
- `Recreate` — evict and recreate Pods with updated resources.
- `Auto` — same as `Recreate` today (may use in-place resize in future k8s versions).

## Commands

```bash
kubectl apply -f nginx-vpa.yaml

kubectl get vpa

kubectl describe vpa nginx-vpa
```
