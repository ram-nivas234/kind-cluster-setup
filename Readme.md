# KIND & minikube Cluster Setup Guide-----

#Installing KIND and kubectl first --

echo "------------kind installing --------------------------"

#!/bin/bash
#For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-linux-amd64
#For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.29.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

echo "------------kubectl installing --------------------------"

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl 
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
#and then append (or prepend) ~/.local/bin to $PATH
kubectl version --client

echo "--------kind & kubectl installation complete-----------"

2. Setting Up the KIND Cluster

Create a kind-cluster-config.yaml file:
and copy below template using vim -
 
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

Create the cluster using the configuration file:

kind create cluster --config kind-cluster-config.yaml --name my-kind-cluster

Verify the cluster:

kubectl get nodes
kubectl cluster-info

3. Accessing the Cluster
Use kubectl to interact with the cluster:

kubectl cluster-info

#That's it your cluster has been setup you can start working on it 


4. Setting Up the Kubernetes Dashboard
Deploy the Dashboard Apply the Kubernetes Dashboard manifest:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
Create an Admin User Create a dashboard-admin-user.yml file with the following content:

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  
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
Apply the configuration:

kubectl apply -f dashboard-admin-user.yml
Get the Access Token Retrieve the token for the admin-user:

kubectl -n kubernetes-dashboard create token admin-user
Copy the token for use in the Dashboard login.

Access the Dashboard Start the Dashboard using kubectl proxy:

kubectl proxy
Open the Dashboard in your browser:

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
Use the token from the previous step to log in.

5. Deleting the Cluster
Delete the KIND cluster:

kind delete cluster --name my-kind-cluster

6. Notes
Multiple Clusters: KIND supports multiple clusters. Use unique --name for each cluster. Custom Node Images: Specify Kubernetes versions by updating the image in the configuration file. Ephemeral Clusters: KIND clusters are temporary and will be lost if Docker is restarted.


-------------------------------------------------------------------------------------------------------

**Minikube ---

1st - sudo apt-get install -y apt-transport-https ca-certificates curl gpg
2nd - docker 
3rd - curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
 chmod +x minikube
sudo mv minikube /usr/local/bin/

4th - install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

5th - Start minikube
. minikube start --driver=docker --vm=true 
. minikube status
