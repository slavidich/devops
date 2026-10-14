# cert-manager + Let's Encrypt (DNS-01) via DuckDNS

Automatic certificates on k3s using DNS-01 through the DuckDNS API.

`slavidich.duckdns.org` points at a private network, so HTTP-01 does not work: Let's Encrypt cannot fetch a challenge file from a host that is not public. DNS-01 proves ownership by creating a TXT record (`_acme-challenge.slavidich.duckdns.org`) via the DuckDNS token.

## Flow

```text
Certificate (wildcard, namespace kube-system)
    → cert-manager
        → ACME DNS-01 Challenge (_acme-challenge.slavidich.duckdns.org)
            → webhook cert-manager-webhook-duckdns
                → DuckDNS API (TXT)
            → Let's Encrypt issues the wildcard cert
        → Secret slavidich-wildcard-tls (kube-system)
            → Traefik TLSStore default (HTTPS for Ingress without tls.secretName)
            → Ingress whoami, echo, …

cert-manager (background)
    → watches expiry
    → ~30 days before expiry: repeat DNS-01 and update the Secret
```

---

## 1. Helm repos

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo add mmontes https://mmontes11.github.io/charts
helm repo update
```

---

## 2. Install cert-manager (jetstack)

```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.18.2 \
  --set crds.enabled=true \
  --set 'extraArgs={--dns01-recursive-nameservers-only=true,--dns01-recursive-nameservers=8.8.8.8:53\,1.1.1.1:53}'
```

`extraArgs` makes cert-manager check TXT records via public DNS (useful at home without a local recursive resolver).

---

## 3. DuckDNS webhook (mmontes)

```bash
helm install cert-manager-webhook-duckdns mmontes/cert-manager-webhook-duckdns \
  --namespace cert-manager \
  --set duckdns.token='token' \
  --set clusterIssuer.email='@mail' \
  --set clusterIssuer.production.create=true \
  --set clusterIssuer.staging.create=false \
  --set logLevel=2
```

---

## 4. Wildcard certificate (one for all services)

Manifest: [`wildcard-certificate.yaml`](wildcard-certificate.yaml)

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl apply -f k3s/1.36.4/infra/cert-manager/wildcard-certificate.yaml
```

cert-manager creates `CertificateRequest` → `Order` → `Challenge`, completes DNS-01, and stores the cert in Secret `slavidich-wildcard-tls` in `kube-system`.

Then enable Traefik default TLS:

```bash
kubectl apply -f k3s/1.36.4/infra/traefik/default-tls-store.yaml
```
