# Prometheus + Grafana (kube-prometheus-stack)

Helm chart `kube-prometheus-stack`: Prometheus Operator, Prometheus, Grafana, Alertmanager, node-exporter, kube-state-metrics.

Argo CD Application `prometheus` installs the chart. This README is the fallback if you need to install by hand.

## 1. Grafana admin Secret

Argo CD creates namespace `monitoring`. The Secret is still created by hand:

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl create namespace monitoring
kubectl create secret generic grafana-admin \
  --from-literal=admin-user=admin \
  --from-literal=admin-password='SuperSecretPassword' \
  -n monitoring
```

## 2. Helm (optional)

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade --install kps prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --version 90.2.0 \
  -f values.yaml
```

## 3. Check

```bash
kubectl get pods -n monitoring
kubectl get ingress -n monitoring
```
