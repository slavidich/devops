# prometheus + grafana (kube-prometheus-stack)

helm chart `kube-prometheus-stack`: prometheus operator, prometheus, grafana, alertmanager, node-exporter, kube-state-metrics.

argo cd application `prometheus` installs the chart (version 90.2.0) with `values.yaml` from this folder. disabled scrape of etcd / controller-manager / scheduler / kube-proxy, because on k3s those control-plane components are not exposed the kubeadm way.

this readme is the fallback if you need to install by hand, and the place for the grafana password which argo cd does not create.

grafana admin secret. argo cd creates namespace `monitoring`. the secret is still created by hand:

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl create namespace monitoring
kubectl create secret generic grafana-admin \
  --from-literal=admin-user=admin \
  --from-literal=admin-password='SuperSecretPassword' \
  -n monitoring
```

helm, only if argo cd is not doing it:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade --install kps prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --version 90.2.0 \
  -f values.yaml
```

grafana ingress is `grafana.slavidich.duckdns.org`. check:

```bash
kubectl get pods -n monitoring
kubectl get ingress -n monitoring
```

if grafana pods fail to mount `kps-grafana-config-dashboards`, the usual cause is not grafana itself. look at application `prometheus` in argo cd. a typical failure is appproject `infra` refusing a namespace the chart wants (for example `kube-system` for the coredns scrape service). the whole sync then aborts and those configmaps never appear.
