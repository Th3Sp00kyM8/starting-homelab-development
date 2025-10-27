# Rack Overview

This homelab is a compact, rack-mounted environment built to simulate an enterprise-grade network and security infrastructure. Its purpose is to provide a controlled space for learning, testing, and documenting cybersecurity concepts.

### Objectives
- Gain practical experience configuring firewalls, routers, and switches.  
- Develop hands-on understanding of VLANs, VPNs, IDS/IPS systems, and secure network segmentation.  
- Host and monitor lightweight servers. 
- Build a foundation for future cloud, automation, and SIEM integrations.  

### Physical Environment
- **Form Factor:** GeeekPi T0 mini rack — compact, open-frame design suitable for small spaces.  
- **Mounting:** Devices secured on 10″ shelves with a recessed power strip for cable management.  
- **Power Backup:** CyberPower EC650LCD UPS providing surge protection and short-term runtime for all core devices.  
- **Network Core:** Protectli Vault FW4B firewall, TP-Link ER605 router, TL-SG108PE switch, and EAP610 access point.  
- **Compute Nodes:** Raspberry Pi 4 and 5 running Ubuntu Server 24 LTS for services like Docker, Pi-hole, and security monitoring.  
- **Connectivity:** GL.iNet Slate AX acts as a bridge from the hotspot to the internal router, ensuring flexible uplink options.

This rack acts as the backbone for my current and future cybersecurity labs, balancing energy efficiency, reliability, and modular scalability. It mirrors a scaled-down enterprise topology, giving room to practice both network administration and defensive security principles in a realistic yet manageable setup.

---

## Hardware + Power Layout

This section documents the physical and electrical organization of the homelab rack. Every device is powered through a dedicated, battery-backed source to maintain uptime and prevent data loss during interruptions.

### Power System

**Primary UPS:**  
**CyberPower EC650LCD Ecological Battery Backup and Surge Protector**  
- 650 VA / 390 W capacity with ECO mode for peripheral control.  
- Eight outlets: four battery-backed, four surge-only.  
- LCD interface for load and runtime monitoring.  
- Supplies consistent power to all networking and compute hardware.

**Connection Hierarchy:**  
1. Wall outlet → UPS input.  
2. UPS → recessed, mountable power strip integrated into rack.

The power design prioritizes stability, energy efficiency, and cable management. Every component is accessible, labeled, and arranged to support quick maintenance or upgrades.

---

## Network Architecture & Data Flow

This section outlines how traffic moves through the homelab—from the external connection to the internal network nodes and how each device contributes to performance, control, and security.

### Network Path

**Primary Flow:**  
> Hotspot → GL.iNet Slate AX (Bridge) → TP-Link ER605 (Router) → Protectli Vault FW4B (Firewall) → TP-Link TL-SG108PE (Switch) → TP-Link EAP610 (Access Point) → Raspberry Pi 4 / Raspberry Pi 5 & Future Lab Devices

### Component Roles

| Device | Function | Notes |
|---------|-----------|-------|
| **Hotspot** | External Internet Source | Provides WAN connectivity to the GL.iNet device; portable and flexible uplink. |
| **GL.iNet Slate AX** | Router/Bridge | Converts hotspot signal into Ethernet WAN for the router. Also used for VPN tunnel testing and as a secure gateway device. |
| **TP-Link ER605** | Router & Firewall Hybrid | Manages WAN routing, VPNs, and load balancing. Acts as a pre-firewall router simulating an ISP-managed edge. |
| **Protectli Vault FW4B (pfSense)** | Dedicated Firewall | Performs internal network segmentation, packet filtering, intrusion prevention, and VLAN management. The true “gatekeeper” of the network. |
| **TP-Link TL-SG108PE** | Managed PoE+ Switch | Distributes wired connections to all devices, supports VLANs, and powers the access point via PoE+. |
| **TP-Link EAP610** | Wireless Access Point | Extends network connectivity to wireless clients, broadcasting VLAN-segmented SSIDs. |
| **Raspberry Pi 4 / 5** | Compute Nodes | Host services like Pi-hole, Docker containers, SIEM agents, and IDS sensors. Provide visibility into traffic and system behavior. |
| **Future Devices** | Peripheral or Lab Equipment | Placeholder for webcams, NAS, IoT, or honeypot devices for future security simulations. |

### Network Logic

This layered path simulates a small enterprise topology, where routing, security, and access are intentionally separated:

1. **Edge Connectivity (Hotspot + GL.iNet)**  
   - Represents the external ISP or WAN link.  
   - Allows testing of VPN, DNS leak prevention, and remote access security.

2. **Routing Layer (ER605)**  
   - Handles load balancing, static routes, and WAN monitoring.  
   - Acts as the first controlled entry point before traffic hits the firewall.

3. **Security Layer (Protectli Vault)**  
   - Applies firewall rules, NAT, and VLAN segmentation.  
   - Enforces the Principle of Least Privilege by isolating IoT, Lab, and Admin networks.  
   - Provides deep packet inspection and intrusion detection capabilities.

4. **Distribution Layer (Switch + Access Point)**  
   - Propagates VLANs from pfSense to both wired and wireless devices.  
   - PoE+ switch simplifies power and data delivery.  
   - The access point acts as a bridge for all wireless network segments.

5. **Compute Layer (Pis + Future Devices)**  
   - Hosts services, monitors network activity, and logs events for analysis.  
   - Emulates internal endpoints or small scale servers commonly found in corporate networks.

### Wireless Design

- Only one wireless access point (EAP610) is used to simplify VLAN-to-SSID mapping.  
- Wireless devices connect through segmented SSIDs:  
  - **Admin SSID** — secure, management-only access.  
  - **Lab SSID** — for experiments and test devices.  
  - **IoT SSID** — for smart or low-trust devices.  
  - **Guest SSID** — isolated public connection if needed.  

### Summary

This network design ensures:
- **Segmentation:** Separate VLANs prevent unauthorized lateral movement.  
- **Scalability:** Additional subnets or VLANs can be added without redesigning the core.  
- **Visibility:** Traffic can be mirrored or monitored for IDS/SIEM analysis.  
- **Realism:** Mirrors multi-layer enterprise environments while remaining energy-efficient and compact.

---

## Design Logic & Device Synergy

This homelab was designed to be modular, efficient, and realistic; replicating how a small enterprise network separates roles across routing, security, access, and compute layers. Each component was selected based on its technical purpose, compatibility, and contribution to overall system synergy.

### Security & Routing Layers

**Protectli Vault FW4B (pfSense Firewall):**  
- Chosen as the central security appliance for its open-source flexibility and enterprise-level firewalling.  
- Provides VLAN segmentation, intrusion prevention, NAT control, and VPN support.  
- Mirrors real-world setups where traffic is filtered and segmented before reaching internal systems.

**TP-Link ER605 (Router/Firewall Hybrid):**  
- Manages WAN routing and VPN tunnels, offloading basic network handling from the firewall.  
- Acts as the “ISP gateway” in this environment, simulating how corporate networks separate routing and security duties.  
- Integrates seamlessly with Omada Controller for centralized SDN management.

**Synergy:**  
These two devices together create a layered defense model:
1. The ER605 performs routing and basic filtering.  
2. The Protectli Vault enforces detailed firewall rules and segmentation.  
This separation models real-world defense-in-depth and enables independent failure testing of routing vs. security functions.

**Note:**
- Because this topology uses both a router (TP-Link ER605) and a firewall (Protectli Vault running pfSense), there is a potential for double NAT.

---

### Network Access & Distribution Layers

**TP-Link TL-SG108PE (Switch):**  
- Acts as the wired backbone for all network devices.  
- Supports VLANs, QoS, and port mirroring for traffic analysis.  
- PoE+ simplifies cabling and ensures reliable power delivery to the access point.

**TP-Link EAP610 (Wi-Fi 6 Access Point):**  
- Broadcasts VLAN-specific SSIDs for isolated wireless zones (Admin, Lab, IoT, Guest).  
- Wi-Fi 6 ensures high throughput and low latency for multiple test devices.  
- Integrates with the router and switch under Omada SDN, enabling unified network policy enforcement.

**Synergy:**  
The switch and access point extend pfSense VLANs across both wired and wireless clients. Together they form a **single, managed network fabric**, allowing consistent security policy application and traffic visibility.

---

### Compute & Experimentation Layer

**Raspberry Pi 4 & Raspberry Pi 5 (Ubuntu Server 24 LTS):**  
- Serve as compact, energy-efficient compute nodes for running Docker containers, Pi-hole, Wazuh, and Suricata.  
- Provide a safe sandbox for testing automation, SIEM data flows, and IDS configurations.  
- The Pi 5, with its upgraded CPU and I/O, handles heavier workloads like security analytics, while the Pi 4 manages supporting services.

**Synergy:**  
Both Pis are low-cost and modular, letting experiments fail safely without risking critical infrastructure. They’re also ideal for building lightweight, reproducible lab environments, where tools can be spun up or reset as needed.

---

### Power & Reliability Layer

**CyberPower EC650LCD UPS + Recessed Power Strip:**  
- Ensures uninterrupted operation of all core devices and allows for controlled shutdowns during power loss.  
- ECO mode reduces draw from idle peripherals, maintaining energy efficiency.  
- Integrated into the rack for neat cable routing and space efficiency.

**Synergy:**  
The UPS and power strip act as a power foundation layer, protecting equipment and preserving lab continuity. They reinforce professional grade resilience.

---

### Design Philosophy

The overall design follows three principles:

1. **Modularity** — Every layer (routing, security, access, compute, power) can be upgraded or replaced independently.  
2. **Realism** — The topology mirrors enterprise networks but stays compact enough for a home environment.  
3. **Resilience** — Redundant power, segmentation, and fault isolation allow experimentation without full-system downtime.

This modular, layered approach supports future scaling: adding a NAS, honeypots, additional Pis, or cloud integration without restructuring the network. It is an intentional architecture suitable for both education and professional growth in cybersecurity.

---

### Future Expansion

The current rack design leaves headroom for additional devices and new learning domains. These additions will help simulate more complex and realistic enterprise or IoT environments.

| Category | Planned Additions | Purpose |
|-----------|------------------|----------|
| **Security Monitoring** | Honeypots, dedicated IDS appliances | Threat analysis and intrusion research | Advance traffic inspection, file sharing, and log archiving |
| **Storage** | NAS (TrueNAS, Synology) | Centralized backups and data hosting |
| **IoT Testing** | Cameras, sensors, smart devices | Network segmentation and vulnerability analysis | VLAN isolation, IoT threat modeling |
| **Automation & Orchestration** | Ansible, Python automation scripts | Infrastructure-as-code and rapid lab redeployment |
| **Cloud Integration** | AWS nodes | Hybrid lab experiments for cloud security testing |
| **Storage & Backup** | NAS (TrueNAS, Synology, or Pi-based) | Centralized backups, file sharing, and log archiving. |
| **Endpoint Security** | Lightweight Linux or Windows VM | Host-based firewall, antivirus, and SOC analysis exercises. |

---

### Summary

This homelab is not a random assortment of parts, it’s a designed ecosystem where every device reinforces another in order to provide a strong environment for experimentation, applying concepts, and obtaining hands on experience.
