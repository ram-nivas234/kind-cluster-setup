# KIND & Minikube Cluster Setup Guide

## 🛠️ 1. Installing KIND and `kubectl`

Install KIND and `kubectl` using the following script:

<details>
<summary><code>Install Script (bash)</code></summary>

```bash
#!/bin/bash

# Install KIND
echo "Installing KIND..."
if [ "$(uname -m)" = "x86_64" ]; then
  curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-linux-amd64
elif [ "$(uname -m)" = "aarch64" ]; then
  curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-linux-arm64
fi
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Install kubectl
echo "Installing kubectl..."
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
chmod +x kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl

# Test installation
kubectl version --client
echo "KIND & kubectl installation complete."
```
</details>

---

## ⚙️ 2. Setting Up the KIND Cluster

Create a `kind-cluster-config.yaml` file:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: kindest/node:v1.33.1@sha256:050072256b9a903bd914c0b2866828150cb229cea0efe5892e2b644d5dd3b34f
  - role: worker
    image: kindest/node:v1.33.1@sha256:050072256b9a903bd914c0b2866828150cb229cea0efe5892e2b644d5dd3b34f
  - role: worker
    image: kindest/node:v1.33.1@sha256:050072256b9a903bd914c0b2866828150cb229cea0efe5892e2b644d5dd3b34f
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
```

### ➕ Create the cluster:

```bash
kind create cluster --config kind-cluster-config.yaml --name my-kind-cluster
```

### ✅ Verify the cluster:

```bash
kubectl get nodes
kubectl cluster-info
```

---

## 📡 3. Accessing the Cluster

Use `kubectl`:

```bash
kubectl cluster-info
```

---

## 📊 4. Setting Up the Kubernetes Dashboard

### 🔹 Deploy the Dashboard:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

### 🔹 Create Admin User (`dashboard-admin-user.yml`):

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kubernetes-dashboard
```

Apply the configuration:

```bash
kubectl apply -f dashboard-admin-user.yml
```

### 🔹 Get the Access Token:

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

### 🔹 Access the Dashboard:

```bash
kubectl proxy
```

Open your browser and visit:

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

Use the token to log in.

---

## 🗑️ 5. Deleting the KIND Cluster

```bash
kind delete cluster --name my-kind-cluster
```

---

## 📝 6. Notes

- **Multiple Clusters:** Use unique `--name` for each cluster.
- **Custom Node Images:** Update the image in your config to specify Kubernetes versions.
- **Ephemeral Nature:** KIND clusters are temporary and deleted when Docker restarts.

---

# 🚀 Minikube Setup Guide

### 🔧 1. Install Prerequisites

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

### 🐳 2. Install Docker (required as Minikube driver)

Make sure Docker is installed and running.

### 📦 3. Install Minikube

```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube
sudo mv minikube /usr/local/bin/
```

### 🔗 4. Install kubectl (if not already installed)

```bash
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### 🚀 5. Start Minikube

```bash
minikube start --driver=docker --vm=true
minikube status
```
