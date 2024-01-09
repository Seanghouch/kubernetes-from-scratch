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
Install git
````
sudo apt install git
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
Disable swap
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
Install CRI-O Runtime
````
cat <<EOF | sudo tee /etc/modules-load.d/crio.conf
overlay
br_netfilter
EOF
````
````
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
````
Execute the following commands to enable overlayFS & VxLan pod communication.
````
sudo modprobe overlay
````
````
sudo modprobe br_netfilter
````
Set up required sysctl params, these persist across reboots.
````
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
````
Reload the parameters.
````
sudo sysctl --system
````
Install Kubeadm & Kubelet & Kubectl

Update your system packages:
````
sudo apt-get update
````
Install apt-transport-https curl
````
sudo apt-get install -y apt-transport-https curl
````
Add gpg keys
````
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
````
Add **"deb https://apt.kubernetes.io/ kubernetes-xenial main"** into **/etc/apt/sources.list.d/kubernetes.list**
example:
````
sudo vim /etc/apt/sources.list.d/kubernetes.list
add => deb https://apt.kubernetes.io/ kubernetes-xenial main
````
Install kubelet kubeadm kubectl
````
sudo apt-get update
````
````
sudo apt-get install -y kubelet="1.28-1.00" kubeadm="1.28-1.00" kubectl="1.28-1.00"
````
````
sudo apt-mark hold kubelet kubeadm kubectl
````
update apt
````
sudo apt-get update
````
install jq
````
sudo apt-get install -y jq
````
Add kubelet extra args
````
cat > /etc/default/kubelet << EOF
KUBELET_EXTRA_ARGS=--node-ip={ip-local-machine}
EOF
````

Config master node

Pull kubeadm images
````
sudo kubeadm config images pull
````
Initialize kubeadm
````
sudo kubeadm init --apiserver-advertise-address={ip-master-node} --apiserver-cert-extra-sans={ip-master-node} --pod-network-cidr="192.168.0.0/16" --node-name {hostname} --ignore-preflight-errors Swap
````
Configuration kubeadm
````
mkdir -p "$HOME"/.kube
````
````
sudo cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
````
````
sudo chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config
````
Now, verify the kubeconfig by executing the following kubectl command to list all the pods in the kube-system namespace.
````
kubectl get po -n kube-system
````
create join-command from worker node to master node
````
kubeadm token create --print-join-command
````
Example: command run on worker node to join master node
````
sudo kubeadm join 192.168.119.129:6443 --token murj3y.yv2cqywq9j2grpuz --discovery-token-ca-cert-hash sha256:588824986bf5f1d8e7e255aa76a76b2e86a43d8c4d76f23bc9b03e31d81691cf
````
Verify node are join with master node
````
kubectl get nodes
````