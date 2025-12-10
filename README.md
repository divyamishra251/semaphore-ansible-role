# Semaphore Ansible Role

**Author:** Divya Mishra
**Last Updated:** 10 Dec 2025

---

## 📌 Overview

This Ansible role installs and configures **Semaphore**, a lightweight and modern web UI for running Ansible playbooks.
The role is designed to be:

* **OS-independent** (supports Ubuntu/Debian, RHEL/CentOS, Amazon Linux)
* **Database-flexible** (PostgreSQL or MySQL)
* **Idempotent & production ready**
* **Compatible with both static and AWS dynamic EC2 inventories**

---

## 📘 What is Semaphore?

Semaphore is an open-source UI that provides:

* Project & template management
* Inventory and credentials management
* Role-based access control
* Job execution history
* Scheduling
* Simple and secure UI for Ansible automation

Official documentation:
[https://docs.ansible-semaphore.com](https://docs.ansible-semaphore.com)

---

## 🎯 Purpose of This Role

The goal of this role is to:

* Install Semaphore consistently across different operating systems
* Allow DB selection (PostgreSQL or MySQL) via variables
* Follow Ansible best practices like FQMN, handlers, OS checks, etc.
* Provide an automated, reproducible installation method
* Support real-world use cases including AWS EC2 dynamic inventory

---

## 🔗 Reference Links

| Purpose                     | Link                                                                           |
| --------------------------- | ------------------------------------------------------------------------------ |
| Semaphore Ansible Role Repo | [https://github.com/OT-OSM/semaphoreui](https://github.com/OT-OSM/semaphoreui) |
| Semaphore Official Docs     | [https://docs.ansible-semaphore.com](https://docs.ansible-semaphore.com)       |
| Ansible Official Docs       | [https://docs.ansible.com](https://docs.ansible.com)                           |

---

## 🖥 Supported Operating Systems

| OS Family    | Versions               |
| ------------ | ---------------------- |
| Debian       | Ubuntu 20+, Debian 10+ |
| RedHat       | CentOS 8+, RHEL 8+     |
| Amazon Linux | Amazon Linux 2         |

OS validation is automatic using `ansible_os_family`.

---

## ⚙️ Prerequisites

### System Requirements

| Requirement | Description    |
| ----------- | -------------- |
| RAM         | 2 GB minimum   |
| CPU         | 1 vCPU or more |
| Access      | SSH + sudo     |
| Internet    | Required       |

### Inside Virtualenv

| Package               | Purpose                         |
| --------------------- | ------------------------------- |
| ansible               | Required for playbook execution |
| boto3 & botocore      | AWS dynamic inventory           |
| amazon.aws collection | Required by aws_ec2 plugin      |

Install dependencies:

```bash
pip install ansible boto3 botocore
ansible-galaxy collection install amazon.aws
```

---

## 🧰 Virtual Environment Setup

```bash
python3 -m venv ansible-venv
source ansible-venv/bin/activate
pip install ansible
```

---

## 📂 Role Directory Structure

```
roles/
└── semaphore/
    ├── defaults/main.yml
    ├── vars/main.yml
    ├── handlers/main.yml
    ├── tasks/main.yml
    └── templates/
        ├── config.json.j2
        └── semaphore.service.j2

inventory/aws_ec2.yml
semaphore.yml
```

---

## 🏆 Best Practices Implemented

| Practice                   | Description                         |
| -------------------------- | ----------------------------------- |
| FQMN                       | Uses `ansible.builtin.*` everywhere |
| Handlers                   | Systemd reload handled cleanly      |
| Flush handlers             | Ensures service starts correctly    |
| OS Detection               | Install logic per OS family         |
| DB Dialect Switching       | PostgreSQL ↔ MySQL support          |
| Clean Directory Management | Only needed paths created           |
| No embedded DB install     | DB treated as external dependency   |

---

## 🗄 Database Configuration

### PostgreSQL

```yaml
pg_host: "127.0.0.1"
pg_port: 5432
pg_user: "semaphore"
pg_pass: "semaphore"
pg_db: "semaphore"
```

### MySQL

```yaml
mysql_host: "127.0.0.1"
mysql_port: 3306
mysql_user: "semaphore"
mysql_pass: "semaphore"
mysql_db: "semaphore"
```

### Switch DB Dialect

Edit:

```
roles/semaphore/defaults/main.yml
```

Set:

```yaml
semaphore_db_dialect: postgres   # or mysql
```

---

## 🛠 Installing PostgreSQL (Optional for Testing)

```bash
sudo apt install postgresql postgresql-contrib -y
sudo -u postgres psql
```

```sql
CREATE ROLE semaphore LOGIN PASSWORD 'semaphore';
CREATE DATABASE semaphore OWNER semaphore;
\q
```

---

## 🛠 Installing MySQL (Optional for Testing)

```bash
sudo apt install mysql-server -y
sudo mysql
```

```sql
CREATE DATABASE semaphore;
CREATE USER 'semaphore'@'localhost' IDENTIFIED BY 'semaphore';
GRANT ALL PRIVILEGES ON semaphore.* TO 'semaphore'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## ▶️ Running the Playbook

```bash
ansible-playbook -i inventory/aws_ec2.yml semaphore.yml -u ubuntu --private-key ~/your-key.pem
```

Expected:

```
failed=0
```

---

## 🔍 Testing Semaphore

### Service status

```bash
sudo systemctl status semaphore
```

### Test if service responds

```bash
curl -I http://localhost:3000
```

### Browser

```
http://<EC2_PUBLIC_IP>:3000
```

---

## ❗ Troubleshooting

| Issue                   | Fix                                |
| ----------------------- | ---------------------------------- |
| Semaphore not starting  | `journalctl -u semaphore -n 50`    |
| Wrong DB credentials    | Check `/etc/semaphore/config.json` |
| Port 3000 busy          | `sudo lsof -i:3000`                |
| Dynamic inventory fails | Reinstall `amazon.aws`             |
| apt/dpkg lock           | `sudo dpkg --configure -a`         |

---

## 📜 Conclusion

This repository provides a complete, OS-independent, production-ready Ansible role to install Semaphore with full DB switching, testing steps, and troubleshooting guidance.

---
