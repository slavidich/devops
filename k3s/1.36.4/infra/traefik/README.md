# traefik default tls (wildcard)

one wildcard certificate in `kube-system` is used by traefik for every ingress that has no `tls.secretName`. that is why whoami, echo, grafana and argo cd do not each carry their own cert secret.

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl apply -f k3s/1.36.4/infra/traefik/default-tls-store.yaml
```

the tlsstore has to be named `default` and live in `kube-system`. that is how traefik in k3s picks the default certificate. any other name and it is ignored.
