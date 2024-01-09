# SOC-Automation-Project
Goal is to create a Wazuh Instance with SOAR Integration along with case management using The Hive

## Tools/Applications Used
- [Draw.io](draw.io)
- [Wazuh](https://wazuh.com/)
- [TheHive](https://thehive-project.org/)
- [Windows 10 using Soc Analyst Project](https://github.com/kyhomelab/SOC-Lab/tree/main)
- [Microsoft Azure]


Here is a diagram of the project
<br> <img src="https://i.imgur.com/JdbiEBI.png"> <br>


## Wazuh
1. Utilizing Azure, I created a VM utilizing Ubuntu Server 20.04
   - Configured with 2 vcpus and 8gb of memory
   - I gave it 64gb of storage
2. Connected to the Ubuntu instance through SSH using Azure CLI
3. Under Bash ran:
```bash
# created a new password
sudo passwd root
```
```bash
# became root user utilizing new password
su -
```
```bash
# updated the index files
apt-get update && apt-get upgrade
```
```bash
# installed the wazuh repository
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```
4. After installing the wazuh repository, your are given the url **https://wazuh-dashboard-ip:443**, user and password
> Networking is not my strongsuite and it took me about 30 min to figure out how to allow my public ip to inbound port rule

![Wazuh](https://i.imgur.com/bV9bXSR.png)
