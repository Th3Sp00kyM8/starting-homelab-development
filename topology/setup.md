This document will provide an overview on general setup. 

# ⚙️ Section 2: Physical Preparation

This phase ensures all devices are safely powered, labeled, and networked before configuration begins.

---

### 1.1 Hardware Overview
All devices are either rack-mounted or adjacent. Data flow follow this physical order:

Hotspot/Internet > GL.iNet Bridge > TP-Link ER605 Router > Protectli Vault > TP-Link TL-SG108PE Switch > AP & End Devices


---

### 1.2 Power Setup
- All devices draw power through the CyberPower EC650LCD UPS, connected to a recessed power strip in the rack.  
- Each power adapter is labeled and connected to a UPS-protected outlet.  

---

### 1.3 Network Cabling
Use Cat6, STP or UTP are fine, Ethernet for all wired connections.  
The switch acts as the central hub for all LAN devices.

| Device | Connection | Destination | Notes |
|---------|-------------|--------------|-------|
| GL.iNet Bridge | LAN Port | ER605 WAN | Provides Internet feed from hotspot |
| TP-Link ER605 | LAN Port | pfSense WAN | Routes inbound traffic |
| Protectli Vault | LAN Port | Switch Port 1 | Primary gateway for internal network |
| TP-Link Switch | Port 6–8 (PoE) | Access Point | VLAN-tagged SSIDs |
| Raspberry Pi 4 | Port 3 | Switch | DNS filtering (Pi-hole) |
| Raspberry Pi 5 | Port 4 | Switch | Lab node / testing |
| GMKtec Mini PC | Port 5 | Switch | Security server / SIEM host |

Keep ports consistent; label both ends of each cable for traceability.

---

### 1.4 Initial Checks
Before powering everything on:
- Verify all Ethernet cables click securely.  
- Confirm power adapters match voltage and connector type.  
- Label all devices and ports clearly (e.g., “P3 – Pi4”, “P5 – GMKtec”).  
- Ensure airflow and cable slack for maintenance.  
- Keep monitor and keyboard nearby for pfSense initial setup.

Once all connections are verified, move to Network Initialization Section 3.



