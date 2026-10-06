## Kali Agent Update

# Steps

1. Check the Wazuh Server version, in my case it was v 4.9.2
2. The command:
```bash
curl -so wazuh-agent.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.9.2-1_amd64.deb && sudo WAZUH_MANAGER='192.168.209.10' dpkg -i ./wazuh-agent.deb
```
- Downloads correct version of software from the internet and immediately installs it while pointing it directly to the server, entirely bypassing the outdated Kali repository filters (with 192.168.209.10 being the server IP)

3. The commands:
```bash
sudo systemctl daemon-reload
sudo systemctl restart wazuh-agent
```
- Restarts agent and is basically a refresh

4. Check on Wazuh dashboard and agent version should have changed and if matches the Wazuh server, the "outdated" marker would have disappeared



# Troubleshooting

1. My first attempt I wanted to execute the Kali update remotely through only using the Wazuh dashboard and Ubuntu server

- Click on the Kali VM, select more and upgrade agent 
- On the Ubuntu console:
```bash
sudo /var/ossec/bin/agent_upgrade -l
```
- Checks total outdated agents

- To upgrade specific agent:
```bash
sudo /var/ossec/bin/agent_upgrade -a 002
```

- This however resulted in a fail and the prompt of "The WPK for this platform is not available" basically meaning have to go on the Kali machine to upgrade it rather than try remotely

2. Update the Kali VM manually through its command line terminal to the NEWEST version

```bash
sudo apt-get update
sudo apt-get install --only-upgrade wazuh-agent
```
- Pulls latest update from repositories

```bash
sudo systemctl restart wazuh-agent 
```
- Restarts the wazuh agent to make sure status is updated

```bash
apt list --installed wazuh-agent 
```
- Tells us what version the agent is on and at this point it had not worked and was still on the old one anyway

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.8-1_amd64.deb
```
- Downloads the correct 64-bit Debian package

```bash
sudo WAZUH_MANAGER='192.168.209.10' dpkg -i wazuh-agent_4.14.8-1_amd64.deb
```
- Installs the package with manager IP

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl restart wazuh-agent
```
- Restarts and clears system cache


```bash
apt list --installed wazuh-agent 
```
- Listed the agent as successfully being updated to the latest version which I didn't immediately recognise as a mistake, however due to the Kali VM machine version being newer than the Wazuh server this meant that the Kali was still "outdated" in the sense that the agent versions and server versions both had to match

