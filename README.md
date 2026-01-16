# Splunk-AD-Project
### This project simulates a network with a Windows Server 2022 acting as the AD DC, with a Windows client which forwards logs to Splunk which is set up on Ubuntu Server 24.04 within the same network. There is an attacker on Kali Linux machine who has gained access to the local network & set themselves up to attack the Windows client through RDP brute force.  
<br>

## Table Of Contents:
- [Objective](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/README.md#objective)
- [Skills Learned](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/README.md#skills-learned)
- [Tools Used](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/README.md#tools-used)
- [Network Diagram](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/README.md#network-diagram)
- [VM Installation](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/README.md#vm-installation)
- [Splunk Server](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/README.md#splunk-server)
- [Target Machine (Windows Client)](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/README.md#target-machine-windows-client)
- [Windows Server (AD DC)](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/README.md#windows-server-ad-dc)
- [Kali Linux (Attacker)](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/README.md#kali-linux-attacker)
- [Atomic Red Team](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/README.md#atomic-red-team)



## Objective
The goal of this project was for me to get an understanding of how Splunk works, get some hands-on & connect it to a Windows server & client. As well as seeing how an attacker may use brute force tools within a network & how new Windows systems defend against it nowadays. Also using Atomic Red Team to generate telemetry & show gaps of security within a network.

## Skills Learned
- Networking: Setting up 4 VM's in accordance with the network diagram. <br>
- Linux: Setting up an Ubuntu 24.04 Server using the CLI & Kali Linux usage aswell tools within. <br>
- Windows Systems: Setting up a Windows Server to act as the Active Directory Domain Controller with a client connected. <br>
- Splunk: The installation of Splunk aswell as the operating of Splunk. <br>
- Atomic Red Team: Installing Atomic Red Team which links to the MITRE ATT&CK framework]. <br>
- PowerShell: Using PS to use tools & how the commands work. <br>
- Security: Knowledge gained on how a system may get attacked & what processes are used to discourage attacks. <br>

## Tools Used
- VirtualBox: To host the VM's. <br>
- Kali Linux: Attackers machine. <br>
- Ubuntu Server: To host the Splunk server. <br>
- Windows Server: To act as the Active Directory Domain Controller. <br>
- Windows Client: To act as the target machine. <br>
- Splunk: To gather data from the Windows Server & client sent by splunk forwarder. <br>
- Sysmon: To provide information about process creations, network connections, and changes to file creation time. <br>
- Atomic Red Team: Emulates attack techniques which tests the effectiveness of security & detection capabilities, each technique corresponds to the MITRE ATT&CK Matrix. <br>

## Network Diagram

![*NETWORK DIAGRAM*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Network%20Diagram.png)

## VM Installation

- 4 VM's will be installed on VirtualBox. <br>

- Windows client (Target Machine): https://www.microsoft.com/en-ca/software-download/windows10
- Windows Server (AD DC): https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022?msockid=0b85e1f00b9d6dee0806f7600a266ca3
- Kali Linux (Attacker): https://www.kali.org/get-kali/#kali-virtual-machines
- Ubuntu Server (Splunk Server): https://ubuntu.com/server <br>

### *Setup*

- On VirtualBox, for each VM, set the network settings to NAT network, this way, the VM's can be on the same network & still have internet access. <br>

- Do this by clicking on tools in virtual box > click on the bullet points > select Network. <br>

- Click on the Nat Networks tab > click create. <br>

- At the bottom, give it a name & for the IPv4 prefix, I set it to what is stated in the network diagram: 192.168.10.0/24 (Leave DHCP checked). <br>

![NAT NETWORKING SETTINGS](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Nat%20Network%20Settings.png)

- Head over to the splunk server in VirtualBox > go into settings >click the expert tab > head down to network then select Nat Network in the dropdown > select the Nat Network that was created. <br>

![NAT NETWORKING SETTINGS SPLUNK](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Net%20Network%20Settings%20Splunk.png)

- Do this for the other 3 VM's. <br>

## Splunk Server

- When booting up the Ubuntu VM for the first time, in the setup menus, use all the default settings, then when it has rebooted, log in & run the command 'sudo apt update && sudo apt upgrade -y'. <br>

### *Splunk Server IP Address*

- In the CLI > type 'ip a'. I got an IP address of '192.168.10.4'. <br>

![SPLUNK IP A](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Splunk%20ip%20a.png)

- In the network diagram the splunk server has a static IP address of '192.168.10.10'. <br>

- That means a static IP address has to be set on the splunk server. <br>

- In the CLI > type 'sudo nano /etc/netplan/50-cloud-init.yaml'. <br>

- The below image will be set up in accordance with the network diagram. <br>

![SPLUNK STATIC IP AFTER EDIT](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Splunk%20Static%20IP%20After%20Edit.png)

Firstly dhcp4 changed from true to no. <br>
The first IP address '192.168.10.10/24' is the servers IP with a subnet mask of 255.255.255.0 as per the /24 notation. <br>
The second address '8.8.8.8' is the DNS IP address of google. <br>
Then for routes the splunk server will use the default route which is '192.168.10.1' which is the gateway. <br>

- Save this file by hitting CTRL-X, typing 'y' then hitting enter. <br>

- Type 'sudo netplan apply'. <br>

- Type in 'ip a' & you will see the IP address has changed '192.168.10.10'. <br>

![*SPLUNK IP A AFTER EDIT*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Splunk%20ip%20a%20After%20Edit.png)

### *Splunk Installation*

- Head over to www.splunk.com on your actual machine & create an account/log in. <br>

- Once signed in, head over to products then click on 'Free Trials & Downloads'. <br>

- Scroll down to 'Splunk Enterprise' > click 'Get My Free Trial'. <br>

- Select the Linux OS > install the .deb file & save in a directory of your choice. <br>

### *Install Guest Addons For VirtualBox*

- In the Ubuntu CLI type 'sudo apt-get install virtualbox-guest-additions-iso'. <br>

- Once installed, head to devices in the toolbar at the top of the screen > shared folders > shared folder settings.

- Add a folder by clicking on the folder with a green plus sign. <br>

For the folder path, select the folder where the splunk installation file is in. <br>
Choose a folder name of your choosing. <br>
Select read-only, auto mount & make permanent. <br>

![*SPLUNK SHARED FOLDERS*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Splunk%20Shared%20Folders.png)

- Back in the Ubuntu VM > type 'sudo reboot'. <br>

### *ADD USER TO VBOX SF (SHARED FOLDERS) GROUP*

- Type 'sudo adduser username vboxsf'. <br>

![*SPLUNK SF ADDUSER*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Splunk%20SF%20Adduser.png)

- If it pops up saying 'vboxsf does not exist', there may be some additional guest installations that VirtualBox offers that have not been installed yet. <br>

- Fix this by typing 'sudo apt-get install virtualbox-guest-utils'. Once this has been installed > type 'sudo reboot'. <br>

- Once rebooted > run the command 'sudo adduser username vboxsf'. <br>

![*SPLUNK SF ADDUSER FIXED*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Splunk%20SF%20Adduser%20Fixed.png)

- The next step is to create a new directory called 'share'. <br>

- Type 'mkdir share'. Once created type 'ls' to see the new directory. <br>

![*SPLUNK SF SHARE DIRECTORY*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Splunk%20SF%20Share%20Directory.png)

- Now that a directory has been created called share, mount the shared folder onto the directory called share. <br>

- Do this typing 'sudo mount -t vboxsf -o uid=1000,gid=1000 AD-Kali-Project share/'
(The 'AD-Kali-Project' in the command is the shared folder name that was created. To check again, go to devices > shared folders > shared folder settings, double click the folder that was created & next to 'Folder Name' is the name to use). <br>

![*SPLUNK SF MOUNT*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Splunk%20SF%20Mount.png)

(If there is an error, exit the session by typing 'exit' to log out & then log back in because when a user gets added into a new group it doesn't take effect until you logout). <br>

- Type 'ls -la' & the share folder that was created is now highlighted. <br>

![*SPLUNK SF LS -LA*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Splunk%20SF%20ls%20-la.png)

- Enter the share directory by typing 'cd share/'. <br>

- Type 'ls -la' & all the files will be listed in that directory of which one will be the splunk installer. <br>

- To install splunk type 'sudo dpkg -i splunk' then hit TAB to finish off the command. <br> 

![*SPLUNK COMMAND INSTALL*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Splunk%20Command%20Install.png)

- Once the word complete appears, you'll be good to go to change into the directory of where splunk is located on the server. <br>

- Type 'cd /opt/splunk'. <br>

- In the splunk directory, type 'ls -la', all the users & groups that belong to splunk will be listed. (This is a good thing as it limits the permissions to just that user). <br>




### *CHANGE INTO THE USER SPLUNK*

- Change into the user splunk > type 'sudo -u splunk bash'. Now you will acting as the user 'splunk'.

- Change to the directory bin by typing 'cd bin' as the files listed in here are all the binaries that splunk can use. The one that will be used is './splunk'. <br>

- Type './splunk start' to run the installer. This will open a license & terms agreement, hold the space key to get to the bottom then type 'y' to accept. <br>

- Set a username of your choice, I used my normal login name & set a password. <br>

- Now that the installation has completed, splunk needs to be set to start up everytime the VM reboots. <br>

- Type 'exit' to exit out of the user splunk. <br>

- Change into the directory bin, type 'cd bin'. <br>

- Then run the command 'sudo ./splunk enable boot-start -user splunk'. (The user flag at the end will make it so anytime the VM reboots, splunk will run with the user splunk). <br>

![*SPLUNK BOOT STARTUP*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Splunk%20Boot%20Startup.png)

- Now that splunk is up & running, splunk universal forwarder & Sysmon needs to be installed on both the target machine & windows server. <br>




## Target Machine (Windows Client)

- When setting up the Windows 10 Machine select the relevant language/keyboard > click next > click install now > click 'I don't have a product key' & for the OS you want to install, select 'Windows 10 Pro' > hit next > accept the license for the the of installation you want, select 'Custom: Install Windows only' > click next > select the virtual disk, now windows should start installing. Once installed, follow the rest of setup process to your liking. <br>

(Note that when setting up the windows client make sure to select 'Windows 10 Pro' in the windows setup). <br>

![*WINDOWS 10 PRO*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Windows%2010%20Pro.png)

- Firstly change the host name of the machine to 'target-PC'. Do this by typing 'pc' in the search bar > click on properties > 'Rename this PC'. Here I will name it 'target-PC' but you can give it a name of your own choosing > click next > click restart now. <br>

- Once rebooted, to check the name has changed, type 'pc' in the search bar again then click on properties. Here you will see the device name is the name you changed it to. <br>

![*TARGET PC NAME CHANGE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20PC%20Name%20Change.png)

### *Target Machine IP Address*

- Next I be changing the IP address of the target machine in accordance with the network diagram. <br>

- Open up CMD. <br>

- To view the IP address of the computer > type 'ipconfig'. <br>

![*TARGET MACHINE OLD IP ADDRESS*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Old%20IP%20Address.png)

- To change the IP address right click the network icon in the bottom right > click on 'Open Network & Internet settings'. <br>

- Scroll down > click 'Change adapter options' > right click the adapter > click properties. 

- Double click 'Internet Protocol Version 4 (TCP/IPv4)' (Or highlight > click properties). <br>

- Set to a static IP. In the image below I set the IP address, subnet mask, default gateway & DNS address as the following: <br>

![*TARGET MACHINE IP ADDRESS SETTINGS*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20IP%20Address%20Settings.png)

- Once the IP address has been set, to test it has actually worked, open CMD & run the command 'ipconfig' again. <br>

![*TARGET MACHINE NEW IP ADDRESS*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20New%20IP%20Address.png)

### *Accessing Splunk From The Target Machine*

- Open up the web browser & in the search bar type '192.168.10.10:8000' this is the splunk IP address & it is on port 8000 (splunk listens on port 8000). <br>

(Make sure the Splunk server VM is running in the background). <br>

![*TARGET MACHINE SPLUNK LOGIN PAGE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Splunk%20Login%20Page.png)



### *Installing Splunk Universal Forwarder*

- Splunk Universal Forwarder is a streamlined component of the Splunk ecosystem that is primarily used to collect data from various sources and forward it to Splunk indexers for processing and analysis. <br>

- Head over to 'splunk.com' on the target machine & login. <br>

- Once logged in, head over to the platform tab then select 'free trials & downloads'. <br>

- Scroll down to 'Universal Forwarder' > click 'Get My Free Download' & select the correct download for your OS. <br>

- Once downloaded, open up the download. <br>

- Click 'Check this box to accept the License Agreement' & make sure that 'An on-premises Splunk Enterprise instance' is selected > click next. <br>

- For the username I put 'admin' then selected 'Generate random password' > click next. <br>

- I do not have a Deployment, so click next.

- For Receiving Indexer, this will be the IP address of the Splunk server & the default port will be 9997 (port 9997 is the default port for recieving events) > click next > click install. <br>

![*SPLUNK UNIVERSALFORWARDER IP SETUP*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Splunk%20UniversalForwarder%20IP%20Setup.png)


### *Sysmon Install*

- Sysmon (System Monitor) logs detailed system activity to the Windows event log which helps identify malicious/anomalous behaviour by monitoring processes, network connections & file changes. <br>

- Open up a browser in the target machine & in the search bar > type 'sysmon'. Open the webpage 'Sysmon - Sysinternals' by Microsoft (https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon). <br>

- Scroll down > select download Sysmon. <br>

- The Sysmon configuration that I'll be using is by olaf. In the search bar type 'sysmon olaf config' > select the GitHub site from 'olafhartong' (https://github.com/olafhartong/sysmon-modular). <br>

- Scroll down & click on'sysmonconfig.xml'. <br>

- Click on 'Raw' on the right hand side (this will open up the config to a full page). <br>

- Right click the page then select save as > save it to a place of your choosing. <br>

- Head over to where the config was downloaded in file explorer. Right click on the Sysmon zip file > click extract all > click extract.

- Once extracted click on the file explorer bar then copy the path file. <br>

![*SYSMON PATH FILE COPY*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Sysmon%20Path%20File%20Copy.png)

- Open up Windows PowerShell as an administrator. <br>

- The reason for copying the file path is so I can quickly change directory by typing & pasting 'cd C:\Users\user\Downloads\Sysmon'. <br>

![*TARGET MACHINE WINDOWS PS PATH FILE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Windows%20PS%20Path%20File.png)

- Type '.\Sysmon64.exe -i ..\sysmonconfig.xml'(the '-i' flag indicates you want to specify a configuration file) > hit enter. (This will install Sysmon once you hit agree). <br>

![*TARGET MACHINE PS SYSMON INSTALL*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20PS%20Sysmon%20Install.png)

- Now that Splunk Universal Forwarder & Sysmon have been installed, I need to instruct the Splunk forwarder on what gets sent to the Splunk server. <br>

- To do this I must configure a file called 'inputs.conf', this file is located in the C drive > program files > SplunkUniversalForwarder > etc > system > default > input.conf. <br>

- As editing the input.conf file in the default directory can break things, go back into system then into the local directory. Here in the local directory, I will create a new file called 'input.conf'. <br>

- As creating a new file requires administrative privileges, open notepad as as administrator, & paste the following into it: <br>

[WinEventLog://Application] <br>
index = endpoint <br>
disabled = false <br>

[WinEventLog://Security] <br>
index = endpoint <br>
disabled = false <br>

[WinEventLog://System] <br>
index = endpoint <br>
disabled = false <br>

[WinEventLog://Microsoft-Windows-Sysmon/Operational] <br>
index = endpoint <br>
disabled = false <br>
renderXml = true <br>
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational <br>

- It will look like the image below: <br>

![*TARGET MACHINE SYSMON CONFIG FILE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Sysmon%20Config%20File.png)

(This instructs the splunk forwarder to push events related to application, security & system as well as Sysmon over to the Splunk server). <br>

(In the config file the index is pointing to an index name called endpoint, this is important to know as whatever events fall under the application, security, system & Sysmon categories will be sent over to Splunk & placed under the index endpoint. If the splunk server doesn't have an index named endpoint). <br>

- Save the notepad file under program files > SplunkUniversalForwarder > etc > system > local. Change the save as type to 'All Files' & name the file 'inputs.conf'. <br>

![*TARGET MACHINE SYSMON CONFIG SAVE LOCATION*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Sysmon%20Config%20Save%20Location.png)

- Anytime that the input.conf file is updated, you MUST restart the Splunk Universal Forwarder Service. <br>

- Go to the search bar at the bottom > type services > run it as an administrator > look for the 'SplunkForwarder' service. <br>

- Once found, scroll to the right & under the 'Log On As' column if you see 'NT SERVICE', it may not be able to collect logs due to some of the permissions of the account. <br>

- To fix this > right click the SplunkForwarder service > properties > go to the 'Log On' tab > select 'Local System Account' > hit apply. (Now under the 'Log On As' column it will have changed from NT SERVICE to Local System Account, just to verify scroll down to the service 'Sysmon64' & it will be running). <br>

![*TARGET MACHINE SPLUNK NT SERVICE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Splunk%20NT%20Service.png)

- Right click the SplunkForwarder service > click restart. (If it pops up saying windows cannot start the service after clicking restart, just right click SplunkForwarder service > click start). <br>

- Sysmon & Splunkuniversalforwarder have now been installed along with an updated inputs.conf file. <br>

- Head over to the splunk web portal at '192.168.10.10:8000' > log in with the credentials created during the splunk install on the splunk server. <br>

![*TARGET MACHINE WEB PORTAL LOGIN*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Web%20Portal%20Login.png)

- Once logged in navigate to 'Settings at the top' > click on 'Indexes'. <br>

![*TARGET MACHINE INDEXES*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Indexes.png)

- On this page, this is where all of the indexes splunk has, are stored. <br>

- If you recall the inputs.conf file, all of the events are being sent over to an index called endpoint. As there is no index called endpoint in here, this is where will create it. <br>

- At the top right, click 'New Index' > name it 'endpoint' > click save. <br>

![*TARGET MACHINE ENDPOINT INDEX*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Endpoint%20Index.png)

- Scroll down to see the new endpoint index in the stored indexes. <br>

![*TARGET MACHINE INDEX LOCATION*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Index%20Location.png)

- Next I need to make sure that we enable the splunk server to receive the data. <br>

- Navigate to settings at the top > click 'Forwarding and receiving'. <br>

![*TARGET MACHINE FORWARDING AND RECEIVING*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Forwarding%20And%20Receiving.png)

- Under receive data > click configure receiving > click new receiving port in the top right. <br>

- During the Splunk setup, I set the default port of 9997 for receiving events, so for 'Listen on this port' type '9997' > click save. <br>

![*TARGET MACHINE 9997*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%209997.png)

- If everything is set up correctly, you should start seeing data coming in from out target machine. <br>

- To see this, in the top left click Apps > select Search & Reporting. <br>

![*TARGET MACHINE SEARCHING AND REPORTING*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Searching%20And%20Reporting.png)

- In the search bar type in 'index=endpoint' then click search. <br>

![*TARGET MACHINE INDEX=ENDPOINT*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Index%3DEndpoint.png)

- In here you will see all event logs that go to the index endpoint. Just under the search bar you can see how many events total there are. <br>

- Scroll down to 'SELECTED FIELDS' > click 'hosts'. There will be 1 host in here which is the target machine. <br>

![*TARGET MACHINE HOST*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Host.png)

- Below 'host' > click'source'. This will display what I specifically set in the inputs.conf file: Security, Application, System & Sysmon data. <br>

![*TARGET MACHINE SOURCE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Source.png)

- Seeing this data means that Sysmon and Splunk have successfully been installed and configured for the target machine. Now I will do the exact same for the Windows Server. <br>



## Windows Server (AD DC)

- When setting up the Windows server VM, select the relevant language/keyboard then click next > click install now. When asked about what OS to install select 'Windows Server 2022 Standard Evaluation (Desktop Experience)' > hit next & accept the T&C's. For type of installation, select 'Custom: Install Windows only' > click next > select the virtual disk > hit next. Windows server should now start installing. <br>

- The Windows Server will act as the Active Directory Domain Controller (AD DC). <br>

- Once installed, log in & change the computer name, the same way I did for the target machine. <br>

- In the taskbar, search 'PC', then right click PC and head into properties then click Rename this PC. <br>

- I set the name for the Windows Server to 'ADDC01', then restarted the VM. <br>

- Once restarted, you can check the name of the server has changed by heading into PC properties. <br>

![*DC NAME CHANGE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/DC%20Name%20Change.png)

- Next I will set up the Domain Controller (DC) with a static IP address in accordance with the network diagram the same way I did for the target machine. <br>

- Right click the network icon in the bottom right > click 'Open Network & Internet settings' > click 'Change adaptor options' > right click the 'Ethernet' interface > click properties then open the IPv4 option. <br>

- In here, click 'Use the following IP address' & input the settings in accordance with the network diagram. <br>

![*DC IP*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/DC%20IP.png)

- To test the settings have worked, open command prompt > type in the command 'ipconfig' & then type 'ping google.com' to test connectivity. <br>

![*DC IP TEST*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/DC%20IP%20TEST.png)

- Now splunk and Sysmon have to be installed and configured on the DC, this is done the exact same way as it was done on the target machine. <br>

- Once set up, test by pinging the splunk server in the command prompt with 'ping 192.168.10.10'. <br>

![*DC SPLUNK SERVER PING*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/DC%20Splunk%20Server%20Ping.png)

- Now head over to our splunk web portal by typing '192.168.10.10:8000'. <br>

- Heading over to the search and reporting tab and here you will see that it is picking up logs from the DC (ADDC01). <br>

![*DC SPLUNK INDEX*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/DC%20Splunk%20Index.png)

(Just as an FYI, you cannot ping between windows computers because ICMP is disabled by default. If you want to enable ICMP, you will have to create an inbound rule to allow ICMP traffic, but in this case since we can ping the splunk server I can assume that there is connectivity between the computers). <br>


### *Active Directory*


- Now that splunk & Sysmon have be installed & configured on the DC, I will install Active Directory Domain Services, head into Server Manager which is located on the taskbar. <br>

- Click manage in the top left > open Add Roles and Features. <br>

- Select next & in installation type make sure 'Role-Based or feature-based installation' is selected. <br>

- Server selection is where you would see all available servers, but since I only have the one, I will select that one. <br>

- In server roles, click on 'Active Directory Domain Services' > click add features. <br>

- Click next until the end of the wizard > click install. <br>

- It will be installed when it says this below the progress bar:

![*DC ADDS INSTALL*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/DC%20ADDS%20Install.png)

- Back in Server Manager, click the flag icon beside manage & you will see the option to Promote this server to a domain controller, click on that. <br>

![*DC PROMOTE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/DC%20Promote.png)

- In the Deployment Configuration wizard select 'Add a new forest' & fill in the field with a domain name of your choice, I set it to 'emre.local' (the domain name must have a top level domain so the domain name cannot just be 'emre', it must be emre.something). <br>

![*DC DOMAIN NAME*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/AD%20Domain%20Name.png)

- Click next into Domain Controller Options & set a password of your choosing. <br>

- Click next through the wizard until 'Paths', this is where database file named NTDS.dit is stored. (Attackers target domain controllers not only because it has access to everything but because of this file as it contains everything related to Active Directory including password hashes). <br>

![*DC NTDS*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/DC%20NTDS.png)

- Click next through the wizard & once the setup has finished verifying prerequisites > click install. Once the setup is completed the server will automatically restart. <br>

- Once restarted, you will see the domain name followed by a back slash which indicates ADDS has successfully installed & promoted the server to a domain controller. <br>

![*DC DOMAIN SLASH*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/DC%20Domain%20Slash.png)


### *Adding Users*

- The next step is to create users. <br>

- In Server Manager >click Tools > select Active Directory Users and Computers. <br>

- Expand the domain in the left hand pane, within the 'Builtin' unit will be groups that have been automatically created by AD & you can double click on any of them to see the description of what each group does. In the 'members' tab of a selected group you can see who is assigned to the group & in the 'member of' tab you can see what other groups the selected group is in. (As an FYI you cannot add groups into a builtin group but you can create a custom group & add the builtin group to the custom group). <br>

- In the left hand pane > click 'Users' > here will be a list of users along with additional groups. <br> 

(In a real word environment, users will likely be broken up into different departments known as Organisational Units (OU)). <br>

- To mimic that I will right click the domain in the left hand pane > select new > select Organisational Unit. I will name this OU as 'IT'. Now in the left hand pane there will be a new OU called 'IT'. <br>

- Within this OU > right click the backdrop > select new > select user. <br>

- In here you can fill out the details for a new user, I used 'John Smith' & for user logon name I will set it as 'jsmith' > click next > set the password to whatever you wish (because I am in a lab environment I will uncheck 'User must change password at next logon) > click next & finish. <br>

![*DC NEW USER*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/DC%20New%20User.png)

- Create a new OU called 'HR' & create another user inside the same way as in the 'IT' OU, I used Bob Smith. <br>

- Now there is 2 users that have been created. (There are many scripts available online that can help you auto create users, groups & computers, like the PowerShell script which I used in a previous project). <br>

- Now that AD is setup & the server is the DC, head over to the windows target machine & join it to the newly created domain & authenticate with the John Smith account that was created. <br>

- On the target machine > type 'PC' in the search bar > click properties. <br>

- Scroll down to advanced system settings > in the computer name tab > click change & select domain, type in the domain controller name that was previously created on the windows server. <br>

![*TARGET MACHINE DOMAIN CHANGE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Domain%20Change.png)

- An error in the image below may pop up, this happens because the target machine does not know how to resolve 'EMRE.LOCAL' & this all stems from how DNS works. <br>

![*TARGET MACHINE DNS ERROR*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20DNS%20Error.png)

- To fix this > right click on the network icon in the bottom right > click network & internet settings > click change adapter options > right click the adaptor > select properties > double click IPv4. <br>

- The DNS server address will be pointing to googles DNS (8.8.8.8), this needs to be changed so it is pointing to the DC (192.168.10.7). <br>

![*TARGET MACHINE DNS CHANGE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20DNS%20Change.png)

- To test it has changed > open command prompt > type in 'ipconfig /all' & it will show the DNS servers IP address. <br>

![*TARGET MACHINE IPCONFIG ALL*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Ipconfig%20All.png)

- Reattempt to join the domain & a prompt to enter credentials will appear, use the administrator account of the server to log in. <br>

- If successful a popup will appear saying welcome to the domain & then it will prompt you that the computer must be restarted. <br>

- Once restarted & at the login screen, log in with a user that you previously created in AD. I will use John Smith with the username of jsmith. To do this select other user at the bottom left & make sure that 'Sign in to:' says your domain. <br>

- Once logged in, you will have successfully created a new user, joined a computer to a domain & logged in as a domain user. <br>

- Now that I have a mini simulated network, I will use Kali Linux to perform a brute force attack on the windows target machine & afterwards viewing the telemetry in Splunk. After that I will set up and install Atomic red team on the target machine to generate telemetry for certain attack methods attackers may use. <br>



## Kali Linux (Attacker)

I will be using a pre-built VM of Kali Linux. <br>

- Open up the Kali Linux VM & login. <br>

### *Kali Linux IP Address*

- A static IP address needs to be set up of '192.168.10.250/24' as per the network diagram. <br>

- Right click the ethernet icon at the top > select edit connections > select the first profile which for me was 'Wired Connection 1' & at the bottom click the cog icon. <br>

- Select IPv4 Settings > change it from automatic(DHCP) to manual > click add. <br>

- IP address: 192.168.10.150, Netmask: 24 (as 255.255.255.0 is a /24), Gateway: 192.168.10.1 & DNS: 8.8.8.8 > click save. <br>

![*KALI IP*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%20IP.png)

- To test it has set > head to the desktop > right click the backdrop > click open terminal. <br> 

- Type in the command 'ip a' & you notice the IP address has not changed yet. <br>

![*KALI IP A BEFORE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%20IP%20A%20Before.png)

- To reflect the changes made > click the ethernet icon in the top right > click disconnect > click on the ethernet icon again > click on the connection that the IP address changes were made to. <br>

- Type in 'ip a' in the terminal again & the IP address will have changed to what it was manually set to. <br>

![*KALI IP A AFTER*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%20IP%20A%20After.png)

- Verify connectivity by pinging google.com in the terminal & to test connectivity to the splunk server, ping the Splunk server. <br>

![*KALI GOOGLE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%20Google.png)

- Now update & upgrade the repositories in the terminal > type 'sudo apt-get update && sudo apt-get upgrade -y'. <br>

- Once the update & upgrade is finished, set up the attack by creating a new directory called 'ad-project'. <br>

- In the terminal, type 'mkdir ad-project'. This will create a directory on the desktop & all of the files that will be created & used will be put into this directory. <br>


### *Patator*

Patator is a multi-threaded brute-force and enumeration tool, which I will be using to perform a brute-force attack for remote desktop on a users account. <br>

(Patator is already installed with kali but to make sure, type in 'sudo apt-get install -y patator').

### *Rockyou*

Rockyou is a popular word list that comes with kali.

- Go to the directory where it is stored by typing 'cd /usr/share/wordlists/'. <br>

- Once in this directory type 'ls' & you will see a file called 'rockyou.txt.gz'. <br>

![*KALI WORDLIST*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%20Wordlist.png)

- This file can be unzipped using gunzip > 'sudo gunzip rockyou.txt.gz'. <br>

- Type in 'ls' to see that it has been unzipped. <br>

![*KALI UNZIP*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%20Unzip.png)

Copy this file to the ad-project file that was previously created. <br>

- Type in 'cp rockyou.txt ~/Desktop/ad-project'. <br>

- Change into the ad-project directory > 'cd ~/Desktop/ad-project'. <br>

- Type 'ls -lh' to see the file & file size, you can see it is 134M, this will contain a lot of passwords, for the sake of this lab I do not want to use all the passwords inside of it so I will just use the first 20 lines. <br>

![*KALI COPY*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%20Copy.png)

- Output the first 20 lines to a file called passwords.txt > type 'head -n 20 rockyou.txt > passwords.txt'. ('head -n 20' specifies the first 20 lines of the file). <br>

- Typing in 'ls' will show the passwords.txt file. <br>

- Read this file by typing 'cat passwords.txt'. <br>

![*KALI CAT*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%20Cat.png)

An attacker would most likely do a lot of reconnaissance & set up basic active directory attacks, but in this scenario I know I want to target a certain password. <br>

- Add a password that was set for the user account on the target machine to the list of passwords.txt> open the file to edit with 'nano passwords.txt' > add the password to the bottom of the list. Once added hit CTRL-X > type 'y' > hit enter. <br>

Read the file again with 'cat passwords.txt', you will see that the chosen password has been added to the bottom of the list. <br>

![*KALI NANO*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%20Nano.png)

Before I launch the attack, remote desktop needs to be enabled on the target machine. <br>

- On the target machine > in the search bar > type 'PC' > click PC properties. <br>

- Scroll down to advanced system settings > log into the administrator account. <br>

- Under the remote tab > select allow remote connections to the computer. <br>

![*TARGET MACHINE REMOTE ENABLE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Remote%20Enable.png)

- Click select users > click add > put in the 2 users that was previously created in Active Directory. <br>

![*TARGET MACHINE ADD USERS*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Add%20Users.png)

- Click ok & apply these settings. <br>

Now remote desktop has been enabled for the target machine (this can also be done with the same method for the DC). <br>

- In the Kali Linux VM > in the terminal type 'patator -h', this will bring up a help guide. 

- Type in 'patator rdp_login host=192.168.10.100 user=jsmith password=FILE0 0=passwords.txt'. <br>

The 'host' syntax points to the target machines IP, the 'user' syntax points to the user I am gaining the login credentials for & the 'FILE0' syntax with the '0' syntax points to the passwords.txt file that I created. <br>

In a real world scenario, modern Windows RDP is not a good brute force target anymore, this is due to: <br>

NLA (Network Level Authentication) - This forces authentication before the RDP session is created. <br>

CredSSP - RDP with NLA relies on CredSSP, which integrates with: Keberos, NTLM & TLS. This means authentication is handled by the Windows security subsystem, not the RDP service. <br>

Poor Tool Support - Most publicly available tools like patator, hydra & crowbar do not fully implement CredSSP, they also assume legacy NTLM only authentication & fail when NLA is enabled. <br>

Account Lockout - Windows environments typically enforce account lockout thresholds, backoff delays & generic authentication failures. These controls limit the number of attempts an attacker can make before detection or lockout. <br>

NTLM (Network Technology LAN Manager) - Many modern windows systems disable NTLM entirely, restrict NTLM to specific hosts & require keberos for authentication. <br>

MFA - When RDP is available for systems, it tends to be protected with MFA, conditional access policies & RDP Gateway/VPN. MFA basically renders password only brute force attacks useless. <br>

Network Exposure - Best practice is to block RDP from the internet, require a VPN & restrict access by IP or security group. <br>

Attackers now tend to do password spraying via AD services, VPN authentication abuse, phishing & token theft & credential reuse from other breachers. Realistically RDP brute force is noisy, slow & low success. <br>

Once the patator command has been entered in the terminal it will most likely give an error (below image). For this instance I know 'Password1' is the correct password for the users account, but it has given an error due to the listed reasons above. <br>

![*KALI PATATOR OUTPUT*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%20Patator%20Output.png)

However for the purpose of this lab, even if the brute force fails it will still give me telemetry data. <br>

- Head over to the splunk web portal. <br>

- Because I did the attack I know it was within the last 15 minutes & against the user 'jsmith' so I can narrow down the search. <br>

- In Search & Reporting, in the search bar type 'index=endpoint'. <br>

- Scroll down & in the left pane select EventCode, in here you should ID values, specifically ID '4625' which means an account failed to log in. 

- Beside 4625, in the count column, it will say how many times the event ID has been produced, for me it was 168 as I was tinkering with the patator commands. <br>

![*KALI 4625*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%204625.png)

- Clicking on 4625, will automatically update the search query. <br>

![*KALI QUERY*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%20Query.png)

If you look within the time column, you can see all these events took place at a similar time which is a clear indication of brute force activity. <br>

Now if you look at any of the events & select Show all 61 lines, it will show whose account has failed & where the source of the attempted login came from. <br>

![*KALI SHOW ALL*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Kali%20Show%20All.png)


## Atomic Red Team

Atomic Red Team is a useful tool to identify the gaps & visibility in a network system, it also generates telemetry to see if you can actually detect this activity, in my case, via splunk. <br>

- To install Atomic Red Team, open powershell as an administrator. <br>

- Run the following command 'Set-ExecutionPolicy Bypass CurrentUser'. (This command allows scripts to be run for the current user). <br>

![*TARGET MACHINE BYPASS*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Bypass.png)

Before Atomic Red Team can be installed, set an exclusion for the entire C drive as Microsoft defender will detect & remove some of the files from Atomic Red Team. <br>

- In bottom right of the screen click the up arrow > click Windows security > click virus & threat protection > click manage settings > scroll down to add or remove exclusions. <br>

- In here click Add an exclusion > click folder > select the C drive. This will add the exclusion for the C drive. <br>

![*TARGET MACHINE C DRIVE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20C%20Drive.png)

- To install Atomic Red Team, go back into PowerShell, run the following command 'IEX (IWR 'https://raw.githubusercontent.com/redcanaryco/invoke-atomicredteam/master/install-atomicredteam.ps1' -UseBasicParsing);'. <br>

- Then run the command 'Install-AtomicRedTeam -getAtomics'. <br>

(This will go out & grab Atomic Red Team & install it onto the target machine). <br>

![*TARGET MACHINE ATOMIC INSTALL*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Atomic%20Install.png)

- Once installed > go into C drive in file explorer > click AtomicRedTeam > click on atomics. <br>

- In here will be lots of technique IDs & these map to the MITRE ATT&CK framework. <br>

- Head over to the MITRE ATT&CK framework on the webpage 'https://attack.mitre.org', scroll down to the Matrix for Enterprise & hover over anything in the table, it will be associated with a technique ID. <br>

![*TARGET MACHINE MITRE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Mitre.png)

For example the one in the image above has an ID of 'T1136.001' which is a local account, head back into the atomics file & see if there is T1136.001 (If there is no technique in the atomics folder, you cannot run a command for it). <br>

![*TARGET MACHINE T1136*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20T1136.png)

- Go into PowerShell & run the command 'Import-Module "C:\AtomicRedTeam\invoke-atomicredteam\Invoke-AtomicRedTeam.psd1" -Force' (This command allows me to run the following commands). <br>

![*TARGET MACHINE INVOKE*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Invoke.png)

- Now I can run the next command 'Invoke-AtomicTest T1136.001'. <br>

This command will automatically generate telemetry based on creating a local account. <br>

![*TARGET MACHINE LOCAL USER*](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Local%20User.png)

Look at the username that was created: NewLocalUser, you can head into splunk & search specifically for NewLocalUser. <br>

- In Search & Reporting in Splunk, search 'index=endpoint NewLocalUser'. <br>

![*TARGET MACHINE EVENTS*]](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Events.png)

If no events were to pop up, this will tell you that you're blind to this activity, if an attacker compromised your system & created a local account, with the current settings will not be able to detect that activity. <br>

- To create alerts for this event, in the top right click save as & click Alert, here you can customise the alert & set it so if there was the creation of a NewLocalAccount an alert will trigger. <br>







































