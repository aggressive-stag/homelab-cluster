# Homelab Kubernetes Cluster

This cluster runs on Raspberry Pi devices with k3s and MetalLB. It exposes web apps via Ingress NGINX using NodePort routing for compatibility and simplicity.

## Components

- [Ingress NGINX](infra/ingress-nginx/README.md)
- [Monitoring Stack (Grafana)](apps/monitoring/README.md)

## Setup Notes

- MetalLB was installed and initially configured but replaced with NodePort for stability.
- Pi-hole DNS is used for `.local.lab` hostnames.
