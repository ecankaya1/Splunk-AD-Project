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











 





