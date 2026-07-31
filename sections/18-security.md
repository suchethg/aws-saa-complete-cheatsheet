# **Security Services — Know Your Defense Layers**

| Service | What it Does | Layer | Exam Scenario |
| ----- | ----- | ----- | ----- |
| **AWS WAF** | Web Application Firewall; rules against SQL injection, XSS, bad bots, geo-blocks | Layer 7; CloudFront, ALB, API GW, AppSync | "block SQL injection", "protect against OWASP Top 10", "geo-block countries" |
| **AWS Shield Standard** | Automatic DDoS protection (Layers 3/4) for all AWS customers at no cost | Layer 3/4; all resources | Always on; no configuration needed |
| **AWS Shield Advanced** | $3,000/month; Enhanced DDoS protection + 24/7 DDoS Response Team + cost protection | Layer 3/4/7; CloudFront, Route53, ALB, NLB, EIP | "sophisticated DDoS attacks", "financial protection", "DRT support" |
| **AWS Firewall Manager** | Centrally deploy and manage WAF rules, Shield Advanced, SGs, Network Firewall across org | Multi-account, Organizations-level | "apply WAF rules across all accounts", "organization-wide security policies" |
| **Amazon GuardDuty** | Threat detection using ML on CloudTrail, VPC Flow Logs, DNS Logs, S3 logs, EKS logs | Account-level threat intelligence | "detect cryptomining", "suspicious API calls", "unauthorized access" |
| **Amazon Inspector** | Automated vulnerability assessment for EC2, ECR images, Lambda functions | Workload-level CVEs | "scan EC2 for vulnerabilities", "container image scanning", "CVE detection" |
| **Amazon Macie** | ML-based PII/sensitive data discovery and classification in S3 | Data classification | "find PII in S3", "GDPR compliance", "sensitive data discovery" |
| **Amazon Detective** | Automatically analyzes log data using ML and graph theory to visualize root causes | Incident Response & Investigation | "investigate root cause of GuardDuty findings", "visualize attack paths and security incidents" |
| **AWS Security Hub** | Aggregates findings from GuardDuty, Inspector, Macie, Config into unified dashboard | Cross-service aggregation | "single pane of glass for security", "normalize findings" |
| **AWS Audit Manager** | Continuously audits AWS usage to automate evidence collection for compliance frameworks | Governance & Compliance | "automate evidence collection for audits", "assess compliance against PCI-DSS/HIPAA" |
| **AWS Network Firewall** | Managed stateful firewall (Suricata rules) for VPC — deep packet inspection | VPC level | "deep packet inspection", "IDS/IPS in VPC", "custom domain filtering" |
| **Amazon S3 Storage Lens** | Organization-wide visibility, analytics, and recommendations for S3 usage and security posture | S3 Storage Analytics | "organization-wide S3 analytics", "identify unencrypted buckets with least operational overhead" |
| **Secrets Manager** | Store, rotate, retrieve secrets (DB passwords, API keys); auto-rotation with Lambda | Application secrets | "rotate DB credentials automatically", "never hardcode passwords" |
| **Systems Manager Parameter Store** | Store config data and secrets (standard: free, advanced: $0.05/param/month) | Config & secrets | "store configuration", "free secrets storage", "hierarchy config" |
| **KMS (Key Management Service)** | Create and manage encryption keys; integrate with 100+ AWS services | Encryption key management | "customer managed keys", "key rotation", "envelope encryption" |
| **CloudHSM** | Dedicated Hardware Security Module; you manage keys entirely; FIPS 140-2 Level 3 | Hardware key protection | "FIPS 140-2 Level 3", "full key control", "regulatory compliance (PKCS11, JCE)" |

| ⚠️ EXAM TRAP: GuardDuty vs. Detective: **GuardDuty** *detects* the threat (sends an alert). **Detective** helps you *investigate* the root cause of that alert using visual graph relationships. GuardDuty triggers the alarm; Detective answers *how* it happened. |
| :---- |

| ⚠️ EXAM TRAP: Secrets Manager vs Parameter Store: Secrets Manager costs ~$0.40/secret/month but auto-rotates credentials and is designed for secrets. Parameter Store is free for standard parameters but no auto-rotation (need to trigger Lambda yourself). For exam: 'auto-rotate database credentials' → Secrets Manager. |
| :---- |

| 🧠 MNEMONIC: GuardDuty = Guard dog (threat DETECTION). Inspector = Home inspector (vulnerability SCANNING). Macie = Mac & Privacy (PII/data CLASSIFICATION). Detective = Sherlock Holmes (ROOT CAUSE INVESTIGATION). Audit Manager = Compliance Auditor (EVIDENCE COLLECTION). |
| :---- |

| NEXT: SECTION 9 — ARCHITECTURAL PATTERNS & DESIGN DECISIONS |
| :---: |

---

[< Monitoring Stack - CloudWatch, CloudTrail, Config](17-monitoring.md) | [Back to README](../README.md) | [Core Architectural Patterns for SAA >](19-architectural-patterns.md)
