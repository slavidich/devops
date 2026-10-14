# postgres

argo cd application `postgre` deploys the statefulset, pvc and services. namespace `postgres` is created by argo cd (`CreateNamespace=true`). the password secret is still created by hand.

```bash
kubectl create secret generic postgres-secret \
  --from-literal=POSTGRES_PASSWORD=SuperSecretPassword \
  -n postgres
```

if the namespace does not exist yet, wait for application `postgre` to sync, or run `kubectl create namespace postgres` first. without the secret the pod will not start.

there is a headless service `postgres` for the statefulset dns, and a nodeport service `postgres-external` on `32432` if you want to reach it from outside the cluster. for day to day use inside k3s, clusterip/headless is enough.

## databases

one postgres instance, many databases. do not put apps in `postgres-db` and do not connect them as `admin`.

### keycloak

keycloak needs its own database and role before the chart goes up. create them by hand, argo cd does not.


```sql
CREATE USER keycloak WITH PASSWORD 'SuperSecretPassword';
CREATE DATABASE keycloak OWNER keycloak;
```

same sql via kubectl 

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl exec -n postgres postgres-0 -- \
  psql -U admin -d postgres-db \
  -c "CREATE USER keycloak WITH PASSWORD 'SuperSecretPassword';" \
  -c "CREATE DATABASE keycloak OWNER keycloak;"
```


from the cluster keycloak talks to:

- host `postgres.postgres.svc.cluster.local`
- port `5432`
- database `keycloak`
- user `keycloak`

put that password in secret `keycloak`, key `db-password`, in namespace `keycloak`. see `infra/keycloak/README.md`. this instance is shared; the database is not. gitlab later gets the same treatment: another database, another role.
