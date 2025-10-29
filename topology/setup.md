This document will provide an overview on general setup. 

# Section 1: Physical Preparation

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
Boot devices upstream to downstream to prevent IP conflicts and ease troubleshooting.

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
- Set LAN IP to 192.168.1.1. Subnet mask 255.255.255.224
- Enable temporary DHCP for pfSense and initial setup.  
- Verify Internet access from a device on the router LAN.

---

### 2.4 Firewall (Protectli Vault)
- Connect monitor and keyboard for first boot. 
- Assign interfaces:
  - WAN → Router LAN port  
  - LAN → Switch Port   
- Set LAN IP to 192.168.10.1 and subnet 255.255.255.224
- Enable DHCP on LAN, range 192.168.10.22 to 192.168.10.30 
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
- Adopt the AP, set: static IP 192.168.10.3, Subnet 255.255.255.224, Gateway: 192.168.10.1
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
- Assign IP 192.168.10.20
  

**Pi 5 (Lab Node)**

- Boot from SD / USB with Ubuntu Server 24 LTS. 
- Connect to switch; confirm DHCP assignment.
- Prepare for containerized services or IDS testing.

---

### 2.8 GMKtec Security Server

- Connect via Ethernet to switch.
- Install Ubuntu Server 24 LTS.
- Assign IP 192.168.10.10

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

Once base connectivity and addressing are verified, proceed to Section 3: VLAN & Firewall Configuration.


# Section 3: VLAN & Firewall Configuration

This phase introduces network segmentation and basic access controls to simulate an enterprise-style defensive layout.

---

### 3.1 VLAN Planning

VLANs provide logical isolation between device groups. The configuration below assumes pfSense as the VLAN controller and the switch as the distribution layer.

| VLAN ID | Name | Subnet | Purpose | Example Devices |
|----------|------|---------|----------|----------------|
| 10 | Admin | 192.168.10.0/27 | Core infrastructure and management | pfSense, Switch, GMKtec, Pi-hole |
| 20 | Lab | 192.168.20.0/27 | Testing, containers, and experimental workloads | Raspberry Pi 5, test VMs |
| 30 | IoT | 192.168.30.0/27 | Smart or untrusted devices | Cameras, sensors, printers |
| 40 | Guest | 192.168.40.0/27 | Temporary or public access | Visitor Wi-Fi, sandbox devices |

---

### 3.2 pfSense VLAN Configuration

- Log in to pfSense (https://192.168.10.1).
- Navigate to Interfaces → Assignments → VLANs → Add.
- Create VLANs on the LAN interface as follows:
   - VLAN 10 → Admin  
   - VLAN 20 → Lab  
   - VLAN 30 → IoT  
   - VLAN 40 → Guest
-  Under Interfaces → Assignments, add each VLAN as a new interface.
-   Assign static IPs:
   - VLAN 10 → 192.168.10.1/27  
   - VLAN 20 → 192.168.20.1/27  
   - VLAN 30 → 192.168.30.1/27 
   - VLAN 40 → 192.168.40.1/27  
- Enable DHCP Server on each VLAN (optional for IoT/Guest).  

---

### 3.3 Switch VLAN Configuration (TP-Link TL-SG108PE)

- Access the switch management page (192.168.10.2).
- Under VLAN → 802.1Q VLAN, create the same VLAN IDs (10, 20, 30, 40).
- Tag the port connected to pfSense (uplink port) with all VLANs.
- Assign untagged ports per device group:

| Port | Device | VLAN | Tagging |
|------|---------|------|----------|
| 1 | pfSense LAN | 10–40 | Tagged (Trunk) |
| 2 | GMKtec Security Server | 10 | Untagged |
| 3 | Raspberry Pi 4 (Pi-hole) | 10 | Untagged |
| 4 | Raspberry Pi 5 (Lab Node) | 20 | Untagged |
| 5 | Access Point | 10–40 | Tagged (Trunk) |
| 6–8 | Spare / Future Use | TBD | Custom |

---

### 3.4 VLANs on the Access Point (EAP610)

If using the Omada Controller:
1. Adopt the AP and go to Wireless Networks → Add SSID.  
2. Create separate SSIDs mapped to VLAN IDs:
   - AdminNet → VLAN 10  
   - LabNet → VLAN 20  
   - IoTNet → VLAN 30  
   - GuestNet → VLAN 40  
3. Optionally limit Guest SSID access under Wireless Control → Access Control.

---

### 3.5 pfSense Firewall Rules

Navigate to Firewall → Rules and configure at least one rule set per VLAN interface.

| Interface | Action | Source | Destination | Description |
|------------|---------|---------|-------------|-------------|
| VLAN 10 (Admin) | Allow | VLAN 10 net | Any | Full access for management |
| VLAN 20 (Lab) | Allow | VLAN 20 net | Any | Limited outbound / testing |
| VLAN 30 (IoT) | Allow | VLAN 30 net | Internet (not local nets) | Isolated outbound only |
| VLAN 40 (Guest) | Allow | VLAN 40 net | Internet only | Fully isolated guest access |
| All VLANs | Block | Any | RFC1918 private nets | Default isolation rule |

Then add a final “deny all” rule on each VLAN to block unauthorized traffic between internal subnets.

---

### 3.6 Verification

After configuration:
1. Connect a device to each VLAN/SSID.  
2. Confirm the assigned IP address matches the expected subnet.  
3. Ping external sites (8.8.8.8) to verify Internet access.  
4. Attempt to ping another VLAN — it should fail unless allowed by rule.  
5. Confirm pfSense Dashboard → Interfaces shows active gateways for all VLANs.

---

Once VLAN segmentation and firewall rules are confirmed, proceed to Section 4: Post-Setup Verification & Next Steps.


# Section 4: Post-Setup Verification & Next Steps

This stage confirms that all devices are reachable, properly segmented, and stable. It also defines the first set of maintenance and expansion actions.

---

### 4.1 Connectivity Verification
From any device on the Admin VLAN (laptop, GMKtec, or Pi):

```
ping 192.168.10.1   # pfSense
ping 192.168.10.2   # Switch
ping 192.168.10.3   # Access Point
ping 192.168.10.4   # Pi-hole
ping 192.168.10.5   # Security Server
```

- All pings should respond within 1 ms – 3 ms.
- Confirm Internet access: ping 8.8.8.8.
- Check pfSense → Status → DHCP Leases to verify each device’s IP is correct.

# 4.2 VLAN Isolation Tests

Connect a client to each SSID (Admin, Lab, IoT, Guest).

Run:
```
ip a        # verify correct subnet (10.x, 20.x, etc.)
ping 192.168.10.1
ping 192.168.20.1
```

- VLANs should reach the Internet but not cross-communicate unless rules permit.
- Confirm pfSense Firewall Logs → Normal View show blocked inter-VLAN attempts.

# 4.3 Service Verification

- Pi-hole:
Visit http://192.168.10.4/admin → confirm queries populate in Query Log.
- AP:
Test each SSID → ensure expected VLAN assignment.
- Switch:
Check port LEDs → confirm link lights and PoE power (for AP).
- Security Server:
SSH in and confirm Internet + DNS resolution.

# 4.4 Configuration Backups

Create full backups for reproducibility:

# pfSense
```
Diagnostics → Backup/Restore → Download configuration.xml
```
# Switch
```
System → Save Config
```
# Pi-hole
```
Settings → Teleporter → Export
```
# Ubuntu Servers
```
sudo tar czvf config_backup_$(date +%F).tar.gz /etc/netplan /etc/pihole /etc/wazuh*
```

- Store all backups in a /backups/configs/ folder (local or GitHub-private).

# 4.5 Maintenance & Next Steps

| Area               | Action                                                   | Purpose                                                  |
| ------------------ | -------------------------------------------------------- | -------------------------------------------------------- |
| **Monitoring**     | Deploy Wazuh, Security Onion, or Grafana stack on GMKtec | Central log collection and visibility                    |
| **Automation**     | Use Ansible or shell scripts for routine updates         | Simplify patching and config management                  |
| **Resilience**     | Clone Pi micro-SDs or move to SSD boot                   | Reduce wear and recovery time                            |
| **VLAN Expansion** | Add Monitoring VLAN or DMZ later                         | Improve isolation for honeypots or external-facing tests |
| **Documentation**  | Update this README after each major change               | Maintain version history and reproducibility             |

Completion Check:
- All core devices reachable
- Internet access verified
- VLAN isolation confirmed
- Backups saved
- Next-phase goals documented

