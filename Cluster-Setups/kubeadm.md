# **Kubeadm Setup and Commands**

## **Install Kubeadm and Kubelet**

### 1. Install Kubeadm and Kubelet
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm
sudo apt-mark hold kubelet kubeadm
```

### 2. Verify the Installation
```bash
kubeadm version
```

---

## **Kubeadm Cluster Setup**

### **1. Initialize the Control Plane Node**
To set up the Kubernetes control-plane node, run the following command:

```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

This command will output a token and command that will allow worker nodes to join the cluster. Be sure to save the token or note it down for later.

---

### **2. Configure kubectl Access**
Once the control-plane node is initialized, configure access to the cluster for the root or your user account:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

### **3. Set Up a Pod Network**
After the control-plane is initialized, you need to install a pod network so that pods can communicate with each other.

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

---

### **4. Join Worker Nodes to the Cluster**
To join worker nodes to the cluster, run the following command on each worker node (generated during the `kubeadm init` step):

```bash
sudo kubeadm join <control-plane-node-ip>:6443 --token <your-token> --discovery-token-ca-cert-hash sha256:<hash>
```

You can view the token and join command again anytime by running:

```bash
kubeadm token list
kubeadm token create --print-join-command
```

---

## **Kubeadm Cluster Management**

### **1. Check Cluster Status**
After completing the setup, you can check the cluster status using:

```bash
kubectl get nodes
```

This will list all the nodes in your Kubernetes cluster (control-plane and worker nodes).

### **2. Reset a Cluster**
If you need to reset the cluster (remove everything), you can use the following command on each node:

```bash
sudo kubeadm reset
```

You will also need to remove the configuration file from the user's home directory:

```bash
sudo rm -rf $HOME/.kube
```

### **3. Delete the Cluster**
To delete the Kubernetes cluster, use the following command:

```bash
sudo kubeadm reset --force
```

---

## **Additional Kubeadm Commands**

### **1. View Cluster Join Command**
If you need to view the join command for worker nodes again, use:

```bash
kubeadm token create --print-join-command
```

### **2. View Logs for Kubeadm**
To check logs for the kubeadm process during initialization:

```bash
journalctl -u kubelet
```

---
