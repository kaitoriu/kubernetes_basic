### Introduction
Tutorial how to create basic cluster kubenetes on Ubuntu Server 20.04.
For this server I will use 3 servers which I configure according my home network:
| IP | Role |
| ------ | ------ |
| 192.168.100.41 | Master |
| 192.168.100.42 | Worker 1 |
| 192.168.100.43 | Worker 2 |


### Setting the Server
first, let's set the ip and hostname of the servers first. Do this to all servers.

Login as super user:
```sh
sudo su
```
change the hostname:
```sh
#edit hostname
nano /etc/hostname

#For Server Master change the name to master 
master

#For Server Worker change to worker[number]:
worker1
```
then let's change the IP, adjust IP according to the servers list above:
```sh
#edit configuration
nano /etc/netplan/00-installer-config.yaml

#configuration content
network:
  ethernets:
    enp0s3:
      addresses:
      - 192.168.100.41/24
      gateway4: 192.168.100.1
      nameservers:
        addresses:
        - 8.8.8.8
  version: 2

#apply the configuration
netplan apply
```
disable swap and firewall:
```sh
#disable swap
swapoff -a; sed -i '/swap/d' /etc/fstab

#disable firewall
ufw disable

#reboot server to apply the settings
reboot now
```
### Install container runtime
Do this on all servers

Install and configure prerequisites:
```sh
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```
Install containerd:
```sh
# (Install containerd)
sudo apt-get update && sudo apt-get install -y containerd

# Configure containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml

# Restart containerd
sudo systemctl restart containerd
```
### Install kubeadm, kubelet, kubectl
add repository:
```sh
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
```
install kubeadm, kubelet, kubectl:
```sh
apt update && apt install -y kubeadm kubelet kubectl
```
### Initialize master (On master node)
run this command just on master node:
```sh
# Configure pod-network-cidr with ip according to the network pod plugin you use
# Input apiserver-advertise-address with master ip address
kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.100.41

# After Success 
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.100.41:6443 --token xfosll.7q1hfhh4wjjrsmoa \
    --discovery-token-ca-cert-hash sha256:0ae573f871a3704ba882cb13e453a1596a768ed873c8c62250f6cde890a58b63   
```
```sh
# Run the command on master node as instructed 
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Join workers node (On workers node)
run the command which generated from master node on each workers node:
```sh
kubeadm join 192.168.100.41:6443 --token xfosll.7q1hfhh4wjjrsmoa \
    --discovery-token-ca-cert-hash sha256:0ae573f871a3704ba882cb13e453a1596a768ed873c8c62250f6cde890a58b63   
```
