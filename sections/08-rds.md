# **RDS — Relational Database Service**

AWS RDS is a managed relational database service supporting MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and DB2.

---

## **RDS Multi-AZ vs Read Replicas — The Classic Confusion**

| Feature | Multi-AZ (High Availability) | Read Replicas (Read Scaling) |
| ----- | ----- | ----- |
| **Purpose** | High Availability & Disaster Recovery | Scaling read-heavy workloads |
| **Replication** | Synchronous (no data loss) | Asynchronous (replication lag possible) |
| **Readability** | **No** — Standby is passive/hidden | **Yes** — Dedicated endpoints for read queries |
| **Failover** | Automatic DNS failover (1–2 mins) | Manual promotion required |
| **Cross-Region** | Same region across multiple AZs | Can be cross-region (also used for DR) |
| **Exam Keywords** | *"high availability", "automatic failover"* | *"read throughput", "reporting offload"* |

| ⚠️ EXAM TRAP: Multi-AZ does NOT improve read performance because the standby DB is passive and unreadable. If a scenario requires both High Availability AND Read Scaling, use Multi-AZ alongside Read Replicas. |
| :---- |

---

## **RDS Security & IAM Database Authentication**

| Security Layer | Mechanism | Exam Key Concept |
| ----- | ----- | ----- |
| **IAM DB Authentication** | Auth tokens (valid 15 mins) via SigV4 | Eliminates database passwords! Uses EC2 IAM Roles for access. Supported on MySQL & PostgreSQL. |
| **Encryption at Rest** | AWS KMS (AES-256) | Must be enabled at creation. Cannot encrypt an existing unencrypted DB directly (snapshot -> copy with encryption -> restore). |
| **Encryption in Transit** | SSL / TLS | Forces encrypted connections between app servers and RDS. Enabled by default with IAM DB Auth. |
| **Network Security** | Security Groups & Subnet Groups | RDS should ALWAYS sit in a **Private Subnet**. Use Security Group rules to allow traffic ONLY from application EC2 instances or ALBs. |

[ EC2 Instance / App ] ──(1) Get IAM Token──► [ AWS STS / IAM ]
│
└──(2) Connect using IAM Token (SSL)──► [ RDS Database ]

---

## **RDS Backup, Restore & Storage Scaling**

| Feature | Details |
| ----- | ----- |
| **Automated Backups** | Daily snapshots + transaction logs (PITR up to 35 days). Restoring creates a **NEW** DB instance. |
| **Manual Snapshots** | User-triggered; kept until manually deleted. Can be copied across regions/accounts. |
| **Storage Auto Scaling** | Automatically increases storage size when free space falls below 10%. Prevents out-of-space downtime. |
| **Deletion Protection** | Prevents accidental deletion of production databases via API or AWS Console. |

| 💡 TIP: Modifying Unencrypted Databases: If you have an unencrypted RDS instance and need encryption, take a manual snapshot, copy the snapshot with KMS encryption enabled, and restore a new database from the encrypted snapshot. |
| :---- |

---

[< Database Selection Guide](07-database-selection.md) | [Back to README](../README.md) | [Aurora - The Exam Favorite >](09-aurora.md)
