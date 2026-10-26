# Overlays and Components

## Overlays

- **Base** = shared config + defaults. **Overlay** = per-env directory that pulls in the base and changes what's different.

```
k8s/
├── base/
│   ├── kustomization.yaml
│   ├── nginx-depl.yaml
│   ├── service.yaml
│   └── redis-depl.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── config-map.yaml
    ├── stg/
    │   ├── kustomization.yaml
    │   └── config-map.yaml
    └── prod/
        ├── kustomization.yaml
        ├── config-map.yaml
        └── grafana-depl.yaml     # resource only prod has
```

`base/kustomization.yaml`:

```yaml
resources:
  - nginx-depl.yaml
  - service.yaml
  - redis-depl.yaml
```

`base/nginx-depl.yaml` — defaults:

```yaml
spec:
  replicas: 1
```

`overlays/prod/kustomization.yaml`:

```yaml
resources:
  - ../../base           # pull in the base (old field "bases:" is deprecated)
  - grafana-depl.yaml    # extra resource only for prod

namespace: prod

patches:
  - target:
      kind: Deployment
      name: nginx-deployment
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 3

images:
  - name: nginx
    newTag: "1.27"
```

```bash
kubectl kustomize k8s/overlays/dev    # preview dev
kubectl apply -k k8s/overlays/prod    # deploy prod
```

- Overlays can add new resources, not just modify existing ones.
- The folder names (`base`, `overlays`) are just convention — what matters is the `resources:` path.

## Components

- Problem: an optional feature (e.g. caching, external DB) needed in **some** envs but not all.
  - Put it in base → every env gets it.
  - Copy it into each overlay → duplicated config again.
- **Component** = reusable bundle of resources + patches + generators that overlays can opt into.

```
k8s/
├── base/
├── components/
│   ├── caching/
│   │   ├── kustomization.yaml
│   │   ├── deployment-patch.yaml
│   │   └── redis-depl.yaml
│   └── db/
│       ├── kustomization.yaml
│       ├── deployment-patch.yaml
│       └── postgres-depl.yaml
└── overlays/
    ├── dev/            # uses db
    ├── premium/        # uses caching + db
    └── standalone/     # uses caching
```

`components/db/kustomization.yaml` — note `kind: Component`, `v1alpha1`:

```yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

resources:
  - postgres-depl.yaml

secretGenerator:
  - name: postgres-cred
    literals:
      - password=postgres123

patches:
  - path: deployment-patch.yaml
```

`components/db/deployment-patch.yaml` — wire the app to the DB:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - name: api
          env:
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-cred
                  key: password
```

`overlays/premium/kustomization.yaml`:

```yaml
resources:
  - ../../base

components:
  - ../../components/caching
  - ../../components/db
```
