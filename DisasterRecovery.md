# Disaster Recovery: Rebuilding the Homelab Cluster

If you're here, something exploded. Here’s how to rebuild your cluster from a bare-metal state using your Git-backed configuration.

---

## 🔁 1. Reinstall k3s on the control node

```bash
curl -sfL https://get.k3s.io | sh -
```

Verify:
```bash
kubectl get nodes
```

For multi-node:
- Join other nodes using the token from `/var/lib/rancher/k3s/server/node-token`
- Example: `k3s agent --server https://<CONTROL_IP>:6443 --token <TOKEN>`

---

## 🌐 2. Ensure network connectivity between all nodes

Make sure all nodes:
- Can ping each other
- Are on the same subnet or VLAN
- Have synchronized clocks (use `ntpd` or `systemd-timesyncd`)

---

## 🧪 3. Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## 🚀 4. Add required Helm repos

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## 🛠 5. Install NGINX Ingress Controller (NodePort)

```bash
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.service.type=NodePort \
  --set controller.service.nodePorts.http=30080 \
  --set controller.service.nodePorts.https=30443 \
  --set controller.service.externalTrafficPolicy=Cluster
```

---

## 📈 6. Install Monitoring Stack (Prometheus + Grafana)

```bash
helm install monitor-stack prometheus-community/kube-prometheus-stack \
  --namespace monitor --create-namespace
```

Wait a minute, then get the Grafana password:

```bash
kubectl -n monitor get secret monitor-stack-grafana -o jsonpath="{.data.admin-password}" | base64 -d && echo
```

---

## 🌍 7. Apply Grafana Ingress Route

```bash
kubectl apply -f apps/monitoring/grafana-ingress.yaml
```

Contents of `grafana-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana-ingress
  namespace: monitor
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: grafana.local.lab
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: monitor-stack-grafana
                port:
                  number: 3000
```

---

## 📡 8. Update Pi-hole DNS (if needed)

Set `grafana.local.lab → <NODE-IP>` in Pi-hole DNS.

---

## ✅ 9. Access Grafana

Visit:
```
http://grafana.local.lab:30080
```

---

## 🧯 10. Git Rehydration

After pulling the Git repo:
- Apply all ingress files (`kubectl apply -f apps/**`)
- Optionally restore any PVs, PVCs, or dashboards
