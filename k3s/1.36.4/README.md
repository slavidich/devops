# base
`export KUBECONFIG=~/.kube/k3s-config`

# order

infrastructure/cert-manager   # helm + wildcard-certificate.yaml (Secret в kube-system)

infrastructure/traefik        # TLSStore default — wildcard cert

infrastructure/argo-cd

infrastructure/prometheus  # namespace monitoring + grafana-admin Secret, затем Helm или Argo CD Application
