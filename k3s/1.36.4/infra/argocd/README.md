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

## keycloak

realm `users`, client `argocd`. the client is confidential. redirect uri is `https://argocd.slavidich.duckdns.org/auth/callback`. client role `argocd-admin-access` is assigned to groups through role mapping. scope `argocd-dedicated` has a User Client Role mapper that puts this role into `resource_access.argocd.roles`, so the oidc config does not request an extra scope for it.

the client secret is not in git. patch it into the secret helm already created. replace the value with the secret from the client credentials tab:

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl -n argocd patch secret argocd-secret --type merge \
  -p '{"stringData":{"oidc.keycloak.clientSecret":"<client-secret>"}}'
```

then apply `values.yaml`. argo cd is bootstrap, this folder is not an argo cd application, so a commit alone does nothing:

```bash
helm upgrade --install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  -f values.yaml
```

`oidc.config` points `clientSecret` at `$oidc.keycloak.clientSecret`, which is that key. rbac reads `resource_access.argocd.roles`, so holders of client role `argocd-admin-access` get `role:admin`. everyone else in the realm gets no role. the local `admin` user still works.

login at `https://argocd.slavidich.duckdns.org` with the button `LOG IN VIA KEYCLOAK`.
