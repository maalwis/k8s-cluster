# Ansible Configuration for Vagrant VMs

## Prerequisites

- Ansible installed on your host machine
- SSH access configured to VMs (see `ssh-vagrant-setup.md`)
- VMs running and accessible via SSH

## Project Structure

```
.
├── ansible.cfg           # Ansible configuration file
├── inventory/
│   └── hosts.ini        # Inventory file with VM definitions
└── playbooks/
    └── prereqs.yml      # Sample playbook for K8s prerequisites
```

---

## Installation

### Install Ansible

**Ubuntu/Debian:**
```bash
sudo apt update
sudo apt install ansible -y
```

**macOS:**
```bash
brew install ansible
```

**Verify installation:**
```bash
ansible --version
```

---

## Configuration Files

### 1. Inventory File (`inventory/hosts.ini`)

The inventory file defines your VMs and their connection details:

```ini
[control_plane]
k8s-master ansible_host=192.168.56.10

[workers]
k8s-node-01 ansible_host=192.168.56.11
k8s-node-02 ansible_host=192.168.56.12

[k8s_cluster:children]
control_plane
workers

[all:vars]
ansible_user=vagrant
ansible_ssh_private_key_file=~/.ssh/id_ed25519
ansible_python_interpreter=/usr/bin/python3
```

**Explanation:**
- `[control_plane]` - Group for master node(s)
- `[workers]` - Group for worker nodes
- `[k8s_cluster:children]` - Parent group containing all nodes
- `[all:vars]` - Variables applied to all hosts
  - `ansible_user` - SSH user (vagrant)
  - `ansible_ssh_private_key_file` - Path to your SSH private key
  - `ansible_python_interpreter` - Python path on remote hosts

### 2. Ansible Configuration (`ansible.cfg`)

This file contains Ansible settings for your project:

```ini
[defaults]
inventory = ./inventory/hosts.ini
host_key_checking = False
become = True
become_method = sudo
become_user = root
remote_user = vagrant
private_key_file = ~/.ssh/id_ed25519
forks = 5
gathering = smart
stdout_callback = yaml
retry_files_enabled = False

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no
pipelining = True
```

**Key settings:**
- `host_key_checking = False` - Skips SSH host key verification (safe for local VMs)
- `become = True` - Automatically use sudo for privileged operations
- `stdout_callback = yaml` - Better formatted output
- `pipelining = True` - Faster SSH connections

---

## Testing Ansible Connectivity

### 1. Test Connection to All Hosts

```bash
ansible all -m ping
```

**Expected output:**
```yaml
PLAY [Ansible Ad-Hoc] *********************************************************************************************

   TASK [ping] ****************************************************************************************************
ok: [k8s-master]
ok: [k8s-node-02]
ok: [k8s-node-01]

   PLAY RECAP *****************************************************************************************************
k8s-master                 : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
k8s-node-01                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
k8s-node-02                : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

---

## Running Playbooks

### 1. Check Playbook Syntax

```bash
ansible-playbook playbooks/prereqs.yml --syntax-check
```

### 2. Dry Run (Check Mode)

See what changes would be made without actually making them:

```bash
ansible-playbook playbooks/prereqs.yml --check
```

### 3. Run the Playbook

```bash
ansible-playbook playbooks/prereqs.yml
```

### 4. Run with Verbose Output

```bash
# -v: Verbose
# -vv: More verbose
# -vvv: Very verbose (connection debugging)
# -vvvv: Extreme verbosity

ansible-playbook playbooks/prereqs.yml -v
```

---

## Common Ansible Commands

### Inventory Commands

```bash
# List all hosts
ansible all --list-hosts

# List hosts in specific group
ansible workers --list-hosts

# Show inventory in graph format
ansible-inventory --graph

# Show all variables for a host
ansible-inventory --host k8s-master
```
