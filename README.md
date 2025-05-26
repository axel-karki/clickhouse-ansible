## ClickHouse + MinIO Backup Setup

### What is ClickHouse?

**ClickHouse** is an open-source, column-oriented database management system designed for high-performance analytics. It allows users to generate reports from large datasets in real time. ClickHouse is widely used in data warehousing, log analytics, and time-series analysis due to its speed, scalability, and efficiency.

---

### Backup with MinIO

**MinIO** is a high-performance, S3-compatible object storage system. It is ideal for storing backups of ClickHouse tables in a cost-efficient and scalable way. Backups are managed using the `clickhouse-backup` tool, which supports uploading snapshots directly to S3-compatible services like MinIO.

---

## Playbook Execution Order

### 1. Install ClickHouse

**Playbook**: `playbooks/install_clickhouse.yml`

* Installs ClickHouse server on a new VM
* Configures databases, users, ports, and system settings
* Enables the `clickhouse-backup` tool for local and remote backups

---

### 2. Setup MinIO Backup

**Playbook**: `playbooks/minio-backup.yml`

* Configures MinIO user, bucket, and S3 alias for `clickhouse-backup`
* Adds scheduled backup jobs for regular snapshots to MinIO

For detailed MinIO setup, refer to the repository:
[axel-karki/minio-ansible](https://github.com/axel-karki/minio-ansible)

---

## Secure Configuration

Edit the following file to set passwords and credentials for both clickhouse configuration and MinIO access:

```bash
roles/clickhouse/vars/main.yml
```

**Encrypt the file using Ansible Vault:**

```bash
ansible-vault encrypt roles/clickhouse/vars/main.yml
```

To edit securely later:

```bash
ansible-vault edit roles/clickhouse/vars/main.yml
```

To run playbooks using the vault:

```bash
ansible-playbook playbooks/install_clickhouse.yml --ask-vault-pass
```

Or use a vault password file:

```bash
ansible-playbook playbooks/install_clickhouse.yml --vault-password-file ~/.vault_pass.txt
```

---

### Future Enhancements: Remote-Only Backups
To reduce local disk usage and streamline storage:

Configure the backup system to skip local storage and push directly to MinIO.

Update your clickhouse-backup configuration (config.yml) as follows:

```
backups_to_keep_local: -1        # Keep backup only until remote upload completes
backups_to_keep_remote: 3        # Retain latest 3 backups on MinIO
log_level: info                  # Logging level for troubleshooting
```

This approach allows you to:

* Avoid using disk space on the ClickHouse server

* Automatically delete local backups after upload

* Retain only a fixed number of recent backups in remote storage

* Manually remove older backups if needed:

```
clickhouse-backup delete remote <backup_name>
```
