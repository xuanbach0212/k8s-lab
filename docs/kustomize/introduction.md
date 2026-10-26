# Kustomize

- Customizes plain k8s yaml **per environment** (dev / staging / prod) without copying files and without a template language.
- Problem: same app in 3 envs, only a few fields differ (replicas, image tag, namespace). Copying the yaml into 3 folders → drift, every change done 3 times.
- Kustomize idea:
  - **Base** — config shared by all envs (plain, valid k8s yaml).
  - **Overlays** — per-env changes applied on top of the base (see [overlays_and_components.md](overlays_and_components.md)).
- Built into `kubectl` (`kubectl kustomize`, `kubectl apply -k`) — the bundled version may lag behind the standalone `kustomize` binary.

## Kustomize vs Helm

| | Kustomize | [Helm](../helm/introduction.md) |
|---|---|---|
| Input | Plain yaml — always valid, can `kubectl apply` directly | Go templates (`{{ .Values.x }}`) — not valid yaml until rendered |
| Customize by | Patches + transformers on top of a base | Values filling templates |
| Logic | None (no loops, conditionals) | Templating functions, `if`, `range`, ... |
| Packaging / releases | No — just generates yaml | Charts, repos, release history, rollback |
| Install | Built into kubectl | Separate binary |

- Kustomize = simple, readable, good for your own apps.
- Helm = full package manager, good for distributing apps / third-party software.

## Install

```bash
# standalone binary (optional — kubectl already includes it)
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash

kustomize version
```

## kustomization.yaml

- Every kustomize directory needs a `kustomization.yaml` — lists **which resources** to manage and **what changes** to make.

```
k8s/
├── kustomization.yaml
├── nginx-deployment.yaml
└── nginx-service.yaml
```

`kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# resources to manage
resources:
  - nginx-deployment.yaml
  - nginx-service.yaml

# customizations to apply to all of them
labels:
  - pairs:
      company: KodeKloud
```

## Commands

```bash
kustomize build k8s/              # render final yaml to stdout — does NOT apply
kubectl kustomize k8s/            # same, using kubectl's built-in kustomize

kustomize build k8s/ | kubectl apply -f -
kubectl apply -k k8s/             # same in one step

kustomize build k8s/ | kubectl delete -f -
kubectl delete -k k8s/
```

- `build` only prints — always check its output before applying.

## Managing directories

- Resources can be spread over sub-directories. Instead of listing every file from the root:

```yaml
# k8s/kustomization.yaml — messy
resources:
  - api/api-depl.yaml
  - api/api-service.yaml
  - db/db-depl.yaml
  - db/db-service.yaml
```

- Give each sub-directory its own `kustomization.yaml` and reference the **directory**:

```
k8s/
├── kustomization.yaml
├── api/
│   ├── kustomization.yaml
│   ├── api-depl.yaml
│   └── api-service.yaml
└── db/
    ├── kustomization.yaml
    ├── db-depl.yaml
    └── db-service.yaml
```

```yaml
# k8s/kustomization.yaml
resources:
  - api/
  - db/

# k8s/api/kustomization.yaml
resources:
  - api-depl.yaml
  - api-service.yaml
```

```bash
kubectl apply -k k8s/   # applies everything in api/ and db/
```

## Generators

- Generate ConfigMaps/Secrets from files or literals (instead of writing the yaml by hand):

```yaml
configMapGenerator:
  - name: app-config
    literals:
      - LOG_LEVEL=debug
    files:
      - application.properties

secretGenerator:
  - name: db-creds
    literals:
      - password=s3cr3t
```

- Generated names get a **content hash suffix**: `app-config-7g8h9k2m4t`. Kustomize updates every reference to it in Deployments/Pods.
- Changing the content → new name → Deployment spec changes → **pods roll automatically**. A plain ConfigMap edit would not restart pods (see [configmap.md](../app_lifecycle_management/configmap.md)).
- Disable with `generatorOptions: { disableNameSuffixHash: true }`.
