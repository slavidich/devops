# Homelab DevOps

Personal homelab on k3s: infrastructure as code on GitHub, GitOps with Argo CD, CI via GitLab in the cluster.

---

## Architecture

```text
Internet
   │
   ▼
DuckDNS  (*.slavidich.duckdns.org)
   │
   ▼
Traefik  (Ingress / TLS)
   │
   ├── Argo CD     ← syncs this repo → cluster
   ├── apps        
   ├── Postgres
   ├── GitLab + runners (only for ci/cd pipelines, code will be here)
   └── Prometheus + Grafana
```

---

## Repository layout

```text
k3s/
  README.md                 # k3s install notes
  1.36.4/
    README.md               # bootstrap order
    infra/                  # cluster infrastructure (Helm values, manifests)
      argocd/
      cert-manager/
      traefik/
      postgre/
      prometheus/
    apps/                   # application manifests
      whoami/
    argocd/                 # Argo CD Applications
      applications/
      infra/
```