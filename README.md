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

![Wazuh](https://i.imgur.com/bV9bXSR.png) <br>

## TheHive
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
5. Going into **nano /etc/cassandra/cassandra.yaml**, Im changing * *listen_address, rpc_address, and seed_provider > seeds** all to the public ip of the server
