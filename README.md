<!--<head><style>
  .center-text { 
    text-align: center; 
  }
</style></head>-->

# Security-Lab
This will detail my home security lab setup, with the goal of making it reproducible by anyone.

## Objective: Demonstrate a two-layer defensive architecture:
•	A pfSense firewall running a network IDS (Snort or Suricata) to detect and optionally block malicious traffic in real time.
•	A Nessus vulnerability scanner to proactively discover misconfigurations, missing patches, and other known vulnerabilities on local network hosts.
•	Other devices, intended for flexibility of usage for a home security lab

Combine real-time detection with scheduled vulnerability assessments to reduce overall network risk and align with CISSP best practices for defense-in-depth.


### Setup: Using VirtualBox, the following setup should be functional:
1.	pfSense Firewall (10.1.1.1)
2.	Kali Blue Team (via dhcp – 10.1.1.26 in example)
3.	Kali Red Team (via dhcp – 10.1.1.27 in example)
4.	Ubuntu Web Server (via dhcp – 10.1.1.25 in example)
5.	Metasploitable 2 (via dhcp – broken currently, but will use in demo)
6.	Misc Other Devices (such as Win11 10.1.1.15 or dhcp)

Visually, a similar setup can be simulated in Cisco Packet Tracer: 
<br><img width="933" height="509" alt="image" src="https://github.com/user-attachments/assets/f4bfd087-42d9-4e15-b314-a059d9d5ba5d">

### Security Lab’s Environmental Setup (all using VirtualBox):
<pre>
1.	Setup a Perimeter Firewall: 
  a.	Deploy pfSense as the gateway for LAN Network “security-lab”
  b.	Configure DHCP IP allocation capability.
  c.	Configure rules to block unauthorized traffic.
2.	Create Security-Lab:
  a.	pfSense – Install Snort and possibly Suricata as well.
  b.	Kali-Blue – Install/Update Nessus
  c.	Ubuntu Web Server – Login Page with simple python web server (more easily made vulnerable than using apache2)
  d.	Kali-Red – Simulation: Used to trigger an attacker firewall rule against another VM (such as simple nmap scan, or spoofing the web server)
  e.	Optional: Metasploitable 2 (intentionally vulnerable)
  f.	Optional: Win11 Workstation
3.	Installations and Configurations:
  a.	Verify pfSense rules to detect SQL Injection or brute-force attempts (add custom rules as needed).
  b.	Configure pfSense via Kali-Blue with the browser GUI.
  c.	Configure Ubuntu Web Server, using simple python login page (or just use apache2).
  d.	Testing: Launch an attack from Kali-Red, such as nmap scans or brute-force attacks.
  e.	Confirm Nessus Essentials is available on Linux-Blue and scan.
  f.	Verify firewall blocks and/or alerts as expected.
<img width="604" height="466" alt="image" src="https://github.com/user-attachments/assets/8dc68894-b455-4508-a816-3070a695fcc0">

# Notes on Configurations:
1.	pfSense WAN will point to 192.168.1.1 – home router.
2.	Home-Lab will use a subset from the 10.0.0.0-10.255.255.255 range to distinguish between Home LAN vs. Security-Lab LAN.
3.	pfSense requires two adapters, bridged for Internet, and Internal Network for VLAN.
4.	Kali-Blue with Nessus requires extra storage space and resources.
5.	Other devices use network adapter Internal Network – security-lab.

## pfSense-Firewall
1.	LAN in promiscuous mode to let Snort capture packets from the whole network.
    <i>Sources: https://www.snort.org/ and https://www.pfsense.org/</i>
<img width="975" height="283" alt="image" src="https://github.com/user-attachments/assets/bc9d5c3c-939c-4cc3-a5a0-536517c5491f">
  a.	Memory: need at least 4 GB, the default will simply not run Snort. Going with 6 GB or 6144 MB for now.
  b.	Processors: 1 minimum, using 2.
  c.	Hard Disk: 16 GB VDI is default, using 20 GB (increasing later is a pain)
2.	BEFORE first boot:
  a.	pfSense Network Adapter 1: Bridged, promiscuous: Allow VM
  b.	pfSense Adapter 2: Internal Network name: security-lab, promiscuous: Allow All
  c.	Audio – disable.

<b><u>pfSense Installation:</u></b>
1.	Follow defaults, assign em0 to WAN active (when detects VirtualBox), and em1 to LAN
2.	Before first reboot, VirtualBox->Devices->Optical Drives remove the ISO.
  a.	If it will not reboot, VirtualBox->Power Off Machine. It should boot up but sometimes glitches and requires power cycling.
3.	Load pfSense, leave em0 alone, but set em1 to 10.1.1.1/24 (option 2, do not use dhcp to assign, set it to a fixed number, with 24 for mask, enable dhcp for range, ie 10.1.1.1/24.
<img width="975" height="255" alt="image" src="https://github.com/user-attachments/assets/822a8827-2ac2-4591-b777-3f2e28625e1d">
4.	Use 7 to ping home router (usually 192.168.1.1)
  •	Reference pfSense manual for issues: https://docs.netgate.com/pfsense/en/latest/

<b><u>Kali-Blue:</u></b>
1.	Network Adapter: Internal Network, name=security-lab (from drop down)
2.	Kali: ip addr show – should have a 10.1.1.x address, ping 10.1.1.1 and ping 192.168.1.1 and verify connectivity.
3.	In browser: 10.1.1.1 for pfSense setup: admin, pfsense (change pw to stop it complaining, save pw reminder somewhere, such as in the pfSense VirtualBox’s Description)
4.	In browser: https://localhost:8834 for Nessus after setup.

<u><b>Snort Installation/Setup</b> (from Kali-Blue):</u>
1.	Install snort package via available packages, or System->Package Manager
  •	Optionally install Suricata as well, just add the package
<img width="921" height="339" alt="image" src="https://github.com/user-attachments/assets/e5c0c210-0cbe-42d5-9c87-5f76b6cfcc1d">

2.	Navigate to Services->Snort->Global Settings
3.	Enable Snort VRT and add your OinkCode (from free snort account)
  a.	Enable Snort GPLv2, Enable ET Open, Enable OpenAppID (no pro stuff)
  b.	Resolve any errors by enabling/disabling incompatible services.
4.	Navigate to Snort->Updates and update rules to do the initial update (in global, could set to auto-update, but this is to make sure it is setup initially as expected)
<img width="861" height="703" alt="image" src="https://github.com/user-attachments/assets/704713c3-ef07-41ae-9ebc-fb944f49a966">

5.	Navigate to Snort Interfaces and enable LAN em1 and check Send Alerts to System Log
6.	Navigate to Snort Interfaces->LAN Categories, and check Use IPS Policy, then Select All from rulesets, and save.
7.	Return to Snort Interfaces and under Snort Status, start Snort (waiting for it to show green checkmark, then it is running)
8.	Optionally enable WAN em0 as well.
<img width="975" height="349" alt="image" src="https://github.com/user-attachments/assets/b1566c5b-af8d-4981-9546-bf6f6a107400">

9.	Once running, navigate to Services->Snort->Alerts->drop down Management (em1), select auto-refresh view and save
10.	Navigate to Services->Snort->Interface Settings->LAN Rules and enable all, save. 
11.	On the home pfSense page, add Snort alerts, customize as needed.
12.	Several guides say Snort should throw an alert even if a Kali Linux host is detected on the network, before doing anything else, but this could not be replicated.
13.	Simulate some attacker type traffic, verify Snort is logging it.
  a.	Below is result of default nmap scan from Kali-Red
  b.	Easier view from main page, but navigating to snort’s alert page gives many more alerts.
  c.	Suricata added just because it works out of the box (default had no issues…)
<img width="973" height="638" alt="image" src="https://github.com/user-attachments/assets/8fa74b87-26c1-49b5-a1f8-7187c7033ced">
<img width="962" height="442" alt="image" src="https://github.com/user-attachments/assets/084f5fe3-8baf-4c15-beb2-ec360756d85d">

Snort’s Alerts with nmap scan and Nessus scan running:
<img width="894" height="822" alt="image" src="https://github.com/user-attachments/assets/bce5e005-599d-4fea-bbeb-cf7681966302">

<b><u>NMAP (via Kali-Red):</u></b>  
1.	Scanning with nmap will trigger some default alerts.
  a.	sudo apt install nmap
2.	nmap 10.1.1.1/24 (default is fine for testing)
<img width="975" height="917" alt="image" src="https://github.com/user-attachments/assets/164ce852-cacc-4143-8ed9-43a4f08d43ee">
  
<b><u>Ubuntu Web Server:</u></b>
  •	mkdir ~/webserver
  •	cd ~/webserver
  •	nano index.html
</pre>
```
<!DOCTYPE html>
<html><body><h2>Login Page</h2>
  <form action="/login" method="POST">
    Username: <input type="text" name="username"><br>
    Password: <input type="password" name="password"><br>
    <input type="submit" value="Login">
  </form>
</body></html>
```
  •	nano server.py
```
from http.server import BaseHTTPRequestHandler, HTTPServer
import urllib.parse
 
class SimpleHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/":
            with open("index.html", "rb") as f:
                self.send_response(200)
                self.send_header("Content-type", "text/html")
                self.end_headers()
                self.wfile.write(f.read())
 
    def do_POST(self):
        length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(length).decode('utf-8')
        data = urllib.parse.parse_qs(post_data)
        print("=== LOGIN ATTEMPT ===")
        print(f"Username: {data.get('username', [''])[0]}")
        print(f"Password: {data.get('password', [''])[0]}")
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"Login submitted. Check server console.")
 
server = HTTPServer(('0.0.0.0', 8080), SimpleHandler)
print("Server started on http://localhost:8080")
server.serve_forever()
```
<pre>
      <i>o	Note: extra spaces may need to be deleted if copied</i>
  •	python3 server.py
  •	visit: https://10.1.1.25:8080 (check and change IP if not static)
  •	Alternatively, can use the default apache2 web server, but this is simpler for testing. 
<img width="939" height="996" alt="image" src="https://github.com/user-attachments/assets/66d12da3-a4d4-41aa-9a1b-c89984820967">

<b><u>Metasploitable 2:</u></b>
For testing purposes, it used to provide more for the Nessus scanner to reveal. This is a moot issue for the writeup though as we know what it will show.

<b><u>Nessus Installation and Setup (via Kali-Blue):</u></b>
1.	Visit the Tenable Nessus website and download (sign up if you don't have an acct already).
  a.	For Kali, download Linux-Debian-amd64 version
    i.	https://www.tenable.com/downloads/nessus
  b.	cd ~/downloads
  c.	sudo dpkg -i Nessus-10.11.0-debian10_amd64.deb
    i.	match downloaded filename
  d.	systemctl start nessusd
2.	In browser, go to https://localhost:8834 (accept certificate risk)
  a.	Set up Nessus Essentials Plus for Education, get code via .edu email address.
  b.	If it does not initialize with a new login, then do the following (similar in Windows)
    i.	sudo /opt/nessus/sbin/nessuscli adduser
    ii.	systemctl restart nessusd
    iii.	sudo /opt/nessus/sbin/nessuscli fetch --register [code from https://docs.tenable.com/nessus/Content/LicensingRequirements.htm]
        •	This is a ####-####-#### code, not the emailed activation one…
        •	If you have ever used Nessus before, this may be a troublesome endeavor since you have to purchase another free license.
        •	Select I’m an student/educator for activation code.
    iv.	After packages update, then systemctl restart nessusd
3.	With a variety of devices running, Nessus is used to scan the VLAN.
4.	Nessus is self-explanatory once updated plugins function properly.
5.	The scan requires the plugins to initialize and compile, which appears to need more resources as the drive fills up quickly. Without more resources, it will simply reset repeatedly.
<img width="476" height="159" alt="image" src="https://github.com/user-attachments/assets/5eebeb5d-52f2-4775-9939-aa9602f8dc55"><img width="380" height="169" alt="image" src="https://github.com/user-attachments/assets/0d2645a2-6ae8-475e-8a67-3b670a4532e6">

6.	The compiling warning implies it can scan before compiling, but this is not accurate.
<img width="416" height="232" alt="image" src="https://github.com/user-attachments/assets/6c5d5633-afa0-4eba-b138-be37155602be">

7.	After boosting up the resources, nmap had a popup for an initial scan which worked well:
  a.	10.1.1.1/24 – basic scan
8.	After Nessus ran, the detections by Snort on pfSense interestingly showed the Nessus scan trying to brute force the login page of any web servers, which means no need to separately test this as the Nessus scan already attempts it for us.
9.	As expected, the Ubuntu Web Server with a basic python login page had the most real vulnerabilities found. Most of the other vulnerabilities are focused on the devices being responsive to potentially normal activity, which scanners use to identify their targets. The responsiveness of the devices may or may not really matter, since preventing many of those responses could unnecessarily degrade normal network activity and would require regular monitoring or some other solution to prevent from returning.
<img width="931" height="528" alt="image" src="https://github.com/user-attachments/assets/4e4a38ec-d33a-48e0-901f-21e641125475">

<b><u>FINDINGS</u></b>
<b><u>Intrusion Detection/Prevention System (IDS/IPS):</u></b>
  The IDS and IPS solutions can be categorized by their detection methods, signature-based, and anomaly-based. For a home security lab, anomaly-based methods are harder to test and so the focus was on signature-based detection.
  pfSense is an open-source network firewall and router solution allowing for continuous development and frequent updates. Being free, it is a cost-effective security solution for small LAN setups such as a home security lab. The large list of third-party packages such as Snort and Suricata, allow for the easy inclusion of additional security tools.
  Snort and/or Suricata are similar IDS/IPS solutions with Snort being more focused on signature-based network IDS with several changes needed for the settings to work with current pfSense, whereas Suricata appears to be plug-and-play currently for a quick IDS/IPS package solution. Sources vary on which one is best to use, therefore tested both, and once configured, both work equally well for the most part.

<b><u>Nessus vulnerability scanning with alerts:</u></b>
  Nessus vulnerability scanner apparently has a massive plugin base considering initially the usual Kali defaults would not even run it. But once that was addressed, Nessus performed as expected from prior labs using the utility in the past.
  With a variety of devices on the network, Nessus has a diverse range of results, and the scanning done by Nessus shows up almost immediately in the pfSense alerts, especially the more detailed Snort alerts and Suricata alerts, as opposed to the summary on the main pfSense interface. 
  Using another Kali device to simulate another potential attack, with the basic nmap scan as a good illustration of something which should be detected, we can see the alert logs filling up with the expected alerts, similar to the alerts from the Nessus vulnerability scan.
  The combination of a good firewall-based IDS/IPS security solution with a scheduled Nessus scan can alert one to any unusual activity while also ensuring the known vulnerabilities regularly updated by Nessus can be proactively addressed.

<b><u>Conclusion:</u></b>
  While the alerts seen in this home security lab setup are extensive, if this were to be used as a more permanent solution, a significant amount of tuning would be needed to reduce false positives since it seems to detect more than what would be expected by default. 
  Nessus vulnerability scanner is a solid free tool which can act as a further preventative measure to proactively scan your network to ensure known vulnerabilities can be removed to reduce the potential for a breach which may occur too quickly for the IDS alert methodology alone, but further tuning can automatically block many of the expected threats from reaching your network in the first place.
  Therefore, such a combined setup would allow your network to be protected from various detectable activities while also closing the known vulnerabilities before any intruder can even attempt to breach into your home network. This would be much more significant on a larger scale but is still a solid home solution, especially for a safer place to park the various IoT devices that are becoming so prevalent in some homes.
</pre>
