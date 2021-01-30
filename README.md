### Introduction
Ini adalah tutorial untuk membuat cluster kubernetes dasar di Ubuntu Server 20.04.
Dalam tutorial ini saya menggunakan 3 buah server dengan IP sebagai berikut:
| IP | Role |
| ------ | ------ |
| 192.168.100.41 | Master |
| 192.168.100.42 | Worker 1 |
| 192.168.100.43 | Worker 2 |
Untuk IP ini sudah saya sesuaikan dengan pengaturan network internet rumah saya. jadi untuk teman-teman silahkan sesuaikan dengan pengaturan internet rumah atau kantor teman-teman sendiri.

### Setting Server
Pertama bagi yang belum tahu mari kita melakukan pengaturan dasar dulu untuk server kita. 
Kita akan melakukan pengaturan hostname dan IP server-server kita.

Untuk mempermudah login sebagai super user dahulu:
```sh
sudo su
```
ubah hostname:
```sh
#edit hostname
nano /etc/hostname

# Untuk Server Master ubah nama hostname menjadi master 
master

# Untuk Server Master ubah nama hostname menjadi worker[nomor]:
worker1
```
Lalu mari kita sesuaikan IP server kita sesuai daftar di atas:
```sh
#edit configuration
nano /etc/netplan/00-installer-config.yaml

#ubah isi sesuai pengaturan IP
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

#apply pengaturan
netplan apply
```
disable swap and firewall untuk kubernetes:
```sh
#disable swap
swapoff -a; sed -i '/swap/d' /etc/fstab

#disable firewall
ufw disable

#reboot server untuk apply pengaturan
reboot now
```
### Install container runtime

Install dan atur prasyarat container runtime:
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
tambah repository:
```sh
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
```
install kubeadm, kubelet, kubectl:
```sh
apt update && apt install -y kubeadm kubelet kubectl
```
### Initialize master 
run perintah ini hanya di master node:
```sh
# Atur pod-network-cidr berdasarkan CNI component yang digunakan
# Atur apiserver-advertise-address sesuai master ip address
kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.100.41

# Setelah berhasil akan muncul tulisan 
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
# Jalankan perintah pada master node sesuai instruksi 
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### Join worker node (worker nodes)
Jalankan perintah yang digenerate oleh master node pada setiap worker node:
```sh
kubeadm join 192.168.100.41:6443 --token xfosll.7q1hfhh4wjjrsmoa \
    --discovery-token-ca-cert-hash sha256:0ae573f871a3704ba882cb13e453a1596a768ed873c8c62250f6cde890a58b63   
```
### Install CNI component (On master node)
Untuk menghubungkan semua node maka install CNI Component pada master node. Dalam tutorial ini saya menggunakan calico
```sh
kubectl apply -f https://docs.projectcalico.org/v3.17/manifests/calico.yaml
```
### Check nodes
Setelah beberapa detik silahkan periksa apakah node-node sudah terhubung dengan perintah di bawah: 
```sh
kubectl get nodes

# If all nodes is ready that command will show
NAME      STATUS   ROLES                  AGE   VERSION
master    Ready    control-plane,master   26m   v1.20.2
worker1   Ready    <none>                 14m   v1.20.2
worker2   Ready    <none>                 14m   v1.20.2
```
