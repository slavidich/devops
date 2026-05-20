# база
`export KUBECONFIG=~/.kube/k3s-config`

# порядок

infrastructure/cert-manager   # helm + wildcard-certificate.yaml (Secret в kube-system)

infrastructure/traefik        # TLSStore default — wildcard на все Ingress

apps/whoami                   # HTTPS через Traefik default TLS

apps/echo                     # второй hostname, тот же wildcard

---

infrastructure/argo-cd

--- 
Дальше подвязываемся в арго с репе и все само работает четенько 