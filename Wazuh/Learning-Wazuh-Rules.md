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

# Changing the rule threshold
- First step is finding the $MS_FREQ variable and what it is set to
    - This variable is used as an alternative to hardcoding integers directly into every rule that needs a frequency threshold, Wazuh defines it once as a named variable (MS_FREQ) that was sensitivity of ALL the related rules can be changed at once
- In rule 60204 it showed up as:
<rule id="60204" level="10" frequency="$MS_FREQ" timeframe="240">

```bash
sudo grep -rn "MS_FREQ" /var/ossec/ruleset/
```
- Finds the $MS_FREQ value

<var name="MS_FREQ">8</var 

- Appears most frequently meaning the variable value is 8

So for rule 60204, this means 8 matching failed-login events within 240 seconds triggers the level 10 alert.
- For home-lab learning purposes, I will drop the MS_FREQ to 4, which would trigger easier and quicker

```bash
sudo grep -rn '<var name="MS_FREQ">8</var>' /var/ossec/ruleset/rules/
``` 
- Searches for a specific exact text pattern across Wazuh's rule files so we know exactly which files need editing

```bash
sudo find /var/ossec/ruleset/rules/ -name "*.xml" -exec sed -i 's/<var name="MS_FREQ">8<\/var>/<var name="MS_FREQ">4<\/var>/' {} \;
```
- Finds every .xml file in the rules folder and for each individual one run the same find and replace on it

```bash
sudo grep -rn '<var name="MS_FREQ">' /var/ossec/ruleset/rules/
```
- To check all the rules hav ebeen changed from 8 -> 4 which was a success

```bash
sudo systemctl restart wazuh-manager
```
- Restart Wazuh to apply

# Post Rule-Change (SUCCESS!)
- Logged into Wazuh
- Powered on Windows-VM and triggered 4 failed login attempts
- Found rule 60204, in the events tab which triggered successfully after 4 attempts this time rather than the original 8

# Troubleshooting

```bash
sudo sed -i 's/<var name="MS_FREQ">8<\/var>/<var name="MS_FREQ">4<\/var>/' /var/ossec/ruleset/rules/*.xml
```
- This was original command I attempted to first use instead of the "find" line, this failed because bash expands *.xml using the user's own permissions before sudo applies - since the user couldnt list that directory, the wildcard passed through literally instead of matching files; find avoided this by running the directory search itself under sudo 

