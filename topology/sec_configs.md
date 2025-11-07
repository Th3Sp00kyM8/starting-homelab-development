# Security Configurations

## Introduction

This document outlines the security configuration and hardening approach for the network environment.  
Rather than completing each device in isolation or following strict linear "phases," this process uses **segmented configuration cycles**.  

Each segment focuses on groups of related systems, configured to approximately **70% functionality** before being integrated and tested together.  
Once baseline connectivity and interoperability are verified, the segment enters a **security hardening pass** — tightening controls, reducing exposure, and aligning configurations across devices.

This segmented model provides several advantages:
- **Stability:** Integration happens gradually, ensuring configurations don’t conflict.
- **Clarity:** Each segment forms a self-contained checkpoint with clear verification steps.
- **Realism:** Mirrors how real-world infrastructures evolve — incrementally, not sequentially.
- **Recovery:** Each segment can be snapshot or backed up independently for easy rollback.

By treating security as a layered, iterative process rather than a finish line, this approach emphasizes controlled progression, consistency, and long-term maintainability.

## Segment 1: Core Connectivity & Baseline Setup

**Objective:**  
Establish stable communication and foundational control between the core network nodes before introducing security layers.

### Scope
This segment focuses on the three primary devices that define network flow:
- **GL.iNet router** (internet bridge / upstream source)
- **TP-Link ER605** (edge router)
- **Protectli FW4B running OPNsense** (firewall / gateway)

### Key Actions
- [x] Update firmware on all three devices to the latest stable releases  
- [x] Change default credentials and disable remote cloud management where applicable  
- [x] Assign static IPs for WAN / LAN interfaces to ensure predictable routing  
- [x] Verify end-to-end connectivity:  
  - Internet → GL.iNet → ER605 → Protectli  
  - LAN clients → Protectli → Internet  
- [x] Confirm DNS resolution via OPNsense or Pi-hole (if present)  
- [x] Document each device’s management IP and login method  
- [ ] Begin defining subnet ranges for WAN, DMZ, and LAN zones  
- [ ] Plan baseline configuration backups for each device  

### Verification
- Successful ping and traceroute from LAN to the internet  
- Dashboard access to OPNsense confirmed through LAN interface  
- Consistent IP addressing and no double-NAT conflicts detected  

**Deliverable:**  
All core network devices online, reachable, and communicating reliably.  
This segment establishes the functional backbone for later security hardening.


## Segment 2: Core Security Hardening

**Objective:**  
Reduce attack surface, standardize administrative access, and ensure baseline protection across core devices before expanding the network.

### Scope
Applies to:
- **Protectli FW4B (OPNsense)**
- **TP-Link ER605**
- **GL.iNet Router**

Each device is secured locally first, then verified together to maintain stable communication and management access.

### Key Actions
- [x] Disable default and root accounts where possible  
- [x] Create named admin users with strong, unique credentials  
- [x] Restrict GUI and SSH access to LAN interface only  
- [x] Change default SSH ports (e.g., `2222`) and disable password authentication after key setup  
- [x] Enforce HTTPS on all management interfaces (self-signed certificate acceptable)  
- [x] Turn off unused services (e.g., UPnP, remote management, cloud access)  
- [x] Apply firmware and package updates across all devices  
- [ ] Configure centralized NTP synchronization for time consistency across logs  
- [ ] Validate firewall rules:  
  - WAN: block inbound by default  
  - LAN: allow outbound, restrict inter-device admin access  

### Verification
- Confirm GUI and SSH access work only from LAN  
- Validate HTTPS enforcement and SSH key authentication  
- Confirm successful time sync and consistent log timestamps  
- Ensure connectivity remains stable post-hardening  

**Deliverable:**  
All core devices hardened and synchronized, with secure and consistent management access.  
This segment establishes the trust boundary for future defense integration.


## Segment 3: Defense Node Integration

**Objective:**  
Introduce the LAN and WAN defense servers into the environment to enable traffic monitoring, packet analysis, and future intrusion detection without disrupting network stability.

### Scope
Applies to:
- **GMKtec G10 (LAN Defense Server)**
- **GMKtec M6 Ultra (WAN Defense Server)**

These devices serve as the foundation for internal and perimeter defense capabilities.  
Each will be configured to baseline readiness (operating system, secure access, and basic connectivity) before adding advanced security software.

### Key Actions
- [x] Install base OS (Ubuntu Server 22.04 LTS recommended) on both defense nodes  
- [x] Create non-root admin accounts and enable SSH with key-based authentication  
- [x] Configure static IPs for each server and verify reachability from Protectli  
- [x] Harden access with `ufw` firewall rules (deny all inbound except SSH and necessary services)  
- [x] Validate network reachability:  
  - M6 ↔ Protectli WAN  
  - G10 ↔ Protectli LAN  
  - G10 ↔ Internet (for updates)  
- [ ] Install essential tools (`htop`, `net-tools`, `tcpdump`, `fail2ban`)  
- [ ] Begin basic logging setup for internal testing  
- [ ] Document initial IP assignments and connection paths  

### Verification
- SSH access to both defense servers confirmed  
- Protectli can communicate with both M6 (WAN side) and G10 (LAN side)  
- Local firewall rules active and blocking unnecessary inbound connections  
- Internet access validated for updates and package installations  

**Deliverable:**  
Defense servers online, reachable, and secured at the system level.  
This segment establishes a functional base for future IDS, honeypot, and SIEM integrations.


## Segment 4: Visibility & Logging Setup

**Objective:**  
Establish centralized monitoring and event logging to gain real-time visibility into network behavior and security posture before enabling active defense systems.

### Scope
Applies primarily to:
- **Protectli FW4B (OPNsense)**
- **GMKtec G10 (LAN Defense Server)**
- **GMKtec M6 Ultra (WAN Defense Server)**

This segment ensures that all key devices forward logs to a central point for correlation, analysis, and troubleshooting.

### Key Actions
- [x] Enable detailed logging on OPNsense (firewall, DHCP, DNS, Suricata)  
- [x] Configure Syslog forwarding from OPNsense to the G10 defense server  
- [x] Set up Wazuh, Graylog, or ELK stack on G10 for log collection and visualization  
- [ ] Forward Suricata and system logs from both M6 and G10 to the central SIEM  
- [ ] Enable NetFlow or sFlow export on OPNsense for traffic analysis  
- [ ] Build baseline dashboards for system events, IDS alerts, and network utilization  
- [ ] Validate time synchronization across all devices (NTP consistency)  

### Verification
- Logs from OPNsense, M6, and G10 visible within central SIEM  
- Timestamp alignment across systems confirmed  
- Real-time firewall and IDS events populating dashboards  
- No significant latency or dropped log entries observed  

**Deliverable:**  
A centralized, synchronized logging and monitoring environment.  
This segment provides full situational awareness of network activity and establishes a foundation for correlation, alerting, and anomaly detection.


## Segment 5: Active Defense & IDS/IPS

**Objective:**  
Transition from passive monitoring to active network defense by deploying intrusion detection and prevention systems (IDS/IPS) and deception-based telemetry.

### Scope
Applies to:
- **Protectli FW4B (OPNsense)**
- **GMKtec M6 Ultra (WAN Defense Server)**
- **GMKtec G10 (LAN Defense Server)**

This segment activates real-time threat detection across both perimeter and internal layers, enabling automated responses and deeper analysis.

### Key Actions
- [x] Enable and configure Suricata on OPNsense in IDS mode  
- [x] Subscribe to the Emerging Threats (ET Open) rule set  
- [x] Verify alert generation for known test signatures  
- [ ] Transition Suricata to IPS (inline) mode for active blocking  
- [ ] Deploy honeypots on M6 Ultra (e.g., Cowrie for SSH, Dionaea for malware collection)  
- [ ] Forward honeypot logs to G10 SIEM for correlation  
- [ ] Deploy Zeek on M6 Ultra or G10 for advanced traffic and behavioral analysis  
- [ ] Tune IDS/IPS rules to minimize false positives while maintaining coverage  

### Verification
- Suricata generates alerts for simulated malicious traffic  
- Honeypot events visible within the SIEM dashboard  
- Network throughput stable after enabling inline IPS  
- No unintentional blocking of legitimate internal traffic  

**Deliverable:**  
Network equipped with live intrusion detection, inline prevention, and deception-based telemetry.  
This segment transforms the environment into an intelligent, layered defense system capable of detecting and responding to real-world threats.


## Segment 6: Ongoing Security Maintenance

**Objective:**  
Establish a continuous security lifecycle for updates, validation, and improvement to maintain a stable, resilient, and adaptive environment.

### Scope
Applies to:
- **All devices and servers** across WAN, DMZ, and LAN zones

This segment focuses on maintaining system integrity, performance, and readiness through consistent upkeep and periodic validation.

### Key Actions
- [x] Schedule regular firmware and OS updates across all devices  
- [x] Maintain configuration backups for each system (local and off-device)  
- [ ] Conduct quarterly rule audits for OPNsense and IDS/IPS tuning  
- [ ] Review system and SIEM logs weekly for anomalies or failed authentications  
- [ ] Rotate administrative credentials and refresh SSH keys periodically  
- [ ] Perform penetration testing or vulnerability scans against the DMZ and LAN zones  
- [ ] Expand segmentation (e.g., VLANs for IoT, guest, or lab environments)  
- [ ] Evaluate zero-trust policy enforcement and automation opportunities  

### Verification
- Configuration files verified and accessible for rapid recovery  
- Update cadence documented and aligned with vendor advisories  
- IDS/IPS and SIEM performance consistent over time  
- No stale or orphaned rules, users, or services active  

**Deliverable:**  
A sustainable, continuously improving security posture with operational visibility and controlled evolution.  
This segment ensures long-term network health, adaptability, and readiness for cloud-scale extension.


## Conclusion & Next Steps

This segmented configuration and hardening plan transforms the environment from a simple connected network into a layered, observable, and actively defended system.  
By progressing in segments rather than linear phases, each milestone builds on a stable foundation — maintaining visibility, functionality, and security throughout the process.

### Key Takeaways
- **Iterative security** allows verification and rollback at every stage.  
- **Defense-in-depth** is achieved through layered visibility, active prevention, and continuous tuning.  
- **Documentation discipline** ensures the environment remains reproducible and auditable as it grows.  

### Next Steps
- Expand segmentation through VLANs and Zero Trust principles.  
- Integrate automated backups and configuration management (e.g., Ansible, Terraform).  
- Begin mapping this physical network to a **cloud-parallel architecture** to simulate hybrid or multi-cloud security models.  
- Continue refining IDS rules, alert thresholds, and SIEM correlation logic as more data is collected.  

The result is a resilient, adaptable home network — one that not only mirrors enterprise-grade security principles but also serves as a live platform for cloud security engineering practice.
