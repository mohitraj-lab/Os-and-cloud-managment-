# Cloud, Security & IoT Deployment Blueprint

## Task 9: Distributed Architecture & Communication Plan
* **Architecture**: **Client-Server Architecture**. The overall system is divided into two roles. The Smart City Operations Dashboard acts as the central server (the main hub), while the 3 Zone Controllers (Zone-A, Zone-B, Zone-C) act as clients that connect to it, send data, and receive commands.
* *Justification*: Ensures centralised control (**single point of failure** mitigated via primary-backup dashboard replicas), high **transparency** as City operators get a single, clear, real-time view of all operations across every zone from one centralised dashboard and clear **scalability** as the city grows; you can easily add Zone-D or Zone-E as new clients without redesigning the entire central system.
* **Data Flow (a) Real-time Public-Safety Alert**:An urgent alert (like a fire sensor or traffic emergency) should not wait for a server to ask for updates (polling) or wait in a queue. ** Asynchronous **: The zone controller fires off the emergency message instantly without stopping its execution or waiting for a complex back-and-forth handshake using **MQTT**.MQTT uses a publish-subscribe model built specifically for lightweight, instant telemetry. It delivers emergency alerts with extremely low latency and minimal bandwidth overhead.
* **Data Flow (b) Full Day Sensor Log Upload**: **Synchronous** using **HTTPS**. Reliable transport protocol ensuring non-repudiation and verified completion during bulk archival data transfers.

## Task 10: VPC Network Boundary
* **VPC Setup**: A Virtual Private Cloud (VPC) is our own private, isolated network inside a cloud computing environment (like AWS, Google, Azure).1 VPC containing 3 isolated private subnets (`Zone-A Subnet`, `Zone-B Subnet`, `Zone-C Subnet`); subnets are logical divisions within a VPC. Placing Zone-A, Zone-B, and Zone-C into separate subnets groups their resources into distinct network boundaries.
* **Boundary Control**:A Network Access Control List (NACL) acts as a stateless firewall at the subnet level. It inspects every IP packet entering or leaving a subnet and evaluates it. Cross-zone traffic is blocked at the network layer using **Network Access Control Lists (NACLs)**. A specific rule, such as adding explicit DENY rules targeting the CIDR ranges (IP address blocks) of Zone-A and Zone-B, any attempt by a compromised node in Zone-A to scan or communicate directly with Zone-B is dropped immediately at the network perimeter; it denies inbound/outbound IP traffic between Zone-A and Zone-B subnet CIDR ranges.

## Task 11: Network Security Objectives
1. **Protect Sensitive Data**: Use AWS KMS / AES-256 encryption for database and log storage.Data stored in databases or log files is encrypted at rest using industry-standard AES-256 keys managed by AWS Key Management Service (KMS). If an attacker physically gains access to disk storage or steals backup snapshots, the stolen data remains unreadable garbage without the encryption keys.
2. **Authentication**: Enforce Multi-Factor Authentication (MFA) with OAuth 2.0 / OIDC tokens for user access. Authentication verifies who the user is; users must prove their identity using credentials and a second factor (such as an authenticator app code), thereby generating temporary OpenID Connect (OIDC) identity tokens.
3. **Authorization**: Authorisation decides what users are allowed to do. Role-Based Access Control assigns users strict roles (e.g., Zone Operator, Auditor), granting only the minimum permissions necessary for their job.
4. **Prevent Cyber Attacks**: Deploy a Web Application Firewall (WAF) to block DDoS and injection attacks. A WAF sits in front of your web application to inspect inbound HTTP/HTTPS traffic for malicious patterns like SQL injection, Cross-Site Scripting (XSS), or rapid-fire request spikes.
5. **Secure Communication**: Enforce TLS 1.3 encryption on all transit data channels. Transport Layer Security (TLS 1.3) encrypts all data moving across network channels between IoT devices, zone controllers, and the central dashboard.
6. **Ensure Availability**: Configure Auto Scaling groups and Multi-AZ Load Balancers. Traffic is distributed across multiple physical Availability Zones (AZs / data centres) via Elastic Load Balancers, while Auto Scaling automatically provisions additional server instances during traffic spikes.

## Task 12: IAM Role Table & Data-Protection Map

### IAM Roles
| Role Name | Permissions |
| :--- | :--- |
| **Zone Operator** | Read/Write access restricted to local zone resources and local scheduling commands.Restricted to their assigned geographic zone (e.g., Zone-A). They can issue scheduling commands or view telemetry locally, but cannot alter central dashboard configurations or manage other zones. This keeps local issues isolated. |
| **City Dashboard Admin** | Full Read/Write administrative access across all zone metrics and global settings.Possesses high-level administrative permissions across the whole platform to manage global rules, monitor all zone controllers, and configure system-wide parameters. |
| **Auditor** | Read-only access to audit trails, system logs, and archival reports.Needs visibility for compliance and post-incident analysis without risk of modifying anything. Read-only permissions ensure they can inspect logs and records without accidentally breaking or altering system operations. |

### Data Protection
* **At Rest**: AES-256 encryption applied to the `JOBS` list database stored on zone controllers.
* **In Transit**: TLS 1.3 protocol encryption applied to public-safety alert messages sent to the dashboard.
* **In Use**: Process memory isolation and enclave memory protection during the scheduler and Banker's Algorithm engine from Part 1 execution.

## Task 13: IoT Connectivity & Architecture Layers

### Sensor Mapping
* **Traffic Camera Trigger**: **5G** (High-definition video streaming requires massive data throughput (bandwidth) and ultra-low latency. 5G handles these heavy data loads easily).
* **Environmental Sensor**: **LoRaWAN** (Air quality sensors send tiny packets of data periodically (e.g., once every 10 minutes) and often run on batteries in remote areas. Long Range Wide Area Network (LoRaWAN) provides multi-kilometre range with minimal power consumption, allowing sensors to run for years on a single battery).
* **Wearable Safety Device**: **NB-IoT** (Public safety workers move throughout the city and need guaranteed connectivity even inside buildings or basements. Narrowband IoT (NB-IoT) leverages existing cellular network towers to provide deep signal penetration and reliable mobile coverage).

### IoT 6-Layer Architecture
1. **Physical Environment**: Urban roads, public safety devices, and environmental monitoring zones.
2. **Perception / Device Layer**: Traffic cameras, air quality sensors, and SOS safety wearables.
3. **Gateway Layer**: Edge zone computing units processing raw sensor input.
4. **Network Communication Layer**: 5G, LoRaWAN, and Cellular NB-IoT infrastructure.
5. **Cloud Platform Layer**: **The scheduler and Banker's-Algorithm engine from Part 1** executing job execution and safety checks.
6. **Application Layer**: Smart City Operations Dashboard displaying real-time metrics and alerts.

## Task 14: Threats & Mitigations
1. **Threat**:An attacker intercepts the network traffic travelling between zone controllers and the central dashboard. They can passively read sensitive sensor data or actively alter messages in transit (e.g., suppressing a public safety alert),Man-in-the-Middle (MitM) eavesdropping on telemetry.
   * *Mitigation*: Enforce TLS 1.3 with strict certificate pinning.
2. **Threat**: An attacker uses a botnet (a massive network of compromised machines) to flood the Smart City Operations Dashboard with millions of fake requests per second. This exhausts server resources, making the dashboard unavailable to city operators during real emergencies, Distributed Denial of Service (DDoS) on dashboard endpoint.
   * *Mitigation*: Deploy Cloudflare / AWS Shield rate-limiting and DDoS mitigation.
3. **Threat**:A bad actor or malware gains access to a low-level account on a zone controller (such as a read-only viewer) and exploits software vulnerabilities to gain administrative/root access. Once escalated, they could tamper with job priorities, override Banker's Algorithm safety checks, or hijack the controller, Unauthorized privileges led to the escalation on Zone Controllers.
   * *Mitigation*: Enforce strict IAM policies with least-privilege RBAC and hardware MFA.
