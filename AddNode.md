# Adding Another Node to the Cluster

Use this guide to add a new Raspberry Pi (or other Linux machine) as a worker node in your existing k3s cluster.

---

## 🧠 Prerequisites

- Your control plane is already running k3s
- The new node is on the same network and can reach the control plane
- You have SSH access to the new node

---

## 🛠 Step-by-Step

### 1. Get the Node Token from the Control Plane

On your k3s control node:

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

You’ll get something like:
```
K10d54f99cb4bafc47a7fcb3...::server:6b21d5...
```

Save this token for the next step.

---

### 2. Install k3s Agent on the New Node

SSH into the new node, then run:

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://<CONTROL_NODE_IP>:6443 K3S_TOKEN=<PASTE_TOKEN_HERE> sh -
```

Example:
```bash
curl -sfL https://get.k3s.io | K3S_URL=https://10.0.0.186:6443 K3S_TOKEN=K10abc123... sh -
```

---

### 3. Confirm the Node Has Joined

Back on the control plane, run:

```bash
kubectl get nodes
```

You should now see the new node listed with `STATUS: Ready`.

---

## 🔎 Notes

- Nodes will automatically start running workloads based on your cluster’s scheduling needs.
- You do not need to install Helm or kubectl on the worker nodes.
- If you want to run system pods (like Ingress) on the new node, make sure taints and tolerations are handled.

---

## 🧼 Optional: Label the New Node

To label the node for targeting workloads:

```bash
kubectl label node <NODE_NAME> role=worker
```

You can then use nodeSelectors or affinity rules in your manifests.

