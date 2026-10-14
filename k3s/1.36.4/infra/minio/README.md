# minio

s3-compatible object storage. one instance, many buckets. do not dump every app into one bucket. argo cd application `minio` deploys the statefulset, pvc, service and ingress. namespace `minio` is created by argo cd (`CreateNamespace=true`). the root secret is still created by hand. password must be at least 8 characters.

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl create namespace minio
kubectl create secret generic minio-secret \
  --from-literal=root-user=admin \
  --from-literal=root-password='SuperSecretPassword' \
  -n minio
```

if the namespace does not exist yet, wait for application `minio` to sync, or create it first as above. without the secret the pod will not start.

urls:

- console `https://minio.slavidich.duckdns.org`
- api `https://s3.slavidich.duckdns.org`

from the cluster apps talk to the api, not the console:

- endpoint `http://minio.minio.svc.cluster.local:9000`
- path-style access, tls off
- user / password from `minio-secret`

put those credentials in the app's own secret in the app namespace. pods cannot read `minio-secret` from namespace `minio`.

## buckets

create a bucket per app (or per purpose: uploads, backups) before the app goes up. argo cd does not.

from a laptop with the [minio client](https://min.io/docs/minio/linux/reference/minio-mc.html):

```bash
mc alias set homelab https://s3.slavidich.duckdns.org admin SuperSecretPassword
mc mb homelab/app-uploads
```

same from the cluster, one-shot pod:

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl -n minio run mc --rm -it --restart=Never --image=minio/mc:RELEASE.2025-08-13T08-35-41Z -- \
  /bin/sh -c 'mc alias set local http://minio:9000 admin SuperSecretPassword && mc mb local/app-uploads'
```
