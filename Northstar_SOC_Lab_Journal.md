# Northstar SOC Homelab Journal

**Retrospective coverage:** Through 2026-09-18  
**Lab owner/operator:** Benjamin San Clemente  
**Environment:** Northstar SOC Homelab  

## AI Involvement / Provenance Note

This journal section was **retrospectively reconstructed by ChatGPT from prior conversations with Benjamin San Clemente**. It was not written contemporaneously by Benjamin. Benjamin performed the lab work, made configuration decisions, ran commands, observed outputs, and completed the troubleshooting described here. ChatGPT was used as an instructional and troubleshooting aid during much of the build and was later asked to reconstruct the history into a journal for continuity.

Dates below are included only where they can be recovered with reasonable confidence from prior conversation timestamps. Entries marked **Approximate** summarize work that clearly occurred during a period but whose exact execution date is not certain from the available history. This document should therefore be treated as a historical reconstruction, not as an independently audited change log.

From this point forward, Benjamin intends to write new lab journal entries himself and may use AI for review, feedback, or clarification. This reconstructed section is retained so the journal does not begin halfway through the project.

---

# Environment Overview

Northstar is a fictional financial-services/accounting-style organization used as the business context for the lab. The primary security objective is to protect client financial data while maintaining availability of business services.

## Current VM Inventory

- **PFSENSE-FIREWALL** — network gateway/firewall and segmentation point.
- **UBUNTU-SERVER-SPLUNK** — Splunk Enterprise server / SIEM.
- **NS-EMPLOYEE-01** — Windows 11 employee endpoint.
- **NS-SOC-01** — Windows 11 SOC/admin workstation.
- **KALI-ADVERSARY** — Kali Linux system representing an external/untrusted attacker.
- **UBUNTU-SERVER-PUBLIC-FACING-WEBSITE** — Ubuntu DMZ web server, currently using Nginx.

## Network Segmentation

pfSense uses four virtual interfaces:

- **em0 — WAN/NAT**
- **em1 — INTERNAL / LAN**
- **em2 — DMZ**
- **em3 — ADVERSARY / ATTACK network**

Known addressing used during the build includes:

- INTERNAL network: **192.168.10.0/24**
- pfSense INTERNAL gateway: **192.168.10.1**
- NS-EMPLOYEE-01 observed at **192.168.10.103** on 2026-09-17 (earlier lab records showed .100, consistent with DHCP changes over time)
- DMZ Ubuntu web server: **192.168.20.100**
- Kali adversary host used a **203.0.113.x** address during September testing

A dedicated management/SOC VLAN has been discussed as a future improvement because all four pfSense virtual NICs are already occupied.

---

# Chronological Lab Journal

## 2026-06-02 — Early Baseline and VM Snapshots

**Date confidence: Exact conversation date**

The lab already had a working set of VirtualBox snapshots by this point, including snapshots described as:

- `PF Sense-working baseline`
- `Kali-connected`
- `Win 11-connected`
- `Ubuntu-connected`

This indicates the initial pfSense, Kali, Windows, and Ubuntu connectivity work was already underway before the later SOC-focused expansion.

**Skills practiced:** VirtualBox snapshots, VM rollback strategy, staged environment building.

---

## 2026-06-11 — DMZ Web Server Became Operational

**Date confidence: Exact conversation date**

The Ubuntu DMZ server became operational at **192.168.20.100**. LAN and DMZ were confirmed as separate routed networks.

Kali was able to reach the DMZ host. Early Nmap testing showed only **TCP/22 (SSH)** open at first.

Apache was installed during this early web-server phase and the default Apache page was successfully loaded from a Windows system. Later Nmap output showed **TCP/22 and TCP/80** open. Nikto testing was also performed and reported items such as missing security headers, ETag/inode information leakage, and permitted HTTP methods.

This was an early exposure to the difference between simply making a service reachable and evaluating the security posture of that service.

**Skills practiced:** DMZ routing, Nmap basics, HTTP service exposure, introductory web-server assessment, Nikto.

---

## 2026-08-06 — SOC Lab Architecture Expanded

**Date confidence: Exact conversation date**

By this point the lab included pfSense, an Ubuntu SOC/SIEM server, a Windows 11 endpoint, Kali on an adversary network, and an Ubuntu DMZ/public server. Internet access and snapshots were established.

The lab was increasingly being treated as a simulated SOC environment rather than simply a networking practice environment.

**Skills practiced:** Lab architecture, role separation, network design, VM lifecycle management.

---

## 2026-08-07 — Splunk Selected as the SIEM

**Date confidence: Exact conversation date**

Splunk Enterprise was selected as the SIEM platform for the Ubuntu server.

The intended telemetry pipeline was defined as:

`pfSense -> syslog/rsyslog -> Ubuntu -> Splunk`

Longer-term telemetry goals included:

- pfSense firewall logs
- Windows event logs
- Linux logs
- web-server logs
- future IDS/IPS or other security telemetry

This was the beginning of the lab's dedicated SIEM/log-management phase.

**Skills practiced:** SIEM architecture, log-source planning, Linux CLI, service-oriented design.

---

## 2026-08-10 — Splunk Storage and pfSense Syslog Ingestion

**Date confidence: Exact conversation date**

The Splunk Ubuntu VM had a storage-space issue. Additional virtual disk capacity was allocated, reducing utilization from roughly 80% to roughly 40%.

pfSense remote syslog forwarding was configured toward the Ubuntu/Splunk system over **UDP/514**. During troubleshooting, rsyslog was active but UDP/514 was initially not listening correctly. This was investigated and corrected.

The initial pfSense logging path used `/var/log/syslog`, then the configuration was moved toward a dedicated file:

`/var/log/pfsense.log`

This made the pfSense telemetry easier to isolate and ingest in Splunk.

A timestamp/time-zone issue also became apparent. pfSense initially produced timestamps that lacked an explicit timezone in a format that caused Splunk event times to appear offset by several hours. The pfSense log format was later changed to a timestamp format containing an explicit UTC offset, improving event-time accuracy.

**Skills practiced:** Linux storage expansion, rsyslog, UDP syslog, Splunk file monitoring, timestamp troubleshooting, log-pipeline validation.

---

## 2026-08-11 — Northstar Business Context and Network Roles Defined

**Date confidence: Exact conversation date**

The lab's fictional organization was formalized as **Northstar Financial Services**, with client financial records treated as the critical asset.

The pfSense interfaces were organized as:

- WAN/NAT
- Internal SOC/LAN
- DMZ
- AdversaryNet

Kali was explicitly treated as an untrusted/foreign system rather than as a Northstar-owned endpoint.

This business framing made later firewall policies and investigation reports easier to reason about in terms of security requirements rather than simply technical connectivity.

**Skills practiced:** Asset prioritization, business-context security reasoning, trust boundaries.

---

## 2026-08-12 to 2026-08-21 — Separate SOC Workstation and Role Separation

**Date confidence: Period-based**

The architecture was refined to separate the monitored employee endpoint from the analyst/admin workstation.

A second Windows 11 VM, **NS-SOC-01**, was added/used as the SOC workstation. This allowed NS-EMPLOYEE-01 to represent an ordinary business endpoint while NS-SOC-01 was used for Splunk and pfSense administration.

By 2026-08-21, the six-VM inventory was established:

1. UBUNTU-SERVER-SPLUNK
2. NS-EMPLOYEE-01
3. KALI-ADVERSARY
4. PFSENSE-FIREWALL
5. UBUNTU-SERVER-PUBLIC-FACING-WEBSITE
6. NS-SOC-01

A future management/SOC VLAN was identified as a better long-term isolation model than leaving the SOC workstation on the same internal network as employee endpoints.

**Skills practiced:** Administrative separation, endpoint role modeling, network-security architecture.

---

## 2026-08-19 — Firewall Segmentation Rules Validated

**Date confidence: Exact conversation date**

Three major network-control goals were tested and validated.

### NS-NET-001 — Adversary -> INTERNAL

Traffic from the adversary network toward INTERNAL was blocked and logged.

### NS-NET-002 — Adversary -> DMZ Web Server

The public-facing DMZ web server was configured so that **TCP/443 (HTTPS)** was reachable from the adversary network while **TCP/80 (HTTP)** was blocked by firewall policy.

Nginx was configured to listen on TCP/443. A self-signed/snakeoil certificate caused normal certificate validation to fail when testing by IP address, while `curl -k` successfully confirmed HTTPS connectivity.

This demonstrated the distinction between transport/service reachability and certificate trust/identity validation.

### NS-NET-003 — DMZ -> INTERNAL

The DMZ originally had a broad outbound rule. Testing showed that a failed ping toward an internal Windows endpoint did **not** prove that pfSense had blocked the packet; pfSense logs showed the packet being forwarded, indicating the endpoint itself was likely rejecting/ignoring the traffic.

A dedicated top-priority firewall rule was then created to block and log **DMZ -> INTERNAL** traffic. Retesting confirmed the traffic was blocked at pfSense and no longer forwarded into the internal network.

The DMZ retained Internet access under a broader lower-priority outbound rule.

This provided a practical demonstration of:

- top-down firewall-rule processing
- first-match behavior
- stateful firewalling
- the difference between firewall policy and endpoint host-firewall behavior
- why a failed ping does not automatically prove network-layer blocking

**Skills practiced:** pfSense policy design, segmentation testing, rule ordering, HTTPS exposure, firewall log interpretation.

---

## Late August to September 2026 — Manual pfSense Log Parsing and Technology Add-on Troubleshooting

**Date confidence: Approximate period**

A major lab focus became making pfSense logs useful inside Splunk rather than relying on raw comma-separated events.

### Manual Parsing Work

Splunk's field extractor was used with comma delimiters against `pfsense:filterlog` events. Useful positions were manually identified for some IPv4 UDP events, including:

- interface
- reason
- action
- direction
- IP version
- protocol
- source IP
- destination IP
- source port
- destination port
- payload/data length

This exercise revealed an important limitation: **IPv4, IPv6, TCP, UDP, and ICMP filterlog records do not all have identical positional layouts**. A field index that means “source IP” in one event structure can represent something else in another.

This led to the understanding that a reliable parser needs to identify/classify the event format first and then apply the correct extraction logic.

### pfSense Technology Add-on

The **Technology Add-on for pfSense**, version 3.0.0, was obtained from its GitHub repository and packaged as a `.tar.gz` file for installation into Splunk.

The add-on expected incoming pfSense events to begin with an initial sourcetype of `pfsense` and then transform them into more specific sourcetypes such as `pfsense:filterlog`.

The add-on's default sourcetype-classification regex expected a different syslog header format than the lab's RFC3339-style timestamp with explicit timezone offset. Because the explicit timezone format had already solved an earlier Splunk timestamp issue, the logging format was intentionally **not reverted**.

Instead, a local override was created under the add-on's `local/transforms.conf` so the classifier could recognize the lab's current header format.

After restarting Splunk, new events began appearing as the correct `pfsense:filterlog` sourcetype, confirming that the **classification stage worked**.

However, the add-on's detailed search-time field extractions still did not reliably populate key firewall fields such as destination, direction, protocol, and action. The parser therefore remained only partially functional.

This became the motivation for potentially building a small custom pfSense parser tailored to the lab's actual event structures.

### Splunk Service Management Note

The Splunk instance was confirmed to be running as root. With the installed Splunk version, lifecycle commands required explicit acknowledgement using `--run-as-root`, for example:

`sudo /opt/splunk/bin/splunk restart --run-as-root`

**Skills practiced:** Splunk sourcetypes, props/transforms concepts, regex troubleshooting, search-time vs parsing-time extraction, Linux config files, add-on installation, root/service management.

---

## 2026-09-17 — INTERNAL Logging Visibility Corrected

**Date confidence: Exact conversation date**

NS-EMPLOYEE-01 was confirmed at:

- IPv4: **192.168.10.103**
- Default gateway: **192.168.10.1**

A search for recent pfSense events containing the employee IP initially returned little or no outbound Internet traffic, despite the machine actively browsing.

The reason was identified as **logging configuration**, not routing failure. The LAN/INTERNAL allow rules were permitting the traffic but were not logging the relevant pass events.

Logging was enabled on the applicable IPv4/IPv6 rules. Afterward, pfSense/Splunk began showing employee traffic such as:

- `192.168.10.103 -> 192.168.10.1`
- `192.168.10.103 -> external Internet destinations`

This demonstrated why WAN-side logs can obscure the original internal source after NAT, while pre-NAT/internal-interface logs preserve the endpoint's private address.

It also reinforced that **“the firewall allowed it” and “the SIEM can see it” are separate questions**.

**Skills practiced:** NAT reasoning, firewall logging strategy, pre-NAT vs post-NAT visibility, Splunk validation.

---

## 2026-09-17 — DHCP / DHCPv6 Log Interpretation

**Date confidence: Exact conversation date**

While reviewing pfSense service logs, DHCP behavior was examined.

Key observations and lessons included:

- DHCPv4 commonly follows **DORA**: Discover -> Offer -> Request -> ACK.
- Lease renewal can produce Request -> ACK without a full new DORA exchange.
- A service log does not necessarily record every packet/message that existed on the wire.
- `dhcp6c` represents a DHCPv6 client process.
- `dhcp6c ... sending SOLICIT` means pfSense is acting as a DHCPv6 client and attempting to locate an upstream DHCPv6 server.
- The four-message DHCPv6 mnemonic used for learning was **SARR**: Solicit -> Advertise -> Request -> Reply.

**Skills practiced:** DHCPv4/DHCPv6 interpretation, service-log vs packet-level visibility, protocol terminology.

---

## 2026-09-17 — Nginx / Apache Port Conflict Troubleshooting

**Date confidence: Exact conversation date**

Apache2 was installed/attempted on the Ubuntu public-facing DMZ server. Apache failed to start with errors indicating that TCP/80 was already in use.

The conflict was traced to **Nginx already listening on port 80**.

Apache was not required for the current design because Nginx was already serving the web-server role. Apache was therefore removed, leaving Nginx as the intended DMZ web service.

During package removal, `apt` temporarily reported that the package-management cache lock was held by an `unattended-upgrade` process. The automatic upgrade was allowed to complete rather than deleting lock files or forcefully interrupting package management.

**Skills practiced:** Linux service troubleshooting, socket/port ownership, `systemctl`, `ss`, apt/dpkg lock behavior, package cleanup.

---

## 2026-09-17 — Full TCP Port Scan Against the DMZ

**Date confidence: Exact conversation date**

KALI-ADVERSARY was used to scan the public-facing DMZ server with Nmap.

A full TCP-port scan produced the following high-level result:

- **443/tcp open (HTTPS)**
- **65,534 TCP ports filtered (no response)**

A SYN scan also identified TCP/443 as open.

This validated the pfSense policy from the attacker's perspective: HTTPS was exposed, while nearly all other unsolicited TCP traffic was filtered.

### SOC-Side Investigation

After the scan, Kali was shut down and the investigation was performed from NS-SOC-01 using Splunk.

The pfSense index was searched using Kali's source IP and the term `block` within the relevant time window.

Approximately **84,393 blocked events** were returned in the 30-minute search window. A much smaller set, approximately **155 pass events**, was also observed.

The event counts were understood as **firewall log records**, not as an exact count of unique ports or unique connection attempts. The large block volume was consistent with the full-port enumeration and possible retries against filtered ports.

This was one of the first clean end-to-end exercises in the lab where the same activity was viewed from both sides:

1. **Attacker perspective:** Nmap identified the reachable service.
2. **Firewall perspective:** pfSense enforced policy and generated block/pass telemetry.
3. **SIEM perspective:** Splunk exposed the concentrated event pattern for investigation.
4. **Analyst perspective:** source IP, timeframe, firewall action, and destination-port behavior were correlated to identify reconnaissance activity.

**Skills practiced:** Nmap, SYN scanning, firewall-policy validation, Splunk investigation, reconnaissance-pattern recognition, event-count interpretation.

---

# Current Capabilities Demonstrated

As of 2026-09-18, the lab demonstrates hands-on experience with:

- Designing and operating a multi-VM SOC lab in VirtualBox.
- Segmenting INTERNAL, DMZ, WAN, and adversary networks through pfSense.
- Writing and validating firewall rules based on trust boundaries.
- Understanding stateful firewall behavior and NAT visibility.
- Running Ubuntu servers primarily through the CLI.
- Installing and operating Splunk Enterprise on Ubuntu.
- Forwarding pfSense syslog through rsyslog into Splunk.
- Troubleshooting timestamps, file monitoring, sourcetypes, and parsing logic.
- Investigating raw firewall logs when normalized parsing is unavailable.
- Installing and troubleshooting a third-party Splunk Technology Add-on.
- Using Nginx as a DMZ web service.
- Troubleshooting Linux service/port conflicts.
- Generating controlled adversarial traffic with Kali/Nmap.
- Investigating that activity from the SOC workstation through Splunk.
- Interpreting basic DHCPv4/DHCPv6 service logs.

---

# Known Gaps / Open Work

The following items remain active areas for development:

- Complete reliable pfSense field extraction/normalization in Splunk.
- Potentially build a custom parser for pfSense filterlog events if the third-party add-on remains unreliable.
- Improve SPL proficiency so investigations can quantify unique destination ports, actions, sources, and time-based patterns without relying heavily on raw-text searches.
- Ingest Nginx access/error logs into Splunk.
- Add Windows telemetry from NS-EMPLOYEE-01, ideally including more detailed endpoint/process visibility.
- Add IDS/IPS telemetry to complement firewall logs.
- Build the planned management/SOC VLAN.
- Develop reusable detections, dashboards, and investigation playbooks.
- Write formal SOC investigation reports for significant lab exercises.
- Improve technical report-writing quality through repeated self-authored reports and review.

---

# Documentation Practice Going Forward

Future entries should ideally be short and contemporaneous. A suggested structure is:

## Date / Session Title

### Objective
What was the goal of the session?

### Work Performed
What systems, commands, rules, or configurations were changed or tested?

### Findings
What did the evidence show?

### Problems / Troubleshooting
What failed, why, and how was it corrected?

### What I Learned
What technical concept became clearer?

### Next Steps
What should be done next?

For significant security events, a separate formal investigation report should be created rather than turning the daily journal entry into a long report.

---

# Accuracy Note

This reconstruction intentionally avoids claiming exact dates where the available chat history did not clearly establish them. It should be reviewed by Benjamin and corrected wherever personal recollection, screenshots, VM snapshot timestamps, or Splunk event timestamps provide better evidence.
### This document has been reviewed by Benjamin San Clemente and is accurate. -Ben S.



## Manual Journal Entries

### 09/24/2026: 
- Work Performed: Log ingestion/parsing in splunk
- Findings: Was able to match up fields of ipv4 and ipv6 logs from pfsense up to the ip version field making a search in spl much easier being able to isolate the two log types.
- Problems / Troubleshooting: still need to create a differentiation for splunk between ipv4 and ipv6 to finish extracting the rest of the fields from each respective log type dependent on ip version.
- What I Learned: I learned the steps and process to parse logs and intend on learning the structure of how to properly parse logs for field extraction to a more valuable skill level.
- Next Steps: as I stated in problems and troubleshooting, the next objective is to finish parsing these logs to completion once i learn the key skill of how to tell splunk to ingest the logs differently depending on ip version.


### 09/25/2026: 
- Work Performed: Log ingestion/parsing in splunk on ipv4 icmp, udp, and common, tcp protocol remaining. also created a pfsense filterlog common field extractor that i am now troubleshooting. may pick this up tomorrow from here
- Findings: was able to successfully create new field extractions within splunk to correctly parse the logs for the fields that varied between different ipv4 log types. began with creating a common ipv4 field extractor until the tos field in which another field extraction was created to specify udp tos ipv4 logs, then proeeded to configure tcp and icmp ipv4 connections. Whilst doing the ipv4 icmp field extraction, i was notified that ipv4 icmp woulod need a common field extractor dependent upon the icmp_id and the icmp_sequence.
- Problems / Troubleshooting: a few syntax errors here (specifically first issue was when i initially wrote the regex to skip 7 spaces to get to the udp defining field from the ip_version field, i didnt add the syntax to specify skip 7 fields and instead wrote it to require 7 fours for the ip_version which was promptly corrected.) a nd there (second issue was once I completed the udp field extractor when i tried to create a table featuring my new fields it produced no results beyond making the table in a time descending order but left the table blank. after a few more searches i went back to the regex used and identified a comma at the last field thus requiring the extractor to require a following field when there was no thus creating no results, after eliminating the ending comma everything worked as intended.) --- I also ran into a minor issue when trying to locate a sample ipv6tcp connection as i had difficulty finding one natrually so i went into the pfsense gui and into the system logs and was able to sort by ip version as well as connection type which then gave me an ipv6 ip address which i copied went back into the splunk search and reporting, did a search "index=pfsese ip_version=6 (ip i had copied) and with that i was able to identify an ipv6 tcp connection thus giving me a sample for extracting those fields in the future. --- I was reminded that the logs are from pfsense' interface perspective and is not reflective of the netwoechork traffic direction but the direction of traffic relative to the pfsense interface. noted.---new issue upon working on the ipv4 icmp echo logs, not all fields are populating when searching splunk with the table command, ---new issue with the pfsense filterlogs common field extractor as it isnt extracting the fields inspite of correct syntax, will continue tomorrow.
- What I Learned: syntax is an extremely critical point in which detail orientation is critical and must be diligently written to avoid common mistakes so you dont have to go through all the regex looking for a single comma that is out of place. and even when you get the syntax right there are a multitude of other things that can happen causing issue with mission objectives.
- Next Steps: parse the logs for pfsense filterlog common, addition of ipv4 tcp, and ipv6 field extraction.


### 9/26/2026: 
- Work Performed: continued field extraction corrections of ipv4 common. changed "direction" field to "pf_direction." completed ipv4 tcp log field extraction. began ipv6 field extraction common.---method of field extraction is using ai to build the regex then testing it to ensure successful extraction.---completed ipv6 pfsense log ingestion into splunk with quality field extraction.---began attempting to install windows sysmon and splunk forwarder to get windows logs into splunk to increase realism and visibility in the lab networks. 
- Findings: upon completion of ipv4 and ipv6 logs from pfsense and getting them completed to a general baseline state i have chosen to progress in adding windows telemetry before extracting the fields of all the source types from pfsense to more quickly begin more on-the-job use of this homelab. intention to circle back down the road once home lab is developed further. ---Ihave also found much difficulty trying to get the windows telemetry to work but despite having issues I believe it is going smoother and faster than my attempt with the pfsense logs and integrating those cleanly. 
- Problems / Troubleshooting:found conflict of "direction" field for ipv4, assumption is use elsewhere causing issues. remedy was changing field name to "pf_direction"--- when testing the regex for ipv6 tcp log extraction, acknowledgment_number field was not populating. discovered my lab only has syn requests no syn ack so no acknowledgement numbers exist, will be force tested later.---when extracting fields from ipv6 icmp logs, it was discovered that the pfsense icmpv6 logs do not contain an icmpv6_type so its been determined that the removal of the icmpv6 feild extraction is unnecessary.---While attempting to set up the sysmon logs for splunk, splunk failed to identify any windows logs, while diagnosing that issue the windows vm treminated itseld, fortunately not during an installation but an issue to be dealt with nonetheless.---windows power issue was resolved, was a simple licesnsing issue that was shutting down the vm and has been corrected by extending the license. now identifying issue with windows log ingestion into splunk.---identified an issue with the forwarder automatically renaming the sysmon logs to a different source type so the app i downloaded into splunk isnt able to extract the fields. trying to navigate the poershell on one computer and getting the information on another computer has proven a pain in the ASS. will continue tomorrow.
- What I Learned: Today was good regex practice, along with more practice in using linux commands, spl searches, and CLI and powershell. I learned more about ipv6 connections and the variance between the logs from ipv4 to ipv6. i learned more about sindows telemetry and how miccrosoft offers security softeware to be added to computers. I am working on expanding my knowledge in problem solving the issues that arise while im trying to establish this lab to a meaningful base level. 
- Next Steps:Tomorrow i intend to correct the issue with the renaming of the sysmon logs at the forwarder layer. If task is completed early and no other task remains, i will circle back to the extracting the few remaining feilds from the pfsense logs.

### 09/27/2026: 
- Work Performed: Correct Sysmon logs within universal log forwarder to stop the forwarder from renaming the logs so splunk can use the sysmon splunk app to extract the fields correctly from the sysmon logs.--- correction to prior statement, the correction was identified as the siem was expecting "XmlWinEventLog:.." while the forwarder was sending "WinEventLog:..." so adjusting what the forwarder was sending and not the renaming was the actual issue.
- Findings: Splunk Universal Forwarder is renaming the XML logs into "Xml Windows Event Logs" when the raw name is what the splunk ingestor wants.---adjustment to prior claim as to what the issue was. It was found that the issue was not the renaming of the logs but of the expected source name was different. the original source was "WinEventLog:..." and the expected source was "XmlWinEventLog:..."---
- Problems / Troubleshooting: the correction was made by going into the windows employee virtual machine, and editing the inputs.conf within the universal forwarder and changed the source to the expected source name.---
- What I Learned: The applications you can use through splunk and I assume with most SIEM's is not smooth in integration but with minor configuration provide a huge amount of benefit and the faster you can identify the misconfigurations (that see ever constant) the sooner you reap the benefits of the entire application youve chosen which in most cases is astronomical in time saving as I've extracted fields manually and this process is faster and has less room for minor errrors in building the feild extraction regex.---
- Next Steps: perform genuine events with the current set up to deepen knowledge and understanding of logs from these new fields and new sources and to begin writing reports. learning the raw logs for xml windows logs would also be useful in my opinion but I'll see if it truly is a strong enough benefit.


### 09/28/2026: 
- Work Performed: Began prodding homelab network with adversary network to generate logs in windows sysmon to deepen understanding of the windows sysmon logs are about and how to use and learn from them, intent is to perform a suspicious powershell execution on the employee VM and observe and read the logs from splunk and identify discrepencies from baseline behaviour and create a report escalating the power shell execution and properly highlight the "who, what, where, when" I am currently under the impression that as a tier 1 soc analyst the scope of work is not to identify intent but to focus on baseline behaviour and deviations frm that.---More specificity on the works performed included using windows CMD on the NS-Employee-01 work station to spawn powershell and to perform a system network config discovery search to replicate what an attack might look like once an attaacker has already infiltrated the network. followed by going into splunk and searching for those logs to learn about sysmon logs and to further my understanding of thistype of event and or situation, identifying more normal powershell behavior compared to deviation from normal ppower shell behaviour as well as being able to identify where certain parts of infotrmation are in the raw sysmon logs (such as what is contained in the command line for the cmd/powershell executions).---
- Findings: I have interacted with my kali linux adversary VM previously and seen all the tools, i was going through them and was able to identify more of the intent of all the tools they have and was able to see their attack process or more so the general attack process just in how they organize the tools they have at their disposal since they are organized by each step. (Recon, Resource dev, initial access, execution, persistence, privilege escalation, defense evasion, credential access, discovery, lateral movement, collection, command and control, exfiltration, impact, forensics, services and other tools)---
- Problems / Troubleshooting:Identified realism error with NS-Employee-01 pc because i had originally built this VM to be the SOC analysts "pc" but realized windows is less efficient and I didnt want logs that im investigating coming from the same pc that im investigating with. so i made a second windows vm and named that soc renamed the original to employee, but i learned the name of the employee vm was still soc in the windows settings so i altered it after having learned of this error from the powershell sysmon logs so redid that and renamed employee workstation to an appropriate name.---
- What I Learned: Began planning for a large incident to cause multiple reports from a progressive attack. I also learned that knowing how to specify the correct fields will vary by log type as they correlate to different aspects of a full system network, some are networking logs like pfsense firewall that focus on IP address source/destination, ip version, protocol used, amount of data transfered, etc. compared to the logs from sysmon which are more so relevant to process creation, user account/permissions, parent/child relationships, essentially more about what the device is doing as opposed to what the device is saying. I'm sure i will experience this again when i ingest the nginx logs into the splunk siem i have set up as those logs will be more similar to networking firewall logs but I wuld guess the focus is on traffic, cross site injection relevant fields, and/or SQL injection relevant fields.---
- Next Steps: finish the report...


### 09/29/2026: 
- Work Performed: finish the report an--- was able to fix a display issue with one of my virtual machines, found out that the display adapter was the basic windows one which limited and constricted my ability to work smoothly and still be able to visibly see the fullscreeen, after installing vbox's display adapter for windows i was able to find a configuration that worked however the issue remains of limiting my SPL searches that feature a table to about 5-7 fields so i can still see them, after that they are off screen and i cannot access.
- Findings: report originally was intended to feature event A-C for different logs for each step of the powershell spawn, however, I found that its probably more realistic to just kinda show the code and fill out the report in a more concise amanner as it is one genuine event with multiple parts, creating one event report is still the logical conclusion. 
- Problems / Troubleshooting: had to think about the realism of the lab and made an adjustment.
- What I Learned: elongating a report to simply satisfy every ounce of detail is in excess of what is expected and of what is beneficial. Practice writing a report with multiple events as a single report if it was a single incident. ensure you capture all relevant information still so a tier 2 soc analyst will be better prepared due to your report but do not waste time trying to report each log relevant. 
- Next Steps: continue building the lab and perhaps reconfigure other vms to be more visually appropriate and correct display resolutions.


