# app of apps

`root.yaml` is the only application you apply by hand. after that argo cd watches `applications/` and keeps the cluster in line with git.

do not put `root.yaml` inside `applications/`. recurse would pick it up and the root app would try to manage itself.

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl apply -f k3s/1.36.4/argocd/root.yaml
```

root clones this repo, takes every yaml under `applications/` (`directory.recurse: true`) and creates those objects in namespace `argocd`. folder names do not matter. first come appprojects (`sync-wave: "-1"`), then child applications. root only creates tickets. each child then pulls its own `source.path` (or helm chart) and deploys into its own `destination.namespace`. path is where to read, namespace is where to put it. they are not tied to the folder name.

appproject is the permission layer: which git/helm repos an application may use, and which namespaces it may write to. `infra` can write to `monitoring`, `postgres`, `keycloak`, `gitlab`, `kube-system`. `apps` can write to `demo`. a child outside that list will not sync. to add a portfolio app later, put an application yaml under `applications/apps/` and add its namespace to project `apps`.
