# Ingress-NGINX Configuration

Ingress-NGINX is an Ingress controller that manages external access to services within the Kubernetes cluster.

## Helm Deployment

The Ingress-NGINX controller can be deployed using Helm with the following command:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.service.type=LoadBalancer
```

This command deploys the Ingress-NGINX controller in the `ingress-nginx` namespace and sets the service type to `LoadBalancer`, allowing it to receive external traffic.

## Configuration

- **values.yaml**: (Optional) Custom Helm values can be specified in the `values.yaml` file to tailor the deployment to specific needs.

## Accessing the Ingress Controller

After deployment, retrieve the external IP assigned by MetalLB:

```bash
kubectl get svc -n ingress-nginx ingress-nginx-controller
```

Update your DNS records or local `/etc/hosts` file to map desired hostnames to this IP.

# Ingress NGINX (NodePort Setup)

This config installs NGINX Ingress Controller using a NodePort service on a k3s cluster, optimized for Raspberry Pi nodes.

## Installation

```bash
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.service.type=NodePort \
  --set controller.service.nodePorts.http=30080 \
  --set controller.service.nodePorts.https=30443 \
  --set controller.service.externalTrafficPolicy=Cluster
```

## Pi-hole DNS Setup

Point `grafana.local.lab` to the LAN IP of the node running Ingress (e.g., `10.0.0.186`).

## Why NodePort?

HostPort binding on Raspberry Pi (k3s + ARM) often fails due to security restrictions.
NodePort offers a stable workaround without requiring privileged containers.
