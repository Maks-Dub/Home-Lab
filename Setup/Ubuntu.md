# Wazuh SIEM Setup on Ubuntu Server using Oracle Virtual Box

# Goal
- Create a Wazuh SIEM on Oracle Virtual Box
- Install a Wazuh agent on both the Windows and Kali VMs

This covers the final successful setup process, see Troubleshooting-Ubuntu.md for what went wrong along the way and command-reference.md for full command references and explanations

# VM Specifications

| Resource | Minimum Used |
|----------|-------------|
| Disk     | 50 GB       |
| RAM      | 8 GB        |
| CPU      | 4 cores     |

# Network Adapters
Crucial to configure BOTH network adapters in Virtual Box before booting the VM
- Adapter 1: Host-Only, for Host <-> VM communications
- Adapter 2: NAT (Internet Access)
Both adapters should have an IP automatically via DHCP, no netplan configuration should be necessary if configured correctly from the start

# OS
- Ubuntu 22.04.5 LTS Server (Newer version 26.04 caused issues see troubleshooting notes on why)

# Steps

1. Confirm Networks are up
```bash
ip a
```
Confirm both interfaces have IP addresses assigned

```bash
ping -c 3 google.com
```
Confirm internet connection before proceeding using google.com

2. Update the base Ubuntu system
```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

3. Download the Wazuh installer
```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh
```

4. Run the all-in-one installer
```bash
sudo bash wazuh-install.sh -a
```

This installs the Wazuh indexer, manager and dashboard together on the VM. This took around 15 minutes on a correclty configured VM but does depend on internet speeds.

5. Save Credentials
- On success the script prints a summary block with dashboard login credentials
User: admin
Password: <generated password>
(On screenshots mine are hidden)
- This MUST be saved immediately as it is not shown again 

6. Access the dashboard
- Alongisde the credentials you are also given a link
- Clicking through the self-signed certificate warning you may log in with the given credentials 

7. After install
- Take a Virtual Box snapshot of this working state before making any further changes 


# Installing Agents for the SIEM

Windows
1. Endpoints tab of Wazuh -> Deploy new agent
2. Ceck that Windows can see the server (Ping IP)
3. Copy install command from Wazuh
4. Paste command in an Administrator Powershell window
5. Start the service using:
```powershell
NET START WazuhSvc 
```
6. Confirm agent shows active in the dashboard

Kali
1. Confirm Kali can see the server with ping
2. Generate a new separate command from the Wazuh server this time for Linux
3. Run the install command in Kali's terminal with:
```bash
sudo 
```
4. Enable and start the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```
5. Confirm on Ubuntu and check Wazuh dashboard for successful connection


# Snapshots
- Fully-Setup
- 2 Agents Active - Disk Fixed

# Quality-Of-Life
- Decided to change the ubuntu terminal colours from black and white to actually enabling the built in colour options
Used:
```bash
grep -n "force_color_prompt" ~/.bashrc
``` 
- This shows the current colour settings
```bash
sed -i 's/#force_color_prompt=yes/force_color_prompt=yes/' ~/.bashrc
source ~/.bashrc
```
- Overwrites the colour prompt to yes instead

