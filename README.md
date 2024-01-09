# kubernetes-from-scratch
## Introduction
1. Configuration Ubuntu 20.04 LTS
2. Installation Container Runtime
3. Installation Kubeadm, Kubelet, Kubectl


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