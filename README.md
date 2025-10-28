# Homelab Portfolio

This repository documents my personal cybersecurity homelab, built to simulate real-world network environments and strengthen my technical foundation while pursuing certifications (Network+, Security+, CySA+, CEH, AWS Cloud Practitioner).

The goal is to demonstrate hands-on experience across networking, system administration, and defensive/offensive security concepts supported by detailed documentation, diagrams, and configurations.

Beyond personal learning, this project also serves as a reference point for other aspiring cybersecurity specialists, providing transparent documentation and practical insights to help them explore their interests, build confidence, and establish a foundation in cybersecurity through real, replicable examples.

---

## Overview

**Homelab Stack:**
- Protectli Vault FW4B – running pfSense for routing, firewalling, and VLAN segmentation
- TP-Link ER605 – secondary router for VPN and load-balancing experiments
- TP-Link TL-SG108PE – 8-port PoE+ managed switch for VLAN testing
- TP-Link EAP610 – Wi-Fi 6 access point for wireless segmentation and SSID control
- GL.iNet Slate AX – portable VPN router for remote access testing
- Raspberry Pi 4 & 5 – Ubuntu Server 24 LTS running Pi-hole, Docker, and security tools

---

## Current Projects

Project | Description | Documentation 

**1. Network Segmentation & Firewall Setup** | Configured VLANs (Admin, Lab, IoT, Guest) using pfSense and tested inter-VLAN isolation. | [`/firewall/README.md`](firewall/README.md) 

**2. SIEM & IDS Stack** | Deployed Wazuh and Suricata on Raspberry Pi 5 for log analysis and intrusion detection. | [`/siem/README.md`](siem/README.md) 

**3. DNS Security & Pi-hole** | Implemented Pi-hole to block malicious domains and monitor DNS queries. | [`/pi-hole/README.md`](pi-hole/README.md) 

**4. Automation Scripts** | Bash and Python scripts for log rotation, backup, and scanning automation. | [`/automation/README.md`](automation/README.md) 

---

## Network Diagram

![Network Diagram]([https://github.com/Th3Sp00kyM8/starting-homelab-development/blob/main/topology/network-diagram.md])
*(Simplified visual of physical & logical topology.)*

---

## Learning Objectives

- Apply **defense-in-depth principles** in small-scale environments  
- Develop **practical network segmentation** and secure routing  
- Experiment with **IDS/IPS systems** (Suricata, Wazuh, Snort)  
- Strengthen **Linux administration** and **scripting** skills  
- Build familiarity with **SIEM and log correlation tools**  

---

## Tools & Technologies

**Network:** pfSense, OpenVPN, WireGuard, VLANs  
**Security:** Wazuh, Suricata, Snort, Pi-hole  
**Automation:** Bash, Python, Ansible  
**Visualization:** Grafana, Kibana  
**Hardware:** Protectli Vault FW4B, Raspberry Pi 4/5, TP-Link Omada series  

---

## Future Plans

- Deploy honeypots for threat emulation  
- Integrate cloud-based logging via AWS
- Publish guides and lessons learned per milestone

