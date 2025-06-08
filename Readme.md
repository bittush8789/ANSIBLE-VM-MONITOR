# Ansible VM Monitor Automation

This project automates monitoring of virtual machines (VMs) using Ansible. It uses AWS EC2 dynamic inventory, tags instances, collects system health metrics (CPU, memory, disk, network), and sends reports via email.

---

## ✅ Project Goals

- Automate VM tagging and monitoring
- Track CPU, memory, disk, and network statistics
- Send performance data via email
- Use Infrastructure-as-Code principles

---

## 🧰 Prerequisites

- Ubuntu-based control node
- AWS EC2 instances with proper tagging (e.g., `Environment=dev`)
- SSH access via key pair (`.pem` file)
- Ansible installed on control machine
- Python 3.6+ with `venv`, `boto3`, `botocore`, and `docker`
- AWS CLI configured (`aws configure`)
- Public SSH key (`~/.ssh/id_rsa.pub`) generated

---

## ⚙️ Setup Steps

### 1. Update System

```bash
sudo apt update && sudo apt upgrade -y

### 2. Install Ansible

```bash

sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y

#### 3. Install AWS CLI

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws configure

### 4. Set Up Python Virtual Environment

sudo apt install python3-venv -y
python3 -m venv ansible-env
source ansible-env/bin/activate
pip install boto3 botocore docker
ansible-galaxy collection install amazon.aws

### 🔖 Tagging EC2 Instances Automatically

### Create and run the tagging script to rename EC2 instances:

#!/bin/bash

instance_ids=$(aws ec2 describe-instances \
  --filters "Name=tag:Environment,Values=dev" "Name=instance-state-name,Values=running" \
  --query 'Reservations[*].Instances[*].InstanceId' \
  --output text)

sorted_ids=($(echo "$instance_ids" | tr '\t' '\n' | sort))

counter=1
for id in "${sorted_ids[@]}"; do
  name="web-$(printf "%02d" $counter)"
  echo "Tagging $id as $name"
  aws ec2 create-tags --resources "$id" --tags Key=Name,Value="$name"
  ((counter++))
done

## 🗂️ Dynamic Inventory Configuration (aws_ec2.yaml)

plugin: amazon.aws.aws_ec2
regions:
  - ap-south-1
filters:
  tag:Environment: dev
  instance-state-name: running
compose:
  ansible_host: public_ip_address
keyed_groups:
  - key: tags.Name
    prefix: name
  - key: tags.Environment
    prefix: env

### Set this in ansible.cfg:

[defaults]
inventory = ./inventory/aws_ec2.yaml
host_key_checking = False

[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null

### 🔐 Inject SSH Public Key to EC2 Hosts

#!/bin/bash

PEM_FILE="DevOps-Shack.pem"
PUB_KEY=$(cat ~/.ssh/id_rsa.pub)
USER="ubuntu"
INVENTORY_FILE="inventory/aws_ec2.yaml"

HOSTS=$(ansible-inventory -i $INVENTORY_FILE --list | jq -r '._meta.hostvars | keys[]')

for HOST in $HOSTS; do
  echo "Injecting key into $HOST"
  ssh -o StrictHostKeyChecking=no -i $PEM_FILE $USER@$HOST "
    mkdir -p ~/.ssh &&
    echo \"$PUB_KEY\" >> ~/.ssh/authorized_keys &&
    chmod 700 ~/.ssh &&
    chmod 600 ~/.ssh/authorized_keys
  "
done

### 🚀 Run the Ansible Playbook
### Clone the project and run:




git clone https://github.com/jaiswaladi246/Ansible-VM-Monitor.git
cd Ansible-VM-Monitor
ansible-playbook playbook.yaml


## 📊 Output
### The playbook collects system stats (CPU, memory, disk, network).

Results are sent to configured email addresses using Ansible mail module.

📌 Tools & Technologies
Ansible

AWS EC2
Terraform (optional for provisioning)
Boto3/Botocore
Docker
Python venv
Prometheus/Grafana (optional for advanced monitoring)

### 📈 Possible Enhancements
Set threshold-based alerting

Integrate with Prometheus/Grafana dashboards

Extend for multi-cloud support

Add log rotation and central logging (ELK)

Project by: GitHub - bittush8789/ANSIBLE-VM-MONITOR


