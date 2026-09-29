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
