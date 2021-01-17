### kubernetes_basic
Tutorial how to create basic cluster kubenetes on Ubuntu Server 20.04.
For this server I will use 3 servers which I configure according my home network:
| IP | Role |
| ------ | ------ |
| 192.168.100.41 | Master |
| 192.168.100.42 | Worker 1 |
| 192.168.100.43 | Worker 2 |


### Setting Server
first, let's set the ip and hostname of the servers first
login as super user:
```sh
$ sudo su
```
change the hostname:
```sh
$ nano /etc/hostname
```
For Server Master change the name to master 
```sh
master
```
For Server Worker change to worker[number]:
```sh
worker1
```
then let's change the IP, adjust IP according to the servers list above:
```sh
$ nano /etc/netplan/00-installer-config.yaml
```
```sh
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
```
```sh
$ nano /etc/netplan/00-installer-config.yaml
```
