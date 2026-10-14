# Traefik default TLS (wildcard)

One wildcard certificate in `kube-system` is used by Traefik for every Ingress that has no `tls.secretName`.

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl apply -f k3s/1.36.4/infra/traefik/default-tls-store.yaml
```

The TLSStore **must** be named `default` and live in `kube-system` (k3s Traefik requirement).
