# base
`export KUBECONFIG=~/.kube/k3s-config`

# order

infrastructure/cert-manager   # helm + wildcard-certificate.yaml (Secret в kube-system)

infrastructure/traefik        # TLSStore default — wildcard cert

apps/whoami                   # HTTPS with Traefik default TLS

apps/echo                     # second hostname, also wildcard

---

infrastructure/argo-cd

--- 
