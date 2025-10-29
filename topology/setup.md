This document will provide an overview on general setup. 

# ⚙️ Section 1: Physical Preparation

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

# Section 2: Network Initialization

This phase establishes base connectivity and verifies that each device communicates correctly within the network chain.

---

### 2.1 Power-On Sequence
Boot devices **upstream to downstream** to prevent IP conflicts and ease troubleshooting.

1. GL.iNet Bridge  
2. TP-Link ER605 Router  
3. Protectli Vault (pfSense)  
4. TP-Link TL-SG108PE Switch  
5. Access Point (EAP610)  
6. Raspberry Pi 4 / Raspberry Pi 5  
7. GMKtec Security Server  

---

### 2.2 GL.iNet Bridge Setup
- Connect a laptop to the GL.iNet via Ethernet or Wi-Fi.  
- Access the dashboard at 192.168.8.1.  
- Under Internet, connect the bridge to your mobile hotspot.  
- Under LAN, disable DHCP to ensure it passes addresses transparently.
- Save and reboot.
- Verify that the router receives Internet access through Ethernet.

---

### 2.3 Router (TP-Link ER605)
- Connect to the router via LAN port → open 192.168.0.1.  
- Set WAN as Dynamic IP (it will receive from GL.iNet).  
- Set LAN IP to 192.168.1.1. Subnet mask 255.255.255.0
- Enable temporary DHCP for pfSense and initial setup.  
- Verify Internet access from a device on the router LAN.

---

### 2.4 Firewall (Protectli Vault)
- Connect monitor and keyboard for first boot. 
- Assign interfaces:
  - WAN → Router LAN port  
  - LAN → Switch Port   
- Set LAN IP to 192.168.10.1 and subnet 255.255.255.0
- Enable DHCP on LAN, range 192.168.10.50 to 192.168.10.200 
- Plug laptop into the switch and it should get an IP assigned.
- Access pfSense via browser: https://192.168.10.1.
- Complete the initial setup wizard (set hostname, DNS, admin password).  

---

### 2.5 Switch (TP-Link TL-SG108PE)
- Connect a laptop to any open port on the switch.
- Download and run TP-Link Easy Smart Configuration Utility on your laptop (TP-Link’s official tool).
- Once started, it will show the switch. 
- Assign a static IP (e.g., 192.168.10.2).
- Reboot.
- Confirm uplink to pfSense and that all downstream devices receive IPs.  

---

### 2.6 Access Point (TP-Link EAP610)
- Power via PoE port on the switch.  
- Access via Omada Controller (temporarily install).  
- Adopt the AP, set: static IP 192.168.10.3, Subnet 255.255.255.0, Gateway: 192.168.10.1
- Create SSIDs for each VLAN (Admin, Lab, IoT, Guest) — leave VLAN tagging for Section 4.
- Save & Reboot.

---

### 2.7 Raspberry Pis
**Pi 4 (Pi-hole)**
- Boot from SD / USB with Raspberry Pi OS Lite or Ubuntu Server 24 LTS.  
- Connect to the switch; confirm DHCP assignment.  
- SSH in using its IP and install Pi-hole:

  ```bash
  curl -sSL https://install.pi-hole.net | bash
  ```

**Pi 5 (Lab Node)**

- Boot from SD / USB with Ubuntu Server 24 LTS. 
- Connect to switch; confirm DHCP assignment.
- Prepare for containerized services or IDS testing.

---

### 2.8 GMKtec Security Server

- Connect via Ethernet to switch.
- Install Ubuntu Server 24 LTS.

Update packages:

```
sudo apt update && sudo apt upgrade -y
```

- Confirm Internet access and ability to ping pfSense and Pis.

---

### 2.9 Verification Checks

Run these quick tests before moving on:

```
ping 192.168.10.1     # pfSense
ping 192.168.10.2     # Switch
ping 192.168.10.4     # Pi-hole
ping 192.168.10.5     # GMKtec
```

- Verify Internet connectivity from a LAN device.
- Confirm pfSense DHCP leases display all devices.

Once base connectivity and addressing are verified, proceed to Section 4: VLAN & Firewall Configuration.
