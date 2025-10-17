# ELK_SIEM_lab

## Introduction
This SOC Lab Challenge is designed to simulate real-world Security Operations Center (SOC) activities. The goal is to strengthen my hands-on skills in monitoring, detecting, and responding to cybersecurity incidents. Over four weeks, I’ll build a complete SOC environment by configuring SIEM tools, analyzing attacks, and managing incidents to better understand how SOC teams operate in real scenarios.

## Objective
The objective of this challenge is to broaden my practical knowledge of SOC through step-by-step labs that cover:
* Setting up and using the ELK Stack (Elasticsearch, Logstash, Kibana) for log analysis.
* Detecting and investigating brute-force attacks, malware infections, and C2 traffic.
* Creating dashboards, alerts, and automated responses.
* Managing incidents using a ticketing system to simulate SOC workflows.
By completing this challenge, I aim to build real-world SOC experience, strengthen my analytical skills, and demonstrate readiness for an entry-level SOC analyst role.

## Requirment
To complete this Lab, the following virtual machines are used:

| VM                      | RAM   | Processors| Purpose        |
|-------------------------|-------|-----------|----------------|
| ELK Stack               | 8 GB  | 4    | Centralized log collection, analysis, and visualization |
| Mythic C2 Server        | 3 GB  | 2    | Simulate command & control attacks               |
| Fleet Server            | 3 GB  | 2    | Endpoint management and log forwarding           |
| Ubuntu Endpoint         | 3 GB  | 2    | Linux endpoint for log generation and testing    |
| Windows Server          | 3 GB  | 2    | Windows environment for brute-force/malware analysis |
| OS Ticket Server        | 2 GB  | 2    | Ticketing system for incident management         |
| Kali Linux              | 2 GB  | 2    | Attack simulation and penetration testing        |


## NAT Network Setup & Base Snapshot

### 1: Create a NAT Network
1. Open **VirtualBox** and go to **File --> Tools --> Network → NAT Networks**.
2. Click **+** to create a new NAT network.
3. Configure the network:
   - Name: ELK_lab
   - Enable **DHCP**
   - Configure subnet: 10.0.2.0/24
4. Click **OK** to save the network.

### 2: Attach VMs to NAT Network
1. Select a VM → **Settings → Network → Adapter 1**.
2. Choose **Attached to:** NAT Network.
3. Select your NAT network ELK_lab from the dropdown.
4. Repeat for all VMs.

### 3: Power On and Configure VMs
1. Boot each VM and update them.
2. Install the basic utlilites like curl, netplan, openssh-server, openssh-client, default-jre, default-jdk etc for all the VMs.
3. Ensure all VMs can communicate with each other using the NAT network.

### 4: Take a Base Snapshot
1. Right-click the VM → **Snapshots → Take Snapshot** and name it.
2. Repeat for all VMs.


## Setting up the environment

### Elasticsearch on ELK server
1. From the official page of [Elasticsearch](https://www.elastic.co/downloads/elasticsearch) copy the link address for deb x86_64 zip file.
2. Then using the wget command in the elk server download the software.
3. To get the full name of the downloaded package use ls command
4. Install the package using the following command
```bash
sudo dpkg -i elasticsearch-9.x.x.-amd64.deb
```
5. After installation you will be provided with account credentials. Save those.
6. you need to make some changes to the config file by editing the elasticsearch.yml file
```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```
7. Change the network.host to the (ELK Server IP) and save it
8. To enable and start elasticsearch type the below command
```bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
```
<img src=’1’/>

### Kibana on the ELK server
1. From the official page of [Kibana](https://www.elastic.co/downloads/kibana) copy the link address for deb x86_64 zip file.
2. Then using the wget command in the elk server download the software.
3. To get the full name of the downloaded package use ls command
4. Install the package using the following command
```bash
sudo dpkg -i kibana-9.x.x.-amd64.deb
```
5. Update the config file using the following command
```bash
sudo nano /etc/kibana/kibana.yml
```
6. Change the server.host to ELK server ip, server.publicbaseUrl to “http://(ELK Server IP):5601”
7. Now generate encryption key to connect to elasticsearch and generate alerts.
8. First navigate to kibana/bin directory
```bash
cd /usr/share/kibana/bin
ls
```
9. Generate keys by using the following command 
```bash
sudo ./kibana-encryption-keys generate
```

<img src=’2’/>

10. Add the keys to the key store by using the below command
```bash
sudo ./kibana-keystore add xpack.encryptionkey...
```
11. Do the same for all the three keys and then enable and start the service\
```bash
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
sudo systemctl start kibana.service
```
12. Access kibana from the browser using the (ELK Server IP) and port 5601.
13. You will be asked to provide an enrollment token from elasticsearch.
14. Generate it by using the below command
```bash
cd /usr/share/elasticsearch/bin
ls
sudo ./elasticsearch-create-enrollment-token --scope kibana
```
15. Paste the token in kibana.
16. It will ask for a verification code. So generate it by using the below command.
```bash
cd /usr/share/kibana/bin
ls
sudo ./kibana-verification-code
```
17. Paste the code and your elasticsearch will configure and ask for login information
18. The username is “elastic” and the password is the one generated when elasticsearch was installed.

### Connection Fleet Server, Windows Server, Ubuntu Machine to Kibana
1. Navigate to the fleet by clicking on the hamburger icon and scrolling down to management.
2. Click on add fleet server
3. Provide the name and the URL is “https://(fleet server ip):8220”

<img src=’3’/>

4. Click on generate policy.
5. Select the linux.tar command type and copy the command. 

<img src=’4’/>

6. On the fleet server command line paste the command to install elastic agent.

<img src=’5’/>

7. As the agent is installed in the kibana interface you will see a message saying fleet server connected.
8. Click on continue enrolling elastic agent to add Windows Server and Ubuntu Machine.
9. In the add agent section add the name for windows server and click on create policy.

<img src=’6’/>

10. Copy the windows command for the elastic agent
11. Paste it in the Windows Server PowerShell

<img src=’7’/>

12. After the agent is installed on the Windows Server and connected to the Fleet Server you will get the confirmation on the kibana interface.

<img src=’8’/>

13. Similar to adding Windows agent add the Ubuntu agent by clicking on add agent
14. Name the agent and click on create policy
15. Select enroll in Fleet option and the copy the command for linux.tar
16. Paste the command to the Ubuntu command line and add "--insecure" to the end of the command and press enter.

<img src=’10’/>

17. Confirm the agent enrollment and incoming data status.

### Sysmon installation on Windows Server
1. Download Sysmon from [Microsoft Sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon).
2. Download a pre-configured `sysmonconfig.xml` from [Sysmon Modular](https://github.com/SwiftOnSecurity/sysmon-config).
3. Extract the Sysmon files and place the `sysmonconfig.xml` file into the extracted directory.
4. Open PowerShell as an administrator and navigate to the Sysmon directory:
```powershell
cd C:\Users\Downloads\sysmon
```
5. Verify Sysmon is running:
```powershell
Get-Process sysmon64
```
6. Install Sysmon with the configuration:
```powershell
.\sysmon64.exe -i sysmonconfig.xml
```
<img src=’9’/>

### Enable RDP on Windows Server
1. Search for Remote Desktop Setting and click on it
2. Turn on “enable RDP” and  confirm it.

### Integrations of Kibana with Sysmon 
1. Go to integrations on the management section of the Kibana app.
2. Search for “Custom Windows Event Logs” and add the integrations.
3. Name the integration.
4. For channel name go to the Windows Server and access the "event viewer" by searching it on the searchbar.
5. Under the "event viewer" expand the “Application and Services → Microsoft → Windows → Sysmon → Operational”
6. Right click the "Operational" and click on "properties" and you will find the “Full Name” which is the channel name.
7. Copy the name and paste it on kibana integrations.
8. Add the integration to the existing host and choose the name for the windows policy.
9. Finally save and deploy it.
10. In the exact manner add the Windows Defender to the integration.

<img src=’11’/>

<img src=’12’/>








