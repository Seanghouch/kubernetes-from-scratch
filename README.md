# kubernetes-from-scratch
## Introduction
1. Configuration Ubuntu 20.04 LTS
2. Installation Container Runtime, Kubeadm, Kubelet, Kubectl

### Requirement(recomment)
* 1 Master Node
    + OS: Ubuntu 20.04 LTS
    + CPU: 2 core
    + RAM: 4 GB
    + HDD: 30GB
* 2 Wowrker Node
    + OS: Ubuntu 20.04 LTS
    + CPU: 1 core
    + RAM: 2 GB
    + HDD: 30GB

### Firewell
Control-plane node(s) or Master node
| Protocol | Direction | Port Range | Purpose                    | Used By                |
|----------|-----------|------------|--------------------------- | ---------------------- |
| TCP      | Inbound   | 6443*      | Kubernets API server       | All                    |
| TCP      | Inbound   | 2379-2380  | etcd server client API     | kube-apiserver, etcd   |
| TCP      | Inbound   | 10250      | kubelet API                | Self, Control plane    |
| TCP      | Inbound   | 10251      | kube-scheduler             | Self                   |
| TCP      | Inbound   | 10252      | kube-controler-manager     | Self                   |

Worker node(s)
| Protocol | Direction | Port Range | Purpose                    | Used By                |
|----------|-----------|------------|--------------------------- | ---------------------- |
| TCP      | Inbound   | 10250      | Kubernets API server       | Self, Control plane    |
| TCP      | Inbound   | 30000-32767| NodePort Services**        | All                    |

### 1. Configuration Ubuntu 20.04 LTS
Install net-tools
````
sudo apt install net-tools
````
Install vim
````
sudo apt install vim
````
Add user to group root all node, and verify user group
````
sudo su
````
````
usermod -aG root {user-name}
````
````
groups {user-name}
````
Install ssh all node
````
sudo apt install openssh-server
````
````
sudo systemctl start ssh
````
````
sudo systemctl enable ssh
````
````
sudo systemctl status ssh
````
remote from other machine.
````
ssh {user-name}@{ip-server}
````
Config network, sometime in folder /etc/netplan/ file name **01-network-manager-all.yaml** or **01-netcfg.yaml**, your just try edit that file and apply test.
+ **Noted:** I use VMware for test, you can skip this step if you have set-up stitic ip already.

Backup file 01-network-manager-all.yaml
````
sudo cp /etc/netplan/01-network-manager-all.yaml /etc/netplan/01-network-manager-all.yaml.bak
````
Check and note your ip 
````
sudo ifconfig
````
Set up static IP
````
sudo vim /etc/netplan/01-network-manager-all.yaml
````
Example:
````
network:
  version: 2
  renderer: networkd
  ethernets:
    eth33: 
      dhcp4: false
      addresses: [192.168.119.129/24]
      gateway4: 192.168.119.2
      nameservers:
        addresses: [192.168.119.2, 8.8.8.8, 8.8.4.4]

````
Apply network
````
sudo netplan apply
````
Restart server
````
sudo init 6
````
### 2. Installation Container Runtime, Kubeadm, Kubelet, Kubectl
Update and upgrade ubuntu
````
sudo apt-get update
````
````
sudo apt-get upgrade
````
Disable swap
````
sudo swapoff -a
````
comment swap
````
sudo vim /etc/fstab
````
Add # berfore /swapfile then save
````
#/swapfile
````
Install and configure prerequisites
````
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
````
````
sudo modprobe overlay
sudo modprobe br_netfilter
````
````
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
````
````
sudo sysctl --system
````