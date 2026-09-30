# SOC-Home-Lab
Virtual SOC home lab built in Oracle VirtualBox using Splunk, pfSense, Windows, Ubuntu, and Kali Linux for security monitoring and network analysis.

## Why
This lab is an ongoing project that I am building in Oracle VirtualBox to simulate a small SOC environment and develop hands-on experience with SIEM administration, firewall configuration, network segmentation, log ingestion, log analysis, and security monitoring.
My goal is to understand not only how to investigate security events, but also how the underlying systems generate, route, and structure the telemetry being analyzed. I feel in order to be able to problem solve issues to the highest degree, knowledge of how something is constructed and how it works are important and give you deeper insight into understanding why something is happening. There is so much information to learn in this field that it can be overwhelming, so I decided starting closest to home but underneath the surface is the best place to begin expanding my knowledge. I know a lot of what I am doing and struggling with are not particularly relevant to working within a SOC as a tier 1 analyst compared to say just doing a TryHackMe SOC analyst practice performing the role with alerts which I have also done, but the knowledge it gains is beneficial to someone in that role, being able to identify a bad field extractor or the logs source being slightly different than what is expected, being able to understand a raw pfsense:filterlog when others cannot proves the value in learning the building in its raw material state. 

## Architecture

```
                              Internet
                                 |  
                            VirtualBox NAT
                                 |       
                             pfSense VM
                            /    |       \
              Adversary Net      |          DMZ Net
            /                    |                   \
          /             Company Internal Net           \
      Kali VM            /       |          \           Ubuntu NGINX VM
                      /          |            \                
          Win11 Employee VM      |              Win11 SOC VM
                          Ubuntu Splunk VM
```

The environment currently contains six virtual machines across seperate internal, DMZ, and adversary networks, with pfSense acting as the firewall and internet gateway.

## Virtual Machines

### PFSENSE-FIREWALL
pfSense firewall and router providing internet access, DHCP, network segmentation, and traffic control between the lab and networks.

### UBUNTU-SERVER-SPLUNK
Ubuntu server hosting Splunk Enterprise for centralized log ingestion, searching, and security monitoring.

### NS-EMPLOYEE-01
Windows 11 workstation representing an employee endpoint and future attack target.

### NS-SOC-01
Windows 11 workstation used for SOC administration, Splunk access, and pfSense management.

### UBUNTU-SERVER-PUBLIC-FACING-WEBSITE
Ubuntu server running NGINX in the DMZ to represent a public-facing service and potential attack surface.

## Current Capabilities
Starting at the network edge, the pfSense VM is functioning as intended and forwards pfSense logs into Splunk as well as functioning as a firewall and enforces rules regarding communication between the internal network and the public facing website network and enforces other rules that were necessary for functionality. The Kali Linux VM has the operating system installed and has full functionality, kali is capable of performing port scans and has on the public facing DMZ and is capable of performing a variety of attack types such as ipv6 spoofing source ip. The current capabilities of the Ubuntu server hosting the NGINX are more limited, currently the machine has a functioning operating system and has nginx usable, however nothing beyond running NGINX and performing a port scan against this VM with the Kali VM has been done to this machine yet. Next is the Windows 11 Employee work station, the capabilities of this machine are fully functioning operating system, has Sysmon installed and is forwarding the logs to the Splunk VM, and is able to generate regular traffic through the firewall creating logs I can interact with and analyze. Then we have the Windows 11 SOC analyst VM which is capable of accessing the GUI for pfSense and for Splunk and is where most of my configuration and operation of those two information management systems occurs, it has a fully functioning OS and has internet access. Finally, we have the Ubuntu Splunk VM which is currently capable of ingesting logs from pfSense and from sysmon, has field extraction done for ipv4 and ipv6 logs from pfSense, has Sysmon XmlWinEventLogs field extraction completed as well amongst other logs types.


## Future Plans
There are a few ideas and thoughts I've had about this home lab because I am learning as I build this I realize improvements that can be made after the fact. One example of this was the creation of the Windows Employee VM, that only occurred because I didn't like the idea that my logs were being created by the device that was viewing them. some Current goals for my home lab are to integrate more telemetry, set up the NGINX server to better represent a public facing website, change the Windows SOC VM to a Linux SOC VM, add a VLAN for a separate IT/SOC network. I also think configuring the SIEM to add alerts or flags, so when an event happens it notifies me as opposed to me having to do the specific SPL search to find relevant log data. 



## Home Lab Journal
[Northstar_SOC_Lab_Journal.md](https://github.com/user-attachments/files/32875299/Northstar_SOC_Lab_Journal.md)

## Image of Current Virtual Machines and Splunk
<img width="3838" height="1157" alt="{6EE2CA46-EA0D-4B5D-AB3B-078EF935DA79}" src="https://github.com/user-attachments/assets/f52d56f4-62f3-4c46-a739-c233594687ec" />


<img width="1019" height="639" alt="image-1" src="https://github.com/user-attachments/assets/151f4ce7-3a27-45db-8db2-558a4ce8a0ef" />
