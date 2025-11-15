## **1. Introduction**

This project adds a reliable GeoIP blocking system to OPNsense using MaxMind data, a custom parser, and local URLTable aliases. The default GeoIP feature in OPNsense is limited, often outdated, and does not provide real CIDR ranges. This workflow replaces it with an accurate solution that uses MaxMind’s country CSV files to generate local blocklists. These lists are loaded directly into PF and updated through a scheduled cron job. The result is a stable and fully automated GeoIP defense that runs entirely on the firewall without depending on external URLs.

## **2. Requirements**

To use this GeoIP automation system, the following components are needed:

- **OPNsense** with shell access  
- **Python 3** installed on the firewall  
- The **maxminddb** Python module  
- A valid **MaxMind GeoLite2** license key  
- Access to the MaxMind Country CSV downloads  
- Writable directories at:  
  - `/usr/local/share/GeoIP` for raw CSV files  
  - `/usr/local/opnsense/geo` for generated CIDR lists  
- Ability to edit firewall aliases in `config.xml`  
- Permission to create cron jobs using `crontab -e`

These items ensure the system can download GeoIP data, convert it into CIDR blocks, and load the lists into OPNsense.

## **3. Directory Layout**

This system uses two main directories to store GeoIP data and generated IP ranges:

- **Raw MaxMind files**  
/usr/local/share/GeoIP

This folder holds the downloaded GeoLite2 CSV archives.

- **Generated CIDR lists**  
/usr/local/opnsense/geo

Each country receives a `.txt` file containing all IPv4 and IPv6 CIDR blocks.

OPNsense URLTable aliases point directly to these local `.txt` files, allowing the firewall to load updated GeoIP ranges without relying on external servers.

## **4. Installation Steps**

Follow these steps to prepare the system for GeoIP generation:

1. **Create required directories**
   ```sh
   mkdir -p /usr/local/share/GeoIP
   mkdir -p /usr/local/opnsense/geo

2. Download MaxMind GeoLite2 Country CSV
Place the ZIP file in /usr/local/share/GeoIP using fetch:
fetch -o GeoLite2-Country-CSV.zip "https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-Country-CSV&license_key=YOUR_LICENSE_KEY&suffix=zip"
unzip GeoLite2-Country-CSV.zip

3. Ensure Python 3 is available
python3 --version

4. Install the maxminddb module
python3 -m pip install maxminddb

5. Confirm directories contain the expected data
ls /usr/local/share/GeoIP
ls /usr/local/opnsense/geo

The system is now ready for the GeoIP generator script.

## **5. GeoIP Generator Script**

A Python script is used to convert MaxMind’s CSV files into country-specific CIDR lists. The script reads the IPv4 and IPv6 blocks, matches them to country codes, and writes each country’s ranges into a `.txt` file.

**Location**
/usr/local/opnsense/scripts/geoip-gen.py


**Make it executable**
```sh
chmod +x /usr/local/opnsense/scripts/geoip-gen.py

Script output
Each run generates files such as:
/usr/local/opnsense/geo/CN.txt
/usr/local/opnsense/geo/RU.txt
/usr/local/opnsense/geo/IR.txt
...

These files contain the complete list of CIDR ranges for each country and are used directly by OPNsense URLTable aliases.

## **6. Running the Script Manually**

The GeoIP generator script can be run at any time to refresh all country CIDR lists. This is useful for testing, troubleshooting, or updating before the scheduled cron job.

**Run the script manually**
```sh
python3 /usr/local/opnsense/scripts/geoip-gen.py && configctl filter reload

## **6. Running the Script Manually**

The GeoIP generator script can be run at any time to refresh all country CIDR lists. This is useful for testing, troubleshooting, or updating before the scheduled cron job.

**Run the script manually**
```sh
python3 /usr/local/opnsense/scripts/geoip-gen.py && configctl filter reload

Wrote 7477 CIDRs for CN
Wrote 13081 CIDRs for RU
Wrote 1995 CIDRs for IR
...
Done.
OK

A successful run confirms that all CIDR lists were regenerated and the firewall applied the updates.

## **7. OPNsense Alias Configuration**

OPNsense uses URLTable aliases to load the generated GeoIP CIDR lists into PF. Each country has its own alias that references the local `.txt` file created by the GeoIP script.

**Location of CIDR files**
/usr/local/opnsense/geo


**Example alias entry**
```xml
<alias uuid="geo-cn-001">
  <enabled>1</enabled>
  <name>Geo_CN</name>
  <type>urltable</type>
  <url>file:///usr/local/opnsense/geo/CN.txt</url>
  <updatefreq>1</updatefreq>
  <descr>China GeoIP Blocklist</descr>
</alias>

How to apply the aliases
After adding or updating alias entries in config.xml, reload the firewall to activate them:

configctl filter reload

Each alias becomes a PF table and is immediately available for use in firewall rules.

## **8. PF Verification Commands**

After generating CIDR lists and creating URLTable aliases, you can verify that OPNsense loaded them correctly into PF. These commands confirm that each alias exists, contains valid entries, and is active in the firewall.

**List all PF tables**
```sh
pfctl -s Tables

Check a specific GeoIP table
pfctl -t Geo_CN -T show

Count the number of CIDR ranges
pfctl -t Geo_CN -T show | wc -l

What to expect
You should see thousands of entries for larger countries and several hundred for smaller ones, depending on the country’s allocation size.

These commands confirm that the GeoIP data is active and the firewall is using the generated CIDR lists.

## **9. Cron Automation**

The GeoIP system can be fully automated by scheduling the generator script to run on a fixed interval. Using cron ensures that MaxMind data is refreshed and the firewall reloads updated CIDR lists without manual intervention.

**Edit the root crontab**
```sh
crontab -e

Add the scheduled job

0 4 * * 1 python3 /usr/local/opnsense/scripts/geoip-gen.py && configctl filter reload >> /var/log/geoip-update.log 2>&1

What this schedule means

Runs every Monday at 04:00

Generates new CIDR lists

Reloads PF so updates take effect

Saves output to /var/log/geoip-update.log

Verify the cron entry

crontab -l

Once added, the system automatically updates all GeoIP blocklists on schedule with no additional steps required.

## **10. Firewall Rule Integration**

Once the GeoIP tables are generated and loaded, you can use them directly in firewall rules to block or reject traffic from specific countries. These rules operate at the PF level, making them efficient and fast.

**Example: Block inbound traffic from high-risk countries**
1. Go to **Firewall → Rules → WAN**
2. Add a new rule:
   - **Action:** Block  
   - **Interface:** WAN  
   - **Source:** `Geo_HighRisk` (or any Geo_* alias)  
   - **Destination:** Any  
   - **Description:** Block high-risk GeoIP ranges  

**Example XML (optional for config.xml)**
```xml
<rule>
  <type>block</type>
  <interface>wan</interface>
  <source>
    <network>Geo_HighRisk</network>
  </source>
  <destination>
    <any/>
  </destination>
  <descr>Block high-risk GeoIP ranges</descr>
</rule>

Reload PF after making changes

configctl filter reload

After adding the rule, traffic from the selected GeoIP ranges will be dropped before reaching any internal services, strengthening perimeter security with minimal overhead.

## **11. Troubleshooting & Verification**

This section covers common checks and fixes to ensure the GeoIP system works reliably. Use these steps if aliases appear empty, rules do not trigger, or cron updates fail.

### **Check if CIDR files were generated**
```sh
ls -l /usr/local/opnsense/geo

You should see files such as:

CN.txt
RU.txt
IR.txt
...

Verify PF loaded the table

pfctl -t Geo_CN -T show

If it outputs thousands of CIDRs, it is working.

If it prints nothing:

The alias XML entry may be incorrect

The file path might be wrong

The GeoIP script may not have generated data

Check the GeoIP script for errors

python3 /usr/local/opnsense/scripts/geoip-gen.py

If you see:
Wrote XXXX CIDRs for <country>
Done.

Then the data is valid.

Check cron activity
crontab -l
tail -50 /var/log/geoip-update.log

If the log shows successful writes, the automation is functioning.

Force a full firewall reload
configctl filter reload
If alias still doesn’t populate

Make sure the alias type is URL Table

Make sure the URL uses the file:// prefix

Make sure permissions allow reading /usr/local/opnsense/geo

Ensure there are no BOM or hidden characters in the .txt files

This section ensures you can verify the end-to-end pipeline and quickly identify any point of failure

If alias still doesn’t populate

Make sure the alias type is URL Table

Make sure the URL uses the file:// prefix

Make sure permissions allow reading /usr/local/opnsense/geo

Ensure there are no BOM or hidden characters in the .txt files

This section ensures you can verify the end-to-end pipeline and quickly identify any point of failure

## **12. Security Considerations**

Using GeoIP-based blocking can significantly reduce noise, brute-force attempts, botnet traffic, and automated scanning. However, understanding its strengths and limitations ensures it is applied safely and effectively.

### **Strengths**
- Blocks large volumes of malicious traffic before it reaches services  
- Reduces brute-force attempts from known high-risk regions  
- Low CPU overhead due to PF table efficiency  
- Easy to automate and maintain  
- Does not depend on Suricata or additional plugins  

### **Limitations**
- GeoIP accuracy is imperfect; IPs can be misclassified  
- Attackers can use VPNs or proxies in allowed countries  
- Over-blocking may interfere with legitimate international access  
- Updates must be kept consistent to avoid stale data  

### **Best Practices**
- Use GeoIP blocking as a **supplement**, not the primary security control  
- Combine rules with **rate limiting**, **fail2ban equivalents**, or **IPS**  
- Only block countries truly irrelevant to your environment  
- Keep cron updates active to ensure freshness  
- Review PF tables periodically:
  ```sh
  pfctl -t Geo_HighRisk -T show | wc -l
