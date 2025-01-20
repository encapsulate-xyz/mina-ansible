# Ansible Playbook to deploy Mina

This Ansible playbook automates the deployment and configuration of Mina execution and consensus clients. It ensures that the necessary dependencies, configuration files, and services are properly set up and running.

## Table of Contents

- [Requirements](#requirements)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Variables](#variables)
- [Usage](#usage)

## Requirements

Before using this playbook, ensure the following requirements are met:

1. **Ansible version**: Make sure you have Ansible 2.15+ installed.
2. **SSH Access**: Passwordless SSH access to all target servers.
3. **Python**: Python 3.x installed on the control node and all target hosts.
4. **Privileges**: The user running the playbook must have sudo privileges on the target machines.

## Prerequisites

**Install HashiCorp Vault**

This playbook relies on HashiCorp Vault to securely retrieve sensitive files, such as validator keys and secrets. Follow the [HashiCorp Vault Installation Guide](https://developer.hashicorp.com/vault/tutorials/getting-started/getting-started-install) to set up Vault on your infrastructure.

**Note on Secrets Management**

The playbook dynamically retrieves private keys and environment secrets for Mina from HashiCorp Vault. The keys and secrets follow a structured path format:
`<environment>/<project>/<organization>/<type>/<file_name>`
For example:
`testnet/mina/encapsulate/validator/private_key`

This structure ensures easy organization and secure retrieval of secrets.

## Setup

### 1. Install Ansible

If Ansible is not installed, visit the official documentation for detailed instructions on how to install Ansible on various Linux distributions:

[Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)

### 2. Clone the repository

Clone this repository to your Ansible control node:

```bash
git clone https://github.com/encapsulate-xyz/mina-ansible.git
cd mina-ansible
```
#### Branch Selection:

Ubuntu 24.04: Use the main branch to install and run the Mina node.
Ubuntu 20.04: Switch to the apt-20.04 branch for installation and execution.

To switch branches, run:

```bash
git checkout <branch-name>
```

Replace <branch-name> with either main or apt-20.04, depending on your target Ubuntu version.

### 3. Inventory

Define your target servers' IP address or DNS in the inventory folder, and select either `mainnet.yml` or `testnet.yml` to update.

Example for `testnet.yml`

```yaml
---
all:
  vars:
    env: testnet
  children:
    mina:
      hosts:
        validator.mina.testnet.encapsulate.xyz:
          type: validator
        fullnode.mina.testnet.encapsulate.xyz:
          type: fullnode
```

## Variables

This playbook allows customization through several variables. You can define these variables in the following locations:

- **`group_vars/all.yml`**: Contains all the port, source URL configurations.
- **`group_vars/mainnet.yml`** or **`group_vars/testnet.yml`**: Contains version-specific variables.

**Note**: Sensitive variables such as private keys and passwords are not stored in `vault.yml`. They are dynamically retrieved from HashiCorp Vault during the playbook run.

### Usage

1. First, install the dependencies:

```bash
ansible-galaxy role install -r roles/requirements.yml
ansible-galaxy collection install -r collections/requirements.yml
```

2. Configure your remote server username and private key file path in the `ansible.cfg` file. Additionally, set the SSH port for your server by adjusting the `ansible_port` variable in `group_vars/all.yml`.

3. Run the playbook:

- To deploy a Mina node:

```bash
ansible-playbook setup_node.yml -i inventory/testnet.yml --limit mina
```

**Example Output:**

```bash
PLAY [Set up mina node] *********************************************************************************************

TASK [Gathering Facts] **********************************************************************************************
ok: [validator.mina.testnet.encapsulate.xyz]

TASK [Include environment setup tasks] ******************************************************************************
included: /Users/aditya/IdeaProjects/mina-ansible/tasks/setup_environment.yml for validator.mina.testnet.encapsulate.xyz

TASK [Set project and service identifier in facts] ******************************************************************
ok: [validator.mina.testnet.encapsulate.xyz]

TASK [Set architecture variable based on system architecture] *******************************************************
ok: [validator.mina.testnet.encapsulate.xyz]

TASK [Display environment being deployed to] ************************************************************************
ok: [validator.mina.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.mina.testnet.encapsulate.xyz",
        "Groups: ['mina']",
        "Project: mina",
        "Environment: testnet",
        "Type: validator",
        "Version: mina-devnet=3.0.4-alpha1-889607b",
        "Username: mina",
        "Service Name: mina",
        "Operating System: linux",
        "System Architecture: amd64"
    ]
}

TASK [Confirm deployment details] ***********************************************************************************
Pausing for 40 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
ok: [validator.mina.testnet.encapsulate.xyz]

TASK [Please confirm again] *****************************************************************************************
ok: [validator.mina.testnet.encapsulate.xyz] => {
    "msg": [
        "Deploying to Host: validator.mina.testnet.encapsulate.xyz",
        "Project: mina",
        "Environment: testnet",
        "Type: validator"
    ]
}

TASK [Confirm deployment details] ***********************************************************************************
Pausing for 20 seconds
(ctrl+C then 'C' = continue early, ctrl+C then 'A' = abort)
[[Press 'C' to continue the play or 'A' to abort]]
```

### Fetching Secrets from HashiCorp Vault

- **Private Key**

```yaml
- name: Fetch private key from HCP Vault
  ansible.builtin.include_tasks: tasks/get_and_save_vault_secret.yml
  vars:
    vault_secret_source_path: "{{ project }}/{{ org }}/{{ type }}/{{ node_private_key_file_name }}"
    secret_destination_path: "{{ node_private_key_path }}"
    secret_type: json
```

- **Environment Secrets**

```yaml
- name: Fetch environment secrets from HCP Vault
  ansible.builtin.include_tasks: tasks/get_and_save_vault_secret.yml
  vars:
    vault_secret_source_path: "{{ project }}/{{ org }}/{{ type }}/{{ node_environment_secrets_file_name }}"
    secret_destination_path: "{{ node_environment_secrets_file_path }}"
    secret_type: environment
```

The two primary secrets retrieved are:

1. **Mina Private Key**
2. **Mina Password (`MINA_PRIVKEY_PASS`)**

