# Ansible Role: Docker Installation on Debian with DNS and Registry Mirror

This Ansible project automates the installation and configuration of Docker on **Debian-based systems**. It ensures deprecated versions are removed and a clean, optimized Docker environment is installed.

> 🛡️ This role is tailored for users in Iran — it configures DNS and Docker registry mirror to bypass Docker access restrictions and deliver a fully functional, ready-to-use Docker environment without censorship issues.
> 

---

## 📁 Project Structure

### `site.yaml`

The entry-point playbook:

```yaml
- hosts: all
  become: true
  roles:
    - common

```

---

## Role: `common`

### Task Breakdown

- **`main.yaml`**
    
    Loads Debian-specific tasks only if the target OS is Debian:
    
    ```yaml
    - name: Execute only if the OS is Debian-based
      include_tasks: docker-debian.yaml
      when: ansible_facts["os_family"] == "Debian"
    
    ```
    
- **`docker-debian.yaml`**
    
    Orchestrates all the major steps:
    
    - DNS setup
    - Removing old Docker versions
    - Installing dependencies
    - Adding Docker repository
    - Installing Docker packages
    - Setting registry mirror
    - Configuring Docker user and services

---

### Key Task Files

- **`set-dns.yaml`**
    
    Adds necessary `nameserver` entries to `/etc/resolv.conf`.
    
- **`requirements.yaml`**
    
    Removes conflicting/old Docker versions before a fresh install.
    
- **`install-docker.yaml`**
    - Installs packages like `curl`, `ca-certificates`
    - Sets up Docker GPG key & APT repository
    - Installs Docker components:
        - `docker-ce`
        - `docker-ce-cli`
        - `containerd.io`
        - `docker-buildx-plugin`
        - `docker-compose-plugin`
- **`set-registery.yaml`**
    
    Sets Docker registry mirror to bypass regional restrictions:
    
    ```json
    {
      "registry-mirrors": [
        "https://docker.iranserver.com"
      ]
    }
    
    ```
    
- **`config.yaml`**
    - Creates `docker` group
    - Adds `{{ user }}` to the group
    - Enables and starts Docker services
    - Optional logging config

---

## Variables

Defined in `roles/common/vars/main.yaml`:

- `packages_to_remove`: List of old Docker packages
- `user`: The user to be added to the Docker group

---

## Handlers

- `update apt cache`: Refreshes package lists
- `restart docker`: Restarts Docker and reloads systemd config

---

## How to Run

### Using become password:

```bash
ansible-playbook site.yaml -i ./inventory/hosts.ini --ask-become-pass
```

### Using Vault for become password:

```bash
ansible-playbook site.yaml -i ./inventory/hosts.ini --ask-vault-pass
```

### Using Vault password file:

```bash
ansible-playbook site.yaml -i ./inventory/hosts.ini --vault-password-file ~/.vault_pass.txt
```

---

## Notes

- Add your system username to `inventory/hosts.ini` and `roles/common/vars/main.yaml`
- You can place your `become` vault password under `inventory/host_vars/`
- Supports both local and remote Debian-based hosts
