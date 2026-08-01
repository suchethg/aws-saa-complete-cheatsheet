Here is the complete, formatted **`README.md`** file for **Amazon VPC & Hybrid Networking** ready for your study notes repo, incorporating all the exact rules, traps, and ephemeral port details tested on the SAA-C03 exam.


# **VPC — Virtual Private Cloud & Hybrid Networking**

Amazon VPC is your logically isolated virtual network in AWS. The SAA-C03 exam tests VPC design heavily—focusing on subnet routing, stateful vs. stateless security rules, private service endpoints, IPv6 migration, and hybrid network topology.

---

## **VPC Fundamentals & Core Components**

| Component | Description | SAA-C03 Key Exam Facts |
| ----- | ----- | ----- |
| **VPC** | Region-wide isolated virtual network. | CIDR block sizing: `/16` (65,536 IPs) to `/28` (16 IPs). Spans all AZs in a Region. Default soft limit: 5 VPCs per Region. |
| **Subnet** | AZ-specific range of IP addresses within a VPC. | **AZ-bound** (cannot span AZs). AWS reserves **5 IP addresses** per subnet (Network, VPC Router, DNS, Reserved, Broadcast). |
| **Internet Gateway (IGW)** | Horizontally scaled, highly available VPC edge component. | Enables bi-directional internet access for public subnets. **1 IGW per VPC max.** |
| **NAT Gateway** | Managed outbound-only internet translator for private subnets. | Deployed in a **Public Subnet**; requires an **Elastic IP (EIP)**. IPv4 only. **AZ-specific** (not HA across AZs by default). |
| **Egress-Only IGW** | Stateful outbound internet gateway for **IPv6 traffic ONLY**. | Prevents external inbound IPv6 connections while allowing private subnets to reach the internet over IPv6. |
| **Route Table (RT)** | Set of rules (routes) determining network traffic direction. | Every subnet must associate with a Route Table (defaults to Main RT). Most specific route (longest prefix match) wins. |

| ⚠️ EXAM TRAP: High Availability NAT Gateway Architecture: A NAT Gateway resides in a single AZ. If that AZ fails, private subnets in other AZs using that NAT Gateway lose internet access. To achieve true High Availability (HA), deploy **one NAT Gateway per Availability Zone** and route each private subnet to its local AZ's NAT Gateway. |
| :---- |

---

## **Security Group vs. Network ACL (NACL) — Deep Dive**

| Feature | Security Group (SG) | Network Access Control List (NACL) |
| ----- | ----- | ----- |
| **Operating Level** | Instance / Elastic Network Interface (ENI) | Subnet boundary |
| **Statefulness** | **STATEFUL**: Return traffic is automatically allowed regardless of inbound/outbound rules. | **STATELESS**: Inbound and outbound traffic must be explicitly allowed in both directions. |
| **Supported Rules** | **Allow rules ONLY** (default: deny all inbound, allow all outbound). | **Allow AND Deny rules** (evaluates rules in numerical order; lowest number takes precedence). |
| **Rule Evaluation** | All rules evaluated simultaneously. | Evaluated sequentially by rule number (**first match wins**). |
| **Reference Targets** | IP CIDRs or **other Security Group IDs** (e.g., `sg-web` allowed into `sg-db`). | IP CIDR blocks ONLY. |
| **Ephemeral Ports** | Tracked automatically due to statefulness. | **MUST explicitly allow outbound response traffic to ephemeral ports** (`32768–65535`). |

### **Ephemeral Port Range Cheat Sheet**
When a client initiates a request to a service, the client's OS assigns a temporary **ephemeral port** to receive the response:
* **Linux Kernel / AWS NAT Gateway / Lambda / ELB:** `1024–65535` or `32768–61000` (Exam standard outbound NACL rule for Linux/Web servers: **`32768–65535`**).
* **Windows OS (Server 2008 and later):** `49152–65535`.

| 🧠 SAA EXAM RULE: If an EC2 instance in a custom NACL can receive SSH (Port 22) or HTTPS (Port 443) traffic, but the client times out waiting for a response, the NACL is **missing an Outbound Rule allowing TCP Return Traffic on Ephemeral Ports (32768–65535)**! |
| :---- |

---

## **VPC Endpoints — Gateway vs. Interface (PrivateLink)**

VPC Endpoints allow instances in private subnets to communicate with AWS services privately without traversing the public internet, requiring NAT Gateways, or attaching Internet Gateways.

| Feature | Gateway VPC Endpoint | Interface VPC Endpoint (AWS PrivateLink) |
| ----- | ----- | ----- |
| **Supported Services** | **Amazon S3** and **Amazon DynamoDB** ONLY. | Almost all other AWS services (KMS, SQS, SNS, EC2, S3 API, etc.) and custom Endpoint Services. |
| **Cost Structure** | **100% FREE** (No hourly charges, no data transfer fees). | Charged **Hourly per ENI** + **Data Processing fees per GB**. |
| **Implementation** | Modifies the VPC **Route Table** with a Prefix List target (`vpce-xxx`). | Deploys an **Elastic Network Interface (ENI)** with a private IP inside your subnet. |
| **On-Premises Access** | Cannot be accessed from on-prem via Direct Connect or VPN. | Accessible from on-premises over AWS Direct Connect or VPN tunnels. |

| 💡 COST OPTIMIZATION TRAP: If a question asks for the **most cost-effective** way to access S3 or DynamoDB from a private subnet without internet exposure, the answer is ALWAYS **Gateway VPC Endpoint** (because Interface endpoints incur hourly usage fees). |
| :---- |

---

## **VPC IPv4 & IPv6 Addressing Dynamics**

* **IPv4 Address Exhaustion:** If a subnet runs out of available IPv4 addresses, you can add secondary IPv4 CIDR blocks to the VPC or create **IPv6-Only subnets**.
* **IPv6 Architectural Rules:**
  * Every IPv6 address is **publicly routable** (no private IPv6 concept).
  * You **cannot disable IPv4** on a VPC or remove primary IPv4 CIDR blocks (VPCs are dual-stack by design).
  * To allow outbound-only internet connectivity for IPv6 private instances, deploy an **Egress-Only Internet Gateway (EIGW)** (NAT Gateways do NOT support IPv6).

---

## **VPC Peering & Transitive Routing Limitations**

* **Direct Private Connection:** Connects two VPCs using AWS's private network backbone. Works across AWS Accounts and Regions.
* **Overlapping CIDRs:** VPC Peering **CANNOT** be established between VPCs with overlapping IP address ranges.
* **Transitive Routing is BANNED:** VPC Peering does **NOT** support edge-to-edge or transitive routing.


```

[ VPC-DEV ] ◄────── Peered ──────► [ VPC-UAT ] ◄────── Peered ──────► [ VPC-PROD ]
│                                                                      │
└───────────────────────── NO DIRECT ACCESS ───────────────────────────┘
(Traffic CANNOT route through UAT to reach PROD)

```

* **Edge-to-Edge Limitation:** A VPC cannot share its Direct Connect, VPN, or IGW connections with peered VPCs. If VPC-1 has a Direct Connect connection to on-premises, VPC-2 **cannot** use VPC-1's peering connection to reach on-premises.

---

## **Hybrid Connectivity: Direct Connect, VPN & Transit Gateway**

| Option | Setup Time | Bandwidth / Latency | Security / Encryption | Key SAA Exam Keyword |
| ----- | ----- | ----- | ----- | ----- |
| **AWS Site-to-Site VPN** | Minutes / Hours | Up to 1.25 Gbps per tunnel; Variable (Internet-based) | IPSec Encrypted by default | "Quick setup", "Cost-effective hybrid connection", "Encrypted over internet" |
| **AWS Direct Connect (DX)** | Weeks / Months | 1 Gbps / 10 Gbps / 100 Gbps; Consistent low latency | **Unencrypted by default** | "Consistent latency", "Bypass public internet", "High throughput" |
| **Direct Connect + VPN** | Weeks / Months | High speed + Consistent | **IPSec Encrypted over DX** | "Encrypted connection over dedicated private infrastructure", "Compliance requirement" |
| **AWS Transit Gateway (TGW)** | Hours | High throughput | Integrates with VPN / DX | "Central hub-and-spoke", "Transitive routing across hundreds of VPCs" |

### **Site-to-Site VPN Prerequisites**
To establish an AWS Site-to-Site VPN connection, you must configure:
1. **Virtual Private Gateway (VGW)** or **Transit Gateway (TGW)** attached on the AWS VPC side.
2. **Customer Gateway (CGW)** representing the on-premises physical appliance or software.
3. **Static, Internet-Routable IP Address** assigned to the Customer Gateway's external interface.

### **Direct Connect High Availability (HA) Patterns**
* **Maximum Resiliency (Critical Workloads):** Two Direct Connect connections terminated at **two separate Direct Connect locations** via separate Customer Gateways.
* **Cost-Effective Redundancy / Backup:** A **Site-to-Site VPN connection over the public internet** configured as a failover backup to a primary AWS Direct Connect connection.

---

## **VPC Flow Logs & Traffic Monitoring**

* **Capture Target:** VPC, Subnet, or ENI level.
* **Destination Options:** Amazon CloudWatch Logs, Amazon S3, or Kinesis Data Firehose.
* **What it Captures:** Source IP, Destination IP, Source Port, Destination Port, Protocol, Action (`ACCEPT` or `REJECT`), Packet count, Bytes.
* **Exclusions (Exam Traps):** Flow logs **do NOT capture** real-time packet payloads, traffic to/from `169.254.169.254` (Instance Metadata), Amazon DNS traffic, or DHCP traffic.

---

## **VPC Decision Matrix for SAA-C03**

| Scenario / Requirement | Recommended Solution |
| ----- | ----- |
| Connect EC2 in a private subnet to Amazon S3 in the most cost-effective manner without public internet exposure | **Gateway VPC Endpoint** |
| Web app in private subnet needs outbound access to download software updates; blocks inbound connections | **NAT Gateway (IPv4) or Egress-Only IGW (IPv6)** |
| Allow inbound traffic on port 443 to a Linux web server in a custom NACL; app currently times out | **Add Inbound 443 ALLOW rule AND Outbound 32768–65535 ALLOW rule in NACL** |
| Centrally manage and route traffic between 50+ VPCs and on-premises data centers without complex peering | **AWS Transit Gateway (TGW)** |
| Dedicated private line with consistent low latency required, but traffic MUST be encrypted for compliance | **AWS Direct Connect + AWS Site-to-Site VPN** |
| Private EC2 instances running out of available IPv4 address space for new deployments | **Add an IPv6-Only Subnet to the VPC** |
| Provide fault tolerance / backup for a single AWS Direct Connect connection | **AWS Site-to-Site VPN over the public internet** |

---



[< ElastiCache - Caching Strategies](11-elasticache.md) | [Back to README](../README.md) | [Route 53 - DNS and Routing Policies >](13-route53.md)
