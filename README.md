# Homelab DevOps

Personal homelab on k3s. This repository is the **platform**: infrastructure as code, GitOps with Argo CD, later CI and a container registry via GitLab in the cluster.

Application **source code** lives in separate GitHub repositories. This repo does not host product code.

---

## How it works

Two layers:

1. **Bootstrap** (once, by hand): cert-manager, Traefik TLS, Argo CD itself. GitOps cannot start until these exist.
2. **GitOps**: Argo CD watches this repo. You change YAML on GitHub → Argo CD applies it to the cluster.

Argo CD is driven by an **App of Apps** tree:

- `argocd/root.yaml` is applied once. It points at `argocd/applications/` (recurse).
- That folder holds **AppProject** objects (`infra`, `apps`) and child **Application** objects.
- Each child Application points at either this repo (infra, demo apps) or, later, another GitHub repo (portfolio apps).
- `spec.destination.namespace` is the single source of truth for where resources land. Manifests do not set `metadata.namespace`. Argo CD creates the namespace (`CreateNamespace=true`).

```text
GitHub (this repo)
        │
        ▼
   Argo CD  ── root ──► applications/
                          ├── projects/   AppProject infra, apps
                          ├── infra/      Prometheus, Postgres, …
                          ├── demo/       whoami, echo
                          └── apps/       pointers to other GitHub repos
        │
        ▼
      k3s
```

Target flow for a real application (not wired yet):

```text
GitHub / app-foo  →  GitLab CI in the cluster  →  GitLab registry
GitHub / app-foo/deploy  →  Argo CD Application  →  k3s namespace app-foo
```

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
   ├── Argo CD
   ├── demo          whoami, echo (Ingress / TLS smoke tests)
   ├── Postgres
   ├── GitLab        runners + registry (planned)
   └── Prometheus + Grafana
```

---

## Repository layout

```text
k3s/
  README.md                 # install k3s
  1.36.4/
    README.md               # bootstrap order
    infra/                  # Helm values and infra manifests
      argocd/
      cert-manager/
      traefik/
      postgre/
      prometheus/
    apps/                   # demo app manifests (whoami, echo)
    argocd/
      root.yaml             # apply once
      applications/
        projects/           # AppProject infra, apps
        infra/              # prometheus, postgres, …
        demo/               # whoami, echo
        apps/               # pointers to other GitHub repos
```

---

## Bootstrap

See [k3s/1.36.4/README.md](k3s/1.36.4/README.md). Order: cert-manager → Traefik TLSStore → Argo CD Helm → `kubectl apply -f argocd/root.yaml`.
