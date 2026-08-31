# Traefik default TLS (wildcard)

Один wildcard-сертификат в `kube-system` используется Traefik для всех Ingress без `tls.secretName`.

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl apply -f infrastructure/traefik/default-tls-store.yaml
```

TLSStore **обязан** называться `default` и жить в `kube-system` (так устроен Traefik в k3s).
