
<img width="3651" height="4625" alt="Cisco network diagram(3)" src="https://github.com/user-attachments/assets/7ad49207-8da7-41d0-9d95-3bd7ee70c537" />
Designed with:https:lucid.app

## Network Overview Draft 1.0

This network is structured for modularity, visibility, and segmentation.  
- **Bridge (GL.iNet):** Converts wireless WAN from a hotspot to Ethernet for stable routing.  
- **Router → Firewall chain:** Separates edge routing (ER605) from internal security (pfSense) to mirror enterprise layering.  
- **Managed Switch:** Central distribution for VLANs, PoE power, and traffic monitoring via SPAN.  
- **Access Point:** Broadcasts VLAN-specific SSIDs for Admin, Lab, IoT, and Guest networks.  
- **Compute Layer:** Raspberry Pis handle light services (DNS, containers); the mini PC serves as a security node for SIEM, IDS, and log aggregation.  
- **UPS:** Maintains uptime and supports graceful shutdowns.  

## Current Limitations & Design Considerations

### 🧩 Functional Gaps
- **No central authentication:** User and device access rely on local credentials; no RADIUS, LDAP, or SSO integration yet.  
- **No automated backups:** Configurations (pfSense, switch, Pi services) are not yet on scheduled or version-controlled backups.  
- **Limited visibility scope:** pfSense and Pi nodes log locally; full log correlation and long-term retention depend on the Security Server build-out.  
- **No vulnerability scanning or compliance auditing:** No OpenVAS, Nessus, or CIS benchmark scanning implemented yet.  
- **No high availability (HA):** Single points of failure exist at the router, firewall, and switch levels.  
- **Double NAT remains:** Current chain (Router → Firewall) introduces double NAT until bridge/passthrough mode is finalized.  
- **No remote management isolation:** Admin access occurs over the same VLAN as local devices; a dedicated management subnet is under review.

### Planned Design Improvements
- **Network Monitoring:** Implementing Grafana + Prometheus or Netdata for device metrics and uptime visualization.  
- **SIEM Integration:** Deploying Wazuh or Security Onion on the Security Server for log correlation and alerting.  
- **Automated Configuration Management:** Using Ansible for backup, patching, and provisioning across Pis and pfSense.  
- **NAS Integration:** Adding local network storage for log retention, backups, and forensic data.  
- **VLAN Refinement:** Expanding segmentation to include a dedicated Monitoring VLAN and DMZ for honeypots or test servers.  
- **Cloud Connectivity:** Linking an AWS node via VPN to simulate hybrid-cloud security models.  
- **Threat Simulation:** Introducing honeypots and IDS testing for active defense exercises.  
- **Identity & Access Control:** Evaluating lightweight RADIUS or FreeIPA setup for centralized authentication.  
- **Power & Environment Monitoring:** Exploring SNMP-based UPS and temperature sensors for system health tracking.

These refinements aim to evolve the lab from a segmented local network into a fully observable, managed, and automatable defensive environment.

