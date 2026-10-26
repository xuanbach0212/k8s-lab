# Kustomize Patches

- [Transformers](transformers.md) change every resource. **Patches** target **specific** objects and change specific fields.
- A patch needs:
  - **target** — which object(s): `kind`, `name`, `namespace`, `labelSelector`, ...
  - the change itself — inline (`patch:`) or in a file (`path:`)
- All patches go under `patches:`. Old fields `patchesStrategicMerge` / `patchesJson6902` are deprecated.

## 2 patch types

### JSON 6902 patch

- List of operations: `op` (`add` / `remove` / `replace` / `move` / `copy` / `test`), `path`, `value`.
- Needs a `target`.

```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
```

- Path uses `/` per level; list items by **index** (`/spec/template/spec/containers/0`).

### Strategic merge patch

- Looks like a partial k8s object — only the fields you want to change. Kustomize merges it into the original.
- Identifies the target from `kind` + `metadata.name` inside the patch.

```yaml
patches:
  - patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: api-deployment
      spec:
        replicas: 5
```

- Easier to read; JSON 6902 is better for precise edits (e.g. by list index).

## Inline vs separate file

```yaml
patches:
  - path: replica-patch.yaml          # strategic merge in a file
  - path: replica-patch-json.yaml     # JSON 6902 in a file — still needs target
    target:
      kind: Deployment
      name: api-deployment
```

## Dictionary (map) examples

Original:

```yaml
template:
  metadata:
    labels:
      component: api
```

| Goal | JSON 6902 | Strategic merge |
|------|-----------|-----------------|
| Replace key | `op: replace`, `path: /spec/template/metadata/labels/component`, `value: web` | `labels: { component: web }` |
| Add key | `op: add`, `path: /spec/template/metadata/labels/org`, `value: KodeKloud` | `labels: { org: KodeKloud }` (merged, existing keys kept) |
| Remove key | `op: remove`, `path: /spec/template/metadata/labels/org` | `labels: { org: null }` |

## List examples

Original:

```yaml
template:
  spec:
    containers:
      - name: nginx
        image: nginx
```

**Replace** an item:

```yaml
# JSON 6902
- op: replace
  path: /spec/template/spec/containers/0     # by index
  value:
    name: haproxy
    image: haproxy

# Strategic merge — containers are merged by "name" key
spec:
  template:
    spec:
      containers:
        - name: nginx
          image: haproxy
```

**Add** an item:

```yaml
# JSON 6902
- op: add
  path: /spec/template/spec/containers/-     # "-" = append to end
  value:
    name: haproxy
    image: haproxy

# Strategic merge — new name → appended
spec:
  template:
    spec:
      containers:
        - name: haproxy
          image: haproxy
```

**Remove** an item:

```yaml
# JSON 6902
- op: remove
  path: /spec/template/spec/containers/1

# Strategic merge
spec:
  template:
    spec:
      containers:
        - $patch: delete
          name: database
```
