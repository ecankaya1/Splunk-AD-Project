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
