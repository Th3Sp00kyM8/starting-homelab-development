# Homelab Portfolio

This repository documents my personal cybersecurity homelab, built to simulate real-world network environments and strengthen my technical foundation while pursuing certifications (Network+, Security+, CySA+, CEH, AWS Cloud Practitioner).

The goal is to demonstrate hands-on experience across networking, system administration, and defensive/offensive security concepts supported by detailed documentation, diagrams, and configurations.

Beyond personal learning, this project also serves as a reference point for other aspiring cybersecurity specialists, providing transparent documentation and practical insights to help them explore their interests, build confidence, and establish a foundation in cybersecurity through real, replicable examples.

---

## Overview
- Protectli Vault FW4B – running pfSense for routing, firewalling, VLAN segmentation, IDS/IPS testing, and network hardening
- TP-Link ER605 – secondary router used for VPN tunneling, failover, and load-balancing experiments
- TP-Link TL-SG108PE – 8-port PoE+ managed switch for VLAN testing, port isolation, and device-level traffic control
- TP-Link EAP610 – Wi-Fi 6 access point for wireless segmentation, multiple SSIDs, and captive-network experiments
- GL.iNet Slate AX – portable VPN router for remote-access testing, travel networking, and WireGuard/OpenVPN experiments
- Raspberry Pi 4 & 5 – Ubuntu Server 24 LTS hosting Pi-hole, Docker containers, local DNS filtering, and various security tools
- GMKtec G10 Mini-Server – primary analysis node running SIEM tools (Wazuh, Zeek), log aggregation (Grafana, Loki, Promtail), and containerized security labs
- M6 Ultra – dedicated DMZ/bastion host used for isolated testing, external-facing services, honeypot experiments, and traffic inspection scenarios

---

## Learning Objectives

- Apply defense-in-depth principles in small-scale environments  
- Develop practical network segmentation and secure routing  
- Experiment with IDS/IPS systems (Suricata, Wazuh, Snort)  
- Strengthen Linux administration and scripting skills  
- Build familiarity with SIEM and log correlation tools  

---

## Current Projects

Project | Description | Documentation 

**1. Network Segmentation & Firewall Setup** | Configured VLANs (Admin, Lab, IoT, Guest) using pfSense and tested inter-VLAN isolation. | [`/firewall/README.md`](firewall/README.md) 

**2. SIEM & IDS Stack** | Deployed Wazuh and Suricata on Raspberry Pi 5 for log analysis and intrusion detection. | [`/siem/README.md`](siem/README.md) 

**3. DNS Security & Pi-hole** | Implemented Pi-hole to block malicious domains and monitor DNS queries. | [`/pi-hole/README.md`](pi-hole/README.md) 

**4. Automation Scripts** | Bash and Python scripts for log rotation, backup, and scanning automation. | [`/automation/README.md`](automation/README.md) 

---

# Homelab Development Roadmap

This roadmap outlines the next logical phases for expanding the homelab beyond base network setup.  
Each stage builds on the previous one, moving from infrastructure to automation, monitoring, and advanced cybersecurity use cases.

---

## Phase 1 — Core Infrastructure Design & Implementation (Complete)
- Physical setup, cabling, and power management
- pfSense firewall, VLANs, and base routing
- Static IP assignments and DHCP reservations
- Pi-hole DNS filtering
- Functional segmented network (Admin, Lab, IoT, Guest)

---

## Phase 2 — System Hardening & Access Control
- Enforce SSH key authentication; disable password logins  
- Enable automatic security updates (`unattended-upgrades`)  
- Tighten pfSense firewall rules and enable logging  
- Configure DNS-over-HTTPS or Unbound resolver  
- Add VPN access for remote management (WireGuard / OpenVPN)

---

## Phase 3 — Security Monitoring & Logging
- Deploy SIEM or monitoring stack (Wazuh, Graylog, or ELK)  
- Forward pfSense, Pi-hole, and Ubuntu logs to central collector  
- Add Grafana dashboards for performance and security metrics  
- Enable alerting via email or Telegram  

---

## Phase 4 — Automation & Backup
- Automate updates and configuration backups with Ansible  
- Snapshot pfSense and Pi-hole configs on schedule  
- Store backups on NAS or GitHub private repo  
- Document infrastructure using Markdown or YAML (Infrastructure-as-Code approach)

---

## Phase 5 — Threat Simulation & Detection
- Build Blue/Red Team test environments  
- Deploy honeypots and IDS tools (T-Pot, Snort, or Suricata)  
- Run ethical attack simulations from Lab VLAN (Kali, Metasploit)  
- Practice detection, log analysis, and incident response  

---

## Phase 6 — Cloud & Remote Integration
- Connect homelab to cloud providers (AWS, Azure, or GCP)  
- Test site-to-site VPN or ZeroTier/Tailscale remote links  
- Backup key services to S3 or Google Drive with Rclone  
- Simulate hybrid infrastructure and cloud security monitoring  

---

## Phase 7 — Documentation & Visualization
- Maintain versioned setup notes and changelog  
- Update network diagrams as new VLANs or devices are added  
- Build GitHub wiki for installation/config guides  
- Add screenshots, metrics, and dashboards to portfolio site  

---

### 🎯 Summary
> **Secure → Monitor → Automate → Experiment → Document.**

This roadmap ensures the lab continues evolving toward a professional-grade cybersecurity environment —  
mirroring real enterprise network practices while remaining modular and scalable.

---

## Network Diagram

![Network Diagram](https://github.com/Th3Sp00kyM8/starting-homelab-development/blob/3cbeccdb5610b4181104c97933527bf9b0bb062a/topology/network-diagram.md)

---

## Tools & Technologies

**Network:** pfSense, OpenVPN, WireGuard, VLANs  
**Security:** Wazuh, Suricata, Snort, Pi-hole  
**Automation:** Bash, Python, Ansible  
**Visualization:** Grafana, Kibana  

---

## Future Plans

- Deploy honeypots for threat emulation  
- Integrate cloud-based logging via AWS
- Publish guides and lessons learned per milestone

