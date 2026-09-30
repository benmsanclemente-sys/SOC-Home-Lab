# SOC-Home-Lab
Virtual SOC home lab built in Oracle VirtualBox using Splunk, pfSense, Windows, Ubuntu, and Kali Linux for security monitoring and network analysis.

## Why
This lab is an ongoing project that I am building in Oracle VirtualBox to simulate a small SOC environment and develop hands-on experience with SIEM administration, firewall configuration, network segmentation, log ingestion, log analysis, and security monitoring.
My goal is to understand not only how to investigate security events, but also how the underlying systems generate, route, and structure the telemetry being analysed.

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

#The environment currently contains six virtual machines across seperate internal, DMZ, and adversary networks, with pfSense acting as the firewall and internet gateway.

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
Starting at the network edge, pfSense VM is functioning as intended and forwards pfSense logs into Splunk as well as functioning as a firewall and enforces rules regarding communication between the internal network and the public facing website network and enforces other rules that were necessary for functionality.
