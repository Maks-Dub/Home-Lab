# Kali Linux Virtual Machine Setup (Oracle Virtual Box)

# Download
- Downloaded Kali Virtual Box file (ZIP Version)
- Extracted to obtain Virtual Disc Image file

# VM Creation
- OS Type: Linux
- Version: Oracle Linux (64 bit)
- Attached existing .vdi file
- Booted successfully using default Kali/Kali credentials

# Networking
- Adapter 1: NAT (Internet Access)
- Adapter 2: Host-Only Adapter (Lab Network)
- Verified interfaces inside Kali using 'ip a' command

# Updates
Ran:
```bash
sudo apt update && sudo apt upgrade -y 
```
(Downloads all updates and bug fixes)

## Snapshots
- Fully Setup
- Post-Install
- Wazuh Agent Active

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