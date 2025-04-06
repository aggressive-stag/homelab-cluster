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