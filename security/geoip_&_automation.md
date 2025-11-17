## **1. Introduction**

This project adds a reliable GeoIP blocking system to OPNsense using MaxMind data, a custom parser, and local URLTable aliases. This workflow uses MaxMind’s country CSV files to generate local blocklists. These lists are loaded directly into PF and updated through a scheduled cron job. 

## **2. Requirements**

To use this GeoIP automation system, the following components are needed:

- **OPNsense**
- **MaxMind**: https://dev.maxmind.com/ 

## **3. CIDIR Lists**

Country

https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-Country-CSV&license_key=(YOUR-LICENSE-KEY)&suffix=zip
- Identifies the country an IP address is associated with. Useful for broad-level blocking or allowing traffic (for example, geo-blocking high-risk countries). Lightweight and least detailed.

https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-ASN-CSV&license_key=(YOUR-LICENSE-KEY)&suffix=zip
- Identifies the Autonomous System Number (ASN) behind an IP address. This tells you which ISP, hosting provider, cloud service, or network owner the traffic comes from. Useful for blocking known malicious ASNs or filtering out traffic from data centers.

https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-City-CSV&license_key=(YOUR-LICENSE-KEY)&suffix=zip
- Most detailed of the three. Useful for high-resolution filtering or logs that need more geographic context.

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

## **5. OPNsense GeoIP Generator**


## **6. OPNsense Alias Configuration**

OPNsense uses URLTable aliases to load the generated GeoIP CIDR lists into PF. Each country has its own alias that references the local `.txt` file created by the GeoIP script.

## **8. OPNsense Firewall Integration**




## **9. Cron Automation Setup*

The GeoIP system can be fully automated by scheduling the generator script to run on a fixed interval. Using cron ensures that MaxMind data is refreshed and the firewall reloads updated CIDR lists without manual intervention.

0 4 * * 1 python3 /usr/local/opnsense/scripts/geoip-gen.py && configctl filter reload >> /var/log/geoip-update.log 2>&1

What this schedule means

Runs every Monday at 04:00

Generates new CIDR lists

Reloads PF so updates take effect

Saves output to /var/log/geoip-update.log

Verify the cron entry

crontab -l

Once added, the system automatically updates all GeoIP blocklists on schedule with no additional steps required.



## **Security Considerations**

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

