# Ansible_Custom_Structure

Server
```bash
apt install ansible
```
```bash
ssh-keygen -t ed25519 -C "Ansible"
```
```bash
ssh-copy-id -i ~/.ssh/Ansible.key ansible@192.168.100.113
```

Host 

```bash
apt install sudo 
```
```bash
useradd -m -s /bin/bash ansiblev
passwd ansible
```
```bash
sudo usermod -aG sudo ansible
```
