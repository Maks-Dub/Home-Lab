# Troubleshooting Ubuntu: Wazuh SIEM Install

Full installation took 3 attempts, each involving a fresh Ubuntu re-install and re-download. This troubleshooting file captures what went wrong at each stage and how it was resolved.

# Attempt 1 (Fail)
- Only one network adapter configured, VM had no internet access
- Then Wazuh used only the Host-Only network to access the internet, so changed to using the NAT route, which is why ping google.com kept failing even after adding a second adapter
- My first attempt at downloading Wazuh I tried using the distributed method so installing all components separately, this ended up failing, with what appeared to be a version-related error
- After this, I attempted to install the Wazuh packages from the internet 
- Ran into VirtualBox clipboard sharing not working, attempted a fix via Guest additions without success, where I ultimately decided on a fresh restart and to switch to Ubuntu 22.04 instead
- Attempt 1 was performed on Ubuntu 26.04. The Wazuh distributed installer encountered a version-related error, likely due to 26.04 being a recently released version not yet fully validated/supported by Wazuh's installer at the time. This contributed to the decision to switch to Ubuntu 22.04 LTS for subsequent attempts.

# Attempt 2 - Ubuntu 22.04 (Fail)
- Downloaded Ubuntu 22.04 from offical website and setup base logins before touching Wazuh
- Edited netplan manually (In hindsight, simply configuring the adapters in VirtualBox first would have avoided this step entirely)
- Confirmed a proper default route was now in place
- Used the update command to update Ubuntu 
- Ran the Wazuh all-in-one installer this time round where I ran into a new error: system did not meet minimum hardware requirements
    - Fix: Increased VM resources, RAM was already 4096MB; increased CPU to 4 cores
- Re-ran the installer, progressed further but failed at the dashboard installation step
    - Cause #1: Disk Space: Dashboard log showed insufficient storage, freed space using relevant commands 
    - Cause #2: Port Conflicts: Re-running the installer failed again, this time due to ports already in use (1515,55000) by leftover processes from the earlier failed attempt. Used relevant commands to kill their processes
    - Cause #3: Corrupted package state: Wazuh claimed it was missing critical files, attempted to include specific file but returned nothing at all meaning the directory was empty or didn't exist, leading to a corrupt package installation
- Decision, rather than continue dealing with an inconsistent package state simply cut losses and start fresh

# Attempt 3 - Ubuntu 22.04 (Success)
- Retry final Ubuntu fresh reinstall now knowing exactly what needs to be done:
    - VM disk needs sufficient capacity
    - Configure BOTH network adapters on the VM settings rather than messing with netplan
    - Meet hardware requirements
    - Each failed attempt before left broken package states, need to do it all in one go
This time:
    - Disk = 50GB
    - RAM = 8GB
    - CPU = 4 Cores

- Google ping responds (Internet connected)
- Ubuntu updates everything successfully with no errors
- Reboot
- Ran the Wazuh all-in-one installer
- Installation completed successfully, admin credentials and dashboard URL printed at the end, dashboard confirmed accessible from host browser


# Key lessons
- Get VM specs correct before installing
- Configure both networks direclty in Virtual Box
- Avoid new Ubuntu releases
- Repeated fails leave debris 
- Read logs not just terminal errors

# Setting up Agents
- Ubuntu machine has a DHCP IP, for an agent to work it requires a static IP which I changed using netplan
- Windows relayed it was all successful (Confirmed in windows-side agent log so was NOT a windows issue) however did not show up on Wazuh dashboard
    - Windows log shows as successfully connected
    - Issue was with disk storage space, I gave Ubuntu 50GB via Virtual Box but Ubuntu was not making use of it fully
        - Had to grow the filesystem to match the new logical volume size
        - Verify it
        - Restart the affected services 
        - Check agent status again and all was up and running
