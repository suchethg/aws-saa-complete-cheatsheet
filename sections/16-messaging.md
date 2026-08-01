# **Messaging & Event-Driven Architecture**

Decoupling components is a core AWS architectural principle. These services act as the asynchronous glue between distributed components across AWS and on-premises environments.

## **Core Messaging & Streaming Services**

| Service | Pattern | Key Traits | Exam Scenario |
| ----- | ----- | ----- | ----- |
| **SQS** | Queue (Point-to-Point pull) | Pull model; messages stored up to 14 days; at-least-once delivery (Standard) or exactly-once (FIFO) | "decouple", "buffer requests", "worker pool", "hybrid on-prem to cloud decoupling" |
| **SNS** | Pub/Sub (Push fan-out) | Push model; up to 12.5M subscribers per topic; no persistence; supports Subscription Filter Policies | "fan-out", "notify multiple services", "filter messages to specific queues based on attributes" |
| **EventBridge** | Event bus (Pub/Sub + Rules) | Event routing with filter rules; integrates 200+ AWS services + SaaS apps; schema registry | "event-driven", "route events from AWS services", "SaaS integration" |
| **SWF** | Workflow Orchestration | Task coordination service for hybrid/complex workflows; supports human interaction tasks | "legacy workflow orchestration", "hybrid on-premise and EC2 worker coordination" |
| **Step Functions** | Modern Serverless Workflows | Visual state machine for serverless microservices; JSON-based State Language | "serverless orchestration", "replace SWF for new applications", "SAGA pattern" |
| **Kinesis Data Streams** | Real-time streaming | Retain 24hrs–365 days; multiple consumers; ordered within shard; replay capability | "real-time analytics", "clickstream", "IoT telemetry", "replay events" |
| **Kinesis Data Firehose** | Streaming ETL delivery | Fully managed delivery to S3/Redshift/OpenSearch; near-real-time (buffer 60s or 5MB) | "load streaming data to S3/Redshift", "transform + deliver", "near-real-time" |
| **Kinesis Data Analytics** | Real-time SQL on streams | SQL or Apache Flink on Kinesis/MSK streams; real-time aggregations/anomaly detection | "real-time SQL on streaming data", "running averages", "anomaly detection" |
| **MSK (Managed Kafka)** | Managed Apache Kafka | AWS manages Kafka brokers; Kafka API compatible; persistent storage on EBS | "Kafka", "existing Kafka workloads", "exactly-once semantics", "long retention" |
| **MQ (Amazon MQ)** | Managed message broker | ActiveMQ / RabbitMQ; for migrating on-prem apps using JMS/AMQP/MQTT/STOMP | "migrate from on-prem ActiveMQ/RabbitMQ", "MQTT", "STOMP", "AMQP" |

---

## **SNS Subscription Filter Policies (Fanout with Selection)**

By default, every subscriber to an SNS topic receives every message. **Subscription Filter Policies** allow subscribers (such as SQS queues or Lambda functions) to only receive messages matching specified criteria based on message attributes.

┌──[ Filter: RequestType = Auto ]────► SQS Queue A (Auto Quotes)
              │
[ SNS Topic ] ───┼──[ Filter: RequestType = Home ]────► SQS Queue B (Home Quotes)
│
└──[ Filter: RequestType = Life ]────► SQS Queue C (Life Quotes)

* **Exam Key Feature:** Avoids deploying multiple SNS topics or writing filtering code inside application workers. A single SNS topic fans out filtered messages to specific queues based on attributes.

---

## **Decoupling Hybrid Applications (On-Premises + AWS)**

When creating a decoupled architecture that spans both **on-premises servers** and **EC2 instances**:
* **Amazon SQS:** On-premises servers and EC2 instances both poll SQS queues over HTTPS (`ReceiveMessage`). Requires no public IP, open firewall ports, or direct network peering.
* **Amazon SWF / AWS Step Functions:** On-prem workers act as "Activity Workers" that poll AWS over HTTPS for tasks, process them, and return results.

| ⚠️ EXAM TRAP: VPC Peering cannot connect an on-premises network to an AWS VPC! VPC Peering is ONLY between two AWS VPCs. For hybrid decoupling without VPN/Direct Connect, use polling services like **SQS** or **SWF/Step Functions**. |
| :---- |

---

## **SQS Deep Dive**

| Property | Standard SQS | FIFO SQS |
| ----- | ----- | ----- |
| Ordering | Best-effort (not guaranteed) | Strict First-In-First-Out per Message Group ID |
| Delivery | At-least-once (duplicates possible) | Exactly-once processing (deduplication) |
| Throughput | Nearly unlimited | 300 msg/sec (3,000 with batching) |
| Deduplication | Not supported | 5-minute deduplication window (content or ID based) |
| Use case | High throughput, order doesn't matter | Order processing, financial transactions |
| Queue name | Any name | Must end in `.fifo` |

### **SQS Important Properties**

| Property | Default | Max | Purpose |
| ----- | ----- | ----- | ----- |
| Visibility Timeout | 30 seconds | 12 hours | Duration message is hidden after consumer receives it; if not deleted, becomes visible again |
| Message Retention | 4 days | 14 days | How long messages stay in queue |
| Message Size | 256 KB | 256 KB (use Extended Client Library for up to 2GB via S3) | Store large payloads in S3; SQS holds reference |
| Long Polling | 0 (short poll) | 20 seconds | Wait for messages instead of returning empty; reduces API calls and cost |
| Dead Letter Queue | Disabled | — | Messages that fail `maxReceiveCount` times sent to DLQ for analysis |
| Delay Queue | 0 seconds | 15 minutes | Delay delivery of new messages by N seconds; useful for dependent processes |

| 💡 TIP: Visibility Timeout confusion: If consumer crashes processing a message, the visibility timeout expires and the message becomes visible AGAIN. Set visibility timeout longer than your maximum processing time to prevent duplicate processing. |
| :---- |

---

## **SNS vs SQS vs EventBridge — When to Choose**

| If the scenario says... | Choose |
| ----- | ----- |
| "Fan-out to multiple services with topic filter rules for each queue" | SNS → SQS queues with Subscription Filter Policies |
| "Decouple producer from consumer; buffer traffic spikes across hybrid/cloud" | SQS (or SWF for complex workflows) |
| "Route events from 200+ AWS services or third-party SaaS apps with filter rules" | EventBridge Event Bus |
| "Real-time stream processing, replay events, multiple consumers" | Kinesis Data Streams |
| "Deliver streaming data to S3/Redshift without managing consumers" | Kinesis Data Firehose |
| "Migrate existing RabbitMQ/ActiveMQ to managed service" | Amazon MQ |
| "Kafka workload, need Kafka API compatibility" | MSK (Managed Streaming for Kafka) |

---

## **Kinesis Shards & Scaling**

**Shard capacity:** 1 MB/s write, 2 MB/s read per shard. Scale by adding shards (Shard Split) or combining them (Shard Merge).

**Enhanced Fan-Out:** Each consumer gets its own dedicated 2 MB/s pipe (instead of sharing 2 MB/s across all consumers). Use when you have multiple Lambda functions or applications reading the same stream and need full throughput per consumer.

| 🧠 MNEMONIC: SNS Filter = Fanout with selection. SQS/SWF = Pull anywhere (On-prem + Cloud). Kinesis = Real-time + Replay + Multiple consumers. Firehose = Load + Near-real-time + Delivery to S3/Redshift. |
| :---- |

| NEXT: SECTION 8 — MONITORING, LOGGING, SECURITY & COMPLIANCE |
| :---: |

---

[< Elastic Load Balancing - Three Types Compared](15-load-balancing.md) | [Back to README](../README.md) | [Monitoring Stack - CloudWatch, CloudTrail, Config >](17-monitoring.md)
