# cert-manager + Let's Encrypt (DNS-01) via DuckDNS

Automatic certificate issuance in k3s with dns01 domain verification via the DuckDNS API
Since the `slavidich.duckdns.org` domain points to a local network, the http01 option is not suitable (my local network is my castle (c) Caesar).

In short, with http01, the CA asks you to host a file on a web server, then the CA verifies it. If it's OK, it issues a certificate. Since I have this all on my local network and don't want to make it public, I'll have to do it through dns01. With dns01, the verification is done through a DNS record. The CA asks you to create a TXT record at a address like 'acme-challenge.slavidich.duckdns.org' (and this record can only be created with a duckdns token). Let's Encrypt verifies that this domain is yours and issues a certificate.

## Scheme

```text
Certificate (wildcard, namespace kube-system)
    → cert-manager
        → ACME DNS-01 Challenge (_acme-challenge.slavidich.duckdns.org)
            → webhook cert-manager-webhook-duckdns
                → DuckDNS API (TXT)
            → Let's Encrypt issues wildcard-cert
        → Secret slavidich-wildcard-tls (kube-system)
            → Traefik TLSStore default (default TLS for services on my ingress)
            → Ingress whoami, echo, … (without secretName — Traefik use wildcard)

cert-manager Controller (backgtound)
    → monitors the expiration date
    → ~30 days before expiration - repeat DNS-01 and update Secret
```

---

## 1. Helm

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

The `extraArgs` parameter helps cert-manager check TXT records via public DNS (convenient at home, without your own resolver).

---

## 3. Webhook DuckDNS (mmontes)

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

## 4. Wildcard-cert (one for all services)

manifest: [`wildcard-certificate.yaml`](wildcard-certificate.yaml)

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl apply -f infrastructure/cert-manager/wildcard-certificate.yaml
```

cert-manager will create `CertificateRequest` → `Order` → `Challenge`, pass DNS-01 and put the certs to Secret `slavidich-wildcard-tls` in `kube-system`.

After wildcard, enable default TLS Traefik:

```bash
kubectl apply -f infrastructure/traefik/default-tls-store.yaml
```
