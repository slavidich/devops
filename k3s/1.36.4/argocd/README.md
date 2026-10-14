# App of Apps

`root.yaml` is the only Application applied by hand. Argo CD then syncs everything under `applications/` (`directory.recurse: true`).

Do **not** put `root.yaml` inside `applications/`, or the root app will try to manage itself.

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl apply -f k3s/1.36.4/argocd/root.yaml
```

## Projects

| AppProject | Allowed sources | Allowed namespaces |
|---|---|---|
| `default` | used only by `root` (it creates the other projects) | `argocd` |
| `infra` | this repo + infra Helm charts | `monitoring`, `postgres`, `gitlab` |
| `apps` | this repo + `github.com/slavidich/*` | `demo` (add a destination per new app) |

Child Applications must set `spec.project` to `infra` or `apps`. Namespace comes from `spec.destination.namespace` plus `CreateNamespace=true`.

## Adding a portfolio app later

1. Keep source code in its own GitHub repo (`deploy/` manifests, no `metadata.namespace`).
2. Add an Application YAML under `applications/apps/`.
3. Add that app's namespace to AppProject `apps` destinations.
