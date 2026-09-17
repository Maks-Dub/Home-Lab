# Main Goals

1. Vulnerable target VM
- Goal is to get a VM with known, well-documented vulnerabilities that I can exploit in a controlled way such as Metasploitable2 
2. Simulated attack from Kali
- Need to pick a well understood attack which I can document
3. Analyse detection capabilities and gaps 
- What Wazuh logs could and coulnd't see

## Actual Setup

1. Download the Metasploitable VM files from:
https://sourceforge.net/projects/metasploitable/postdownload

In Oracle Virtual Box:
- 512MB of memory
- OS -> Linux
- Distribution -> Ubuntu
- Version -> Ubuntu 32-bit
- Hard-Disk -> Existing file that was just downloaded

2. Logging in
Default credentials for Metasploitable are:
- msfadmin/msfadmin

3. Network
Checked network using
```bash
ifconfig
```
- Confirmed IP to be 192.168.209.11

SSH into my Ubuntu Wazuh server and checked ping connection from the Wazuh server to the Metasploitable VM which was a success

4. Last checks
On Metasploitable used:
```bash
uname -a
```
- Gave us the OS version which is Linux 2.6.24, from April 2008
- This does mean that Metasploitable cannot run a Wazuh agent but can still rely on network-based and Kali-side visibility instead

```Zsh
ping -c 192.168.209.11
```
- Pinged Metasploitable from Kali which was a success
