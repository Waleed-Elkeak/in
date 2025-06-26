#!/bin/bash

# --- Update and Upgrade System ---
echo "Updating and upgrading packages..."
sudo apt update && sudo apt upgrade -y

# --- Install Dependencies ---
echo "Installing dependencies..."
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# --- Install Docker ---
echo "Installing Docker..."
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker

# --- Add Kubernetes Repository ---
echo "Adding Kubernetes repository..."
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update

# --- Install Kubernetes Components ---
echo "Installing kubelet, kubeadm, and kubectl..."
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# --- Disable Swap ---
echo "Disabling swap..."
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# --- Configure Kernel Modules ---
echo "Loading kernel modules..."
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system

# --- Master Node Initialization (Only on master) ---
if [[ $(hostname) == "k8smaster" ]]; then
  echo "Initializing Kubernetes cluster on master node..."
  sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --upload-certs | tee kubeadm_init_output.txt
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-autoscaler.yaml
  echo "Kubernetes cluster initialized. Run the following command on worker nodes:"
  echo "kubeadm join <master-ip>:<master-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>"
else
  echo "This is a worker node.  Join the cluster using the command from the master node."
fi


echo "Kubernetes installation script complete."
