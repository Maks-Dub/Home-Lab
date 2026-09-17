# Detection

Since Metasploitable has no Wazuh agent, personal logs much be checked for evidence of the attack

```bash
tail -30 /var/log/auth.log
```
- Looks through Metasploitable logs
- Nothing mentions port 21, FTP or vsftpd. The backdoor exploit does not log through the standard auth.log mechanism it is a backdoor bypassing normal authentication, so the exploit is invisible in this particular log file

```bash
sudo cat /var/log/vsftpd.log
```
- vsftpd specific log file
- Last 3 current date logs are actual attack connections from Kali hitting the FPT service
- The vsftpd backdoor exploit registered as bare CONNECT events in vsftpd.log, bypassed the authentication logging shown in normal login attempts, showing that the backdoor leaves only a raw connection timestamp as evidence
- A good example of why legacy systems without proper monitoring are dangerous and why backdoors specifically evade conventional logging

- Wazuh could not be connected to Metasploitable as an agent due to it being a legacy system

