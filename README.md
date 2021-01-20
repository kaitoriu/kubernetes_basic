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
use systemd for cgroup driver:
```sh
nano /etc/containerd/config.toml

[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
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
