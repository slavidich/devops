# Install Argo CD

Bootstrap only. After this, GitOps is started with `argocd/root.yaml`.

```bash
kubectl create namespace argocd
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

helm upgrade --install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  -f values.yaml
```
