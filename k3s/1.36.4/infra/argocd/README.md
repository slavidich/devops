# install argo cd

bootstrap only. this is the helm install of argo cd itself. gitops starts afterwards with `argocd/root.yaml` from the folder next to this one, not with another helm command.

```bash
kubectl create namespace argocd
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

helm upgrade --install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  -f values.yaml
```

`values.yaml` turns on ingress at `argocd.slavidich.duckdns.org` and `server.insecure`, because tls terminates on traefik.

unlike grafana and postgres, you do not create the admin secret by hand. helm writes the initial password into `argocd-initial-admin-secret`. user is `admin`:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
echo
```
