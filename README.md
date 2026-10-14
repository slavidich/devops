# homelab devops

personal homelab on k3s. this repo is the platform: gitops with argo cd, later ci and a registry via gitlab in the cluster. product code lives in other github repos. gitlab will build images; argo cd deploys them.

`slavidich.duckdns.org` is a public name that points at a private `192.168.0.x` address. the cluster is not on the internet.

tls comes from let's encrypt via cert-manager, using dns-01. http-01 would need let's encrypt to fetch a file from a public web server — that cannot work here. dns-01 instead creates a txt record `_acme-challenge.slavidich.duckdns.org` through the duckdns api. let's encrypt checks that and issues a wildcard cert.

two layers.

bootstrap is once, by hand: cert-manager, traefik tls, argo cd. gitops cannot start until those exist — argo cd needs ingress and tls, and cannot install itself.

after that argo cd watches this repo. change yaml on github, it applies to the cluster.

app of apps: apply `argocd/root.yaml` once. it recursively watches `argocd/applications/` — appprojects (`infra`, `apps`) and child applications. children point at this repo (infra, demo) or, later, other github repos.

`spec.destination.namespace` is where resources land. manifests omit `metadata.namespace`. argo cd creates the namespace with `CreateNamespace=true`.

how the pieces relate:

```text
github (this repo)
        |
        v
   argo cd  -- root --> applications/
                          projects/   appproject infra, apps
                          infra/      prometheus, postgres, ...
                          demo/       whoami, echo
                          apps/       pointers to other github repos
        |
        v
      k3s
```

target flow for a real application (not wired yet):

```text
github / app-foo  ->  gitlab ci in the cluster  ->  gitlab registry
github / app-foo/deploy  ->  argo cd application  ->  k3s namespace app-foo
```

from the outside:

```text
internet
   |
   v
duckdns  (*.slavidich.duckdns.org)
   |
   v
traefik  (ingress / tls)
   |
   +-- argo cd
   +-- demo          whoami, echo (ingress / tls smoke tests)
   +-- postgres
   +-- gitlab        runners + registry (planned)
   +-- prometheus + grafana
```

repository layout:

```text
k3s/
  README.md                 install k3s
  1.36.4/
    README.md               bootstrap order
    infra/                  helm values and infra manifests
      argocd/
      cert-manager/
      traefik/
      postgre/
      prometheus/
    apps/                   demo app manifests (whoami, echo)
    argocd/
      root.yaml             apply once
      applications/
        projects/           appproject infra, apps
        infra/              prometheus, postgres, ...
        demo/               whoami, echo
        apps/               pointers to other github repos
```

bootstrap order is in `k3s/1.36.4/README.md`: cert-manager, then traefik tlsstore, then argo cd helm, then `kubectl apply -f argocd/root.yaml`. how argo cd walks the tree after that is in `k3s/1.36.4/argocd/README.md`.
