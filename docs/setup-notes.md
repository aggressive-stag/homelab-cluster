# Homelab Kubernetes Cluster

## Observability Stack

- Grafana & Prometheus installed with Helm:
  ```bash
  helm install monitor-stack prometheus-community/kube-prometheus-stack --namespace monitor --create-namespace

