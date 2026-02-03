# Kubeadm Cluster Setup Guide for CKA Exam Preparation

[![Kubernetes](https://img.shields.io/badge/Kubernetes-kubeadm-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![CKA](https://img.shields.io/badge/CKA-Certified-326CE5?logo=kubernetes&logoColor=white)](https://www.cncf.io/certification/cka/)
[![Docker](https://img.shields.io/badge/Docker-Container-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![containerd](https://img.shields.io/badge/containerd-Runtime-575757?logo=containerd&logoColor=white)](https://containerd.io/)
[![Flannel](https://img.shields.io/badge/Flannel-CNI-4A90E2?logo=flannel&logoColor=white)](https://github.com/flannel-io/flannel)

> **Important**: The CKA (Certified Kubernetes Administrator) exam is conducted on **clusters built with kubeadm**.
> Therefore, practicing building and managing clusters directly with kubeadm is essential for exam preparation.

## File Structure

The files provided with this guide are organized as follows:

```text
k8s/
├── README.md                    # This guide document (main guide)
├── config/                      # Configuration files for actual use
│   ├── kubeadm-config.yaml     # kubeadm initialization config (with actual IP)
│   └── kube-flannel.yml        # Flannel CNI config (with ens15f0 interface specified)
├── templates/                   # Template files (require variable substitution)
│   ├── crictl.yaml             # crictl configuration template
│   ├── kubelet-config.yaml     # kubelet configuration template
│   └── kubeadm-config-template.yaml  # kubeadm configuration template
├── scripts/                     # Executable scripts
│   ├── docker-recovery-commands.sh    # Docker recovery script
│   └── fix-containerd-config.sh      # containerd configuration fix script
└── docs/                        # Detailed guide documents
    ├── file-structure.md        # File structure and usage explanation
    ├── swap-disabling-guide.md  # Detailed swap disabling guide
    ├── external-disk-setup.md    # External disk setup guide
    ├── troubleshooting.md        # Troubleshooting guide
    └── cka-exam-preparation.md  # CKA exam preparation guide
```

**File Usage Guide**:

- **Template files**: Use by substituting variables with actual values
- **Config files**: Modify IP addresses and other settings for your actual environment
- **Script files**: Directly executable (`bash scripts/script-name.sh`)

For detailed information, refer to the [File Structure Guide](docs/file-structure.md).

## Why kubeadm?

- ✅ kubeadm-based cluster identical to the CKA exam environment
- ✅ Directly manage and understand cluster components
- ✅ Practice exam problems in a real environment
- ✅ Improve cluster troubleshooting skills

## Prerequisites

### 1. System Requirements (Similar to CKA exam environment)

- Ubuntu 20.04+ or other Linux distributions
- **Minimum 3 nodes recommended** (1 master + 2 workers)
  - CKA exam typically works with multi-node clusters
- Minimum 2GB RAM (per node)
- Minimum 2 CPU cores (per node)
- Swap disabled (required)

### 2. Learning Environment Setup Options

#### Option 1: Physical/VirtualBox/VMware VMs (Recommended)

- Create multiple VMs to build a multi-node cluster
- Most similar to the exam environment

#### Option 2: Cloud VMs (AWS EC2, GCP, Azure, etc.)

- Use each VM as a node

#### Option 3: Multi-node on local machine

- Single machine environment with multiple node roles (for learning)

### 3. Swap Disabling

**⚠️ Required**: Kubernetes requires swap to be disabled.

```bash
# Check current swap status
sudo swapon --show

# Disable swap
sudo swapoff -a

# Permanently disable (persist after reboot)
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Verify
free -h
```

**For detailed explanation and troubleshooting, refer to:**

- [Swap Disabling Guide](docs/swap-disabling-guide.md)

## Installation Steps

### 1. Container Runtime Installation (containerd)

#### When only Docker is installed (including containerd auto-installation)

If you have only Docker installed without containerd, you can handle everything from containerd installation to configuration with the script below.

```bash
# Install and configure containerd (Docker and Kubernetes coexistence)
bash /home/user/apps/k8s/scripts/fix-containerd-config.sh
```

> The script installs containerd via `apt`/`apt-get` if it's not present, creates default configuration, and applies kubeadm-required options like `SystemdCgroup = true`. Since containerd and Docker restart during execution, run this when there's no service impact.

**If containerd is already installed**, you only need to adjust the configuration as follows:

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl restart docker
sudo systemctl status containerd --no-pager | head -5
sudo systemctl status docker --no-pager | head -5
```

**⚠️ Important**:

- Docker and Kubernetes share the same containerd
- Docker containers and Kubernetes Pods work together
- `/etc/containerd/config.toml` configuration applies to both Docker and Kubernetes

#### When neither Docker nor containerd is installed

```bash
sudo apt-get update
sudo apt-get install -y containerd docker.io
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd docker
sudo systemctl enable containerd docker
```

#### Verify containerd configuration

```bash
# Check containerd configuration
sudo cat /etc/containerd/config.toml | grep -A 5 "SystemdCgroup"

# Check containerd status
sudo systemctl status containerd

# Check containerd version
containerd --version
```

### 2. Add Kubernetes Repository and Install kubeadm, kubelet, kubectl

**Note**: The CKA exam typically uses Kubernetes 1.28 or 1.29. Install the latest stable version.

```bash
# Add Kubernetes apt repository
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Update packages
sudo apt-get update

# Install kubeadm, kubelet, kubectl
sudo apt-get install -y kubelet kubeadm kubectl

# Check versions (Important: CKA exam often requires checking kubectl version)
kubeadm version
kubectl version --client

# Prevent automatic upgrades (prevent unintended upgrades)
sudo apt-mark hold kubelet kubeadm kubectl
```

**Repeat the above process on all nodes (master + workers)!**

### 2-1. External Disk Setup (Optional)

**⚠️ Important**: If you want to use an external disk, set it up **after Kubernetes installation but before kubeadm init**.

**For detailed setup instructions, refer to:**

- [External Disk Setup Guide](docs/external-disk-setup.md)

### 3. Load Kernel Modules

```bash
# Load required kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Set permanently
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

# Network configuration
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply settings
sudo sysctl --system
```

## Master Node Initialization (Run only on master node)

### 0. External Disk Setup Verification (Optional)

**⚠️ Important**: External disk setup must be completed **after Kubernetes installation but before kubeadm init**.

**For detailed setup instructions, refer to:**

- [External Disk Setup Guide](docs/external-disk-setup.md)

#### External Disk Quick Reference

```bash
# Create kubeadm config file (using template file)
EXTERNAL_ETCD_PATH="/media/de/k8s/etcd"
MASTER_IP="10.10.100.80"  # Change to your master node IP

sed -e "s|<마스터-노드-IP>|${MASTER_IP}|g" \
    -e "s|/media/de/k8s/etcd|${EXTERNAL_ETCD_PATH}|g" \
  /home/user/apps/k8s/templates/kubeadm-config-template.yaml | \
  sudo tee /tmp/kubeadm-config.yaml

# kubeadm init (using config file)
sudo kubeadm init --config=/tmp/kubeadm-config.yaml

# Or use pre-created config file
sudo kubeadm init --config=/home/user/apps/k8s/config/kubeadm-config.yaml
```

### 1. kubeadm Initialization

```bash
# Check master node IP address
ip addr show | grep inet

# Basic initialization (Pod network will be installed later)
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Or explicitly specify master node IP:
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=<master-node-IP> \
  --control-plane-endpoint=<master-node-IP>:6443

# Or store etcd data on external disk (using template file):
MASTER_IP="<master-node-IP>"  # Change to actual IP
EXTERNAL_ETCD_PATH="/media/de/k8s/etcd"

sed -e "s|<마스터-노드-IP>|${MASTER_IP}|g" \
    -e "s|/media/de/k8s/etcd|${EXTERNAL_ETCD_PATH}|g" \
  /home/user/apps/k8s/templates/kubeadm-config-template.yaml | \
  sudo tee /tmp/kubeadm-config.yaml

sudo kubeadm init --config=/tmp/kubeadm-config.yaml

# Or for high availability (HA) cluster setup:
sudo kubeadm init \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=<master-node-IP> \
  --control-plane-endpoint=<master-node-IP>:6443 \
  --upload-certs
```

**⚠️ Important**: **Must save** the output information after initialization completion:

1. `kubeadm join` command (for adding worker nodes)
2. `--discovery-token-ca-cert-hash` value
3. Token (expires after 24 hours)

### 2. kubeconfig Setup

```bash
# kubeconfig setup for regular user
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Or if running as non-root user:
export KUBECONFIG=$HOME/.kube/config
```

### 3. Pod Network Plugin Installation (CNI)

#### Using Flannel (Recommended)

```bash
# Use local configuration file (with ens15f0 interface specified)
kubectl apply -f /home/user/apps/k8s/config/kube-flannel.yml

# Or download latest version (default settings)
# kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

#### Or Using Calico

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml
```

### 4. Kubernetes Dashboard Installation (Optional)

If you want to check node/Pod status with a simple dashboard, you can use Kubernetes Dashboard. It installs automatically using the provided script.

```bash
# Deploy Dashboard (default: v2.7.0)
bash /home/user/apps/k8s/scripts/deploy-dashboard.sh
```

The script applies the official recommended manifest and creates an `admin-user` ServiceAccount with `cluster-admin` privileges. Generate a token as follows for login:

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

For access, open a proxy and access the Dashboard URL in your browser.

```bash
kubectl proxy
# Open in browser:
# http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

If you want to deploy directly on the node, you can manually run the following commands:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
kubectl apply -f /home/user/apps/k8s/dashboard/dashboard-admin.yaml
```

### 5. Cluster Status Verification

```bash
# Check node status
kubectl get nodes

# Check Pod status
kubectl get pods --all-namespaces

# Wait until all systems are in Running state (about 1-2 minutes)
```

### 6. Docker and Kubernetes Coexistence Verification

Since Docker and Kubernetes use the same containerd, both should work normally:

```bash
# Check Docker containers
docker ps

# Check Kubernetes Pods
kubectl get pods --all-namespaces

# Check all containers in containerd
sudo crictl ps -a

# Verify Docker and Kubernetes use the same containerd
docker info | grep -i containerd
kubectl get nodes -o wide
```

**⚠️ Fixing crictl errors** (dockershim endpoint error):

crictl may cause errors when trying multiple endpoints by default because `dockershim.sock` doesn't exist. Create a configuration file using the template:

```bash
# Create user configuration file (using template file)
cp /home/user/apps/k8s/templates/crictl.yaml ~/.crictl.yaml

# Create configuration file for root user (when using sudo)
sudo cp /home/user/apps/k8s/templates/crictl.yaml /root/.crictl.yaml

# Or create system-wide configuration file
sudo cp /home/user/apps/k8s/templates/crictl.yaml /etc/crictl.yaml

# Verify configuration
crictl ps -a  # Should run without errors
```

**Note**:

- Docker containers and Kubernetes Pods run in different namespaces
- Docker uses the `moby` namespace, while Kubernetes uses the `k8s.io` namespace
- The two systems do not interfere with each other

## Adding Worker Nodes

### 1. Worker Node Preparation

**Complete the following on worker nodes as well:**

- Disable swap
- Install and configure containerd
- Install kubeadm, kubelet, kubectl
- Load kernel modules

### 2. Join Worker Node to Cluster

Run the `kubeadm join` command output during master node initialization on the worker node:

```bash
# Run the join command output from master node
sudo kubeadm join <master-IP>:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

### 3. If Token is Expired (after 24 hours)

```bash
# Generate new token on master node
kubeadm token create --print-join-command

# Run the output command on worker node
```

### 4. Node Verification

```bash
# Check on master node
kubectl get nodes

# Check detailed node information (frequently used in CKA exam)
kubectl get nodes -o wide

# Check specific node information
kubectl describe node <node-name>
```

## Essential Commands for CKA Exam Preparation

### Cluster Management

```bash
# Regenerate token (when token expires)
kubeadm token create --print-join-command

# Check cluster information (frequently required in exam)
kubectl cluster-info

# Check cluster configuration information
kubectl get nodes -o wide

# Check detailed node status
kubectl describe node <node-name>

# Remove master node taint (allow Pod execution on master)
# ⚠️ Use only for single-node clusters or learning purposes
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

# Add/remove taint on specific node (CKA exam problem)
kubectl taint nodes <node-name> key=value:NoSchedule
kubectl taint nodes <node-name> key:NoSchedule-
```

### Quick Cluster Reset (For CKA Practice)

Useful when you need to repeatedly rebuild clusters during exam preparation:

```bash
# Run on all nodes (master + workers)
sudo kubeadm reset -f

# Clean up network and configuration files
sudo rm -rf /etc/cni/net.d
sudo rm -rf /var/lib/cni
sudo rm -rf /var/lib/etcd
sudo rm -rf $HOME/.kube/config
sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X

# Restart containerd
sudo systemctl restart containerd

# Reinitialize on master node
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### Debugging Commands (Important for CKA Exam!)

```bash
# Check kubelet logs
sudo journalctl -u kubelet -f

# Check kubelet status
sudo systemctl status kubelet

# Container runtime logs
sudo journalctl -u containerd -f

# Check etcd status
sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health

# API server logs
kubectl logs -n kube-system kube-apiserver-<node-name>

# Scheduler logs
kubectl logs -n kube-system kube-scheduler-<node-name>

# Controller manager logs
kubectl logs -n kube-system kube-controller-manager-<node-name>
```

## Troubleshooting

For detailed troubleshooting guide, refer to:

- [Troubleshooting Guide](docs/troubleshooting.md)

### Troubleshooting Quick Reference

```bash
# Check kubelet status
sudo systemctl status kubelet

# Check kubelet logs
sudo journalctl -xeu kubelet

# Reset cluster
sudo kubeadm reset -f
sudo rm -rf /etc/cni/net.d /var/lib/cni /var/lib/etcd $HOME/.kube/config
sudo iptables -F && sudo iptables -t nat -F && sudo iptables -t mangle -F && sudo iptables -X
sudo systemctl restart containerd
```

## CKA Exam Preparation

**For detailed CKA exam preparation guide, refer to:**

- [CKA Exam Preparation Guide](docs/cka-exam-preparation.md)

### CKA Quick Reference

```bash
# Set up kubectl auto-completion (time-saving in exam!)
echo 'source <(kubectl completion bash)' >>~/.bashrc
source ~/.bashrc

# Or if using zsh
echo 'source <(kubectl completion zsh)' >>~/.zshrc
source ~/.zshrc
```

## Additional Learning Resources

1. **Official Documentation**: [https://kubernetes.io/docs/]
2. **kubeadm Documentation**: [https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/]
3. **CKA Exam Guide**: [https://www.cncf.io/certification/cka/]
4. **killer.sh CKA Simulator**: Practice problems similar to the actual exam environment

## Related Documents

- [File Structure Guide](docs/file-structure.md)
- [Swap Disabling Guide](docs/swap-disabling-guide.md)
- [External Disk Setup Guide](docs/external-disk-setup.md)
- [Troubleshooting Guide](docs/troubleshooting.md)
- [CKA Exam Preparation Guide](docs/cka-exam-preparation.md)
