# Custom Wazuh Rule Creation

## Step 1:
- The new rule I'm going to make is a new alert for a new user account being created on the windows machine, in a real life scenario this would be used in the sense when an attacker has access to a machine and decides to create a backdoor account for later use

1. Open local_rules.xml on the manager
- I chose to SSH into my Wazuh manager, then open the custom rule files:

```bash
sudo nano /var/ossec/etc/rules/local_rules.xml
```

## Step 2:
- Add the custom rule:

```xml
  <rule id="100010" level="8">
    <if_sid>60109</if_sid>
    <field name="win.system.eventID">^4720$</field>
    <description>New local user account created on $(win.system.computer) - possible persistence attempt</description>
    <mitre>
    <id>T1136.001</id>
    </mitre>
  </rule>
```
- In my case there was a custom rule already in place, so therefore I did not need to create new <global> tags and simply used the xml script above
- The rule states, if a Windows logon-related event comes in AND its event ID is exactly 4720 (account created) raise a level 8 alert with a clear description

## Step 3:
- Save the file with Ctrl O, Enter, Ctrl X
- Validate the rule has no syntax errors before restarting anything

```bash
sudo /var/ossec/bin/wazuh-logtest
```
- If there is a syntax error, an error would show up, otherwise it is blank

## Step 4:
- Restart the Wazuh Manager

```bash
sudo systemctl restart wazuh-manager
sudo systemctl status wazuh-manager
```

## Step 5:
- Trigger an event on the Windows VM, open powershell as admin and run:
```powershell
net user backdoor Test1234! /add
```
- This creates a new local account called "Backdoor" - simulating what an attacker creating a persistence would look like

## Step 6:
- Confirm the alert fires in Wazuh
- Alert should be rule ID 100010 with the description we set

## Step 7:
- Remove the test account once confirmed
```powershell
net user backdoor /delete
```

## TROUBLESHOOTING
- For me an event was fired but not the custom one I had created, it matched an already built in rule (60109), this was because my rule meant "only evaluate my rule if 60106 already matched first" but 60106 is "Windows logon success" which is a different event category entirely. Since 60106 didn't fire for an account-creation event my rule condition were never checked, the fix however was simple:

```bash
sudo nano /var/ossec/etc/rules/local_rules.xml
```

then change:
```xml
<if_sid>60106</if_sid>
```

to:
```xml
<if_sid>60109</if_sid>

