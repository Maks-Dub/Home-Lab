## Windows Virtual Machine Setup (Oracle Virtual Box)

# Download
- Downloaded Windows 10 file from Microsoft Azure (Version 22H2)

# VM Creation
- OS Type: Microsoft Windows
- Version: Windows 10 (64 bit)
- Attached .vdi file
- Created separate VM Gmail account and new Windows account to run the lab

# Tools 
- Chrome 
- VS Code 
- Git
- Wireshark
- Nmap
- Powershell 7
- 7-Zip 
- Python
- Burp Suite
- Notepad++

# Firewall
Added several firewall changed
- Enabled ICMP Echo Requests (Pings) 
- Enabled ALL File and Printer Sharing rules, after which new firewall snapshot created

# Snapshots
- Fresh Install
- Base Tools Installed - Clean
- Fully Setup
- Firewall Settings Changed
- Wazuh Agent Active

# Agent setup for Wazuh
1. Endpoints tab of Wazuh -> Deploy new agent
2. Ceck that Windows can see the server (Ping IP)
3. Copy install command from Wazuh
4. Paste command in an Administrator Powershell window
5. Start the service using:
```powershell
NET START WazuhSvc 
```
6. Confirm agent shows active in the dashboard