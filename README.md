\# 🔐 Sudoers Archive \& Azure Upload Automation



!\[Ansible](https://img.shields.io/badge/Ansible-2.9+-red?style=flat-square\&logo=ansible)

!\[Python](https://img.shields.io/badge/Python-3.8+-blue?style=flat-square\&logo=python)

!\[Azure](https://img.shields.io/badge/Azure-Cloud-0078D4?style=flat-square\&logo=microsoft-azure)

!\[License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

!\[Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen?style=flat-square)

!\[Maintained](https://img.shields.io/badge/Maintained-Yes-brightgreen?style=flat-square)



> \*\*Enterprise-grade automation for sudoers configuration backup and archival to Azure Blob Storage with email reporting\*\*



\## 📑 Table of Contents



\- \[Quick Summary](#quick-summary)

\- \[Architecture Overview](#architecture-overview)

\- \[Quick Start](#quick-start)

\- \[Project Structure](#project-structure)

\- \[Features \& Capabilities](#features--capabilities)

\- \[Technology Stack](#technology-stack)

\- \[Prerequisites](#prerequisites)

\- \[Role Descriptions](#role-descriptions)

\- \[Variables Guide](#variables-guide)

\- \[How to Use](#how-to-use)

\- \[Security Considerations](#security-considerations)

\- \[Troubleshooting](#troubleshooting)

\- \[Documentation Links](#documentation-links)

\- \[Examples](#examples)

\- \[Contributing](#contributing)

\- \[Roadmap](#roadmap)

\- \[License \& Contact](#license--contact)



\---



\## 🎯 Quick Summary



\### What Does This Project Do?



This Ansible automation framework \*\*automatically backs up sudoers configurations\*\* from multiple RHEL/Linux machines and \*\*securely archives them to Azure Blob Storage\*\*. It's designed for organizations that need to:



\- ✅ \*\*Maintain compliance\*\* with sudoers configuration audits

\- ✅ \*\*Implement disaster recovery\*\* for critical access control files

\- ✅ \*\*Track changes\*\* across multiple Linux systems

\- ✅ \*\*Generate execution reports\*\* automatically via email

\- ✅ \*\*Scale backups\*\* across enterprise infrastructure



\### Problem Statement



Linux sudoers configurations (`/etc/sudoers` and `/etc/sudoers.d/`) are critical access control files. Without proper backup and versioning:

\- Configuration changes are \*\*not tracked\*\*

\- \*\*Accidental deletions\*\* cannot be recovered

\- \*\*Compliance audits\*\* lack historical data

\- \*\*Multi-system changes\*\* are difficult to document



\### Solution Approach



This automation \*\*centralizes sudoers backup management\*\* by:

1\. Archiving sudoers files from remote RHEL machines

2\. Uploading archives to Azure Blob Storage (with versioning)

3\. Generating HTML execution reports

4\. Sending reports via email automatically



\---



\## 🏗️ Architecture Overview



\### High-Level Deployment Model



```

┌─────────────────────────────────────────────────────────────────┐

│                    Ansible Control Node (AAP)                   │

│                  aap-controller.learninggallery.com              │

└─────────────────────────────────────────────────────────────────┘

&#x20;         ↓ (SSH)              ↓ (SSH)              ↓ (SSH)

┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐

│  RHEL Target 1   │  │  RHEL Target 2   │  │  RHEL Target N   │

│  (ansiblenode1)  │  │  (ansiblenode2)  │  │  (ansiblnodeN)   │

│  /etc/sudoers    │  │  /etc/sudoers    │  │  /etc/sudoers    │

│  /etc/sudoers.d/ │  │  /etc/sudoers.d/ │  │  /etc/sudoers.d/ │

└──────────────────┘  └──────────────────┘  └──────────────────┘

&#x20;         ↓ Archive ↓           ↓ Archive ↓           ↓ Archive ↓

┌─────────────────────────────────────────────────────────────────┐

│            Ansible Control Node Local Temp Storage              │

│              /tmp/ansible\_sudoers\_backups/                      │

│  - ansiblenode1\_2024-01-15\_14-30-45\_sudoers\_backup.tar.gz      │

│  - ansiblenode2\_2024-01-15\_14-30-45\_sudoers\_backup.tar.gz      │

└─────────────────────────────────────────────────────────────────┘

&#x20;         ↓ Azure CLI (Upload)

┌─────────────────────────────────────────────────────────────────┐

│              Azure Blob Storage Container                       │

│           storage\_account\_name: learninggallery                │

│           container\_name: rotation-reports                     │

│  - ansiblenode1\_2024-01-15\_14-30-45\_sudoers\_backup.tar.gz     │

│  - ansiblenode2\_2024-01-15\_14-30-45\_sudoers\_backup.tar.gz     │

└─────────────────────────────────────────────────────────────────┘

&#x20;         ↓ HTML Report Generation

┌─────────────────────────────────────────────────────────────────┐

│            Email Notification (Gmail SMTP)                     │

│  To: abutalha@amityonline.com                                  │

│  Subject: Sudoers Backup \& Azure Upload Report                 │

│  Body: Execution status + error details (HTML table)           │

└─────────────────────────────────────────────────────────────────┘

```



\### Managed Systems



| Hostname | OS Type | Target Role | Key Config |

|----------|---------|-------------|-----------|

| ansiblenode1+ | RHEL/CentOS 7+ | Target Host | SSH accessible, sudo required |

| aap-controller.learninggallery.com | RHEL/CentOS | Control Node | Ansible 2.9+, Azure CLI, SMTP |



\### Key Components \& Dependencies



| Component | Purpose | Location | Dependency |

|-----------|---------|----------|-----------|

| \*\*sudoers\_archive\*\* role | Archive /etc/sudoers + /etc/sudoers.d/ | roles/sudoers\_archive/ | Gzip, Ansible fetch module |

| \*\*storage\_upload\*\* role | Upload to Azure Blob Storage | roles/storage\_upload/ | Azure CLI, storage account key |

| \*\*Main playbook\*\* | Orchestration and reporting | ExtractSudoerDirnSudoerFile.yml | Both roles, templates |

| \*\*HTML template\*\* | Execution report generation | templates/tracker\_report.html.j2 | Jinja2 templating |

| \*\*Azure CLI\*\* | Azure authentication and upload | AAP server | Azure subscription access |

| \*\*SMTP (Gmail)\*\* | Email delivery | AAP server | SMTP credentials |



\---



\## ⚡ Quick Start



\### Step 1: Prerequisites Checklist



\- \[ ] Ansible 2.9+ installed on control node

\- \[ ] Python 3.8+ on control node

\- \[ ] SSH key pair configured for target hosts

\- \[ ] Azure CLI installed (`az --version`)

\- \[ ] Azure Storage Account created

\- \[ ] Azure Storage Account Key available

\- \[ ] Gmail SMTP credentials (or SMTP server details)

\- \[ ] Inventory file configured with target hosts

\- \[ ] Network connectivity verified (SSH ports open)



\### Step 2: Clone Repository



```bash

git clone https://github.com/yourusername/sudoers-azure-backup.git

cd sudoers-azure-backup

```



\### Step 3: Install Dependencies



```bash

\# Install Ansible collections

ansible-galaxy collection install -r collections/requirements.yml



\# Verify Azure collection

ansible-galaxy collection list | grep azure

```



\### Step 4: Configure Inventory



```bash

\# Edit inventory file

cat > inventory.ini << 'EOF'

\[sudoers\_backup\_targets]

ansiblenode1 ansible\_host=192.168.1.10 ansible\_user=ansible ansible\_become=yes

ansiblenode2 ansible\_host=192.168.1.11 ansible\_user=ansible ansible\_become=yes

ansiblenode3 ansible\_host=192.168.1.12 ansible\_user=ansible ansible\_become=yes



\[sudoers\_backup\_targets:vars]

\# Inherited from group\_vars/all.yml

EOF

```



\### Step 5: Configure Variables



```bash

\# Edit group variables

cat > group\_vars/all.yml << 'EOF'

\# Azure Storage Configuration

az\_container: "rotation-reports"

aap\_server\_hostname: "aap-controller.learninggallery.com" 

local\_temp\_dir: "/tmp/ansible\_sudoers\_backups"

storage\_account\_name: "learninggallery"

storage\_account\_key: "YOUR\_AZURE\_STORAGE\_KEY\_HERE"  # ⚠️ Use Ansible Vault!



\# Email Configuration

smtp\_sender: "abutalha3005@gmail.com"

email\_recipient: "abutalha@amityonline.com"

smtp\_host: "smtp.gmail.com"

smtp\_port: 587

smtp\_user: "your-gmail@gmail.com"

smtp\_pass: "your-gmail-password"  # ⚠️ Use App Password + Vault!

EOF

```



\### Step 6: Test Connectivity



```bash

\# Ping all hosts

ansible -i inventory.ini all -m ping



\# Gather facts from targets

ansible -i inventory.ini sudoers\_backup\_targets -m setup | head -50

```



\### Step 7: Run the Playbook (Dry-Run First)



```bash

\# Dry-run to preview changes

ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml --check -v



\# Expected output:

\# TASK \[sudoers\_archive : Archive sudoers file and sudoers.d directory...] 

\# TASK \[storage\_upload : CLI Upload and Clean Up Block] 

```



\### Step 8: Execute Production Playbook



```bash

\# Full execution with verbosity

ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml -v



\# With extra debugging if needed

ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml -vvv

```



\### Step 9: Validate Execution



```bash

\# Check backup files in local temp directory

ls -lah /tmp/ansible\_sudoers\_backups/



\# Expected output:

\# -rw-r--r-- 1 ansible ansible 4.2K Jan 15 14:30 ansiblenode1\_2024-01-15\_14-30-45\_sudoers\_backup.tar.gz

\# -rw-r--r-- 1 ansible ansible 3.8K Jan 15 14:31 ansiblenode2\_2024-01-15\_14-30-45\_sudoers\_backup.tar.gz



\# Verify Azure upload

az storage blob list \\

&#x20; --account-name learninggallery \\

&#x20; --account-key $AZURE\_STORAGE\_KEY \\

&#x20; --container-name rotation-reports \\

&#x20; --output table



\# Check email inbox for execution report

echo "✅ Check email for HTML tracker report"

```



\### Step 10: Next Steps



\- Review \*\*\[Detailed Architecture Guide](docs/02-ARCHITECTURE.md)\*\*

\- Explore \*\*\[Role Reference](docs/05-ROLE-REFERENCE.md)\*\*

\- Set up \*\*\[Scheduling with Cron](docs/13-RUNBOOK.md#scheduling-automated-backups)\*\*

\- Implement \*\*\[Security Best Practices](docs/11-SECURITY-HARDENING.md)\*\*



\---



\## 📂 Project Structure



```

sudoers-azure-backup/

│

├── README.md                                    # This file

├── ExtractSudoerDirnSudoerFile.yml             # Main playbook (orchestration)

├── collections/

│   └── requirements.yml                         # Ansible collections (azure.azcollection)

├── group\_vars/

│   └── all.yml                                  # Global variables for all hosts

├── roles/

│   ├── sudoers\_archive/                        # Role 1: Archive sudoers files

│   │   ├── defaults/

│   │   │   └── main.yml                        # Default variables

│   │   ├── tasks/

│   │   │   └── main.yml                        # Archive tasks (gzip, fetch)

│   │   └── (no handlers, templates, vars)

│   │

│   └── storage\_upload/                         # Role 2: Upload to Azure

│       ├── defaults/

│       │   └── main.yml                        # Default variables

│       ├── tasks/

│       │   ├── main.yml                        # CLI upload method (current)

│       │   └── main.yml\_usingUMI               # REST API method (alternative)

│       └── (no handlers, templates, vars)

│

├── templates/

│   └── tracker\_report.html.j2                  # HTML report template (Jinja2)

│

├── inventory.ini                                # Inventory file (example)



```



\### Folder Descriptions



| Folder | Purpose | Contains |

|--------|---------|----------|

| `roles/` | Ansible roles (reusable automation units) | sudoers\_archive, storage\_upload |

| `group\_vars/` | Group-level variables | Shared config for all hosts |

| `templates/` | Jinja2 templates | HTML report template |

| `collections/` | Ansible collection requirements | Azure collection manifest |

| `docs/` | Complete documentation | Guides, architecture, ADRs, diagrams |



\---



\## ✨ Features \& Capabilities



\### Backup Capabilities



\- ✅ \*\*Multi-host sudoers archival\*\* - Backs up `/etc/sudoers` + `/etc/sudoers.d/` from N targets

\- ✅ \*\*Compressed archives\*\* - Gzip compression reduces storage footprint

\- ✅ \*\*Timestamped filenames\*\* - Archives include date/time for version tracking

\- ✅ \*\*Hostname-based naming\*\* - Uses actual OS hostname instead of IP (better readability)



\### Upload Capabilities



\- ✅ \*\*Azure Blob Storage integration\*\* - Direct upload to Azure cloud

\- ✅ \*\*Automatic versioning\*\* - Azure maintains version history

\- ✅ \*\*Retry logic\*\* - Handles network latency with 3 retry attempts

\- ✅ \*\*Dual upload methods\*\* - CLI-based (current) + REST API (alternative)

\- ✅ \*\*Managed Identity support\*\* - Optional UMI authentication (more secure)



\### Reporting Capabilities



\- ✅ \*\*HTML execution reports\*\* - Professional, styled execution status

\- ✅ \*\*Email delivery\*\* - Automated report delivery via SMTP

\- ✅ \*\*Error capture\*\* - Detailed error messages for failed nodes

\- ✅ \*\*Per-node status tracking\*\* - Success/failure visibility per host



\### Resilience \& Error Handling



\- ✅ \*\*SSH connection retry\*\* - 5-minute timeout for slow connections

\- ✅ \*\*Azure CLI retry logic\*\* - 3x retry with 15s delay for network issues

\- ✅ \*\*Error block/rescue\*\* - Captures failures without stopping workflow

\- ✅ \*\*Automatic cleanup\*\* - Removes local temp files after upload

\- ✅ \*\*Graceful degradation\*\* - Reports partial success (some hosts succeeded)



\### Security Features



\- ✅ \*\*Privilege escalation\*\* - Uses `become: yes` for root-level operations

\- ✅ \*\*Secure file permissions\*\* - Archives created with restricted permissions

\- ✅ \*\*No hardcoded secrets\*\* - Variables separated from code

\- ✅ \*\*Vault-ready\*\* - Supports Ansible Vault for credential storage

\- ✅ \*\*SSH key-based auth\*\* - No password-based authentication



\---



\## 🛠️ Technology Stack



| Component | Technology | Version | Purpose |

|-----------|-----------|---------|---------|

| \*\*Automation Platform\*\* | Ansible | 2.9+ | Orchestration engine |

| \*\*Control Node OS\*\* | RHEL/CentOS | 7+ | AAP server OS |

| \*\*Target OS\*\* | RHEL/CentOS | 7+ | Linux machines being backed up |

| \*\*Python\*\* | Python | 3.8+ | Runtime for Ansible |

| \*\*Archive Format\*\* | Gzip | - | Compression (tar.gz) |

| \*\*Cloud Storage\*\* | Azure Blob Storage | - | Backup destination |

| \*\*CLI Tool\*\* | Azure CLI | 2.30+ | Upload tool |

| \*\*API Method\*\* | Azure REST API | 2020-04-08 | Alternative upload |

| \*\*Authentication\*\* | Storage Account Key OR UMI | - | Azure access |

| \*\*Reporting\*\* | Jinja2 + HTML | - | Report templates |

| \*\*Email\*\* | SMTP (Gmail) | TLS 587 | Report delivery |

| \*\*Collections\*\* | azure.azcollection | 1.10+ | Azure modules |



\---



\## 📋 Prerequisites \& Requirements



\### System Requirements



\*\*Control Node (AAP Server)\*\*

\- OS: RHEL/CentOS 7+ or compatible

\- RAM: 2GB minimum (4GB recommended)

\- Disk: 10GB available for temp backups

\- Network: Outbound to Azure (443), SMTP (587), SSH (22)



\*\*Target Nodes (RHEL Machines)\*\*

\- OS: RHEL/CentOS 7+ or compatible

\- Network: Inbound SSH (port 22) from control node

\- User: An ansible user with sudo privileges

\- Sudo: passwordless or password-enabled (configured)



\### Software Requirements



```bash

\# Control Node

\- Ansible 2.9 or higher

\- Python 3.8 or higher

\- Azure CLI 2.30+

\- Gzip/tar utilities (usually pre-installed)

\- SSH client



\# Target Nodes

\- SSH server (sshd)

\- Gzip/tar utilities

\- Sudo binary with sudoers files

```



\### Network Requirements



| Source | Destination | Port | Protocol | Purpose |

|--------|-------------|------|----------|---------|

| Control Node | Target Hosts | 22 | SSH | Ansible communication |

| Control Node | Azure | 443 | HTTPS | Azure API + Blob upload |

| Control Node | SMTP (Gmail) | 587 | SMTP | Email delivery |



\### Credentials \& Secrets



\*\*Required Before Execution:\*\*



1\. \*\*Azure Storage Account Key\*\*

&#x20;  ```bash

&#x20;  # Retrieve from Azure Portal or CLI

&#x20;  az storage account keys list \\

&#x20;    --resource-group myResourceGroup \\

&#x20;    --account-name learninggallery \\

&#x20;    --query '\[0].value' -o tsv

&#x20;  ```



2\. \*\*Gmail SMTP Credentials\*\* (or alternative SMTP)

&#x20;  ```bash

&#x20;  # For Gmail, create an App Password (not main password)

&#x20;  # https://support.google.com/accounts/answer/185833

&#x20;  smtp\_user: "your-email@gmail.com"

&#x20;  smtp\_pass: "xxxx xxxx xxxx xxxx"  # 16-char app password

&#x20;  ```



3\. \*\*SSH Key Pair\*\*

&#x20;  ```bash

&#x20;  # Generate if not exist

&#x20;  ssh-keygen -t rsa -b 4096 -f \~/.ssh/ansible\_key

&#x20;  

&#x20;  # Copy to target hosts

&#x20;  ssh-copy-id -i \~/.ssh/ansible\_key.pub ansible@target\_host

&#x20;  ```



\### Inventory Configuration



Ensure your inventory includes all target hosts:



```ini

\[sudoers\_backup\_targets]

ansiblenode1 ansible\_host=192.168.1.10

ansiblenode2 ansible\_host=192.168.1.11

ansiblenode3 ansible\_host=192.168.1.12



\[sudoers\_backup\_targets:vars]

ansible\_user=ansible

ansible\_become=yes

ansible\_become\_method=sudo

```



\---



\## 🎭 Role Descriptions



\### Role 1: `sudoers\_archive`



| Property | Value |

|----------|-------|

| \*\*Purpose\*\* | Archive /etc/sudoers + /etc/sudoers.d/ on target hosts |

| \*\*Target Hosts\*\* | All RHEL/Linux machines in inventory |

| \*\*Execution Scope\*\* | Runs on each target individually |

| \*\*Privileges Required\*\* | sudo (to read /etc/sudoers files) |



\*\*Tasks Executed:\*\*



1\. \*\*Create temp directory\*\* on control node (`/tmp/ansible\_sudoers\_backups`)

2\. \*\*Archive sudoers\*\* on target host using gzip compression

3\. \*\*Fetch archive\*\* from target to control node (using Ansible fetch)

4\. \*\*Clean up\*\* remote temp files after fetch



\*\*Example Execution:\*\*

```bash

\# Single role execution

ansible-playbook -i inventory.ini -e "ansible\_user=ansible" \\

&#x20; -m include\_role -a name=sudoers\_archive sudoers\_backup\_targets

```



\### Role 2: `storage\_upload`



| Property | Value |

|----------|-------|

| \*\*Purpose\*\* | Upload archives to Azure Blob Storage |

| \*\*Target Hosts\*\* | Runs on control node (delegated) |

| \*\*Execution Scope\*\* | Centralized on AAP server |

| \*\*Privileges Required\*\* | Azure Storage Account Key |



\*\*Tasks Executed:\*\*



1\. \*\*Create backup directory\*\* on AAP server

2\. \*\*Copy file\*\* from EE container to AAP host disk (if needed)

3\. \*\*Upload to Azure Blob\*\* using Azure CLI (with retry)

4\. \*\*Clean up\*\* local temp files after successful upload

5\. \*\*Record status\*\* for HTML report (success/failure)



\*\*Two Upload Methods Provided:\*\*



\- \*\*Method 1 (CLI):\*\* `tasks/main.yml` - Uses `az storage blob upload`

\- \*\*Method 2 (REST API):\*\* `tasks/main.yml\_usingUMI` - Uses HTTP PUT + MSI token



\---



\## 📊 Variables Guide



\### Global Variables (`group\_vars/all.yml`)



```yaml

\# Azure Storage Configuration

az\_container: "rotation-reports"              # Blob container name

storage\_account\_name: "learninggallery"       # Azure storage account

storage\_account\_key: "{{ vault\_storage\_key }}"  # ⚠️ Use Vault!



\# AAP Control Node

aap\_server\_hostname: "aap-controller.learninggallery.com"

local\_temp\_dir: "/tmp/ansible\_sudoers\_backups"



\# Email Configuration

smtp\_sender: "abutalha3005@gmail.com"

email\_recipient: "abutalha@amityonline.com"

smtp\_host: "smtp.gmail.com"

smtp\_port: 587

smtp\_user: "{{ vault\_smtp\_user }}"  # ⚠️ Use Vault!

smtp\_pass: "{{ vault\_smtp\_pass }}"  # ⚠️ Use Vault!

```



\### Dynamic Variables (Generated at Runtime)



| Variable | Set By | Value Example | Purpose |

|----------|--------|---------------|---------|

| `backup\_timestamp` | pre\_tasks | `2024-01-15\_14-30-45` | Unique timestamp for run |

| `backup\_filename` | pre\_tasks | `ansiblenode1\_2024-01-15\_14-30-45\_sudoers\_backup.tar.gz` | Archive filename |

| `upload\_status` | storage\_upload role | `Success` / `Failed` | Upload result |

| `error\_message` | storage\_upload role | CLI error or `-` | Error details |



\### Role-Specific Variables



\*\*sudoers\_archive role\*\* (`roles/sudoers\_archive/defaults/main.yml`)

```yaml

local\_temp\_dir: "/tmp/ansible\_sudoers\_backups"

```



\*\*storage\_upload role\*\* (`roles/storage\_upload/defaults/main.yml`)

```yaml

local\_temp\_dir: "/tmp/ansible\_sudoers\_backups"

```



\### How to Override Variables



```bash

\# Method 1: Command-line extra variables

ansible-playbook ExtractSudoerDirnSudoerFile.yml \\

&#x20; -e "email\_recipient=newemail@example.com" \\

&#x20; -e "az\_container=custom-container"



\# Method 2: Create host\_vars for specific hosts

mkdir -p host\_vars/

echo "storage\_account\_name: custom\_account" > host\_vars/ansiblenode1.yml



\# Method 3: Use inventory variables

\# Edit inventory.ini with per-host vars

\[sudoers\_backup\_targets]

ansiblenode1 email\_recipient=node1@example.com



\# Method 4: Ansible Vault (recommended for secrets)

ansible-vault create group\_vars/all/vault.yml

\# Then reference: {{ vault\_storage\_key }}

```



\---



\## 🚀 How to Use



\### Use Case 1: Initial Sudoers Backup (Fresh Run)



\*\*Objective:\*\* Back up sudoers from all target hosts for the first time



```bash

\# Step 1: Verify connectivity

ansible -i inventory.ini sudoers\_backup\_targets -m ping



\# Step 2: Dry-run playbook

ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml \\

&#x20; --check -v



\# Step 3: Execute playbook

ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml -v



\# Step 4: Verify outputs

ls -lah /tmp/ansible\_sudoers\_backups/

az storage blob list --account-name learninggallery \\

&#x20; --container-name rotation-reports --output table



\# Step 5: Check email for HTML report

echo "✅ Check your email for execution report"

```



\*\*Expected Output:\*\*

```

PLAY \[Archive and Upload Sudoers to Azure] \*\*\*\*



TASK \[Wait for host to be reachable]

ok: \[ansiblenode1]

ok: \[ansiblenode2]



TASK \[Gather facts explicitly]

ok: \[ansiblenode1]

ok: \[ansiblenode2]



... (more tasks)



PLAY RECAP

ansiblenode1: ok=8 changed=3 unreachable=0 failed=0

ansiblenode2: ok=8 changed=3 unreachable=0 failed=0

```



\---



\### Use Case 2: Update Existing Infrastructure



\*\*Objective:\*\* Add new target hosts to the backup automation



```bash

\# Step 1: Add new hosts to inventory

cat >> inventory.ini << 'EOF'

ansiblenode4 ansible\_host=192.168.1.13 ansible\_user=ansible

ansiblenode5 ansible\_host=192.168.1.14 ansible\_user=ansible

EOF



\# Step 2: Test connectivity to new hosts

ansible -i inventory.ini ansiblenode4,ansiblenode5 -m ping



\# Step 3: Run playbook (will include new hosts)

ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml -v



\# Verify new archives

ls -lah /tmp/ansible\_sudoers\_backups/ | grep ansiblenode\[45]

```



\---



\### Use Case 3: Troubleshoot Failed Node



\*\*Objective:\*\* Diagnose and fix why a specific host failed to backup



```bash

\# Step 1: Run playbook with verbose output on single host

ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml \\

&#x20; --limit ansiblenode1 -vvv



\# Step 2: Manual tests on target

ssh ansible@ansiblenode1

sudo ls -la /etc/sudoers\*  # Verify files exist

sudo tar czf /tmp/test.tar.gz /etc/sudoers /etc/sudoers.d/

exit



\# Step 3: Check AAP connectivity

ansible -i inventory.ini ansiblenode1 -m setup | grep -i gather



\# Step 4: Test Azure CLI credentials on control node

az storage blob list --account-name learninggallery \\

&#x20; --account-key $AZURE\_STORAGE\_KEY --container-name rotation-reports



\# Step 5: Re-run playbook on single host

ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml \\

&#x20; --limit ansiblenode1 -v

```



\---



\### Use Case 4: Schedule Daily Automated Backups



\*\*Objective:\*\* Run backup every day at 2:00 AM on control node



```bash

\# Step 1: Create wrapper script

cat > /usr/local/bin/backup\_sudoers.sh << 'EOF'

\#!/bin/bash

cd /opt/ansible/sudoers-backup

/usr/bin/ansible-playbook -i inventory.ini ExtractSudoerDirnSudoerFile.yml

EOF



chmod +x /usr/local/bin/backup\_sudoers.sh



\# Step 2: Add cron job

crontab -e

\# Add: 0 2 \* \* \* /usr/local/bin/backup\_sudoers.sh



\# Step 3: Verify cron is running

crontab -l

ls -la /var/log/cron  # Check cron logs

```



\---



\## 🔒 Security Considerations



\### SSH Key Requirements



\*\*Setup SSH keys properly:\*\*



```bash

\# On control node, generate key pair

ssh-keygen -t rsa -b 4096 \\

&#x20; -f \~/.ssh/ansible\_id\_rsa \\

&#x20; -C "ansible@aap-controller"



\# Copy to target hosts

for host in ansiblenode1 ansiblenode2 ansiblenode3; do

&#x20; ssh-copy-id -i \~/.ssh/ansible\_id\_

