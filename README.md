# devops

## dns 
duckdns.org is used as the dns server 

slavidich.duckdns.org -> k3s control plane 

I dont have port forwarding on my router so i use dns01 (not http01) to 
For example, gitlab.slavidich.duckdns.org will direct to my local server

## argo 
not connected to argo (maybe later)
```
infra/argocd 
infra/cert-manager 
infra/traefik 
```