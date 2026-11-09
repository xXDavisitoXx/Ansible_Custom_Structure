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

Create namespace.colection estructure
```bash
ansible-galaxy collection init Homelab.Updater-debian
```
Create collections home and move updater

```bash
mkdir -p /srv/Homelab/ansible_collections/homelab/updater
mv /srv/Homelab/Updater /srv/Homelab/ansible_collections/homelab/updater/
```
Create inventory file

in your root namespace

```bash
mkdir inventory
nano inventory/hosts
```
example:
```conf
root@Ansible:/srv/Homelab# cat inventory/hosts 
# ==========================================
# DEBIAN
# ==========================================
[debian]
debian1 ansible_host=192.168.100.113


# ==========================================
# ALMALINUX
# ==========================================
[alma]
alma ansible_host=192.168.100.114


# ==========================================
# SUSE
# ==========================================
[suse]
suse ansible_host=192.168.100.115


# ==========================================
# ARCH LINUX
# ==========================================
[arch]
arch ansible_host=192.168.100.116


# ==========================================
# ANSIBLE
# ==========================================
[sin_clasificar]
ansible ansible_host=192.168.100.112


# ==========================================
# VARIABLES GLOBALES
# ==========================================
[all:vars]
ansible_user=ansible
ansible_ssh_private_key_file=/root/.ssh/Ansible.key
ansible_become=true
ansible_become_method=sudo
```

create .vault-pass in your root namespace and insert your master key
```bash
echo 'YOUR-PASS' > .vault-ansible
```

create your vault for group hosts:
```bash
echo 'ansible_become_pass: "YOUR-PASS"' > inventory/group_vars/debian/vault-debian.yml
```

encrypt the password file
```bash
 ansible-vault encrypt inventory/group_vars/vault-debian.yml
```

define ansible.cfg in your root namespace
```bash
nano ansible.cfg
```
```conf
defaults]
# Ruta principal donde Ansible busca las colecciones (debe apuntar al directorio padre de ansible_collections)
collections_path = /srv/Homelab/

# Inventario por defecto
inventory = inventory/hosts

# Desactiva la comprobaci  n estricta de claves SSH
host_key_checking = False

# Desactiva la creaci  n de archivos .retry innecesarios
retry_files_enabled = False

# Si usas un archivo maestro para desbloquear los vaults de forma autom  tica:
vault_password_file = .vault-ansible
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

