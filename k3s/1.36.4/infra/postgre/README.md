# postgres

argo cd application `postgre` deploys the statefulset, pvc and services. namespace `postgres` is created by argo cd (`CreateNamespace=true`). the password secret is still created by hand.

```bash
kubectl create secret generic postgres-secret \
  --from-literal=POSTGRES_PASSWORD=SuperSecretPassword \
  -n postgres
```

if the namespace does not exist yet, wait for application `postgre` to sync, or run `kubectl create namespace postgres` first. without the secret the pod will not start.

there is a headless service `postgres` for the statefulset dns, and a nodeport service `postgres-external` on `32432` if you want to reach it from outside the cluster. for day to day use inside k3s, clusterip/headless is enough.
