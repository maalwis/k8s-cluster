# SSH Access Setup for Vagrant VMs

## Prerequisites

- Vagrant VMs provisioned and running
- Existing SSH key pair on host machine (`id_ed25519` and `id_ed25519.pub`)

## VM Configuration

The setup includes three VMs:
- **Control Plane**: `k8s-master` - 192.168.56.10
- **Worker Node 01**: `k8s-node-01` - 192.168.56.11
- **Worker Node 02**: `k8s-node-02` - 192.168.56.12

---

## Step 1: Verify SSH Keys Exist

First, confirm your SSH keys are present on your host machine:

```bash
ls -la ~/.ssh/id_ed25519*
```

**Expected output:**
- `id_ed25519` (private key)
- `id_ed25519.pub` (public key)

---

## Step 2: View Your Public Key

Display your public key content:

```bash
cat ~/.ssh/id_ed25519.pub
```

**Example output:**
```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... your_email@example.com
```

Copy this entire line - you'll need it in the next step.

---

## Step 3: Add Public Key to Each VM

For each VM, SSH in using Vagrant and add your public key to the authorized keys.

### Control Plane (k8s-master)

```bash
# SSH into the VM
vagrant ssh control-plane

# Add your public key to authorized_keys
echo "YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys

# Set proper permissions
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

# Exit the VM
exit
```

### Worker Node 01

```bash
# SSH into the VM
vagrant ssh worker-node-01

# Add your public key to authorized_keys
echo "YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys

# Set proper permissions
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

# Exit the VM
exit
```

### Worker Node 02

```bash
# SSH into the VM
vagrant ssh worker-node-02

# Add your public key to authorized_keys
echo "YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys

# Set proper permissions
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

# Exit the VM
exit
```

> **Note:** Replace `YOUR_PUBLIC_KEY_HERE` with the actual public key content from Step 2.

---

## Step 4: Remove Old Host Keys (If Recreating VMs)

If you've previously connected to VMs at these IP addresses, remove the old host keys:

```bash
ssh-keygen -f ~/.ssh/known_hosts -R '192.168.56.10'
ssh-keygen -f ~/.ssh/known_hosts -R '192.168.56.11'
ssh-keygen -f ~/.ssh/known_hosts -R '192.168.56.12'
```

This prevents "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!" errors.

---

## Step 5: Test Direct SSH Access

Test SSH access to each VM using their IP addresses:

```bash
# Test control plane
ssh vagrant@192.168.56.10

# Test worker node 01
ssh vagrant@192.168.56.11

# Test worker node 02
ssh vagrant@192.168.56.12
```

On first connection, you'll be prompted to verify the host fingerprint. Type `yes` to continue.

---

## Step 6: Configure SSH Shortcuts (Optional)

Create or edit `~/.ssh/config` to add convenient aliases:

```bash
cat >> ~/.ssh/config << 'EOF'

Host k8s-master
    HostName 192.168.56.10
    User vagrant
    IdentityFile ~/.ssh/id_ed25519

Host k8s-node-01
    HostName 192.168.56.11
    User vagrant
    IdentityFile ~/.ssh/id_ed25519

Host k8s-node-02
    HostName 192.168.56.12
    User vagrant
    IdentityFile ~/.ssh/id_ed25519
EOF
```

### Usage with SSH Config

After adding the config, you can connect using simple aliases:

```bash
ssh k8s-master
ssh k8s-node-01
ssh k8s-node-02
```