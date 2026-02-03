# Kubernetes Cluster Setup with Vagrant and Ansible

Automated Kubernetes cluster deployment using Vagrant for VM provisioning and Ansible for configuration management.

## Overview

This project provides a complete automation solution for setting up a multi-node Kubernetes cluster with:
- **Control Plane**: 1 master node
- **Worker Nodes**: 2 worker nodes
- **Container Runtime**: containerd
- **CNI Plugin**: Cilium
- **Kubernetes Version**: v1.35

## Prerequisites

- [Vagrant](https://www.vagrantup.com/) (2.3.0+)
- [VirtualBox](https://www.virtualbox.org/) (6.1+)
- [Ansible](https://www.ansible.com/) (2.15+)
- Host machine with at least 8GB RAM

## 🧱 Architecture

- **1 Control Plane (Master)**
- **2 Worker Nodes**
- OS: **Ubuntu 22.04 (Jammy)**
- Provider: **VirtualBox**
- Networking: **Private network (192.168.56.0/24)**

| Node Name        | Hostname     | IP Address        | RAM  | CPU |
|------------------|-------------|-------------------|------|-----|
| control-plane    | k8s-master  | 192.168.56.10     | 8 GB | 2   |
| worker-node-01   | k8s-node-01 | 192.168.56.11     | 2 GB | 2   |
| worker-node-02   | k8s-node-02 | 192.168.56.12     | 2 GB | 2   |

---

## Project Structure

```
.
├── ansible/
│   ├── ansible.cfg                    # Ansible configuration
│   ├── inventory/
│   │   └── hosts.ini                  # Inventory file with node definitions
│   └── playbooks/
│       ├── 01_swapoff.yml             # Disable swap (required by Kubernetes)
│       ├── 02_kernel_module.yml       # Load required kernel modules
│       ├── 03_sysctl_config.yml       # Configure kernel parameters
│       ├── 04_containerd.yml          # Install and configure containerd
│       ├── 05_kubernetes_tools.yml    # Install kubeadm, kubelet, kubectl
│       ├── 06_kubeadm_init.yml        # Configure kubelet node IP
│       ├── 07_control_plane_init.yml  # Initialize Kubernetes control plane
│       ├── 08_cilium.yml              # Install Cilium CNI
│       └── 09_worker_join.yml         # Join worker nodes to cluster
├── vagrant/
│   └── Vagrantfile                    # VM definitions and networking
├── vagrant-up.sh                      # Helper script to start VMs
├── vagrant-down.sh                    # Helper script to stop VMs
├── ansible-notes.md                   # Ansible usage notes
├── ansible-vagrant-setup.md           # Setup documentation
├── ssh-vagrant-setup.md               # SSH configuration guide
└── README.md                          # This file
```

## Quick Start

### 1. Clone the Repository

```bash
git clone <repository-url>
cd <repository-name>
```

### 2. Start Vagrant VMs

```bash
./vagrant-up.sh
# or
cd vagrant && vagrant up
```

This creates three VMs:
- `k8s-master` (192.168.56.10) - 2 vCPUs, 2GB RAM
- `k8s-node-01` (192.168.56.11) - 2 vCPUs, 2GB RAM
- `k8s-node-02` (192.168.56.12) - 2 vCPUs, 2GB RAM

### 3. Run Ansible Playbooks

```bash
cd ansible

# Run all playbooks in sequence
ansible-playbook playbooks/01_swapoff.yml
ansible-playbook playbooks/02_kernel_module.yml
ansible-playbook playbooks/03_sysctl_config.yml
ansible-playbook playbooks/04_containerd.yml
ansible-playbook playbooks/05_kubernetes_tools.yml
ansible-playbook playbooks/06_kubeadm_init.yml
ansible-playbook playbooks/07_control_plane_init.yml
ansible-playbook playbooks/08_cilium.yml
ansible-playbook playbooks/09_worker_join.yml
```

### 4. Access the Cluster

```bash
# SSH into master node
vagrant ssh k8s-master

# Check cluster status
sudo kubectl get nodes --kubeconfig=/etc/kubernetes/admin.conf
sudo kubectl get pods -A --kubeconfig=/etc/kubernetes/admin.conf
```

## Playbook Details

### System Preparation (All Nodes)

1. **01_swapoff.yml** - Disables swap memory (Kubernetes requirement)
2. **02_kernel_module.yml** - Loads overlay and br_netfilter kernel modules
3. **03_sysctl_config.yml** - Configures IP forwarding and bridge netfilter

### Container Runtime (All Nodes)

4. **04_containerd.yml** - Installs containerd and enables systemd cgroup driver

### Kubernetes Tools (All Nodes)

5. **05_kubernetes_tools.yml** - Installs kubeadm, kubelet, kubectl and pins versions
6. **06_kubeadm_init.yml** - Configures kubelet to use host-only network interface

### Control Plane Setup (Master Only)

7. **07_control_plane_init.yml** - Initializes control plane with kubeadm
8. **08_cilium.yml** - Installs Cilium CNI with cluster-pool IPAM

### Worker Node Setup (Workers Only)

9. **09_worker_join.yml** - Joins worker nodes to the cluster

## Network Configuration

- **Host-Only Network**: 192.168.56.0/24 (private cluster communication)
- **NAT Network**: For internet access
- **Pod Network CIDR**: 10.244.0.0/16
- **Service Network CIDR**: 10.96.0.0/12

## Verification

```bash
# Check all nodes are Ready
sudo kubectl get nodes --kubeconfig=/etc/kubernetes/admin.conf

# Check all system pods are running
sudo kubectl get pods -A --kubeconfig=/etc/kubernetes/admin.conf

# Check Cilium status
sudo cilium status

# Run connectivity test
sudo cilium connectivity test
```

## Cleanup

```bash
# Stop and destroy VMs
./vagrant-down.sh
# or
cd vagrant && vagrant destroy -f
```

## Troubleshooting

### Nodes Not Ready
- Verify Cilium is running: `sudo cilium status`
- Check kubelet logs: `sudo journalctl -u kubelet -f`

### Network Issues
- Verify host-only network: `ip addr show enp0s8`
- Check kernel modules: `lsmod | grep -E 'overlay|br_netfilter'`

### Join Failed
- Regenerate token on master: `sudo kubeadm token create --print-join-command`
- Verify containerd is running: `sudo systemctl status containerd`

## Documentation

- [ansible-vagrant-setup.md](ansible-vagrant-setup.md) - Detailed setup guide
- [ssh-vagrant-setup.md](ssh-vagrant-setup.md) - SSH configuration for Vagrant

