### kubernetes_basic
Tutorial how to create basic cluster kubenetes on Ubuntu Server 20.04.
For this server we will use 3 servers
| IP | Role |
| ------ | ------ |
| 192.168.100.41 | Master |
| 192.168.100.42 | Worker 1 |
| 192.168.100.43 | Worker 2 |

### Setting Server
first, let's set the ip and hostname of the servers first

change the hostname:
```sh
$ sudo nano /etc/hostname
```
For Server Master change the name to masternode 
```sh
masternode
```
For Server Worker change to workernode[number]:
```sh
workernode1
```
