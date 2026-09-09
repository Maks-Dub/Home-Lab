# Learning Wazuh Alerts and Rules Timeline


# First steps
- Trigger a log on failure alert to explore further 
- Highest one I managed to trigger was using Windows powershell commands and forced Windows to attempt to log in as administrator with the incorrect password 10 times in quick succession
- This triggered an alert level 10, which is just one under high-severity in Wazuh and triggered rule "60204" 
Using:
```bash
sudo grep -rB 5 -A 15 'id="60204"' /var/ossec/ruleset/rules/
```
This command searches recursively through /var/ossec/ruleset/rules for rule "60204" as mentioned previously 
- This produced several very useful information such as:
    - if_matched_group: authentication_failed - this rule watches for a group of underlying failed-login events, not just one specific rule ID directly
    - MITRE ATT&CK mapping: T1110 - Official MITRE technique ID for "brute force" confirming Wazuh correctly classified my rapid repeated failures as brute-force pattern behaviour not just random error

- Next steps after this would be:
    - Adjust rule threshold
    - Write a custom rule that goes further, e.g. auto-blocking the source IP after the rule fires (However my alert was originally triggered via Windows VM so will have to attack Windows from the Kali VM instead OR alternatively could set up a block for around 60 seconds instead)

