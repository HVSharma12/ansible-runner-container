# ansible-runner-container

This repository provides a simple **Ansible Runner** setup that runs a playbook inside a **containerized Ansible Runner environment**.

- No need to install Ansible or Ansible-runner directly on the VM.
- All SSH connections go through the VM running the container, not directly inside the container.
- Each playbook execution spawns a new ephemeral Podman container.
- Supports parallel execution, allowing multiple containers to run side by side.
- Integration with ITSM tools via a REST API for workflows (See [ansible-runner-service](https://github.com/HVSharma12/ansible-runner-service))
- Execution logs and artifacts are stored on the VM, not inside the container.
- Based on [ansible-container](https://github.com/ansible-container/)
- Image used opensuse/tumbleweed:latest, Tested on SLE15-SP5

## Ansible runner commands

The ansible-runner commands are provided as symlinks to ansible-wrapper.sh. The commands will instantiate the container and execute the ansible-runner command.

## Getting Started

### Build the Ansible Runner Container Image

Before running Ansible Runner, pull the container image:

```sh
cd container
podman build -t ansible-runner .
```

---

### Install Ansible Runner (Root User)
note: the container be installed as as non-root user-todo.

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
### Setting Up Passwordless SSH
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""

ssh-copy-id -i ~/.ssh/id_rsa.pub root@xx.xxx.xxx.xx

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
### Outputting json (raw event data) to the console instead of normal output

Add -j argument on the command line:
```sh
ansible-runner run ansible-runner -j -p playbook.yml
```

---
### Artifact Integration with Elasticsearch & Similar Tools
One of the benefits of using Ansible Runner is that artifacts (logs, execution data, and playbook outputs) are stored in structured directories.
These artifacts can be easily indexed by Elasticsearch, Logstash, or other tools for analysis and visualization.

![Elasticsearch Integration](https://github.com/HVSharma12/ansible-runner-container/blob/main/Screenshot_20250207_060322.png)

---
## Storing and Using Ansible Vault for Secrets

To **encrypt sensitive data**, you can use **Ansible Vault** within the containerized Ansible Runner.

### **Encrypt sensitive data**
```sh
ansible-vault create ansible-runner/project/secrets.yml
```
Add secrets:
```yaml
db_password: "SuperSecretPassword"
api_token: "123456abcdef"
```

### **Using Vault in `env/` Directory**
Store vault password in `ansible-runner/env/passwords`:
```sh
echo "myvaultpassword" > ansible-runner/env/passwords
```
Define the Vault Password File in cmdline:
```sh
echo "--vault-password-file /work/ansible-runner/env/passwords" > ansible-runner/env/cmdline

or pass option -k

```
Run playbooks without manual password entry:
```sh
ansible-runner run ansible-runner -p vault_test_playbook.yml
```
---

---
### Sync Ansible projects from GitHub to the project

Run the below playbook to sync projects from github
```sh
ansible-runner run ansible-runner --project-dir ansible-runner/runner_manager -p sync_github_projects.yml
```
The list of projects and their GitHub repositories is managed in:

```yaml
cat ansible-runner/runner_manager/roles/github_sync/vars/project_config.yml 
---
projects:
  test-playbooks:
    git_url: "https://github.com/ansible/test-playbooks.git"
    sync: true
  system-roles:
    git_url: "https://github.com/linux-system-roles/cockpit.git"
    sync: false  # Will NOT sync

```

---
### Uninstalling Ansible Runner

```sh
podman container runlabel uninstall ansible-runner
```
---
## Setting up Ansible Runner Service

Users can also set up an ansible-runner-service to enable the following capabilities:

  - Execution of playbooks via a REST API instead of CLI.
  - Integration with ITSM tools for workflows.
  - Running multiple playbooks programmatically with API-driven management.
  - Inventory management through API endpoints.
  - Secure execution using HTTPS and mutual TLS authentication.
  - Querying playbook execution state and logs through API endpoints.

For setup and installation, refer to the GitHub repository: [ansible-runner-service](https://github.com/HVSharma12/ansible-runner-service)

---

## Directory Structure

```
ansible-runner/
├── env/               # (Optional) Environment variables and settings
│   ├── envvars        # Environment variables in JSON/YAML format
│   ├── extravars      # Extra variables passed to playbook execution
│   ├── passwords      # Securely stored passwords for SSH/Vault
│   ├── cmdline        # Command-line arguments for Ansible execution
│   ├── ssh_key        # SSH private key for remote authentication
│   ├── settings       # Runner-specific settings like timeouts
├── inventory/         # Hosts file for Ansible inventory
│   ├── hosts          # Define target systems for playbook execution
├── project/           # Contains playbooks and related Ansible content
│   ├── playbook.yml   # Sample Ansible playbook
│   ├── roles/         # Optional Ansible roles directory
│   ├── modules/       # Optional modules if custom Ansible modules are needed
├── artifacts/         # Stores execution logs and results
│   ├── <job_id>/      # Each playbook run generates a new folder
│   │   ├── rc         # Return code of execution
│   │   ├── status     # Execution status (success, failure, timeout)
│   │   ├── stdout     # Standard output logs from execution
│   │   ├── fact_cache/ # JSON-based fact cache for host facts
│   │   ├── job_events/ # Structured event logs for each task
│   │   ├── profiling_data/ # Resource profiling (CPU, Memory, PIDs)
├── profiling_data/    # If enabled, stores execution profiling data
│   ├── <job_id>-cpu.json    # CPU profiling per task
│   ├── <job_id>-memory.json # Memory profiling per task
│   ├── <job_id>-pids.json   # Process count profiling per task
```
Refer -https://ansible.readthedocs.io/projects/runner/en/stable/intro/

---
To-Do
- add support for non-root container
- add more examples
