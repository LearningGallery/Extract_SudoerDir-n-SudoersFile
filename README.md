# 🔐 Sudoers Archive \& Azure Upload Automation

![Ansible](https://img.shields.io/badge/Ansible-2.9+-red?style=flat-square\&logo=ansible)
![Python](https://img.shields.io/badge/Python-3.8+-blue?style=flat-square\&logo=python)
![Azure](https://img.shields.io/badge/Azure-Cloud-0078D4?style=flat-square\&logo=microsoft-azure)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)
![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen?style=flat-square)
![Maintained](https://img.shields.io/badge/Maintained-Yes-brightgreen?style=flat-square)


> **Enterprise-grade automation for sudoers configuration backup and archival to Azure Blob Storage with email reporting**

## 📑 Table of Contents

- [Quick Summary](#quick-summary)  
- [Architecture Overview](#architecture-overview)  
- [Quick Start](#quick-start)  
- [Project Structure](#project-structure)  
- [Features \& Capabilities](#features--capabilities)  
- [Technology Stack](#technology-stack)  
- [Prerequisites](#prerequisites)  
- [Role Descriptions](#role-descriptions)  
- [Variables Guide](#variables-guide)  
- [How to Use](#how-to-use)  
- [Security Considerations](#security-considerations)  
- [Troubleshooting](#troubleshooting)  
- [Documentation Links](#documentation-links)  
- [Examples](#examples)  
- [Contributing](#contributing)  
- [Roadmap](#roadmap)  
- [License \& Contact](#license--contact)  


---

## 🎯 Quick Summary

### What Does This Project Do?

This Ansible automation framework **automatically backs up sudoers configurations** from multiple RHEL/Linux machines and **securely archives them to Azure Blob Storage**. It's designed for organizations that need to:

- ✅ **Maintain compliance** with sudoers configuration audits  
- ✅ **Implement disaster recovery** for critical access control files  
- ✅ **Track changes** across multiple Linux systems  
- ✅ **Generate execution reports** automatically via email  
- ✅ **Scale backups** across enterprise infrastructure  


### Problem Statement

Linux sudoers configurations (`/etc/sudoers` and `/etc/sudoers.d/`) are critical access control files. Without proper backup and versioning:

- Configuration changes are **not tracked**  
- **Accidental deletions** cannot be recovered  
- **Compliance audits** lack historical data  
- **Multi-system changes** are difficult to document


### Solution Approach

This automation **centralizes sudoers backup management** by:  

1. Archiving sudoers files from remote RHEL machines  
2. Uploading archives to Azure Blob Storage (with versioning)  
3. Generating HTML execution reports  
4. Sending reports via email automatically  

---

## 🏗️ Architecture Overview

### High-Level Deployment Model

```

┌─────────────────────────────────────────────────────────────────┐
│                    Ansible Control Node (AAP)                   │
│                  aap-controller.learninggallery.com             │
└─────────────────────────────────────────────────────────────────┘
       ↓ (SSH)              ↓ (SSH)              ↓ (SSH)
 ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
 │  RHEL Target 1   │  │  RHEL Target 2   │  │  RHEL Target N   │
 │  (ansiblenode1)  │  │  (ansiblenode2)  │  │  (ansiblnodeN)   │
 │  /etc/sudoers    │  │  /etc/sudoers    │  │  /etc/sudoers    │
 │  /etc/sudoers.d/ │  │  /etc/sudoers.d/ │  │  /etc/sudoers.d/ │
 └──────────────────┘  └──────────────────┘  └──────────────────┘
      ↓ Archive ↓           ↓ Archive ↓           ↓ Archive ↓
┌─────────────────────────────────────────────────────────────────┐
│            Ansible Control Node Local Temp Storage              │
│              /tmp/ansible_sudoers_backups/                      │
│  - ansiblenode1_2024-01-15_14-30-45_sudoers_backup.tar.gz       │
│  - ansiblenode2_2024-01-15_14-30-45_sudoers_backup.tar.gz       │
└─────────────────────────────────────────────────────────────────┘
                     ↓ Azure CLI (Upload)
┌─────────────────────────────────────────────────────────────────┐
│              Azure Blob Storage Container                       │
│           storage_account_name: learninggallery                 │
│           container_name: rotation-reports                      │
│  - ansiblenode1_2024-01-15_14-30-45_sudoers_backup.tar.gz       │
│  - ansiblenode2_2024-01-15_14-30-45_sudoers_backup.tar.gz       │
└─────────────────────────────────────────────────────────────────┘
                     ↓ HTML Report Generation
┌─────────────────────────────────────────────────────────────────┐
│            Email Notification (Gmail SMTP)                      │
│  To: abutalha@amityonline.com                                   │
│  Subject: Sudoers Backup \& Azure Upload Report                 │
│  Body: Execution status + error details (HTML table)            │
└─────────────────────────────────────────────────────────────────┘

```


### Managed Systems

| Hostname | OS Type | Target Role | Key Config |
|----------|---------|-------------|-----------|
| ansiblenode1+ | RHEL/CentOS 7+ | Target Host | SSH accessible, sudo required |
| aap-controller.learninggallery.com | RHEL/CentOS | Control Node | Ansible 2.9+, Azure CLI, SMTP |


### Key Components \& Dependencies

| Component | Purpose | Location | Dependency |
|-----------|---------|----------|-----------|
| **sudoers_archive** role | Archive /etc/sudoers + /etc/sudoers.d/ | roles/sudoers_archive/ | Gzip, Ansible fetch module |
| **storage_upload** role | Upload to Azure Blob Storage | roles/storage_upload/ | Azure CLI, storage account key |
| **Main playbook** | Orchestration and reporting | ExtractSudoerDirnSudoerFile.yml | Both roles, templates |
| **HTML template** | Execution report generation | templates/tracker_report.html.j2 | Jinja2 templating |
| **Azure CLI** | Azure authentication and upload | AAP server | Azure subscription access |
| **SMTP (Gmail)** | Email delivery | AAP server | SMTP credentials |


---

## ⚡ Quick Start

### Step 1: Prerequisites Checklist

- [ ] Ansible 2.9+ installed on control node  
- [ ] Python 3.8+ on control node  
- [ ] SSH key pair configured for target hosts  
- [ ] Azure CLI installed (`az --version`)  
- [ ] Azure Storage Account created  
- [ ] Azure Storage Account Key available  
- [ ] Gmail SMTP credentials (or SMTP server details)  
- [ ] Inventory file configured with target hosts  
- [ ] Network connectivity verified (SSH ports open)  


### Step 2: Clone Repository

```bash

git clone https://github.com/yourusername/sudoers-azure-backup.git
cd sudoers-azure-backup

```


### Step 3: Install Dependencies

```bash

# Install Ansible collections
ansible-galaxy collection install -r collections/requirements.yml

# Verify Azure collection
ansible-galaxy collection list | grep azure

```


### Step 4: Configure Inventory

```bash

# Edit inventory file

cat > inventory.ini << 'EOF'
[sudoers_backup_targets]
ansiblenode1 ansible_host=192.168.1.10 ansible_user=ansible ansible_become=yes
ansiblenode2 ansible_host=192.168.1.11 ansible_user=ansible ansible_become=yes
ansiblenode3 ansible_host=192.168.1.12 ansible_user=ansible ansible_become=yes

[sudoers_backup_targets:vars]
# Inherited from group_vars/all.yml
EOF

```


### Step 5: Configure Variables

```bash

# Edit group variables
cat > group_vars/all.yml << 'EOF'

# Azure Storage Configuration
az_container: "rotation-reports"
aap_server_hostname: "aap-controller.learninggallery.com" 
local_temp_dir: "/tmp/ansible_sudoers_backups"
storage_account_name: "learninggallery"
storage_account_key: "YOUR_AZURE_STORAGE_KEY_HERE"  # ⚠️ Use Ansible Vault!


# Email Configuration
smtp_sender: "abutalha3005@gmail.com"
email_recipient: "abutalha@amityonline.com"
smtp_host: "smtp.gmail.com"
smtp_port: 587
smtp_user: "your-gmail@gmail.com"
smtp_pass: "your-gmail-password"  # ⚠️ Use App Password + Vault!
EOF

```


### Step 6: Test Connectivity

```bash

# Ping all hosts
ansible -i inventory.ini all -m ping

# Gather facts from targets
ansible -i inventory.ini sudoers_backup_targets -m setup | head -50

```


### Step 7: Run the Playbook (Dry-Run First)

```bash

# Dry-run to preview changes
ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml --check -v

# Expected output:
# TASK [sudoers_archive : Archive sudoers file and sudoers.d directory...] 
# TASK [storage_upload : CLI Upload and Clean Up Block] 

```


### Step 8: Execute Production Playbook



```bash

# Full execution with verbosity
ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml -v

# With extra debugging if needed
ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml -vvv

```


### Step 9: Validate Execution

```bash

# Check backup files in local temp directory
ls -lah /tmp/ansible_sudoers_backups/

# Expected output:
# -rw-r--r-- 1 ansible ansible 4.2K Jan 15 14:30 ansiblenode1_2024-01-15_14-30-45_sudoers_backup.tar.gz
# -rw-r--r-- 1 ansible ansible 3.8K Jan 15 14:31 ansiblenode2_2024-01-15_14-30-45_sudoers_backup.tar.gz

# Verify Azure upload
az storage blob list \\
--account-name learninggallery \\
--account-key $AZURE_STORAGE_KEY \\
--container-name rotation-reports \\
--output table

# Check email inbox for execution report
echo "✅ Check email for HTML tracker report"

```

---


## 📂 Project Structure

```
sudoers-azure-backup/
│
├── README.md                                    # This file
├── ExtractSudoerDirnSudoerFile.yml             # Main playbook (orchestration)
├── collections/
│   └── requirements.yml                         # Ansible collections (azure.azcollection)
├── group_vars/
│   └── all.yml                                  # Global variables for all hosts
├── roles/
│   ├── sudoers_archive/                        # Role 1: Archive sudoers files
│   │   ├── defaults/
│   │   │   └── main.yml                        # Default variables
│   │   ├── tasks/
│   │   │   └── main.yml                        # Archive tasks (gzip, fetch)
│   │   └── (no handlers, templates, vars)
│   │
│   └── storage_upload/                         # Role 2: Upload to Azure
│       ├── defaults/
│       │   └── main.yml                        # Default variables
│       ├── tasks/
│       │   ├── main.yml                        # CLI upload method (current)
│       │   └── main.yml_usingUMI               # REST API method (alternative)
│       └── (no handlers, templates, vars)
│
├── templates/
│   └── tracker_report.html.j2                  # HTML report template (Jinja2)
│
├── inventory.ini                                # Inventory file (example)

```

### Folder Descriptions

| Folder | Purpose | Contains |
|--------|---------|----------|
| `roles/` | Ansible roles (reusable automation units) | sudoers_archive, storage_upload |
| `group_vars/` | Group-level variables | Shared config for all hosts |
| `templates/` | Jinja2 templates | HTML report template |
| `collections/` | Ansible collection requirements | Azure collection manifest |
| `docs/` | Complete documentation | Guides, architecture, ADRs, diagrams |

---

## ✨ Features \& Capabilities

### Backup Capabilities
- ✅ **Multi-host sudoers archival** - Backs up `/etc/sudoers` + `/etc/sudoers.d/` from N targets  
- ✅ **Compressed archives** - Gzip compression reduces storage footprint  
- ✅ **Timestamped filenames** - Archives include date/time for version tracking  
- ✅ **Hostname-based naming** - Uses actual OS hostname instead of IP (better readability)  

### Upload Capabilities
- ✅ **Azure Blob Storage integration** - Direct upload to Azure cloud  
- ✅ **Automatic versioning** - Azure maintains version history  
- ✅ **Retry logic** - Handles network latency with 3 retry attempts  
- ✅ **Dual upload methods** - CLI-based (current) + REST API (alternative)  
- ✅ **Managed Identity support** - Optional UMI authentication (more secure)  

### Reporting Capabilities  
- ✅ **HTML execution reports** - Professional, styled execution status  
- ✅ **Email delivery** - Automated report delivery via SMTP  
- ✅ **Error capture** - Detailed error messages for failed nodes  
- ✅ **Per-node status tracking** - Success/failure visibility per host  

### Resilience \& Error Handling  
- ✅ **SSH connection retry** - 5-minute timeout for slow connections  
- ✅ **Azure CLI retry logic** - 3x retry with 15s delay for network issues  
- ✅ **Error block/rescue** - Captures failures without stopping workflow  
- ✅ **Automatic cleanup** - Removes local temp files after upload  
- ✅ **Graceful degradation** - Reports partial success (some hosts succeeded)  

### Security Features
- ✅ **Privilege escalation** - Uses `become: yes` for root-level operations  
- ✅ **Secure file permissions** - Archives created with restricted permissions  
- ✅ **No hardcoded secrets** - Variables separated from code  
- ✅ **Vault-ready** - Supports Ansible Vault for credential storage  
- ✅ **SSH key-based auth** - No password-based authentication  

---


## 🛠️ Technology Stack

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Automation Platform** | Ansible | 2.9+ | Orchestration engine |
| **Control Node OS** | RHEL/CentOS | 7+ | AAP server OS |
| **Target OS** | RHEL/CentOS | 7+ | Linux machines being backed up |
| **Python** | Python | 3.8+ | Runtime for Ansible |
| **Archive Format** | Gzip | - | Compression (tar.gz) |
| **Cloud Storage** | Azure Blob Storage | - | Backup destination |
| **CLI Tool** | Azure CLI | 2.30+ | Upload tool |
| **API Method** | Azure REST API | 2020-04-08 | Alternative upload |
| **Authentication** | Storage Account Key OR UMI | - | Azure access |
| **Reporting** | Jinja2 + HTML | - | Report templates |
| **Email** | SMTP (Gmail) | TLS 587 | Report delivery |
| **Collections** | azure.azcollection | 1.10+ | Azure modules |


---

## 📋 Prerequisites \& Requirements

### System Requirements

**Control Node (AAP Server)**  
- OS: RHEL/CentOS 7+ or compatible  
- RAM: 2GB minimum (4GB recommended)  
- Disk: 10GB available for temp backups  
- Network: Outbound to Azure (443), SMTP (587), SSH (22)  

**Target Nodes (RHEL Machines)**  
- OS: RHEL/CentOS 7+ or compatible  
- Network: Inbound SSH (port 22) from control node  
- User: An ansible user with sudo privileges  
- Sudo: passwordless or password-enabled (configured)  

### Software Requirements  

```bash
# Control Node
- Ansible 2.9 or higher  
- Python 3.8 or higher  
- Azure CLI 2.30+  
- Gzip/tar utilities (usually pre-installed)  
- SSH client    

# Target Nodes
- SSH server (sshd)  
- Gzip/tar utilities  
- Sudo binary with sudoers files  

```

### Network Requirements  

| Source | Destination | Port | Protocol | Purpose |
|--------|-------------|------|----------|---------|
| Control Node | Target Hosts | 22 | SSH | Ansible communication |
| Control Node | Azure | 443 | HTTPS | Azure API + Blob upload |
| Control Node | SMTP (Gmail) | 587 | SMTP | Email delivery |

### Credentials \& Secrets

**Required Before Execution:**  

1. **Azure Storage Account Key**

```bash

# Retrieve from Azure Portal or CLI
az storage account keys list \\
--resource-group myResourceGroup \\
--account-name learninggallery \\
--query '\[0].value' -o tsv

```

2. **Gmail SMTP Credentials** (or alternative SMTP)

```bash

# For Gmail, create an App Password (not main password)
# https://support.google.com/accounts/answer/185833
smtp_user: "your-email@gmail.com"  
smtp_pass: "xxxx xxxx xxxx xxxx"  # 16-char app password  

```

3. **SSH Key Pair**

```bash
# Generate if not exist
ssh-keygen -t rsa -b 4096 -f \~/.ssh/ansible_key

# Copy to target hosts
ssh-copy-id -i \~/.ssh/ansible_key.pub ansible@target_host

```

### Inventory Configuration

Ensure your inventory includes all target hosts:

```ini

[sudoers_backup_targets]
ansiblenode1 ansible_host=192.168.1.10
ansiblenode2 ansible_host=192.168.1.11
ansiblenode3 ansible_host=192.168.1.12

[sudoers_backup_targets:vars]
ansible_user=ansible
ansible_become=yes
ansible_become_method=sudo

```

---

## 🎭 Role Descriptions

### Role 1: `sudoers_archive`

| Property | Value |
|----------|-------|
| **Purpose** | Archive /etc/sudoers + /etc/sudoers.d/ on target hosts |
| **Target Hosts** | All RHEL/Linux machines in inventory |
| **Execution Scope** | Runs on each target individually |
| **Privileges Required** | sudo (to read /etc/sudoers files) |

**Tasks Executed:**

1. **Create temp directory** on control node (`/tmp/ansible_sudoers_backups`)  
2. **Archive sudoers** on target host using gzip compression  
3. **Fetch archive** from target to control node (using Ansible fetch)  
4. **Clean up** remote temp files after fetch  

**Example Execution:**

```bash

# Single role execution
ansible-playbook -i inventory.ini -e "ansible_user=ansible" \\
-m include_role -a name=sudoers_archive sudoers_backup_targets

```

### Role 2: `storage_upload`

| Property | Value |
|----------|-------|
| **Purpose** | Upload archives to Azure Blob Storage |
| **Target Hosts** | Runs on control node (delegated) |
| **Execution Scope** | Centralized on AAP server |
| **Privileges Required** | Azure Storage Account Key |

**Tasks Executed:**

1. **Create backup directory** on AAP server  
2. **Copy file** from EE container to AAP host disk (if needed)  
3. **Upload to Azure Blob** using Azure CLI (with retry)  
4. **Clean up** local temp files after successful upload  
5. **Record status** for HTML report (success/failure)  

**Two Upload Methods Provided:**

- **Method 1 (CLI):** `tasks/main.yml` - Uses `az storage blob upload`  
- **Method 2 (REST API):** `tasks/main.yml_usingUMI` - Uses HTTP PUT + MSI token

---

## 📊 Variables Guide

### Global Variables (`group_vars/all.yml`)

```yaml

# Azure Storage Configuration
az_container: "rotation-reports"              # Blob container name
storage_account_name: "learninggallery"       # Azure storage account
storage_account_key: "{{ vault_storage_key }}"  # ⚠️ Use Vault!

# AAP Control Node
aap_server_hostname: "aap-controller.learninggallery.com"
local_temp_dir: "/tmp/ansible_sudoers_backups"

# Email Configuration
smtp_sender: "abutalha3005@gmail.com"
email_recipient: "abutalha@amityonline.com"
smtp_host: "smtp.gmail.com"
smtp_port: 587
smtp_user: "{{ vault_smtp_user }}"  # ⚠️ Use Vault!
smtp_pass: "{{ vault_smtp_pass }}"  # ⚠️ Use Vault!

```

### Dynamic Variables (Generated at Runtime)

| Variable | Set By | Value Example | Purpose |
|----------|--------|---------------|---------|
| `backup_timestamp` | pre_tasks | `2024-01-15_14-30-45` | Unique timestamp for run |
| `backup_filename` | pre_tasks | `ansiblenode1_2024-01-15_14-30-45_sudoers_backup.tar.gz` | Archive filename |
| `upload_status` | storage_upload role | `Success` / `Failed` | Upload result |
| `error_message` | storage_upload role | CLI error or `-` | Error details |

### Role-Specific Variables

**sudoers_archive role** (`roles/sudoers_archive/defaults/main.yml`)

```yaml

local_temp_dir: "/tmp/ansible_sudoers_backups"

```

**storage_upload role** (`roles/storage_upload/defaults/main.yml`)

```yaml

local_temp_dir: "/tmp/ansible_sudoers_backups"

```


### How to Override Variables

```bash

# Method 1: Command-line extra variables
ansible-playbook ExtractSudoerDirnSudoerFile.yml \\
-e "email_recipient=newemail@example.com" \\
-e "az_container=custom-container"

# Method 2: Create host_vars for specific hosts
mkdir -p host_vars/
echo "storage_account_name: custom_account" > host_vars/ansiblenode1.yml

# Method 3: Use inventory variables
# Edit inventory.ini with per-host vars
[sudoers_backup_targets]
ansiblenode1 email_recipient=node1@example.com

# Method 4: Ansible Vault (recommended for secrets)
ansible-vault create group_vars/all/vault.yml

# Then reference: {{ vault_storage_key }}

```

---

## 🚀 How to Use

### Use Case 1: Initial Sudoers Backup (Fresh Run)

**Objective:** Back up sudoers from all target hosts for the first time

```bash

# Step 1: Verify connectivity
ansible -i inventory.ini sudoers_backup_targets -m ping

# Step 2: Dry-run playbook
ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml \\
--check -v

# Step 3: Execute playbook
ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml -v

# Step 4: Verify outputs
ls -lah /tmp/ansible_sudoers_backups/
az storage blob list --account-name learninggallery \\
--container-name rotation-reports --output table

# Step 5: Check email for HTML report

echo "✅ Check your email for execution report"

```


**Expected Output:**

```

PLAY [Archive and Upload Sudoers to Azure] ****
TASK [Wait for host to be reachable]
ok: [ansiblenode1]
ok: [ansiblenode2]

TASK \[Gather facts explicitly]
ok: \[ansiblenode1]
ok: \[ansiblenode2]
... (more tasks)

PLAY RECAP
ansiblenode1: ok=8 changed=3 unreachable=0 failed=0
ansiblenode2: ok=8 changed=3 unreachable=0 failed=0

```

---


### Use Case 2: Update Existing Infrastructure

**Objective:** Add new target hosts to the backup automation

```bash

# Step 1: Add new hosts to inventory
cat >> inventory.ini << 'EOF'
ansiblenode4 ansible_host=192.168.1.13 ansible_user=ansible
ansiblenode5 ansible_host=192.168.1.14 ansible_user=ansible
EOF

# Step 2: Test connectivity to new hosts
ansible -i inventory.ini ansiblenode4,ansiblenode5 -m ping

# Step 3: Run playbook (will include new hosts)
ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml -v

# Verify new archives
ls -lah /tmp/ansible_sudoers_backups/ | grep ansiblenode\[45]

```

---

### Use Case 3: Troubleshoot Failed Node

**Objective:** Diagnose and fix why a specific host failed to backup

```bash

# Step 1: Run playbook with verbose output on single host
ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml \\
--limit ansiblenode1 -vvv

# Step 2: Manual tests on target
ssh ansible@ansiblenode1
sudo ls -la /etc/sudoers\*  # Verify files exist
sudo tar czf /tmp/test.tar.gz /etc/sudoers /etc/sudoers.d/

exit

# Step 3: Check AAP connectivity
ansible -i inventory.ini ansiblenode1 -m setup | grep -i gather

# Step 4: Test Azure CLI credentials on control node
az storage blob list --account-name learninggallery \\
--account-key $AZURE_STORAGE_KEY --container-name rotation-reports

# Step 5: Re-run playbook on single host
ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml \\
--limit ansiblenode1 -v

```

---


### Use Case 4: Schedule Daily Automated Backups

**Objective:** Run backup every day at 2:00 AM on control node

```bash

# Step 1: Create wrapper script
cat > /usr/local/bin/backup_sudoers.sh << 'EOF'
#!/bin/bash
cd /opt/ansible/sudoers-backup
/usr/bin/ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml
EOF

chmod +x /usr/local/bin/backup_sudoers.sh

# Step 2: Add cron job
crontab -e
# Add: 0 2 * * * /usr/local/bin/backup_sudoers.sh

# Step 3: Verify cron is running
crontab -l
ls -la /var/log/cron  # Check cron logs

```


---

## 🔒 Security Considerations

### SSH Key Requirements

**Setup SSH keys properly:**

```bash

# On control node, generate key pair
ssh-keygen -t rsa -b 4096 \\
-f \~/.ssh/ansible_id_rsa \\
-C "ansible@aap-controller"

# Copy to target hosts
for host in ansiblenode1 ansiblenode2 ansiblenode3; do
ssh-copy-id -i \~/.ssh/ansible_id_