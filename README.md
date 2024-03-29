# SOC-Automation-Project
Goal is to create a Wazuh Instance with SOAR Integration along with case management using The Hive

## Tools/Applications Used
- [Draw.io](draw.io)
- [Wazuh](https://wazuh.com/)
- [TheHive](https://thehive-project.org/)
- [Shuffle](https://shuffler.io/)
- [Virus Total](https://www.virustotal.com/gui/home/upload)
- [Windows 10 using Soc Analyst Project](https://github.com/kyhomelab/SOC-Lab/tree/main)
- [Microsoft Azure](https://azure.microsoft.com/en-us)


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

![Wazuh](https://i.imgur.com/bV9bXSR.png) <br>

## TheHive
> Configure a VM with at least 4cores of cpu. With ElasticSearch you aren't able to start the service if you dont have enough cpu. I reconfigured 4 vms trying to fix this.

1. Same first step of creating Ubuntu VM with same configurations
2. Connected to the Ubuntu instance through SSH using Azure CLI
3. Changed root password and ran updates
4. Under Bash ran these in order:
```bash
# Install the dependencies
apt install wget gnupg apt-transport-https git ca-certificates ca-certificates-java curl  software-properties-common python3-pip lsb-release
```
```bash
# Install Java
wget -qO- https://apt.corretto.aws/corretto.key | sudo gpg --dearmor  -o /usr/share/keyrings/corretto.gpg
echo "deb [signed-by=/usr/share/keyrings/corretto.gpg] https://apt.corretto.aws stable main" |  sudo tee -a /etc/apt/sources.list.d/corretto.sources.list
sudo apt update
sudo apt install java-common java-11-amazon-corretto-jdk
echo JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto" | sudo tee -a /etc/environment 
export JAVA_HOME="/usr/lib/jvm/java-11-amazon-corretto"
```
```bash
# Install Cassandra
wget -qO -  https://downloads.apache.org/cassandra/KEYS | sudo gpg --dearmor  -o /usr/share/keyrings/cassandra-archive.gpg
echo "deb [signed-by=/usr/share/keyrings/cassandra-archive.gpg] https://debian.cassandra.apache.org 40x main" |  sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
sudo apt update
sudo apt install cassandra
```
```bash
# Install ElasticSearch
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch |  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" |  sudo tee /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install elasticsearch
```
```bash
# Install TheHive
wget -O- https://archives.strangebee.com/keys/strangebee.gpg | sudo gpg --dearmor -o /usr/share/keyrings/strangebee-archive-keyring.gpg
echo 'deb [signed-by=/usr/share/keyrings/strangebee-archive-keyring.gpg] https://deb.strangebee.com thehive-5.2 main' | sudo tee -a /etc/apt/sources.list.d/strangebee.list
sudo apt-get update
sudo apt-get install -y thehive
```
<br>

5. Going into "nano /etc/cassandra/cassandra.yaml", Im changing "listen_address, rpc_address, and seed_provider > seeds" all to the public ip of the server
6. Restart the server using:
```bash
systemctl stop cassandra.service
```
```bash
rm -rf /var/lib/cassandra/*
```
```bash
systemctl enable cassandra.service
```
```bash
systemctl start cassandra.service
```
<br>

7. Now to configure Elastic Search which will query data. We need to go to the config files:
```bash
nano /etc/elasticsearch/elasticsearch.yml
```
8. Within the config, uncommented * *cluster name, node.name, network host (changed it to public ip of server)* *. Save config and exit. Run cmd to start it:
```bash
systemctl start elasticsearch
```
- Check to make sure both Cassandra and Elasticsearch are running
```bash
systemctl status elasticsearch
```
```bash
systemctl status cassandra.service
```

9. Now we need to configure theHive. We need to ensure theHive user and group have access to a file path
```bash
chown -R thehive:theehive /opt/thp
```
10. Now we need to go into the config files:
```bash
nano /etc/thehive/application.conf
```
- Changed the hostname default ip to public ip of the server
- Changed the application.baseUrl to the public ip of the server
- Save and exit the config
11. Now start theHive:
```bash
systemctl start thehive
```
```bash
systemctl enable thehive
```
```bash
systemctl status thehive
```
- Ensure all 3 services are running
- You'll want to navigate to: http://IP_ADDRESS:9000

<img src="https://i.imgur.com/RTReWi6.png" width="45%" height="45%">

## Configuring Dashboards
1. Navigating to the Wazuh Dashboard and logging in (I recommend doing this on the Windows VM)
2. Click on Add Agent and you should get this page here:
<br> <img src="https://i.imgur.com/0fZuzN8.png" width="45%" height="45%"> <br>
- Choose Windows > Enter Wazuh Public Server Address > and you will get a command to run in powershell (example):
```bash
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.1-1.msi -OutFile ${env.tmp}\wazuh-agent; msiexec.exe /i ${env.tmp}\wazuh-agent /q WAZUH_MANAGER='1.1.1.1' WAZUH_AGENT_NAME='mydfir' WAZUH_REGISTRATION_SERVER='1.1.1.1'
```
- Run this in the Windows 10 powershell with admin privileges followed by:
```bash
net start wazuhsvc
```
- Now when I go back to the dashboard it shows a connection, and I can see this dashboard:
<br> <img src="https://i.imgur.com/qDmbJyT.png" width="45%" height="45%"> <br>
> The dashboard is typically blank but this is running on my [SOC Lab](https://github.com/kyhomelab/SOC-Lab/tree/main) so theres events already)
- Utilizing [Mimikatz](https://github.com/gentilkiwi/mimikatz/releases/tag/2.2.0-20220919) I'm going to set up custom alerts
3. After downloading Mimikatz, and extracting it, I use powershell to run it. Wazuh picks up the event shown here:
<br> <img src="https://i.imgur.com/YZG1sIH.png" width="45%" height="45%"> <br>
4. From the Home Dashbolard go to the dropdown arrow > Management > Rules > Manage Rules Files > Custom Rules > Edit (pencil icon) > and paste in this rule above **</group>** :
  ```bash
  <rule id="100003" level="15">
    <if_group>sysmon_event1</if_group>
    <field name="win.eventdata.originalFileName" type="pcre2">(?i)mimikatz\.exe</field>
    <description>Mimikatz Usage Detected</description>
    <mitre>
      <id>T1003</id>
    </mitre>
  </rule>
  ```
- After saving and reataring the mananger, I restart Mimikatz, go to the dashboard and now it has a description and event ID:
<br> <img src="https://i.imgur.com/ukqYkPK.png" width="45%" height="45%"> <br>

## Shuffle
1. Naviagting to [Shuffle](https://shuffler.io/) I create an account, and create a new workflow
2. Utilizing a webhook, and the URI, I will add it to the Wazuh CLI utilizing this line:
```bash
<integration>
  <name>shuffle</name>
  <hook_url>WEBHOOK_URL</hook_url>
  <rule_id>100003</rule_id>
  <alert_format>json</alert_format>
</integration>
```
- After, I restart the services, stop and restart mimikatz, and looking on Shuffle, it comes through:
<br> <img src="https://i.imgur.com/snIP8fE.png" width="45%" height="45%"> <br>
3. Created an account on [Virus Total](https://www.virustotal.com/gui/home/upload) and created an account to get my api
4. Back to Shuffle, on my workflow I added the Virus Total app, authenticated using my API, and re ran the exececution so that Virus Total is now pushing
> Setting up the SOAR platform I recieve the Wazuh alert, utilize Regex to parse out the 256 hash, and use Virus Total to check the reputation

## theHive
1. Log into theHive instance, I created a personal analyst account as well as a SOAR user account. I created an API Key for the SOAR user
2. Back on Shuffle, I added theHive app to the workflow, added my API Key as well as the public IP of theHive
   - The action I set was for an alert with a custom date, description, summary, tags, and title
   - Running the workflow again, it successfully goes through all the way to theHive and I get this Alert:
<br> <img src="https://i.imgur.com/zLlTz0R.png" width="45%" height="45%"> <br>
<br> <img src="https://i.imgur.com/qokbn1o.png" width="45%" height="45%"> <br>
