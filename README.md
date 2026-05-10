# EXP 7 - Configuration Management with Ansible

## Aim

To understand the basics of Ansible including:
- Inventory
- Playbooks
- Modules

And automate server configuration by installing Nginx on a remote node machine using Ansible playbooks.

---

# Architecture

```text
Local PC
   ↓ SSH
Master Machine (Ansible Installed)
   ↓ SSH using Public Key Authentication
Node Machine
   ↓
Automated Nginx Installation
```

---

# Prerequisites

- AWS Account
- 2 Ubuntu EC2 Instances
- Same Key Pair for both instances
- Internet Connection

---

# Machines Used

| Machine | Purpose |
|---|---|
| Master Machine | Runs Ansible |
| Node Machine | Target machine managed by Ansible |

---

# Step 1: Create EC2 Instances

Create 2 Ubuntu instances in AWS EC2:

1. Master Machine
2. Node Machine

---

# Step 2: Configure Security Groups

## For BOTH Machines

Allow:

| Type | Port |
|---|---|
| SSH | 22 |

## Additional Rule for Node Machine

Later allow:

| Type | Port |
|---|---|
| HTTP | 80 |

---

# Step 3: Connect to Master Machine

## Windows PowerShell

```powershell
ssh -i yourkey.pem ubuntu@MASTER_PUBLIC_IP
```

## Linux/Mac

```bash
ssh -i yourkey.pem ubuntu@MASTER_PUBLIC_IP
```

---

# Step 4: Install Ansible on Master Machine

Update packages:

```bash
sudo apt update
```

Install Ansible:

```bash
sudo apt install -y ansible
```

Check Ansible version:

```bash
ansible --version
```

---

# Step 5: Test Ansible

Run:

```bash
ansible localhost -m ping
```

Expected Output:

```json
"ping": "pong"
```

---

# Step 6: Generate SSH Key on Master Machine

Generate SSH key:

```bash
ssh-keygen
```

Press ENTER for all prompts.

Go to SSH folder:

```bash
cd ~/.ssh
```

List files:

```bash
ls
```

Display public key:

```bash
cat id_ed25519.pub
```

Copy the complete output.

Example:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI.... ubuntu@ip-172-31-xx-xx
```

---

# Step 7: Connect to Node Machine

Open another terminal and connect:

```bash
ssh -i yourkey.pem ubuntu@NODE_PUBLIC_IP
```

---

# Step 8: Add Master Public Key to Node Machine

Run on NODE MACHINE:

```bash
echo "PASTE_MASTER_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
```

Example:

```bash
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI.... ubuntu@ip-172-31-xx-xx" >> ~/.ssh/authorized_keys
```

This enables passwordless SSH access from Master to Node.

---

# Step 9: Return to Master Machine

Go back to master terminal.

Move to home directory:

```bash
cd ~
```

---

# Step 10: Create Inventory File

Open inventory file:

```bash
nano ~/inventory.ini
```

Add the following:

```ini
[node_machines]
NODE_PUBLIC_IP ansible_ssh_user=ubuntu
```

Example:

```ini
[node_machines]
54.123.45.67 ansible_ssh_user=ubuntu
```

Save File:

```text
CTRL + O
ENTER
CTRL + X
```

---

# Step 11: Create Ansible Playbook

Open playbook file:

```bash
nano ~/install_package.yml
```

Paste the following YAML code:

```yaml
---
- name: Install a package on the node machine
  hosts: node_machines
  become: yes

  tasks:

    - name: Update apt cache
      apt:
        update_cache: yes

    - name: Install Nginx
      apt:
        name: nginx
        state: present
```

Save File:

```text
CTRL + O
ENTER
CTRL + X
```

---

# Step 12: Run Ansible Playbook

Execute:

```bash
ansible-playbook -i ~/inventory.ini ~/install_package.yml
```

---

# What Happens Internally

Ansible will:

1. Read inventory file
2. Connect to node machine via SSH
3. Execute tasks from playbook
4. Install Nginx automatically

---

# Expected Output

```text
TASK [Update apt cache]
changed: [NODE]

TASK [Install Nginx]
changed: [NODE]
```

---

# Step 13: Verify Nginx Installation

SSH into node machine:

```bash
ssh ubuntu@NODE_PUBLIC_IP
```

Check Nginx status:

```bash
systemctl status nginx
```

Expected Output:

```text
Active: active (running)
```

---

# Step 14: Allow HTTP Traffic

In AWS:

EC2 → Security Groups → Node Machine → Inbound Rules

Add Rule:

| Type | Port | Source |
|---|---|---|
| HTTP | 80 | Anywhere |

Save rules.

---

# Step 15: Open Nginx Webpage

Open browser:

```text
http://NODE_PUBLIC_IP
```

Expected Output:

```text
Welcome to nginx!
```

---

# Important Concepts

## Inventory

Contains list of target machines.

Example:

```ini
[node_machines]
54.123.45.67 ansible_ssh_user=ubuntu
```

---

## Playbook

YAML file containing automation tasks.

Example:

```yaml
tasks:
  - name: Install Nginx
```

---

## Module

Modules perform tasks.

Example:

```yaml
apt:
  name: nginx
```

---

# Features of Ansible

## 1. Agentless

No software required on node machines.

Uses:
- SSH
- Python

---

## 2. Idempotency

Running the playbook multiple times will not duplicate changes.

Example:
- Nginx already installed
- Ansible skips reinstallation

---

## 3. Check Mode

Dry-run mode.

Command:

```bash
ansible-playbook --check -i inventory.ini install_package.yml
```

Shows changes without applying them.

---

# Common Errors and Fixes

## Permission Denied

Run:

```bash
chmod 400 yourkey.pem
```

---

## Host Key Verification Failed

Run manually once:

```bash
ssh ubuntu@NODE_PUBLIC_IP
```

Type:

```text
yes
```

---

## UNREACHABLE Error

Possible reasons:
- Wrong IP
- SSH not allowed
- Public key not copied correctly

---

## YAML Formatting Error

YAML uses indentation.

Correct:

```yaml
tasks:
  - name:
```

Wrong:

```yaml
tasks:
-name:
```

---

# Important Commands Summary

## Install Ansible

```bash
sudo apt install -y ansible
```

---

## Generate SSH Key

```bash
ssh-keygen
```

---

## View Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

---

## Create Inventory File

```bash
nano ~/inventory.ini
```

---

## Create Playbook

```bash
nano ~/install_package.yml
```

---

## Run Playbook

```bash
ansible-playbook -i ~/inventory.ini ~/install_package.yml
```

---

# Viva Questions

## What is Ansible?

Ansible is an automation and configuration management tool used to manage servers automatically.

---

## What is an Inventory File?

A file containing list of target machines.

---

## What is a Playbook?

A YAML file containing automation tasks.

---

## Which protocol does Ansible use?

SSH.

---

## What is Agentless Architecture?

No software/agent required on node machines.

---

## What is Idempotency?

Running tasks multiple times gives same result without duplication.

---

## What is become: yes?

Runs tasks with sudo privileges.

---

# Result

Successfully installed and managed Nginx on a remote node machine using Ansible Playbooks and Inventory files.
