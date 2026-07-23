# kustomize-configs

Shared [kustomize transformer configurations](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/configurations/).

Kustomize's builtin transformers (`images:`, `nameReference`, `varReference`, …) only know about native Kubernetes kinds. When we introduce a CRD whose spec contains
image references or ConfigMap/Secret name pointers, kustomize needs to be taught where those fields live: that's what the files in this repository do.

The whole repo is released as a single unit under the tag `vX.Y.Z`.

## Consuming

Reference a specific file at a pinned tag from a consumer `kustomization.yaml`.
The recommended URL uses `raw.githubusercontent.com` (single HTTPS GET, no git clone):

```yaml
configurations:
  - https://raw.githubusercontent.com/AODocs/kustomize-configs/vX.Y.Z/<file>.yaml
```

Git-URL form also works (heavier - kustomize does a `git clone`):

```yaml
configurations:
  - github.com/AODocs/kustomize-configs//<file>.yaml?ref=vX.Y.Z
```

**Always pin `?ref=` / the tag segment** to a released tag. Never use `main` - an upstream change would silently break every consumer.

## Runtime dependency

Every consumer reconciliation (Flux, ArgoCD) that runs `kustomize build` will fetch this URL. This adds a runtime dependency on GitHub's HTTPS availability during reconciliations.
It's the same trust as your app repo's `GitRepository` so the marginal risk is small - but be aware: a `raw.githubusercontent.com` outage will freeze new deploys until it recovers.

If you need to eliminate this dependency, vendor the file into your app repo (git submodule or a copy bumped via Renovate PRs).

## Available configs

### `javaapplication.yaml`

Teaches kustomize about the `JavaApplication` CRD:

- `images:` rewrites `spec.image` (a full `repo:tag` reference).
- `nameReference` propagates hashed ConfigMap/Secret names from `configMapGenerator`
  / `secretGenerator` into `spec.envFrom[].name`.


If a CRD schema changes, update the corresponding config **and cut a new release** - consumers pinning the previous tag stay safe.

## Contributing

1. Add or edit a `<crd>.yaml` at the repo root.
2. Document it in the "Available configs" section above.
3. Commit with conventional commits, e.g. `feat: add MyCRD transformer config`
   or `fix(javaapplication): correct nameReference path`.
4. On merge to `main`, semantic-release cuts the next `vX.Y.Z` tag.
