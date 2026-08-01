Here is the updated, complete **`README.md`** file for **Amazon DynamoDB** tailored for the AWS SAA-C03 exam. It incorporates core concepts, indexing, VPC gateway endpoints, AWS Backup integrations, auto-scaling mechanics, identity federation, and common exam traps.


# **DynamoDB — The Serverless NoSQL Giant**

Amazon DynamoDB is a fully managed, serverless, key-value and document NoSQL database designed for single-digit millisecond performance at any scale. Understanding its consistency models, capacity modes, indexing, streams, security controls, and backup mechanisms is critical for the SAA-C03 exam.

---

## **DynamoDB Core Concepts**

| Concept | Detail | SAA-C03 Exam Implication |
| ----- | ----- | ----- |
| **Primary Key** | Simple (Partition Key / `HASH`) or Composite (Partition Key `HASH` + Sort Key `RANGE`). | Design partition keys with high cardinality for uniform data distribution — avoid **hot partitions**. |
| **Read Consistency** | **Eventually Consistent** (default, 0.5 RCU per 4 KB) vs. **Strongly Consistent** (1 RCU per 4 KB). | Financial/inventory apps needing instant read accuracy after a write require Strongly Consistent reads (at 2x RCU cost). |
| **Capacity Modes** | **Provisioned** (fixed RCU/WCU; auto-scaling optional) vs. **On-Demand** (pay per request; zero capacity planning). | **On-Demand** for unpredictable or unknown traffic spikes; **Provisioned + Auto Scaling** for predictable, steady workloads. |
| **DynamoDB Streams** | Time-ordered sequence of item-level modification logs (24-hour retention). | Used for event-driven architectures: triggering AWS Lambda, cross-region replication, or streaming to Kinesis Data Streams. |
| **TTL (Time to Live)** | Automatically expires and deletes items based on an epoch timestamp attribute at **no additional cost** (no WCU consumed). | Ideal for session management, temporary tokens, ephemeral logs, and data retention compliance. |
| **DynamoDB Accelerator (DAX)** | Managed in-memory write-through cache delivering **microsecond latency**. | Caches individual item reads (`GetItem`/`BatchGetItem`) and query results. Requires **no code changes** for read acceleration. |
| **Global Tables** | Multi-region, active-active replication using DynamoDB Streams. | Enables global applications to read/write locally with multi-region disaster recovery and high availability. |
| **ACID Transactions** | All-or-nothing operations across multiple items and tables (`TransactWriteItems`, `TransactGetItems`). | Financial ledgers, banking transactions, complex order processing, and fulfillment systems. |

---

## **DynamoDB Secondary Indexes**

Indexes allow querying data using attributes other than the table's primary key.

| Index Type | Created When | Key Structure | Read Consistency | Capacity Management | SAA Exam Use Case |
| ----- | ----- | ----- | ----- | ----- | ----- |
| **LSI** (Local Secondary Index) | Table Creation **ONLY** | Same Partition Key, different Sort Key | Strongly or Eventually Consistent | Shares the main table's provisioned capacity | Query the same partition key with a different sort key/order. |
| **GSI** (Global Secondary Index) | Any time (Creation or Post-creation) | Different Partition Key and/or Sort Key | **Eventually Consistent ONLY** | Requires **its own** provisioned RCU/WCU | Query on any arbitrary attribute; highly flexible. |

| 🧠 MNEMONIC: LSI = Local = Same partition key = created at table Launch. GSI = Global = Greater flexibility = can be added anytime, own capacity. |
| :---- |

| ⚠️ EXAM TRAP: GSI Throttling Impact: If a GSI's provisioned write capacity is throttled, **write operations on the main table will also be throttled**, even if the main table has unused WCU capacity! Always monitor and auto-scale GSI capacity alongside table capacity. |
| :---- |

---

## **VPC Security & Network Connectivity**

By default, DynamoDB endpoints are public service endpoints accessible over the internet.

| Mechanism | Configuration Details | Exam Key Takeaways |
| ----- | ----- | ----- |
| **Gateway VPC Endpoint** | Creates a private connection between a VPC and DynamoDB over the AWS internal network. | • **Free of charge**.<br>• Must associate with VPC **Route Tables** using the DynamoDB **Prefix List**.<br>• Ensures traffic between EC2/EKS and DynamoDB **does not traverse the public internet**. |
| **Fine-Grained Access Control** | Use Web Identity Federation (Cognito/OIDC) with IAM policy condition keys (`dynamodb:LeadingKeys`). | Enables mobile/client applications to access DynamoDB directly while restricting users to reading/writing **only their own partition key items**. |
| **Encryption at Rest** | Encrypted by default using AWS KMS. | Supports AWS Owned Keys (default/free), AWS Managed Keys (`aws/dynamodb`), or Customer Managed Keys (CMKs). |


```

┌─────────────────┐        ┌────────────────────────┐        ┌──────────────────────┐
│  Private Subnet │        │  VPC Route Table       │        │  DynamoDB Gateway    │
│  EC2 / EKS Task ├───────►│  Dest: Prefix List     ├───────►│  Endpoint            ├───────► Amazon DynamoDB
└─────────────────┘        │  Target: vpce-xxxxxxx  │        └──────────────────────┘
└────────────────────────┘

```

---

## **Backup, Disaster Recovery & AWS Backup Integration**

| Feature | Scope & Retention | SAA Exam Strategy |
| ----- | ----- | ----- |
| **Point-in-Time Recovery (PITR)** | Continuous automatic backups; per-second recovery within a **35-day window**. | **Same Account & Region ONLY.** Does not support cross-account or cross-region automated copy on its own. |
| **Native On-Demand Backups** | Full table snapshots taken manually or programmatically. | Zero impact on table performance or capacity. Cannot be natively copied across accounts/regions without AWS Backup. |
| **AWS Backup for DynamoDB** | Centralized policy-based backup management across AWS services. | **Mandatory choice** when the scenario requires **Cross-Account Backup**, **Cross-Region Backup**, lifecycle rules to cold storage, or centralized compliance auditing. |

---

## **Capacity Planning & Auto Scaling Pitfalls**

* **CLI / IaC Default Behavior:** Tables created via the **AWS CLI or SDK do NOT have Auto Scaling enabled by default**. Auto Scaling must be explicitly defined using Application Auto Scaling policies, otherwise sudden spikes will throw a `ProvisionedThroughputExceededException`.
* **On-Demand vs. Provisioned Choice Matrix:**
  * **Choose On-Demand:** New application releases, unknown or unpredictable traffic patterns, low utilization with sudden massive spikes.
  * **Choose Provisioned + Auto Scaling:** Well-known, predictable, or steadily growing workloads. Most cost-effective at sustained scale.

---

## **Architecture Integration & Exam Patterns**


```

┌─────────────────────────────────────────────────────────────────────────────┐
│                       SERVERLESS ARCHITECTURE PATTERN                       │
│                                                                             │
│  [ Client App ] ──► [ API Gateway ] ──► [ AWS Lambda ] ──► [ DAX / DynamoDB ]│
│                          │                                                  │
│                    (API Caching)                                            │
└─────────────────────────────────────────────────────────────────────────────┘

```

| Pattern / Integration | Valid Architecture? | Exam Nuance |
| ----- | ----- | ----- |
| **CloudFront + DynamoDB Direct** | ❌ **Invalid** | CloudFront **cannot** use DynamoDB directly as an origin. Must route via API Gateway or Application Load Balancer to a compute tier first. |
| **Timestream Integration** | ❌ **Invalid** | Amazon Timestream is a dedicated time-series database (IoT/metrics), **not** a backup or PITR destination for DynamoDB. |
| **DAX Caching Layer** | ✅ **Valid** | Use DAX for read-heavy workloads needing microsecond latency. Caches `GetItem` and `Query` operations without complex cache invalidation logic. |
| **API Gateway + Lambda + DynamoDB** | ✅ **Valid** | Standard serverless tier. Use API Gateway response caching for external request throttling and DAX for DB-level read acceleration. |

---

## **DynamoDB Decision Matrix for SAA-C03**

| Scenario / Requirement | Recommended Solution |
| ----- | ----- |
| Microsecond read latency required for high-frequency reads without changing application code | **DynamoDB Accelerator (DAX)** |
| Prevent database calls from an EKS cluster/VPC from traversing the public internet | **DynamoDB Gateway VPC Endpoint + Route Table update** |
| Disaster recovery policy requiring DynamoDB backups to be stored in a separate secondary AWS account | **AWS Backup (configured for cross-account backup copies)** |
| Mobile application requiring users to securely read/write only their own user profile items | **IAM Policy with `dynamodb:LeadingKeys` condition + Cognito Identity** |
| Active-active global database deployment with local read/write access across US and Europe | **DynamoDB Global Tables (Streams enabled)** |
| Automatically purge user session data older than 7 days without incurring write costs | **DynamoDB Time to Live (TTL)** |

---

[< Aurora - The Exam Favorite](09-aurora.md) | [Back to README](../README.md) | [ElastiCache - Caching Strategies >](11-elasticache.md)

```
