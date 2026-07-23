# kustomize-configs

Shared [kustomize transformer configurations](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/configurations/), packaged as [kustomize Components](https://kubectl.docs.kubernetes.io/guides/config_management/components/).

Kustomize's builtin transformers (`images:`, `nameReference`, `varReference`, …) only know about native Kubernetes kinds. When we introduce a CRD whose spec contains image references or ConfigMap/Secret name pointers, kustomize needs to be taught where those fields live: that's what the Components in this repository do.

The whole repo is released as a single unit under the tag `vX.Y.Z`.

## Consuming

Reference a specific Component at a pinned tag from a consumer `kustomization.yaml`:

```yaml
components:
  - github.com/AODocs/kustomize-configs//<crd>?ref=vX.Y.Z
```

**Always pin `?ref=`** to a released tag. Never use `main` - an upstream change would silently break every consumer.

## Available Components

### `javaapplication/`

Teaches kustomize about the `JavaApplication` CRD:

- `images:` rewrites `spec.image` (a full `repo:tag` reference).
- `nameReference` propagates hashed ConfigMap/Secret names from `configMapGenerator` / `secretGenerator` into `spec.envFrom[].name`.

If a CRD schema changes, update the Component **and cut a new release** - consumers pinning the previous tag stay safe.

## Contributing

1. Add a new Component under `<crd>/` with its `kustomization.yaml` (`kind: Component`) and transformer config file, or edit an existing one.
2. Document it in the "Available Components" section above.
3. Commit with conventional commits, e.g. `feat: add MyCRD component` or `fix(javaapplication): correct nameReference path`.
4. On merge to `main`, semantic-release cuts the next `vX.Y.Z` tag.
