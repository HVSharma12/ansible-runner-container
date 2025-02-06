# ansible-runner-container

This repository provides a simple **Ansible Runner** setup that runs a playbook inside a **containerized Ansible Runner environment**.

- No need to install Ansible or Ansible-runner directly on the VM.
- All Ansible tasks execute via SSH to the VM hosting the container or other remote systems.
- All SSH connections go through the VM running the container, not directly inside the container.
- A new container is created for each playbook run.
- Execution logs and artifacts are stored on the VM, not inside the container.
- Based on [ansible-container](https://github.com/SUSE/ansible-container/)
- Image used OpenSUSE Tumbleweed, Tested on SLE15-SP5

## Ansible runner commands

The ansible-runner commands are provided as symlinks to ansible-wrapper.sh. The commands will instantiate the container and execute the ansible-runner command.

## Getting Started

### Pull the Ansible Runner Container Image

Before running Ansible Runner, pull the container image:

```sh
podman pull registry.opensuse.org/home/hsharma/15.6/ansible-runner:latest
```

---

### Install Ansible Runner (Root User)

To install **Ansible Runner as root**, run:

```sh
podman container runlabel install ansible-runner
```

This places the `ansible-runner` command inside `/usr/local/bin`, making it available system-wide.

---

### Add Hosts to the Inventory File

Before running a playbook, you need to define your inventory in the **`ansible-runner/inventory/hosts`** file.

Example **inventory/hosts** file:

```ini
[servers]
server1 ansible_host=192.168.1.10 ansible_user=root
server2 ansible_host=192.168.1.20 ansible_user=root
```

---

### Run the Sample Playbook

Once the inventory file is updated, you can run the **sample playbook** using:

```sh
ansible-runner run ansible-runner -p playbook.yml
```

---

### Check Execution Logs

After execution, logs and artifacts will be stored in:

```sh
ansible-runner/artifacts/
```

To inspect the logs:

```sh
ls -l ansible-runner/artifacts/
cat ansible-runner/artifacts/<job_id>/stdout
```

---
### Uninstalling Ansible Runner

```sh
podman container runlabel uninstall ansible-runner
```
---

## Directory Structure

```
ansible-runner/
├── env/               # (Optional) Environment variables
├── inventory/
│   ├── hosts         # Inventory file (edit this)
├── project/
│   ├── playbook.yml  # Sample Ansible playbook
├── artifacts/        # Stores logs & execution artifacts
```

---
- add support for non-root
