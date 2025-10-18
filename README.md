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
<img src=’https://github.com/TanunM/ELK_SIEM_lab/blob/main/Gallery/1.%20elasticsearch.png’/>

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

<img src=’https://github.com/TanunM/ELK_SIEM_lab/blob/main/Gallery/2.%20kibana.png’/>

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

<img src=’https://github.com/TanunM/ELK_SIEM_lab/blob/main/Gallery/3.%20add%20fleet.png’/>

4. Click on generate policy.
5. Select the linux.tar command type and copy the command. 

<img src=’https://github.com/TanunM/ELK_SIEM_lab/blob/main/Gallery/4.%20elastic%20agent.png’/>

6. On the fleet server command line paste the command to install elastic agent.

<img src=’https://github.com/TanunM/ELK_SIEM_lab/blob/main/Gallery/5.%20fleet_server.png’/>

7. As the agent is installed in the kibana interface you will see a message saying fleet server connected.
8. Click on continue enrolling elastic agent to add Windows Server and Ubuntu Machine.
9. In the add agent section add the name for windows server and click on create policy.

<img src=’https://github.com/TanunM/ELK_SIEM_lab/blob/main/Gallery/6.%20policy%20name.png’/>

10. Copy the windows command for the elastic agent
11. Paste it in the Windows Server PowerShell

<img src=’https://github.com/TanunM/ELK_SIEM_lab/blob/main/Gallery/7.%20win_elastic_agent.png’/>

12. After the agent is installed on the Windows Server and connected to the Fleet Server you will get the confirmation on the kibana interface.

<img src=’https://github.com/TanunM/ELK_SIEM_lab/blob/main/Gallery/8.%20confirmation.png’/>

13. Similar to adding Windows agent add the Ubuntu agent by clicking on add agent
14. Name the agent and click on create policy
15. Select enroll in Fleet option and the copy the command for linux.tar
16. Paste the command to the Ubuntu command line and add "--insecure" to the end of the command and press enter.

<img src=’https://github.com/TanunM/ELK_SIEM_lab/blob/main/Gallery/10.%20ubuntu_elastic_agent.png’/>

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
<img src=’https://github.com/TanunM/ELK_SIEM_lab/blob/main/Gallery/9.%20sysmon%20install.png’/>

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

### osTicket Server Setup
1. On the osTicket server download xampp from the [official website](https://www.apachefriends.org/download.html)
2. Similarly download osTicket from its [official website](https://osticket.com/download/)
3. Install xampp with default settings
4. Extract osTicket.
5. Configure the xampp properties file located in the xampp folder on the C drive
6. Change the apache domain name to the osTicket server IP.

<img src=’22’/>

7. Launch xampp control panel and start Apache and MYSQL
8. Go to phpMyAdmin and create a new database

<img src=’23’/>

9. Go to home -> User account and change the root account for the host by clicking on the root and changing the password.
10. Some warning will appear but those are expected.
11. Next change the login information
12. Change the host name type to “Use text field” and name to osTicket Server IP
13. Provide the password you set for root and on the bottom click on "keep the old" and then click on go.

<img src=’24’/>

14. Go to the phpMyAdmin folder on xampp and configure the config.inc.php (C:\xampp\phpMyAdmin\config.inc.php) file and change the password

<img src=’25’/>

15. Try reconnecting and you will be able to reconnect with no issue

<img src=’26’/>

16. Make sure that you have full access to the newly created database.
17. You will be provided two files when you extract osTicket zip. Copy both file and navigate to the htdocs folder of xampp.
18. Create a new folder named "osticket" and paste both the files.
19. Access the server using the http://(osTicket Server IP)/osticket/upload
20. Click on continue and you will be prompted to make some changes to the configuration file.
21. In the “upload” folder of the “osticket” folder navigate to “include” and you will find “ost.sample.config” file
22. Rename the file to “ost-config.php”

<img src=’27’/>

23. In the osTicket interface if you click continue you will be prompted to provide some information to complete the basic installation
24. Fill up the fields by providing the required information

<img src=’28’/>

25. Click on “install now” and you will be provided with a URL to access osTicket.
26. You can login using the username and password you set.

<img src=’29’/>

27. Fill up the System Settings and Preferences information using the osTicket Server IP as the URL (http://(osTicket Server IP)/osticket/upload).

<img src=’30’/>

28. Go to the manage -> API section  and click on Add New API Key
29. Provide the IP address of the "ELK Server" in the IP Address section then select "Can Create Tickets" and Add Internal Notes as a reminder of the API key's use.
30. Copy the API key and go to the Kibana Interface
31. In the interface under “Alerts and Insights”, select “Connectors”.
32. Search for “Webhook connector” and select it. Webhook connector may require trial or a paid subscription depending on your Kibana deployment.
33. In the Webhook connector, provide the URL as “http://(osTicket Server IP)/osticket/upload/api/tickets.xml” then select Authentication as none.
34. Add HTTP header key as “X-API-Key” and value as the provided key value in the OSTicket interface.
35. On the top there is a test option to check if the connection has been correctly configured.
36. Copy the code from the  [Github repo of osTicket](https://github.com/osTicket/osTicket/blob/develop/setup/doc/api/tickets.md)

<img src=’31’/>

37. Run the test and in the osTicket interface navigate to “Admin panel” and you will see a new Ticket as “Testing API”
<img src=’32’/>

### Configure Mythic Server
1. First install Docker on the mythic server 
```bash
sudo apt install docker-compose
```
2. Enable and start docker
```bash
sudo systemctl enable docker.service
sudo systemctl start docker.service
sudo systemctl status docker.service
```
3. Clone the mythic github repo to the server and navigate to the mythic directory
```bash
git clone https://github.com/its-a-feature/Mythic 
ls 
cd ./Mythic
```
4. Install make and start make inside the directory
```bash
sudo apt install make
sudo make
```
5. Finally start mythic using the below command.
```bash
sudo ./mythic.cli start
```
<img src=’13’/>

6. It will take some time and when it starts you can access it using the following url “https://(Mythic IP):7443”

<img src=’14’/>

7. The login credentials are randomly generated and can be accessed using the below command
```bash
cat .env
```
8. From the credentials collect the mythic admin username and password.
9. It's time to add payload. We will be using Apollo agent and  http C2 profile
10. To download the payload we will use the below command.
```bash
sudo ./mythic-cli install github https://github.com/MythicAgents/Apollo.git
sudo ./mythic-cli install github https://github.com/MythicC2Profiles/http 
```
<img src=’15’/>
<img src=’16’/>
<img src=’17’/>

11. We are going to make the payload by clicking on the payload icon on the screen and "Generate New Payload" which is on the dropdown of the "ACTIONS" button.
12. Click on New payload and select the following
   * Windows for OS type
   * Apollo for payload type
   * Include all commands
   * For C2 profile select http
   * For the callback host select the IP of the Mythic Server.
   * Now select Apollo payload and create the payload 

<img src=’18’/>

13. Download the payload by clicking on the download icon and copying the payload link address
14. Go to the command line and type the following
```bash
cd ./
ls
wget https://(Mythic IP):7443/direct/download/<payload> --no-check-certificate
```
15. Rename the file
```bash
mv (payload name) svchost-rocks.exe
```
16. Start a python http server for transferring the payload to the windows machine using the below command.
```bash
python3 -m http.server 9999
```
<img src=’19’/>







