
<img width="1947" height="2387" alt="Cisco network diagram" src="https://github.com/user-attachments/assets/444ee836-6a17-426a-9c46-66c5008d8c1b" />
Draft 1.0

## Network Overview & Design Nuances

This network is designed to emulate a layered enterprise environment within a compact home lab. Each component serves a distinct purpose within a modular, defense-in-depth structure, emphasizing separation of roles, segmentation, and realistic visibility.

The topology begins with a **GL.iNet Slate AX** acting as a *wireless bridge*, converting a mobile hotspot connection into a wired uplink for the **TP-Link ER605** router. The ER605 performs basic routing and WAN management before handing traffic to the **Protectli Vault running pfSense**, which enforces advanced firewall rules, VLAN segmentation, and packet inspection.

A managed **TP-Link TL-SG108PE PoE+ switch** distributes both data and power across the network, linking wired endpoints and the **TP-Link EAP610 access point**, which extends connectivity through multiple SSIDs tied to VLANs (Admin, Lab, IoT, Guest). This segmentation ensures that wireless devices operate within their designated trust zones.

Downstream, compute and monitoring devices — including two **Raspberry Pi nodes** and a **dedicated Security Server (mini PC)** — form the analytical and experimental layer. The Pis host lightweight services such as DNS filtering, containers, and test automation, while the Security Server aggregates logs, runs SIEM and IDS workloads, and provides centralized visibility across all VLANs. This setup models a small-scale SOC environment, where data from multiple sources can be correlated for threat detection and incident response.

Every element of the network is powered through a **CyberPower UPS**, maintaining operational continuity during outages and reflecting enterprise-grade reliability practices. The resulting design provides not just connectivity, but a controlled environment for practicing network security, monitoring, and system hardening in a realistic, scalable way.
