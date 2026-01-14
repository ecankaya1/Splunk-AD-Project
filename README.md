# Splunk-AD-Project
### This project simulates a network with a Windows Server 2022 acting as the AD DC, with a Windows client which forwards logs to Splunk which is set up on Ubuntu Server 24.04 within the same network. There is an attacker on Kali Linux machine who has gained access to the local network & set themselves up to attack the Windows client through RDP brute force.  
<br>

## Table Of Contents:

## Objective
The goal of this project was for me to get an understanding of how Splunk works, get some hands-on & connect it to a Windows server & client. As well as seeing how an attacker may use brute force tools within a network & how new Windows systems defend against it nowadays. Also using Atomic Red Team to generate telemetry & show gaps of security within a network.

## Skills Learned
- Networking: Setting up 4 VM's on the same network. <br>
- Linux: Setting up an Ubuntu 24.04 Server using the CLI & Kali Linux usage. <br>
- Windows Systems: Setting up a Windows Server to act as the Active Directory Domain Controller with a client connected. <br>
- Splunk: The installation of Splunk aswell as the operating of Splunk. <br>
- Atomic Red Team: Installing Atomic Red Team which links to the MITRE ATT&CK framework & using it within PowerShell. <br>
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

## Windows Server

- When setting up the Windows server VM, select the relevant language/keyboard then click next > click install now. When asked about what OS to install select 'Windows Server 2022 Standard Evaluation (Desktop Experience)' > hit next & accept the T&C's. For type of installation, select 'Custom: Install Windows only' > click next > select the virtual disk > hit next. Windows server should now start installing. <br>





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

![*TARGET MACHINE SPLUNK NT SERVICE* ](https://github.com/ecankaya1/Splunk-AD-Project/blob/main/Images/Target%20Machine%20Splunk%20NT%20Service.png)








 





