
# **Elastic Load Balancing — Three Types Compared**

Knowing WHICH load balancer to pick is a key SAA skill. The exam describes a requirement and expects you to match it.

| Feature | ALB (Application) | NLB (Network) | GWLB (Gateway) |
| ----- | ----- | ----- | ----- |
| **OSI Layer** | Layer 7 (HTTP/HTTPS/WebSocket/gRPC) | Layer 4 (TCP/UDP/TLS) | Layer 3+4 (IP packets) |
| **Protocol** | HTTP, HTTPS, WebSocket, gRPC | TCP, UDP, TLS | GENEVE (6081) for packet encapsulation |
| **Performance** | High throughput; milliseconds latency | Extreme performance; millions of req/sec; ultra-low latency | Transparent to traffic; used for appliances |
| **Health checks** | HTTP/HTTPS-based | TCP, HTTP, HTTPS | TCP, HTTP, HTTPS |
| **Target types** | EC2, IP, Lambda, containers | EC2, IP, ALB (as target) | EC2 instances running security appliances |
| **Sticky sessions** | Yes (application-based or duration-based cookies) | Yes (source IP) | No |
| **WebSocket** | Native support | Yes (Layer 4 pass-through) | N/A |
| **Fixed IP** | No (use NLB for fixed IP) | Yes (Elastic IP per AZ) | No |
| **Use case** | Web apps, microservices, content routing | TCP apps, gaming, IoT, NLB → ALB pattern | Deploy 3rd-party virtual appliances (Palo Alto, Fortinet) |
| **Exam keyword** | "HTTP routing", "host/path-based routing", "microservices", "Lambda" | "static IP", "TCP", "extreme performance", "UDP", "gaming" | "intrusion detection", "firewall appliance", "security inspection" |

| ⚠️ EXAM TRAP: NLB provides static IP addresses (one per AZ) — use this when clients need to whitelist fixed IPs. ALB has no fixed IPs. A common pattern: NLB in front of ALB to get fixed IPs while retaining Layer 7 routing. |
| :---- |

---

## **ALB Advanced Routing Features**

| Feature | Description | Example |
| ----- | ----- | ----- |
| **Host-based routing** | Route based on HTTP Host header | `api.example.com` → API Target Group; `app.example.com` → App Target Group |
| **Path-based routing** | Route based on URL path | `/images/*` → Image servers; `/api/*` → API servers |
| **HTTP header routing** | Route based on any HTTP header value | `User-Agent: iPhone` → mobile backend |
| **HTTP method routing** | Route based on GET/POST/PUT etc. | `POST /orders` → Order service TG |
| **Query string routing** | Route based on query string parameters | `?platform=mobile` → Mobile TG |
| **Fixed response** | Return HTTP 200/301/302 directly from ALB | Maintenance page, custom HTTP messages |
| **Redirect action** | ALB redirects HTTP → HTTPS | Enforce HTTPS without backend code |
| **Lambda target** | ALB invokes Lambda directly as target | Serverless web apps behind ALB |
| **gRPC support** | End-to-end HTTP/2 bi-directional gRPC streams | Microservices using gRPC target protocol |

---

## **HTTP Request Headers (`X-Forwarded-*`)**

When an ALB terminates HTTPS traffic, target servers only see the private IP address of the ALB node. To retrieve original client details, application logs inspect these headers:

| Header | Purpose | Key Exam Scenario |
| ----- | ----- | ----- |
| **`X-Forwarded-For`** | Stores the original **Client IP address** | "Need to log original caller IP for security/analytics" |
| **`X-Forwarded-Proto`** | Stores the original **Connection Protocol** (`http` or `https`) | "Detect if client connected over secure HTTPS" |
| **`X-Forwarded-Port`** | Stores the original **Destination Port** (e.g., `80` or `443`) | Application-level port redirection rules |

---

## **Cross-Zone Load Balancing**

| Feature | Cross-Zone ENABLED | Cross-Zone DISABLED |
| ----- | ----- | ----- |
| **Behavior** | Distributes traffic evenly across **ALL** targets in all enabled AZs. | Splits traffic 50/50 per AZ, then routes strictly within that AZ. |
| **Impact on Imbalance** | Prevents instance overload when AZs have unequal target counts. | AZs with fewer instances experience significantly higher load per instance. |
| **ALB Default** | **Enabled by default** (No extra data charges). | Can be toggled manually. |
| **NLB / GWLB Default** | **Disabled by default**. | Enabling incurs standard inter-AZ data transfer charges. |

| 🧠 EXAM RULE: If a question describes an uneven distribution of EC2 instances across AZs causing high CPU on instances in one zone, the answer is ALWAYS to enable **Cross-Zone Load Balancing**. |
| :---- |

---

## **Connection Draining / Deregistration Delay**

**What it is:** When an instance is deregistered or marked unhealthy, existing connections are kept alive for up to 3600 seconds (default 300 seconds) to complete in-flight requests before the instance is removed.

* **Increase delay:** For long-running HTTP requests, large file uploads, or batch jobs being interrupted during scale-in/deployments.
* **Decrease delay:** When deployments or auto-scaling scale-in events take too long because instances remain stuck in `draining` state.

---

## **Security & SSL/TLS Configuration**

### **Security Group Chaining Pattern**
Never allow public access directly to EC2 instances. Always enforce security group chaining:




[ Internet ] ──> ( Inbound Port 443: Open to 0.0.0.0/0 ) ──> [ ALB (sg-alb) ]
│
( Inbound Port 80: Source = sg-alb )
▼
[ EC2 Instances (sg-ec2) ]



* **ALB Security Group (`sg-alb`):** Allows inbound `0.0.0.0/0` on port 80/443.
* **EC2 Security Group (`sg-ec2`):** Allows inbound port 80 **ONLY from `sg-alb`** (referencing the ALB's Security Group ID as the source).

### **SNI (Server Name Indication) & Certificates**
* **SSL Offloading:** ALB/NLB terminates SSL/TLS, offloading crypto computation from backend instances. Integrates with **AWS Certificate Manager (ACM)**.
* **SNI:** Bind **multiple SSL certificates** to a single ALB listener. The ALB inspects the client's requested hostname (TLS extension) and serves the matching certificate automatically.

---

## **ELB Monitoring & Logging**

* **Access Logs:** Captures detailed request metadata (client IP, latencies, ciphers, paths). Disabled by default; **must be stored in an Amazon S3 bucket**.
* **CloudWatch Metrics:**
  * **`HTTPCode_Target_5XX`:** Application errors coming from the backend EC2 instances.
  * **`HTTPCode_ELB_5XX`:** Load balancer issue (e.g., `502 Bad Gateway` when targets fail health checks, `504 Gateway Timeout`).

---

| NEXT: SECTION 7 — MESSAGING: SQS, SNS, EVENTBRIDGE, KINESIS & MORE |
| :---: |

---

[< CloudFront - Content Delivery Network](14-cloudfront.md) | [Back to README](../README.md) | [Messaging and Event-Driven Architecture >](16-messaging.md)

