# Postgres

Argo CD creates namespace `postgres` (`CreateNamespace=true`). The password Secret is still created by hand:

```bash
kubectl create secret generic postgres-secret \
  --from-literal=POSTGRES_PASSWORD=SuperSecretPassword \
  -n postgres
```

If the namespace does not exist yet, wait for Application `postgre` to sync, or run `kubectl create namespace postgres` first.
