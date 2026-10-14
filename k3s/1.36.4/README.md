# bootstrap

```bash
export KUBECONFIG=~/.kube/k3s-config
```

install in this order. these pieces are not managed by argo cd. chicken and egg: argo cd needs ingress and tls before it is usable, and it cannot install itself.

first `infra/cert-manager`: helm plus `wildcard-certificate.yaml`. the tls secret lands in `kube-system`.

then `infra/traefik`: tlsstore `default` so traefik uses that wildcard cert for https.

then `infra/argocd`: helm with `values.yaml`.

after that start gitops:

```bash
kubectl apply -f argocd/root.yaml
```

what happens next is described in `argocd/README.md`.

secrets argo cd does not create. grafana wants `grafana-admin` in `monitoring`. postgres wants `postgres-secret` in `postgres`. make those by hand, see the readmes next to prometheus and postgres.
