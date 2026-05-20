# база
`export KUBECONFIG=~/.kube/k3s-config`

# порядок
infrastructure/cert-manager   # helm + wildcard-certificate.yaml
apps/whoami                   # TLS через общий wildcard
apps/echo                     # второй hostname на том же wildcard

infrastructure/argo-cd
