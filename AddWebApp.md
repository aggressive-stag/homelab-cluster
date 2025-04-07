# Adding Another Web App to the Cluster

This guide outlines how to deploy a new internal web app to your homelab cluster and expose it cleanly via Ingress and Pi-hole.

---

## 🧠 Overview

Every app:
- Runs as a Kubernetes `Deployment` + `Service`
- Is exposed via `Ingress` (host-based routing)
- Uses Pi-hole to resolve `*.local.lab` domains on your LAN

---

## 🛠 Steps to Add a New Web App

### 1. Create Namespace (optional but recommended)

```bash
kubectl create namespace myapp
```

---

### 2. Deploy the App

Example `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myorg/myapp:latest
        ports:
        - containerPort: 8080
```

---

### 3. Create the Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: myapp
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

> This maps `myapp:80` internally to the container's `8080`.

---

### 4. Add an Ingress Route

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: myapp
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: myapp.local.lab
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp
                port:
                  number: 80
```

Apply it:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

---

### 5. Add DNS Record to Pi-hole

Go to **Local DNS > DNS Records**  
Add:
```
myapp.local.lab → 10.0.0.186
```

> Replace `10.0.0.186` with the IP of your Ingress node

---

### 6. Access the App

Once deployed and DNS is set:
```
http://myapp.local.lab:30080
```

---

## 🧼 Optional Enhancements

- Label namespace (`kubectl label ns myapp purpose=webapp`)
- Use separate folders like `apps/myapp/` in your Git repo
- Add TLS with cert-manager and `letsencrypt-staging` for testing
