# Kustomize Transformers

Built-in transformers change **all resources** listed in a `kustomization.yaml` at once — no patch needed.

## Common transformers

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - api-depl.yaml
  - api-service.yaml

labels:
  - pairs:
      org: KodeKloud
    includeSelectors: false   # true = also add to selectors (careful: selectors are immutable)

commonAnnotations:
  branch: master

namePrefix: KodeKloud-       # api-deployment → KodeKloud-api-deployment
nameSuffix: -dev             # → KodeKloud-api-deployment-dev

namespace: lab               # set metadata.namespace on every resource
```

- Name changes are propagated to references — e.g. a Deployment using a renamed ConfigMap is updated too.
- `commonLabels` is the older form of `labels` — it **always** adds to selectors. Deprecated, prefer `labels`.
- Transformers apply at the level of the `kustomization.yaml` they're in:
  - in `k8s/kustomization.yaml` → every resource under `k8s/`
  - in `k8s/api/kustomization.yaml` → only `api/` resources

```yaml
# k8s/api/kustomization.yaml
resources:
  - api-depl.yaml
labels:
  - pairs:
      feature: api      # api resources only

# k8s/kustomization.yaml
resources:
  - api/
  - db/
labels:
  - pairs:
      org: KodeKloud    # everything
```

## Image transformer

- Change container images without touching the Deployment yaml.
- Matches by **image name** (`image:` field), **not** container name.

```yaml
images:
  - name: nginx          # find every container using image "nginx"
    newName: haproxy     # replace the image
    newTag: "2.4"        # and/or set the tag
```

Before / after:

```yaml
containers:
  - name: web
    image: nginx         # → haproxy:2.4
```

- Only `newTag` → keeps the image, changes the tag (common for per-env versions).
- `digest: sha256:...` → pin by digest instead of tag.

```bash
kustomize edit set image nginx=nginx:1.27   # update the images field from CLI (handy in CI)
```
