# Bootstrap

```bash
export KUBECONFIG=~/.kube/k3s-config
```

Install in this order. These pieces are **not** managed by Argo CD (chicken-and-egg: Argo CD needs Ingress/TLS to be usable).

1. `infra/cert-manager` — Helm + `wildcard-certificate.yaml` (Secret in `kube-system`)
2. `infra/traefik` — TLSStore `default` for the wildcard cert
3. `infra/argocd` — Helm with `values.yaml`

Then start GitOps:

```bash
kubectl apply -f argocd/root.yaml
```

Secrets Argo CD does not create:

- `monitoring`: Secret `grafana-admin`
- `postgres`: Secret `postgres-secret`
