# VirtualBox Networking Configuration

# Adapter 1 - NAT
Purpose: Internet access for Kali tools and updates

# Adapter 2 - Host-Only
Purpose: Internal lab network for attacking other VMs (E.g. Windows) 

# Verification
Inside Kali:
ip a

Expected:
- eth0 -> NAT
- eth1 -> Host-Only
