# cert-manager + let's encrypt (dns-01) via duckdns

automatic certificates on k3s using dns-01 through the duckdns api.

`slavidich.duckdns.org` points at a private network, so http-01 does not work. let's encrypt would have to fetch a challenge file from a public web server, and this network is not public. dns-01 proves ownership by creating a txt record `_acme-challenge.slavidich.duckdns.org` via the duckdns token. let's encrypt checks that record and issues the cert.

flow:

```text
certificate (wildcard, namespace kube-system)
    -> cert-manager
        -> acme dns-01 challenge (_acme-challenge.slavidich.duckdns.org)
            -> webhook cert-manager-webhook-duckdns
                -> duckdns api (txt)
            -> let's encrypt issues the wildcard cert
        -> secret slavidich-wildcard-tls (kube-system)
            -> traefik tlsstore default (https for ingress without tls.secretName)
            -> ingress whoami, echo, ...

cert-manager (background)
    -> watches expiry
    -> ~30 days before expiry: repeat dns-01 and update the secret
```

helm repos:

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo add mmontes https://mmontes11.github.io/charts
helm repo update
```

install cert-manager (jetstack):

```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.18.2 \
  --set crds.enabled=true \
  --set 'extraArgs={--dns01-recursive-nameservers-only=true,--dns01-recursive-nameservers=8.8.8.8:53\,1.1.1.1:53}'
```

`extraArgs` makes cert-manager check txt records via public dns. convenient at home without your own recursive resolver.

duckdns webhook (mmontes):

```bash
helm install cert-manager-webhook-duckdns mmontes/cert-manager-webhook-duckdns \
  --namespace cert-manager \
  --set duckdns.token='token' \
  --set clusterIssuer.email='@mail' \
  --set clusterIssuer.production.create=true \
  --set clusterIssuer.staging.create=false \
  --set logLevel=2
```

wildcard certificate, one for all services. manifest is `wildcard-certificate.yaml`.

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl apply -f k3s/1.36.4/infra/cert-manager/wildcard-certificate.yaml
```

cert-manager creates a `CertificateRequest`, then an `Order`, then a `Challenge`, completes dns-01, and stores the cert in secret `slavidich-wildcard-tls` in `kube-system`.

then enable traefik default tls:

```bash
kubectl apply -f k3s/1.36.4/infra/traefik/default-tls-store.yaml
```
