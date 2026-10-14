# keycloak

helm chart `keycloakx` (codecentric): keycloak 26, no bundled postgres. argo cd application `keycloak` installs chart 7.3.2 with `values.yaml` from this folder. users live in keycloak itself. apps (gitlab later) talk oidc to it.

create the database first, see `infra/postgre/README.md`. then the secret. argo cd creates namespace `keycloak`. the secret is still created by hand. `db-password` must match the role you created in postgres.

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl create namespace keycloak
kubectl create secret generic keycloak \
  --from-literal=admin-user=admin \
  --from-literal=admin-password='SuperSecretPassword' \
  --from-literal=db-password='SuperSecretPassword' \
  -n keycloak
```

bootstrap admin (`KC_BOOTSTRAP_ADMIN_*`) is only read on the first start, while the database is empty. changing the secret later does not change the admin password.

url: `https://keycloak.slavidich.duckdns.org`. after login, create a realm for apps. do not put gitlab and the rest in `master`.
