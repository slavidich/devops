# Prometheus + Grafana (kube-prometheus-stack)

Helm-чарт `kube-prometheus-stack`: Prometheus Operator, Prometheus, Grafana, Alertmanager, node-exporter, kube-state-metrics.

## 1. Namespace и секрет Grafana

```bash
export KUBECONFIG=~/.kube/k3s-config
kubectl create namespace monitoring
kubectl create secret generic grafana-admin \
  --from-literal=admin-user=admin \
  --from-literal=admin-password='SuperSecretPassword' \
  -n monitoring
```

## 2. Helm

Арго сам поднимет в принципе все, но вдруг

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

```bash
helm upgrade --install kps prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --version 90.2.0 \
  -f values.yaml
```

## 3. Проверка

```bash
kubectl get pods -n monitoring
kubectl get ingress -n monitoring
```
