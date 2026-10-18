# Auto-block via Kali-attacks-Windows brute force

- Simulated a brute-force login attack from the Kali VM against the Windows VM, captured the resulting failed login events in Wazuh and then configured Wazuh Active Response to automatically trigger a Windows Firewall block against the attacking IP, once the brute-force detection rule fires it will be lifted automatically after a timeout period

## Step 1
- On the Windows VM opened Network and sharing centre --> Advanced Audit Policy Configuration --> Logon/Logoff --> Audit Logon, and set iot to log both Success and Failure. This is what geenrates our event ID every time a login attempt fails which is the event Wazuh needs to see to detect the brute force

## Step 2
- Check Windows Defender Firewall allows inbound SMB (445), if it's already blocked at the Windows firewall, Kali's attempts never reach the point of generating an event, so nothing for Wazuh to detect
- Control Panel --> Network and Sharing Centre --> Advanced Sharing Settings, turn on Network discovery and File and Printer Sharing for current Network profile. This is what exposes SMB (port 445) for Kali to attack

## Step 3
- Right click a folder (e.g. create one called Test-Share) go to Properties --> Sharing --> Share and add a specific local user account with read/write permission and click share. This gives Kali an actual SMB login to brute-force against - without a shared resource, there's nothing for a login attempt to authenticate to

## Step 4
- Create a small password list on the Kali VM e.g. Passwordlist.txt and include actual password of windows VM inside (Ideally not as first option)
- On Kali use the Metasploit tool 

```bash
msf console -q
```
- Enters the metasploitable tool

```bash 
use auxiliary/scanner/smb/smb_login
```
- Loads the SMB login brute-force module

```bash
set RHOSTS 192.168.209.4
```
- Sets the target IP (The Windows VM)

```bash
set SMBUser
```
- Sets the username to attempt to login

```bash
set PASS_FILE /home/kali/passwordlist.txt
```
- Points the module at the wordlist of passwords to try

```bash
set VERBOSE true
```
- Makes the module print each individual attempt success/fail instead of as a summary

```bash
run
```
- Executes the module

## Step 5
- Confirm Wazuh detected it, check the Wazuh dashboard for events concerning failed logon attempts

## Step 6
- Configure Wazuh to react to the detection, at the moment Wazuh just logs and alerts it doesn't do anything about it. Need to add an active response block to the manager's config that says when rule (60204) fires run a command to block that IP on the Windows firewall

- On the Wazuh Manager:
```bash
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak
```
- Backs up original config

```bash 
sudo sed -n `1.100p` /var/ossec/etc/ossec.conf
```
- Lets us read the config before making any changes

```bash
sudo nano /var/ossec/etc/ossec.conf
```
- Open the config for editing

'
- Inside the actual config file we first need to whitelist ourself (or me in this case) so that I wont ever get blocked. This is added inside the existing <global> module block after <update_check>yes</update_check> and before </global>, I added a new check block at the top but there might already be an exisitng one near the bottom delete that one.

```xml
<update_check>yes</update_check>
<white_list>192.168.209.10</white_list>
<white_list>127.0.0.1</white_list>
```
- Whitelist config specifically for my virtual machines

- Then find the <remote> and </remote> tags and under it place the following:

```xml
<command>
    <name>win-firewall-block</name>
    <executable>netsh.exe</executable>
    <timeout_allowed>yes</timeout_allowed>
</command>

<active-response>
    <command>win-firewall-block</command>
    <location>local</location>
    <rules_id>60204</rules_id>
    <timeout>300</timeout>
</active-response>
```
- This deinfes a custom active-response command that runs netsh.exe on the agent, then ties it to Wazuh rule 60204 so that whenever the rule fires Wazuh automatically triggers the command to block the attacking IP on the Windows Firewall for 300 seconds

```bash
sudo systemctl restart wazuh-manager
```
- Restart Wazuh

```bash
sudp systemctl status wazuh-manager
```
- Double checks the status of the manager

## Step 7 (Final)
- Re-running Metasploit brute force from Kali and checking whether the Kali IP actually gets firewalled this time

- Re-run exact same Metasploit commands (If Kali was left should just be able to type the run command)
- Check Wazuh dashboard, find a rule mentioning netsh
- Check Windows firewall, should be a new Wazuh rule
- Use same Metasploit commands should return 0 credentials were successful
- Wait 5 minutes and it should allow it again


## TROUBLESHOOTING

1. First issue was I initially also checked RDP on the Windows firewall and found that my Windows VM did not support remote desktop so had to use solely SMB
2. At step 4 I originally ran Hydra before I found out that Windows 10 Home doesn't support incoming RDP, when I tried switching to SMB I found that Hydra's SMB module is old and built primarily around SMBv1 which Windows 10 disables by default 
3. Next step was to try nmap, this worked in the sense that it confirmed port 445 was open and reachable but that was its sole purpose
4. Installed a tool built for smb rather than nmap
```bash
sudo apt install netexec
```
- The tool hit an internal bug, the tool returned 0 hosts before ever reaching my Windows VM 
5. Using the medusa tool instead, came back failed aswell as it was similarly old like Hydra, final move was to use Metasploit
```bash
medusa -h 192.168.209.4 -u virtu -P /home/kali/passwordlist.txt -M smbnt
```
6. A simple error I faced at one point with the Windows Command Prompt was I was entering my User PIN rather than my actual Windows Password
7. Another problem was my Test-Share file was not appearing when using the `net share` command in windows, this was fixed by:
```powershell
New-SmbShare -Name "Test-Share" -Path "C:\Users\virtu\Documents\Test-Share" -FullAccess virtu
```
- This shared the file properly (basically a work around if going into file properties and sharing it from there doesn't work)

