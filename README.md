# EXP 7 – Configuration Management with Ansible

## Aim
To automate server configuration using Ansible playbooks by installing Nginx on a node machine from a master machine.

---

# Step 1: Create 2 Ubuntu EC2 Instances

Create the following instances in AWS EC2:

1. Master Machine (Ubuntu)
2. Node Machine (Ubuntu)

Make sure:
- Both instances are running
- Both instances allow SSH (Port 22)

---

# Step 2: Install Ansible on Master Machine

Run the following commands on the MASTER machine:

```bash
sudo apt update
sudo apt install -y ansible
ansible --version
ansible localhost -m ping
```

Expected Output:
```bash
"ping": "pong"
```

---

# Step 3: Configure SSH Access to Node Machine

Run the following commands on the MASTER machine:

```bash
ssh-keygen
cd ~/.ssh
ls
cat id_ed25519.pub
```

Copy the complete output of:
```bash
cat id_ed25519.pub
```

---

# Step 4: Add Public Key to Node Machine

Connect to the NODE machine and run:

```bash
echo "PASTE_COPIED_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
```

Example:
```bash
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHaG/JyaFIkfiwNi/ao9mjC7qfP9kL11o0v3aWBNNhex ubuntu@ip-172-31-74-57" >> ~/.ssh/authorized_keys
```

---

# Step 5: Create Inventory File

Return to the MASTER machine and run:

```bash
nano ~/inventory.ini
```

Add the following content:

```ini
[node_machines]
NODE_PUBLIC_IP ansible_ssh_user=ubuntu
```

Example:
```ini
[node_machines]
3.239.119.125 ansible_ssh_user=ubuntu
```

Save the file:
- CTRL + O
- ENTER
- CTRL + X

---

# Step 6: Create Ansible Playbook

Run on MASTER machine:

```bash
nano ~/install_package.yml
```

Add the following content:

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

Save the file:
- CTRL + O
- ENTER
- CTRL + X

---

# Step 7: Run the Ansible Playbook

Run on MASTER machine:

```bash
ansible-playbook -i ~/inventory.ini ~/install_package.yml
```

This command:
- Reads the inventory file
- Connects to the node machine using SSH
- Executes the playbook tasks
- Installs Nginx automatically

---

# Step 8: Verify Nginx Installation

Run on MASTER machine:

```bash
ssh ubuntu@NODE_PUBLIC_IP
```

Then run:

```bash
systemctl status nginx
```

Expected Output:
```bash
Active: active (running)
```

---

# Step 9: Allow HTTP Port 80 in AWS

Go to:
- EC2 Dashboard
- Security Groups
- Node Machine Security Group
- Inbound Rules

Add Rule:

| Type | Protocol | Port Range | Source |
|------|-----------|------------|---------|
| HTTP | TCP | 80 | Anywhere IPv4 |

Save the rule.

---

# Step 10: Open Nginx Web Page

Open browser and type:

```bash
http://NODE_PUBLIC_IP:80
```

Expected Output:
- Welcome to nginx page

---

# Result

Successfully automated server configuration using Ansible playbooks and installed Nginx on the node machine.

---

# Important Commands Summary

```bash
sudo apt update
sudo apt install -y ansible
ansible --version
ansible localhost -m ping

ssh-keygen

cat ~/.ssh/id_ed25519.pub

nano ~/inventory.ini

nano ~/install_package.yml

ansible-playbook -i ~/inventory.ini ~/install_package.yml

ssh ubuntu@NODE_PUBLIC_IP

systemctl status nginx
```
