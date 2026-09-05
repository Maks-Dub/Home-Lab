# Command Reference: Wazuh SIEM Install

Full list of commands used across all attempts, with explanations. See Troubleshooting-Ubuntu.md for context of when/why each was used and Ubuntu.md for final working sequence.

# Networking
Editing Netplan 
```bash
sudo nano /etc/netplan/*.yaml
```
Opens the netplan network configuration file for editing
```bash
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.209.10/24
      gateway4: 192.168.209.1
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1]
```
Manually assigns a static IP, gateway and DNS servers to the enp0s3 interface, rather than relying on DHCP

```bash
sudo netplan apply
```
Applies the netplan configuration changes live, without requiring a reboot

# Checking connectivity
```bash
ip a
```
Displays IP addresses assigned to each network interface, used to confirm both adapters have valid IPs 

```bash
ping -c 3 google.com
```
Sends 3 ICMP packets to google.com to test Internet and DNS resolution

```bash
ip route
```
Displays the system's routing table 

```bash
sudo ip route del default via 192.168.209.1 dev enp0s3
```
Used to delete specific default route

# System preparation
```bash
sudo apt update
```
Refreshes apt's local package index with the latest available package versions from configured repositories, does not install anything itself

```bash
sudo apt install curl apt-transport-https unzip -y
```
Installs three utlitiy packages 

```bash
sudo apt install build-essential dkms linux-headers-$(uname -r) -y
```
Installs toolchain needed to compile and manage kernel modules, used ahead of installing VirtualBox Guest additions

```bash
sudo apt update && sudo apt upgrade -y
```
Refreshed the package index then installs any available upgrades chained with && so upgrade only runs if update succeeds 

# Wazuh Installation
```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
```
Downloads the Wazuh install script
```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh
```
Same as above but version 4.9, version used for successful install

```bash
sudo bash wazuh-install.sh -a
```
Runs the install script via bash

```bash
sudo bash wazuh-install.sh -a -o
```
Same as above but with -o meaning overwrite, forces script to wipe any existing Wazuh installation

# Diagnosing Failures
```bash
sudo grep -i "dashboard" /var/log/wazuh-install.log
```
Searches the log for lines mentioning "dashboard"

```bash
sydi grep -i "port" /var/log/wazuh-install.log
```
Searches the log for lines mentioning "port"

```bash
sudo apt clean
```
Deletes all cached .deb package files 

```bash
sudo apt autoremove -y
```
Removes packages that were installed as dependencies for something else but are no longer needed

```bash
df -h
```
Displays disk space 

```bash
sudo ss -tulpn | grep -E '1515|55000'
```
Lists all processes containing the relevant port numbers

```bash
sudo kill -9 53226
```
Forcibly terminates the process with PID 53226

# Cleanup / Reset
```bash
sudo systemct1 stop wazuh-manager wazuh-indexer wazuh-dashboard filebeat 2>/dev/null
```
Stops all four Wazuh related services at once

```bash
sudo apt purge wazuh-manager wazuh-indexer wazuh-dashboard filebeat -y
```
Completely removes all four Wazuh packages, including their configuration files

```bash
sudo rm -rf /var/ossec
sudo rm -rf /etc/filebeat
sudo rm -rf /usr/share/filebeat
sudo rm -rf /var/lib/wazuh-indexer
sudo rm -rf /etc/wazuh-indexer
sudo rm -rf /var/log/wazuh-indexer
sudo rm -rf /etc/wazuh-dashboard
sudo rm -rf /usr/share/wazuh-dashboard
```
Manually deletes leftover Wazuh directories

```bash
sudo dpkg -L wazuh-manager | grep keystore
```
Lists every file the wazuh-manager package's manifest says should be installed

```bash
sudo apt purge wazuh-manager -y
```
Purges just the wazuh-manager package specifically 

```bash
sudo rm -rf /var/ossec
```
Forcibly and permanently deletes Wazuh's main installation directory 

# Commands used when adding agents 
```bash
sudo pvdisplay
```
Displays details about LVM physical volumes, used when investigating disk space issues before expanding it

```bash
sudo lsblk 
```
Lists all block devices (disks, partitions and logical volumes) on the system, shown as a hierarchical tree to figure out LVM structure

```bash
sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```
Extends the logical volume to use all remaining free space in the volume group (In the troubleshooting this made the volume grow from 24GB to the full 48GB)

```bash
df -h
```
Mentioned earlier however is used to verify disk space usage

```bash
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-manager
```
Restarts affected services, used in my troubleshooting when increased disk volume

```bash
sudo /var/ossec/bin/agent_control -i 001
```
Checks agent status

