# Kubernetes Multi-Node Cluster Setup (Local or AWS EC2 using kubeadm)

![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.30-blue)
![Cluster](https://img.shields.io/badge/Cluster-Multi--Node-green)
![Platform](https://img.shields.io/badge/Platform-AWS%20%7C%20Local-orange)
![Runtime](https://img.shields.io/badge/Container%20Runtime-containerd-yellow)
![CNI](https://img.shields.io/badge/CNI-Flannel-blueviolet)
![Provisioning](https://img.shields.io/badge/Provisioning-kubeadm-critical)
![OS](https://img.shields.io/badge/OS-Ubuntu%2022.04%20%7C%2024.04-lightgrey)
![Beginner Friendly](https://img.shields.io/badge/Level-Beginner%20Friendly-success)
![License](https://img.shields.io/badge/License-MIT-brightgreen)

This guide provides step-by-step instructions to set up a Kubernetes cluster using:

- `kubeadm` for cluster initialization
- `containerd` as the container runtime
- Flannel as the networking solution

The cluster consists of:

- 1 Control Plane (Master Node)
- 2 Worker Nodes

---

# Table of Contents

1. Overview
2. Local Setup (Optional)
3. AWS Infrastructure Setup
4. Common Configuration (All Nodes)
5. Control Plane Setup
6. Worker Node Setup
7. Verification
8. Troubleshooting
9. Conclusion

---

# 1. Overview

Cluster architecture:

```
AWS VPC (172.31.0.0/16)
 ├── master01 (Control Plane)
 ├── worker01
 └── worker02
```

All nodes must:

- Reside in the same network (VPC/Subnet or VM network)
- Communicate via private IP addresses
- Have unrestricted internal connectivity

---

# 2. Local Setup (Optional)

You can replicate this setup locally using virtual machines.

## Recommended OS

- Ubuntu 22.04 LTS
- Ubuntu 24.04 LTS

## Minimum Requirements

| Node Type | CPU | RAM  | Disk  |
| --------- | --- | ---- | ----- |
| Master    | 2   | 2 GB | 20 GB |
| Worker    | 1   | 1 GB | 20 GB |

## Supported Tools

- VirtualBox
- VMware
- Multipass
- KVM

## Networking Requirements

- All VMs must be on the same network
- Nodes must reach each other via private IP

Test connectivity:

```bash
ping <node-ip>
```

## Notes

- Do not use `localhost` or `127.0.0.1`
- Ensure no firewall blocks traffic

After VM creation, proceed to:

- Common Setup
- Control Plane Setup
- Worker Setup

---

# 3. AWS Infrastructure Setup

## 3.1 Launch EC2 Instances

Create three EC2 instances:

| Name     | Role          | Instance Type     |
| -------- | ------------- | ----------------- |
| master01 | Control Plane | t2.medium         |
| worker01 | Worker        | t2.micro/t2.small |
| worker02 | Worker        | t2.micro/t2.small |

Use:

- Ubuntu Server 22.04 LTS or 24.04 LTS

---

## 3.2 Networking Requirements

Ensure:

- Same VPC
- Same Subnet
- Use private IPs for communication

---

## 3.3 Security Group Configuration

Create a security group to allow Kubernetes API traffic on port 6443.

### Steps

1. Navigate to EC2 Dashboard
2. Open **Security Groups**
3. Click **Create Security Group**

### Configuration

- Name: `k8s-6443-sg`
- Description: Allow Kubernetes API access
- VPC: Same as EC2 instances

### Inbound Rule

```
Type: Custom TCP
Port: 6443
Source: My IP (recommended)
```

Avoid `0.0.0.0/0` unless necessary.

### Attach to Instance

- Go to EC2 → Instances
- Select instance
- Actions → Security → Change security groups
- Attach `k8s-6443-sg`

---

## 3.4 Connect to Instances

```bash
ssh -i <key.pem> ubuntu@<public-ip>
```

---

## 3.5 Set Hostnames

Run on respective nodes:

```bash
sudo hostnamectl set-hostname master01
sudo hostnamectl set-hostname worker01
sudo hostnamectl set-hostname worker02
```

---

## 3.6 Configure Hosts File

Edit `/etc/hosts` on all nodes:

```bash
sudo nano /etc/hosts
```

Add:

```
<MASTER-IP> master01
<WORKER1-IP> worker01
<WORKER2-IP> worker02
```

---

# 4. Common Configuration (All Nodes)

## 4.1 Update System

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https curl
```

---

## 4.2 Install containerd

```bash
sudo apt-get install -y containerd
```

---

## 4.3 Configure containerd

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Enable systemd cgroup:

```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

Restart:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

---

## 4.4 Add Kubernetes Repository

```bash
sudo mkdir -p /etc/apt/keyrings
```

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" \
| sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```bash
sudo apt-get update
```

---

## 4.5 Install Kubernetes Components

```bash
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

---

## 4.6 Enable kubelet

```bash
sudo systemctl enable --now kubelet
```

---

## 4.7 Disable Swap

```bash
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
```

---

## 4.8 Enable Networking

```bash
sudo modprobe br_netfilter
sudo sysctl -w net.ipv4.ip_forward=1
```

---

# 5. Control Plane Setup

## 5.1 Initialize Cluster

```bash
sudo kubeadm init \
  --apiserver-advertise-address=<MASTER-IP> \
  --pod-network-cidr=10.244.0.0/16
```

---

## 5.2 Configure kubectl

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

## 5.3 Install Flannel

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

---

## 5.4 Verify Control Plane

```bash
kubectl get pods -A
```

Wait until all pods are in `Running` state.

---

## 5.5 Get Join Command

```bash
kubeadm token create --print-join-command
```

---

# 6. Worker Node Setup

## 6.1 Reset Node (if required)

```bash
sudo kubeadm reset -f
sudo rm -rf /etc/kubernetes/ $HOME/.kube /etc/cni/net.d
sudo systemctl restart containerd
```

---

## 6.2 Join Cluster

```bash
sudo kubeadm join <MASTER-IP>:6443 \
  --token <TOKEN> \
  --discovery-token-ca-cert-hash sha256:<HASH>
```

---

# 7. Verification

On the master node:

```bash
kubectl get nodes
```

Expected output:

```
master01   Ready
worker01   Ready
worker02   Ready
```

---

# 8. Troubleshooting

## Node in NotReady State

Common causes:

- Flannel not initialized
- CNI issues
- containerd not restarted

Fix:

```bash
sudo rm -rf /etc/cni/net.d
sudo systemctl restart containerd
sudo systemctl restart kubelet
```

---

## Verify Flannel

```bash
kubectl get pods -n kube-flannel -o wide
```

Restart if needed:

```bash
kubectl delete pod -n kube-flannel --all
```

---

## containerd Not Found

```bash
sudo apt-get install -y containerd
```

---

## kubeadm Join Failure

```bash
kubeadm reset -f
```

---

## SSH Host Key Error

```bash
ssh-keygen -R <IP>
```

---

## APT Repository Error

Ensure repository entry is a single line.

---

# 9. Conclusion

You now have a fully functional Kubernetes cluster with:

- kubeadm-based setup
- containerd runtime
- Flannel networking

This cluster can be used for:

- Application deployment
- Kubernetes learning and experimentation
- Scaling and orchestration practice
