# Monitoring Stack (Prometheus & Grafana)

This directory contains configuration for deploying a monitoring stack using the kube-prometheus-stack Helm chart.

## Helm Install

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitor-stack prometheus-community/kube-prometheus-stack \
  --namespace monitor --create-namespace
```

## Accessing Grafana

Grafana is exposed via Ingress and is accessible at:

```
http://grafana.local.lab
```

To retrieve the default Grafana admin password:

```bash
kubectl -n monitor get secret monitor-stack-grafana -o jsonpath="{.data.admin-password}" | base64 -d && echo
```

## Ingress Config

See `grafana-ingress.yaml` in this directory to modify routing or host configuration.