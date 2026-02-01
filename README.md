# Kubernetes Cluster using Vagrant & Ansible

This project provisions a **local Kubernetes cluster** using **Vagrant** for virtual machines and **Ansible** for system configuration and Kubernetes prerequisites.

---

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

## 📁 Project Structure

```text
k8s-cluster/
├── ansible/
│   ├── ansible.cfg
│   ├── inventory/
│   │   └── hosts.ini
│   └── playbooks/
│       ├── 01-swapoff.yml
│       ├── 02-kernel_module.yml
│       └── 03-sysctl-config.yml
├── vagrant/
│   └── Vagrantfile
├── vagrant-up.sh
├── vagrant-down.sh
├── ansible-notes.md
├── ansible-vagrant-setup.md
├── ssh-vagrant-setup.md
└── README.md
