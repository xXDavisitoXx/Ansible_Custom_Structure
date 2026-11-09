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
ansible-galaxy collection init Homelab.Updater
```
```text
.
|-- README.md
|-- docs
|-- galaxy.yml
|-- meta
|   `-- runtime.yml
|-- plugins
    `-- README.md
```
create rour diferent roles (category OS example)
```bash
cd Homelab/Updater
ansible-galaxy role init roles/debian
```

```bash
.
|-- README.md
|-- docs
|-- galaxy.yml
|-- meta
|   `-- runtime.yml
|-- plugins
|   `-- README.md
`-- roles
    `-- debian
        |-- README.md
        |-- defaults
        |   `-- main.yml
        |-- files
        |-- handlers
        |   `-- main.yml
        |-- meta
        |   `-- main.yml
        |-- tasks
        |   |-- main.yml
        |-- templates
        |-- tests
        |   |-- inventory
        |   `-- test.yml
        `-- vars
            `-- main.yml
```
create your diferent tasks:
```bash
cd roles/debian/tasks
nano update.yml
nano dist-upgrade.yml
nano autoremove.yml
nano autoclean.yml
```

update.yml 
```yml
---
- name: Update APT package cache
  ansible.builtin.apt:
    update_cache: true
    cache_valid_time: 3600
  become: true
 ```
dist-upgrade.yml 
```yml
---
- name: Upgrade installed packages
  ansible.builtin.apt:
    upgrade: dist
  become: true
 ```
autoremove.yml  
```yml
---
- name: Remove unused packages and purge configuration
  ansible.builtin.apt:
    autoremove: true
    purge: true
  become: true
 ```
autoclean.yml 
```yml
---
- name: Clean obsolete APT package archives
  ansible.builtin.apt:
    autoclean: true
  become: true
 ```
reboot-check.yml
```yml 
- name: Check if a system reboot is required by Debian
  stat:
    path: /var/run/reboot-required
  register: reboot_file

- name: Notify administrator if reboot is pending
  debug:
    msg: "⚠️ ATENCIÓN: El servidor {{ inventory_hostname }} requiere un reinicio para aplicar actualizaciones críticas del kernel o librerías."
 ```
reboot.yml 
```yml 
---
- name: Check if a system reboot is required
  ansible.builtin.stat:
    path: /var/run/reboot-required
  register: reboot_required_file

- name: Reboot the Debian server if required
  ansible.builtin.reboot:
    msg: "🔄 Reiniciando el sistema por actualización pendiente del kernel gestionada por Ansible."
    connect_timeout: 5
    reboot_timeout: 300
    pre_reboot_delay: 0
    post_reboot_delay: 30
    test_command: uptime
  when: reboot_required_file.stat.exists
  register: reboot_result

- name: Notify administrator that reboot was performed
  ansible.builtin.debug:
    msg: "✅ El servidor {{ inventory_hostname }} se ha reiniciado correctamente y vuelve a estar operativo."
  when: reboot_result is changed
  ```
add your tasks to main.yml task:
```yml
---
# tasks file for roles/debian
- name: Update APT cache
  ansible.builtin.import_tasks: update.yml

- name: Upgrade system packages
  ansible.builtin.import_tasks: dist-upgrade.yml

- name: Remove unused packages
  ansible.builtin.import_tasks: autoremove.yml

- name: Clean APT cache
  ansible.builtin.import_tasks: autoclean.yml

- name: Check pending system reboot
  ansible.builtin.import_tasks: reboot-check.yml

- name: Handle system reboot if needed
  ansible.builtin.import_tasks: reboot.yml
```
Create collections home and move updater to collections default home ansible

```bash
mkdir -p ~/Homelab/ansible_collections/homelab/updater
mv ~/Homelab/Updater /srv/Homelab/ansible_collections/homelab/updater/
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
ansible_ssh_private_key_file=~/.ssh/Ansible.key
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
apt install openssh-server sudo 
```
```bash
useradd -m -s /bin/bash ansible
passwd ansible
```
```bash
sudo usermod -aG sudo ansible
```

