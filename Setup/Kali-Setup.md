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
sudo apt update && sudo apt upgrade -y (Downloads all updates and bug fixes)

## Snapshots
- Fully Setup
- Post-Install
