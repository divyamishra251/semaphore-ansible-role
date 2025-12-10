# Semaphore Ansible Role

---

## What is Semaphore?

Semaphore is an open-source UI used for running and managing Ansible automation.
It provides:

* Projects, templates, and inventories
* Role-based access control
* Job logs and execution history
* Scheduling support
* Centralized credential management
* Simple, safe and clean UI for operational teams

Official docs: [https://docs.ansible-semaphore.com](https://docs.ansible-semaphore.com)

---

## Purpose of This Role

The objective of this role is to:

* Install Semaphore consistently across multiple OS families
* Allow DB selection (PostgreSQL or MySQL) via variables
* Follow Ansible best practices (FQMN, handlers, OS checks)
* Provide a fully automated and reproducible installation
* Work seamlessly with AWS dynamic inventory

---

## Supported Operating Systems

| OS Family    | Versions               |
| ------------ | ---------------------- |
| Debian       | Ubuntu 20+, Debian 10+ |
| RedHat       | CentOS 8+, RHEL 8+     |
| Amazon Linux | Amazon Linux 2         |

OS detection is done using `ansible_os_family`.

---

## Prerequisites

### System Requirements

| Requirement | Description    |
| ----------- | -------------- |
| RAM         | 2 GB minimum   |
| CPU         | 1 vCPU or more |
| Access      | SSH + sudo     |
| Internet    | Required       |

### Python venv Requirements

| Package               | Purpose                |
| --------------------- | ---------------------- |
| ansible               | Playbook execution     |
| boto3 & botocore      | AWS dynamic inventory  |
| amazon.aws collection | `aws_ec2` plugin usage |

Install:

```bash
pip install ansible boto3 botocore
ansible-galaxy collection install amazon.aws
```

---

## Virtual Environment Setup

```bash
python3 -m venv ansible-venv
source ansible-venv/bin/activate
pip install ansible
```

---

## Role Directory Structure

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

## Database Configuration

### PostgreSQL Example

```yaml
pg_host: "127.0.0.1"
pg_port: 5432
pg_user: "semaphore"
pg_pass: "semaphore"
pg_db: "semaphore"
```

### MySQL Example

```yaml
mysql_host: "127.0.0.1"
mysql_port: 3306
mysql_user: "semaphore"
mysql_pass: "semaphore"
mysql_db: "semaphore"
```

### Choose Dialect

Edit:

```
roles/semaphore/defaults/main.yml
```

Set:

```yaml
semaphore_db_dialect: postgres   # or mysql
```

---

## Installing PostgreSQL (Optional)

```bash
sudo apt install postgresql postgresql-contrib -y
sudo -u postgres psql
```

```sql
CREATE ROLE semaphore LOGIN PASSWORD 'semaphore';
CREATE DATABASE semaphore OWNER semaphore;
```

---

## Installing MySQL (Optional)

```bash
sudo apt install mysql-server -y
sudo mysql
```

```sql
CREATE DATABASE semaphore;
CREATE USER 'semaphore'@'localhost' IDENTIFIED BY 'semaphore';
GRANT ALL PRIVILEGES ON semaphore.* TO 'semaphore'@'localhost';
FLUSH PRIVILEGES;
```

---

## Running the Playbook

```bash
ansible-playbook -i inventory/aws_ec2.yml semaphore.yml -u ubuntu --private-key ~/your-key.pem
```

Expected:

```
failed=0
```

---

## Testing Semaphore

### Service status

```bash
sudo systemctl status semaphore
```

### Check port

```bash
curl -I http://localhost:3000
```

### Browser

```
http://<EC2_PUBLIC_IP>:3000
```

---

## Best Practices Implemented

| Practice                  | Description                     |
| ------------------------- | ------------------------------- |
| FQMN                      | Uses `ansible.builtin.*`        |
| Handlers                  | Clean systemd reload            |
| Flush handlers            | Ensures correct service startup |
| OS Detection              | Installs per OS family          |
| DB Dialect Switching      | PostgreSQL/MySQL support        |
| Proper directory handling | Limited to required paths       |
| External DB assumption    | No embedded database            |

---

## Troubleshooting

| Issue                  | Fix                                 |
| ---------------------- | ----------------------------------- |
| Semaphore not starting | `journalctl -u semaphore -n 50`     |
| DB connection refused  | Verify `/etc/semaphore/config.json` |
| Port 3000 busy         | `sudo lsof -i:3000`                 |
| AWS inventory missing  | Reinstall `amazon.aws` collection   |
| apt/dpkg lock          | `sudo dpkg --configure -a`          |

---

## Conclusion

This role provides a fully automated, production-ready setup of Semaphore across multiple environments.
It is simple to extend, easy to override using variables, and ideal for both local and cloud deployments.

---


## Reference Links

| Purpose                     | Link                                                                           |
| --------------------------- | ------------------------------------------------------------------------------ |
| Semaphore Ansible Role Repo | [https://github.com/OT-OSM/semaphoreui](https://github.com/OT-OSM/semaphoreui) |
| Semaphore Official Docs     | [https://docs.ansible-semaphore.com](https://docs.ansible-semaphore.com)       |
| Ansible Official Docs       | [https://docs.ansible.com](https://docs.ansible.com)                           |

---

**Author:** Divya Mishra

**Last Updated on:** 10-Dec-2025

---
