# redis

one redis 7.2 instance, many logical databases (`SELECT 0` .. `SELECT 15`). do not dump every app into db `0` without a prefix. argo cd application `redis` deploys the statefulset, pvc and services. namespace `redis` is created by argo cd (`CreateNamespace=true`). the password secret is still created by hand.

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl create namespace redis
kubectl create secret generic redis-secret \
  --from-literal=REDIS_PASSWORD=SuperSecretPassword \
  -n redis
```

if the namespace does not exist yet, wait for application `redis` to sync, or create it first as above. without the secret the pod will not start.

there is a headless service `redis` for in-cluster dns, and a nodeport service `redis-external` on `32379` if you want to reach it from outside the cluster. for day to day use inside k3s, headless is enough.

from the cluster apps talk to:

- host `redis.redis.svc.cluster.local`
- port `6379`
- password from `redis-secret`, key `REDIS_PASSWORD`

put that password in the app's own secret in the app namespace. pods cannot read `redis-secret` from namespace `redis`.
