
# create root of all applications 
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application 
metadata:
  name: root-app #  root-infra
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/slavidich/devops.git
    targetRevision: main
    path: k3s/1.36.4/argocd/applications
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true 
      selfHeal: true 
```