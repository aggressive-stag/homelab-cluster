# Adding a New Node to a Kubernetes (kubeadm) Cluster

This guide walks you through setting up a new Raspberry Pi or Linux VM as a worker node in your `kubeadm`-based Kubernetes cluster.

---

## 🧠 Prerequisites

- The control plane is already running with kubeadm
- The new node is on the same network and reachable
- You have SSH access to the new node
- You have the `kubeadm join` command from your control plane

---

## ⚙️ System Preparation

SSH into the new node and run the following:

### 1. Update and Install Base Packages

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

### 2. Disable Swap (Required for Kubernetes)

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#/g' /etc/fstab
```

### 3. Load Kernel Modules

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

### 4. Apply sysctl Settings

```bash
cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

---

## 📦 Install Kubernetes Components

### 5. Add the Kubernetes Apt Repository

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key |
    gpg --dearmor | sudo tee /etc/apt/keyrings/kubernetes-apt-keyring.gpg > /dev/null

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" |
    sudo tee /etc/apt/sources.list.d/kubernetes.list
```

> 🔐 This replaces the deprecated `apt.kubernetes.io` repo.

### 6. Install Kubernetes Binaries

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## 📦 Install containerd

```bash
sudo apt-get install -y containerd
```

> Optionally configure `/etc/containerd/config.toml` and restart the service:

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
```

---

## 🤝 Join the Cluster

On the control plane node, retrieve the join command:

```bash
kubeadm token create --print-join-command
```

It will look like this:

```bash
kubeadm join 10.0.0.10:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

Run this on the **new node**:

```bash
sudo kubeadm join 10.0.0.10:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

---

## ✅ Validate

On the control plane node:

```bash
kubectl get nodes
```

You should now see the new node in the list with `STATUS: Ready`.

---

## 🧼 Optional: Label the New Node

```bash
kubectl label node <NODE_NAME> role=worker
```

You can then target workloads using node selectors or affinity.
