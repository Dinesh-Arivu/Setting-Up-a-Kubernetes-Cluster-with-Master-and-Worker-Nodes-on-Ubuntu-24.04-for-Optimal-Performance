# 🚀 Ultimate Guide: Setting Up a Kubernetes Cluster on Ubuntu 24.04

## 📝 Introduction
Kubernetes is a powerful open-source platform designed to automate the deployment, scaling, and operation of application containers.  

In this comprehensive guide, we’ll explore how to set up a **Kubernetes cluster** using three nodes, leveraging the flexibility and scalability of **Amazon EC2 instances**.  
We’ll ensure each step is clear and detailed, making it easy for anyone to follow along.

---

## ⚙️ What You Will Need
- **Two EC2 Instances**
  - One instance as the **Master Node**
  - One instance as the **Worker Node**
- **Instance Type**: At least **2GB RAM** (Recommended: `t2.medium`)
- **Caution**: `t2.medium` costs money 💰 → Monitor usage to avoid unexpected bills

---

## 🖥️ Step 1: Create Amazon EC2 Instances

### Configuration for All Instances:
- Select **Ubuntu 24.04**
- Instance Type: **t2.medium (2GB RAM, 2 CPUs)**
- Storage: **16GB default**
- Networking: **Default VPC**
- Auto-assign Public IP: **Enabled**

### Detailed Steps:
1. Open the **AWS Management Console**
2. Navigate to **EC2 Dashboard**
3. Click **Launch Instance**
4. Choose **Ubuntu Server 24.04 LTS (HVM), SSD**
5. Select **t2.medium**
6. Configure:
   - Instances: **2**
   - Subnet: Available subnet
   - Security Group:
     - Allow **SSH (port 22)** from your IP  
     - (Optional) Open all ports for **learning only 🚫** (not for production)
7. Add Tags (optional)
8. Review & Launch
9. Select/create a **Key Pair**
10. Launch Instances ✅

---

## 🐳 Step 2: Install and Configure Docker & Kubernetes

### Connect to Instances:
```bash
ssh -i <your-key.pem> ubuntu@<instance-ip>
````

### Update Packages:

```bash
sudo apt update && sudo apt upgrade -y
```

### Disable Swap:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### Load Kernel Modules:

```bash
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```

### Sysctl Parameters:

```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

### Install Docker + Containerd:

```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y containerd.io
```

Configure containerd:

```bash
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### Install Kubernetes Packages:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
```

(If fails → use **Snap** method)

---

## 🏗️ Step 3: Initialize Master Node

Initialize Cluster:

```bash
sudo kubeadm init
```

Setup kubeconfig:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Deploy **Calico** network:

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

---

## 🔗 Step 4: Add Worker Nodes

On each Worker Node:

```bash
sudo kubeadm join <master-ip>:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

Verify cluster:

```bash
kubectl get nodes
```

---

## 🎉 Conclusion

✅ You have successfully set up a Kubernetes cluster with **1 master + 1 worker** on Ubuntu 24.04 using AWS EC2.

---

## ✅ Summary of Key Points

* ✅ Created EC2 instances (Master + Worker)
* ✅ Installed Docker & containerd
* ✅ Installed kubeadm, kubelet, kubectl
* ✅ Initialized Kubernetes cluster on Master
* ✅ Joined Worker node
* ✅ Installed Calico networking

---

## 💡 Additional Tips

* 🔍 Set up **Monitoring & Logging** (Prometheus, Grafana, etc.)
* 🔒 Keep your nodes **patched & secure**
* 🚫 Don’t keep all ports open in production!

---


