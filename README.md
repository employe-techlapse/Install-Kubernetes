Install Required Packages
On all nodes (Master and Worker):

Update the system packages:

bash
Copy code
sudo apt update && sudo apt upgrade -y
Install Docker (as the container runtime):

bash
Copy code
sudo apt install -y docker.io
Start and enable Docker:
bash
Copy code
sudo systemctl start docker
sudo systemctl enable docker
Install kubeadm, kubectl, and kubelet:

Add Kubernetes apt repository:
bash
Copy code
sudo apt update
sudo apt install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt update
Install Kubernetes components:
bash
Copy code
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
Disable swap (required by Kubernetes):

bash
Copy code
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
2. Initialize Kubernetes Cluster on the Master Node
On the master node:

Run kubeadm init:

bash
Copy code
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
--pod-network-cidr: Specifies the CIDR range for the pod network. Adjust as needed.
After initialization:

Set up kubectl for the current user:
bash
Copy code
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
Save the kubeadm join command displayed at the end of initialization. This will be used to join worker nodes.

3. Install a Pod Network Add-On
A pod network allows communication between pods. Install a network plugin, such as Calico:

bash
Copy code
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
4. Add Worker Nodes to the Cluster
On each worker node:

Use the kubeadm join command from the master node initialization. Example:

bash
Copy code
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
Verify the nodes are added: On the master node, check the cluster status:

bash
Copy code
kubectl get nodes
5. Deploy an Application on Kubernetes
Step 1: Create a Deployment
Create a deployment to manage your application:

bash
Copy code
kubectl create deployment nginx-deployment --image=nginx --replicas=2
This creates two replicas of the nginx container.
Step 2: Expose the Deployment
Expose the deployment as a service:

bash
Copy code
kubectl expose deployment nginx-deployment --type=NodePort --port=80
--type=NodePort: Exposes the service on a port accessible externally.
Step 3: Access the Application
Get the service details:
bash
Copy code
kubectl get service
Note the NodePort value.
Access the application in a browser:
plaintext
Copy code
http://<node-ip>:<node-port>
6. Common Commands
Check the status of nodes:
bash
Copy code
kubectl get nodes
View all pods:
bash
Copy code
kubectl get pods -o wide
View services:
bash
Copy code
kubectl get services
Delete a deployment:
bash
Copy code
kubectl delete deployment <deployment-name>
