## Overview

The `/security` directory documents the network’s defensive architecture and the underlying systems. Each subsystem is designed to model real-world enterprise security functions on a smaller scale; emphasizing observability, segmentation, and coordination across all layers.

This documentation outlines the purpose and interconnection of all defense-based nodes. These systems demonstrate how layered security can be achieved through open-source tooling, structured configuration, and continuous monitoring.

The goal of this section is to provide a clear reference for:
- The design intent and purpose behind each security node.
- The toolsets deployed within each layer.
- How defensive components interoperate to maintain network integrity.

This folder acts as the foundation for all future documentation related to:
- Intrusion detection and response architecture.
- Log and metric aggregation (GLP stack: Grafana, Loki, Promtail).
- Integration of monitoring, alerting, and automation functions across nodes.

---

## Design Philosophy

The security design emphasizes visibility, segmentation, and orchestration as the foundation of defense. Rather than relying on a single point of protection, the system distributes responsibility across multiple nodes.

Every component is treated as both sensor and actor: able to observe, record, and relay information to the central orchestration layer. This approach allows security to scale organically; adding new detection or analytics tools without disrupting the overall structure.

The architecture follows several key principles:

1. **Defense in Depth**  
   Multiple independent controls protect against failure in any single layer. The firewall, DMZ node, and internal orchestration node operate with minimal overlap, ensuring redundancy in monitoring and containment.

2. **Observability over Obscurity**  
   Security depends on what can be seen and measured, which is ideal for a homelab. Logging, telemetry, and analysis take priority over concealment. The Grafana–Loki–Promtail stack provides a continuous record of system behavior and network events.

3. **Segmentation and Least Privilege**  
   Each node functions within a defined network zone (Perimeter, DMZ, Internal). Access, communication, and data flow are restricted to essential paths, limiting exposure and lateral movement.

4. **Orchestration before Automation**  
   Before introducing automated response systems, the environment must understand, contextualize, and verify its data. The Defense Orchestration Node serves as the control layer that unifies insight across all defensive components.

5. **Scalability through Modularity**  
   Nodes and services can be added or replaced without rearchitecting the network. Each defensive layer communicates through open protocols and modular services, mirroring real-world SOC and NOC design patterns.

This philosophy ensures the homelab remains adaptable.

---

## Security Node Framework

The network’s defensive structure is organized into functional layers, each represented by a purpose-driven node.  
Every node has a defined role: routing, exposure, enforcement, or orchestration; ensuring that security and observability scale in tandem with complexity.

### **1. Edge Router - Perimeter Routing**

**Name:** Charon 
**Device:** TP-Link ER605  
**Role:** Edge Router 

Charon acts as the network’s external gateway, responsible for managing incoming and outgoing traffic from the ISP while maintaining control over multiple WAN connections.  
It establishes the first line of boundary control before traffic reaches the internal defense chain.

**Core Functions:**
- Primary routing, NAT, and load balancing  
- Basic stateful firewall and access control (ACL)  
- VPN endpoint management for remote access  
- DoS protection and basic packet filtering  
- Log forwarding to the Defense Orchestration Node  

---

### **2. Bastion Host – Exposure and Detection Layer**

**Name:** Heimdall 
**Device:** GMKtec mini-PC (dedicated)  
**Role:** Bastion Host 

Heimdall functions as a bastion, an intentionally exposed intermediary between the untrusted network and the protected core.  
It hosts inspection, detection, and deception tools, analyzing all inbound traffic while remaining fully isolated from internal systems.

**Core Functions:**
- Intrusion detection and traffic analysis (Zeek, Suricata, or CrowdSec)  
- Optional honeypot or sandbox services for behavioral observation  
- Reverse proxy and limited application hosting if required  
- Log and event forwarding to the Defense Orchestration Node  
- Strict VLAN and routing isolation from internal segments  

---

### **3. Gateway Firewall – Enforcement and Segmentation Layer**

**Name:** Amaterasu 
**Device:** Protectli Vault (OPNsense)  
**Role:** Gateway Firewall 

Amaterasu serves as the authoritative control layer within the security chain.  
It defines internal and external trust boundaries, enforces VLAN segmentation, and coordinates traffic inspection and logging between the DMZ and internal network.

**Core Functions:**
- Stateful firewall and NAT control  
- VLAN segmentation and subnet isolation  
- Traffic shaping and bandwidth control  
- IDS/IPS integration and rule management  
- Syslog forwarding to the Defense Orchestration Node  

---

### **4. Defense Orchestration Node – Command and Control Layer**

**Name:** Hestia
**Device:** GMKtec G10  
**Role:** SOAR Server

Hestia consolidates logs, metrics, and telemetry from every layer; converting raw data into actionable intelligence. It is the command and control layer.
It forms the analytical foundation of the network’s defensive posture, enabling visualization, correlation, and alerting across systems.

**Core Functions:**
- Centralized observability stack (Grafana, Loki, Promtail)  
- System monitoring and metrics collection  
- Alert generation and trend analysis  
- Historical log retention and event correlation  
- Future integration with Wazuh, SOAR scripts, and cross-node intelligence  

---

## Node Overview Table

The table below summarizes each active defense-based node within the security architecture.  
Each system fulfills a distinct role within the layered defense model, contributing to visibility, segmentation, and orchestration across the network.

| **Node** | **Device** | **Network Layer / Placement** | **Primary Role** | **Key Tools / Services** | **Core Responsibilities** |
|-----------|-------------|-------------------------------|------------------|---------------------------|----------------------------|
| **Charon – Edge Router** | TP-Link ER605 | Outer perimeter (entry point to network) | Routing, NAT, and basic packet filtering | Built-in OS | - Manage inbound/outbound traffic<br>- Apply ACLs and DoS protection<br>- Control WAN/VPN connectivity<br>- Forward filtered traffic to DMZ node |
| **Heimdall – Bastion Host** | GMKtec M6 Ultra | Between ER605 and Protectli (isolated VLAN) | Controlled exposure, traffic analysis, and deception | Zeek, Suricata, CrowdSec, optional Honeypots | - Inspect and log inbound packets<br>- Detect intrusion attempts<br>- Host deception or analysis services<br>- Forward logs to Defense Orchestration Node |
| **Amaterasu – Gatewall Firewall** | Protectli Vault (OPNsense) | Core perimeter (between DMZ and internal LAN) | Stateful firewall, segmentation, and policy enforcement | OPNsense, IDS/IPS, Syslog | - Enforce VLAN and subnet isolation<br>- Apply security policies<br>- Manage internal/external routing<br>- Forward logs to Defense Orchestration Node |
| **Hestia - SOAR Server** | GMKtec G10 | Internal network (behind Protectli) | Observability, analytics, and orchestration | Grafana, Loki, Promtail (GLP Stack) | - Aggregate logs and telemetry<br>- Visualize metrics and trends<br>- Generate alerts and correlation<br>- Serve as internal SOC and analysis hub |

---

## Core Components

The core components represent the foundational technologies that power the network’s defensive, monitoring, and orchestration layers.  
Each stack has a defined purpose — whether for visibility, enforcement, or detection — and contributes to the broader security posture of the homelab.

---

### **1. Observability Stack (GLP Stack)**
**Components:** Grafana, Loki, Promtail  
**Node:** Defense Orchestration Node (G10)  

The GLP Stack provides centralized log management, visualization, and analysis.  
It forms the internal observability layer responsible for recording, storing, and interpreting activity across all connected nodes.

| Tool | Role | Description |
|------|------|-------------|
| **Promtail** | Collector | Collects local and remote logs, tags them with metadata, and forwards them to Loki for indexing. |
| **Loki** | Log Storage | Efficient log database optimized for querying and indexing structured events without high storage overhead. |
| **Grafana** | Visualization Layer | Provides dashboards and alerting for metrics, system activity, and security events across the environment. |

**Functions:**
- Aggregates system and firewall logs from multiple nodes.  
- Enables real-time visualization of system metrics and logs.  
- Generates alerts based on defined thresholds or anomaly patterns.  
- Serves as the foundation for future SIEM and automation integrations.

---

### **2. Firewall and Enforcement Stack**
**Components:** OPNsense (Protectli Vault)  
**Node:** Core Firewall Node  

The firewall stack defines network boundaries and enforces segmentation.  
It operates as the authoritative routing and security layer between the DMZ and internal networks.

**Functions:**
- Stateful inspection and NAT management  
- VLAN-based segmentation and subnet control  
- Intrusion detection/prevention (IDS/IPS)  
- Rule-based traffic shaping and QoS  
- Log forwarding to the Defense Orchestration Node

---

### **3. Detection and Deception Stack**
**Components:** Zeek, Suricata, CrowdSec, optional Honeypot services  
**Node:** DMZ Bastion Node  

The detection stack captures and analyzes network traffic before it enters protected zones.  
It provides early-warning telemetry and behavioral data used for correlation and forensic review.

**Functions:**
- Deep packet inspection and protocol analysis  
- Intrusion detection and threat behavior mapping  
- Community-based reputation scoring (via CrowdSec)  
- Sandbox and honeypot experimentation for attacker profiling  
- Event forwarding to the Defense Orchestration Node

---

### **4. Routing and Edge Control Stack**
**Components:** TP-Link ER605 (built-in OS)  
**Node:** Border Router / Edge Filtering Node  

The routing layer manages WAN ingress and egress while applying coarse-grained filtering and basic DoS protection.  
It ensures upstream stability and reduces noise before packets reach the DMZ.

**Functions:**
- WAN load balancing and failover management  
- Basic firewall and access control (ACLs)  
- VPN termination for remote access  
- Log and event forwarding to the orchestration node

---

### **5. Future Integrations (Planned)**
The architecture is designed to scale toward enterprise-grade observability and security automation.

**Planned Additions:**
- **Wazuh:** Host-based intrusion detection and SIEM integration  
- **Zeek/Suricata correlation:** Cross-node detection patterns  
- **SOAR Layer:** Automated response scripting for event triggers  
- **Netdata/Prometheus:** Extended system metrics for performance visibility  
- **Centralized Syslog ingestion:** Unifying all node outputs into Loki or Wazuh  

---

## Network Integration

The network’s defensive architecture follows a **layered visibility model**, where each node performs a specialized role while maintaining controlled communication with its adjacent layers.  
Traffic, logs, and telemetry flow directionally — from the public edge inward — ensuring inspection, enforcement, and analysis occur in sequence.

---


Each node passes sanitized and contextualized data downstream:
- **ER605 → DMZ:** Routes only pre-approved inbound flows, dropping unsolicited packets at the edge.  
- **DMZ → Protectli:** Forwards analyzed and filtered traffic while logging potential threats.  
- **Protectli → G10:** Sends syslogs, alerts, and metrics for long-term visibility and analysis.  

---

### **2. VLAN & Segmentation Structure**
Segmentation ensures that each defense layer operates within its own trust boundary.  
Routing between layers is explicitly controlled through the Protectli firewall.

| **Zone** | **Primary Device** | **CIDR / Example Subnet** | **Purpose** |
|-----------|--------------------|----------------------------|--------------|
| **WAN Zone** | ER605 | Public IP from ISP | External connectivity, NAT, VPN termination |
| **DMZ Zone** | GMKtec Bastion | 192.168.30.0/24 | Isolated inspection and deception environment |
| **Core Zone** | Protectli | 192.168.10.0/24 | Central routing, VLAN management, and enforcement |
| **Internal Zone** | G10 / LAN Hosts | 192.168.20.0/24 | Internal analytics and user endpoints |
| **Management Zone** *(optional)* | Admin Laptop / Console | 192.168.40.0/24 | Administrative access and orchestration control |

**Segmentation Rules:**
- DMZ traffic cannot directly reach the internal or management zones.  
- Only the Protectli can broker communication between VLANs.  
- G10 receives telemetry via syslog or API — no open inbound services from less-trusted zones.

---

### **3. Data & Log Flow**
Telemetry and logs follow a separate, secured path distinct from user or application traffic.

| **Source** | **Transport Path** | **Destination** | **Protocol** |
|-------------|-------------------|-----------------|---------------|
| ER605 | Syslog / HTTP | G10 | UDP 514 / API |
| DMZ Bastion | Promtail / Syslog | G10 (Loki) | HTTP / UDP |
| Protectli | Syslog | G10 (Loki) | UDP 514 |
| G10 | Localhost | Grafana Dashboard | HTTP 3000 |

This approach centralizes visibility without increasing attack surface.  
Only outbound log and metric forwarding are permitted from defensive nodes.

---

### **4. Physical & Logical Isolation**
Each device connects through managed switch ports or VLAN-tagged interfaces to maintain traffic separation.  
No node holds dual-purpose roles — routing, inspection, and analysis remain discrete functions.

- **ER605**: Edge device connected directly to ISP modem.  
- **DMZ Node**: Connected to dedicated switch port / VLAN bridged to Protectli WAN.  
- **Protectli**: Manages VLANs for DMZ, internal, and management subnets.  
- **G10**: Connected to LAN VLAN for analytics and observability.  

---

### **5. Inter-Node Communication Summary**
| **From** | **To** | **Purpose** | **Direction** |
|-----------|---------|-------------|----------------|
| ER605 → DMZ | Forward filtered inbound traffic | Downstream |
| DMZ → Protectli | Forward analyzed traffic and logs | Downstream |
| Protectli → G10 | Send system logs and events | Downstream |
| G10 → Grafana (local) | Visualize and correlate data | Internal |
| G10 → Admin Device | Dashboard access and management | Controlled Access |

---

## Future Expansion

The current network establishes a functional and observable defensive baseline.  
Future development will focus on extending the system toward greater autonomy, intelligence correlation, and enterprise-grade scalability.  
Each milestone builds on existing infrastructure, ensuring continuity without reconfiguration of the foundational layers.

---

### **1. Enhanced Detection and Analysis**
**Goal:** Broaden visibility from network behavior to host-level activity.  

**Planned Integrations:**
- **Wazuh:** Add host-based intrusion detection and security event correlation to complement network-level IDS.  
- **OSQuery:** Implement endpoint telemetry for detailed process and configuration auditing.  
- **Zeek–Suricata Correlation:** Fuse network and packet-level analytics for deeper threat behavior mapping.  
- **Log Enrichment Pipelines:** Introduce parsers to normalize syslog data before ingestion by Loki or Wazuh.  

These enhancements will transition the monitoring layer from passive logging to active event correlation and incident detection.

---

### **2. Automated Response and SOAR Integration**
**Goal:** Introduce conditional response mechanisms for faster containment and alert handling.  

**Planned Integrations:**
- **SOAR Scripts (Custom or Wazuh-integrated):** Automate responses to defined triggers — such as isolating hosts, blocking IPs, or alert escalation.  
- **API-based Integration with Protectli:** Allow automated firewall rule updates or blocklists based on log thresholds.  
- **Alert Routing:** Integrate alert channels via email, Discord, or webhooks for distributed notifications.  

This phase establishes a lightweight **Security Orchestration, Automation, and Response (SOAR)** layer — enabling the system to act, not just observe.

---

### **3. Advanced Observability and Performance Monitoring**
**Goal:** Expand visibility beyond security events into system and service health.  

**Planned Integrations:**
- **Netdata or Prometheus:** Real-time system performance dashboards for each node.  
- **Node Exporters:** Standardized metric collection across all servers for CPU, memory, and disk monitoring.  
- **Grafana Dashboard Expansion:** Merge performance, security, and network data into unified panels.  

This will bridge operational observability and security telemetry into one cohesive analytical environment.

---

### **4. Centralized Intelligence and Data Fusion**
**Goal:** Enable multi-source data correlation and external threat intelligence.  

**Planned Integrations:**
- **Threat Intelligence Feeds:** Incorporate IP and domain reputation data into alert rules.  
- **TheHive or MISP:** Integrate case management and threat-sharing for investigative workflows.  
- **Data Lake Expansion:** Establish long-term storage for logs, metrics, and packet captures to support historical analysis.  

These additions will evolve the homelab toward a **lightweight SOC model**, capable of correlating both internal telemetry and external threat data.

---

### **5. Network Hardening and Advanced Segmentation**
**Goal:** Increase resilience and reduce potential attack surfaces.  

**Planned Enhancements:**
- Fine-grained VLAN microsegmentation for IoT, lab, and administrative zones.  
- Layer 3 ACLs between VLANs to enforce strict communication policies.  
- Bastion host authentication hardening and MFA integration.  
- Encrypted log transport via TLS for Promtail and Syslog endpoints.  

This ensures that observability and automation remain secure even as complexity increases.

---

### **6. Long-Term Vision**
The architecture’s long-term trajectory is to transition from **manual observation to autonomous defense orchestration**.  
Each node will contribute data, context, and eventually decision logic — allowing the environment to function as a small-scale **cyber defense operations network**.

> The end state: A realistic environment built to test theories, prove concepts, and turn countless hours of frustration into moments of hard-earned, unhinged laughter.


