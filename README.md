# Threat Detection & Response with Wazuh

## Comprehensive End-to-End SOC / SIEM / EDR Lab Report

**Project type:** Defensive Security Engineering / SOC Operations / Detection Engineering  
**Primary platform:** Wazuh  
**Virtualization platform:** VirtualBox  
**Host architecture:** Apple Silicon (Apple M5)  
**Internal lab network:** `wazuh-lab` — `192.168.50.0/24`  
**Attacker / test workstation:** Kali Linux — `192.168.50.10`  
**Wazuh Server:** Ubuntu Server — `192.168.50.20`  
**Linux / container endpoint:** Ubuntu Desktop hosting OWASP Juice Shop in Docker — `192.168.50.30` 
**Windows Endpoint:** Windows ARM64 — `192.168.50.40`  
**Application:** OWASP Juice Shop — TCP/3000  

---

# 1. Executive Summary

This project implements a complete threat detection and response laboratory using Wazuh as the central SIEM/EDR platform. The environment was designed to simulate a small but realistic Security Operations Center (SOC) architecture with dedicated systems for attack simulation, endpoint monitoring, containerized application hosting, event collection, detection, investigation, MITRE ATT&CK mapping, and incident analysis.

The lab was rebuilt from the ground up on a MacBook using Apple Silicon. VirtualBox was used to host four primary virtual machines:

- a Wazuh security monitoring server,
- a Kali Linux security-testing workstation,
- a Windows endpoint,
- and an Ubuntu Desktop endpoint hosting OWASP Juice Shop in Docker.

The project focuses on validating the complete detection pipeline rather than simply generating dashboard alerts.

```text
Security activity
      ↓
Endpoint/application telemetry
      ↓
Wazuh agent
      ↓
Wazuh manager
      ↓
Decoder and rule processing
      ↓
Wazuh indexer
      ↓
Threat Hunting
      ↓
MITRE ATT&CK
      ↓
Investigation
      ↓
Response/validation
```

Validated capabilities include:

- Wazuh central infrastructure deployment.
- Multi-platform Wazuh agent deployment.
- Windows process creation monitoring through Event ID `4688`.
- PowerShell Module Logging through Event ID `4103`.
- PowerShell Script Block Logging through Event ID `4104`.
- Script-block content visibility inside Wazuh.
- Native ARM64 Sysmon deployment and Event ID `1`.
- Dockerized OWASP Juice Shop deployment.
- Container JSON-log collection.
- Ubuntu journald and audit telemetry.
- Wazuh Threat Hunting.
- MITRE ATT&CK Dashboard, Framework, and Events views.
- Cross-source event correlation.
- Incident timeline construction.
- Architecture-specific troubleshooting on Apple Silicon.

This project is intended as a practical cybersecurity portfolio demonstrating hands-on security monitoring, detection engineering, endpoint telemetry configuration, container security monitoring, threat hunting, and SOC investigation.

---

# 2. Project Objectives

The objectives of the project were to:

1. Build an isolated cybersecurity laboratory.
2. Deploy Wazuh as a central SIEM/EDR platform.
3. Configure static internal networking for all lab systems.
4. Deploy Wazuh agents to Windows, Ubuntu, and Kali.
5. Deploy OWASP Juice Shop as an intentionally vulnerable web application.
6. Generate controlled security telemetry.
7. Validate Windows Security logging.
8. Validate PowerShell logging.
9. Deploy and validate Sysmon.
10. Collect Linux system and audit telemetry.
11. Collect Docker/container logs.
12. Perform structured threat hunting in Wazuh.
13. Map events to MITRE ATT&CK.
14. Correlate related events.
15. Build a final incident timeline.
16. Document all implementation and troubleshooting decisions.
17. Produce a GitHub-ready technical portfolio.

---

# 3. Skills Demonstrated

## 3.1 Security Operations

- SIEM deployment and administration
- Endpoint detection and response concepts
- Alert triage
- Threat hunting
- Event correlation
- Detection validation
- Incident investigation
- Incident timeline reconstruction
- MITRE ATT&CK mapping
- SOC evidence collection

## 3.2 Windows Security

- Windows Security auditing
- Event ID `4688`
- PowerShell Module Logging
- PowerShell Script Block Logging
- Event IDs `4103` and `4104`
- Sysmon
- Windows Defender telemetry
- Process and parent-process investigation
- Command-line logging
- Group Policy configuration

## 3.3 Linux Security

- journald
- auditd
- systemd
- Linux authentication telemetry
- Wazuh Linux agent administration
- log collection
- service troubleshooting
- file permissions

## 3.4 Container Security

- Docker
- OWASP Juice Shop
- Docker JSON logs
- container lifecycle visibility
- application-log collection

## 3.5 Network / Lab Engineering

- VirtualBox networking
- NAT
- Internal Network
- static IPv4 addressing
- multi-NIC VM design
- inter-VM connectivity validation
- TCP port verification
- Nmap-based service discovery

## 3.6 Apple Silicon / ARM64 Troubleshooting

- Windows ARM64
- Kali ARM64
- ARM64 Sysmon
- kernel/header troubleshooting
- VirtualBox Linux ARM Guest Additions limitations
- architecture-aware software selection

---

# 4. Lab Architecture

## 4.1 Host System

The lab runs on a MacBook with:

```text
Processor: Apple M5
Memory: 24 GB
Storage: 1 TB SSD
Virtualization: VirtualBox
Architecture: ARM64 / Apple Silicon
```

The ARM64 architecture significantly influenced deployment decisions, especially for Windows Sysmon and Linux display integration.

### Evidence Placeholder

> **Insert Screenshot:** Host system / VirtualBox environment  
> **Suggested caption:** *Figure 1 — Apple Silicon host and VirtualBox environment used to run the Wazuh security lab.*

---

# 5. Virtual Machines

| VM | Function | Role |
|---|---|---|
| Wazuh Server | Ubuntu Server | SIEM / EDR management |
| Kali Linux | Kali ARM64 | Controlled security-testing workstation |
| Windows Endpoint | Windows ARM64 | Monitored Windows endpoint |
| Ubuntu Desktop | Ubuntu Desktop | Linux endpoint and Docker host |
| OWASP Juice Shop | Docker container | Intentionally vulnerable web application |

### Evidence Placeholder

> **Insert Screenshot:** VirtualBox VM inventory  
> **Suggested caption:** *Figure 2 — VirtualBox showing the Wazuh Server, Kali, Windows Endpoint, and Ubuntu Desktop virtual machines.*

---

# 6. Network Architecture

The lab uses two network adapters per graphical endpoint/server VM.

## Adapter 1 — NAT

Purpose:

- package installation,
- updates,
- Docker image downloads,
- Wazuh repository access,
- Internet connectivity.

## Adapter 2 — Internal Network

VirtualBox internal network name:

```text
wazuh-lab
```

Subnet:

```text
192.168.50.0/24
```

Confirmed addresses:

```text
Kali Linux        192.168.50.10
Wazuh Server      192.168.50.20
Windows Endpoint  192.168.50.40
Ubuntu Desktop    [INSERT CURRENT UBUNTU INTERNAL IP]
```

The lab network does not use a gateway on the internal adapter. The NAT adapter remains responsible for Internet routing.

```text
                         INTERNET
                            |
                       VirtualBox NAT
                            |
     +----------------------+----------------------+
     |                      |                      |
 Wazuh Server             Kali                Windows
     |                      |                      |
     +----------------------+----------------------+
                            |
                         wazuh-lab
                     192.168.50.0/24
                            |
                     Ubuntu Desktop
                            |
                         Docker
                            |
                      OWASP Juice Shop
                         TCP/3000
```

### Evidence Placeholder

> **Insert Screenshot:** Wazuh Server internal IP  
> **Suggested caption:** *Figure 3 — Wazuh Server configured with internal address 192.168.50.20.*

### Evidence Placeholder

> **Insert Screenshot:** Kali internal IP  
> **Suggested caption:** *Figure 4 — Kali Linux configured with internal address 192.168.50.10.*

### Evidence Placeholder

> **Insert Screenshot:** Windows internal IP  
> **Suggested caption:** *Figure 5 — Windows Endpoint configured with internal address 192.168.50.40.*

### Evidence Placeholder

> **Insert Screenshot:** Ubuntu Desktop internal IP  
> **Suggested caption:** *Figure 6 — Ubuntu Desktop configured on the wazuh-lab network.*

---

# 7. Wazuh Server Deployment

The Wazuh server was installed as an all-in-one central monitoring node containing:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

Service validation:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
sudo systemctl status filebeat
```

Concise health validation:

```bash
sudo systemctl is-active wazuh-manager
sudo systemctl is-active wazuh-indexer
sudo systemctl is-active wazuh-dashboard
sudo systemctl is-active filebeat
```

Expected result:

```text
active
```

for each required component.

The Wazuh server internal address is:

```text
192.168.50.20
```

The dashboard administrator password was changed from the generated installation password to a user-defined password using the supported Wazuh password-management process.

### Evidence Placeholder

> **Insert Screenshot:** Wazuh service status  
> **Suggested caption:** *Figure 7 — Wazuh Manager, Indexer, Dashboard, and Filebeat services running successfully.*

### Evidence Placeholder

> **Insert Screenshot:** Wazuh Dashboard login / main page  
> **Suggested caption:** *Figure 8 — Wazuh Dashboard successfully accessible from the host system.*

---

# 8. Wazuh Agent Deployment

Agents were installed on:

- Kali Linux
- Ubuntu Desktop
- Windows Endpoint

Central agent status can be reviewed from the Wazuh Server:

```bash
sudo /var/ossec/bin/agent_control -lc
```

Linux agent status:

```bash
sudo systemctl status wazuh-agent
```

Windows agent status:

```powershell
Get-Service *wazuh*
```

A major troubleshooting lesson was that the exact Windows service name should be queried instead of assumed. During this project the service was observed as:

```text
WazuhSvc
```

### Evidence Placeholder

> **Insert Screenshot:** Wazuh Endpoints / Agents view  
> **Suggested caption:** *Figure 9 — Wazuh Dashboard showing the monitored endpoints and their connection status.*

---

# 9. Kali Linux Security-Testing Workstation

Kali acts as the controlled adversary-simulation workstation.

Confirmed internal IP:

```text
192.168.50.10
```

Validation:

```bash
ip -br addr
ip route
```

Connectivity checks:

```bash
ping -c 3 192.168.50.20
ping -c 3 192.168.50.40
```

Controlled network discovery:

```bash
nmap -n -Pn -sT -sV <TARGET-IP>
```

Juice Shop port validation:

```bash
nmap -n -Pn -sT -sV -p 3000 <UBUNTU-IP>
```

All testing is restricted to the private `192.168.50.0/24` lab network.

### Evidence Placeholder

> **Insert Screenshot:** Kali IP and connectivity  
> **Suggested caption:** *Figure 10 — Kali Linux using 192.168.50.10 on the isolated wazuh-lab network.*

---

# 10. OWASP Juice Shop Deployment

OWASP Juice Shop was deployed on Ubuntu Desktop in Docker.

Container verification:

```bash
docker ps
```

Container name:

```text
juice-shop
```

Application port:

```text
3000/tcp
```

Reachability test:

```bash
curl -I http://<UBUNTU-IP>:3000
```

Container status:

```bash
docker ps --filter name=juice-shop
```

Logs:

```bash
docker logs --tail 50 juice-shop
```

Container ID:

```bash
docker inspect -f '{{.Id}}' juice-shop
```

The application was successfully accessed from Kali across the internal lab network.

### Evidence Placeholder

> **Insert Screenshot:** `docker ps` showing Juice Shop  
> **Suggested caption:** *Figure 11 — OWASP Juice Shop running as a Docker container on Ubuntu Desktop.*

### Evidence Placeholder

> **Insert Screenshot:** Juice Shop opened from Kali  
> **Suggested caption:** *Figure 12 — OWASP Juice Shop successfully accessed from Kali over the internal lab network.*

---

# 11. Docker JSON Logging

Docker stores container logs under:

```text
/var/lib/docker/containers/<container-id>/<container-id>-json.log
```

Reusable commands:

```bash
ID=$(docker inspect -f '{{.Id}}' juice-shop)
sudo tail -n 20 /var/lib/docker/containers/$ID/$ID-json.log
```

Live monitoring:

```bash
sudo tail -f /var/lib/docker/containers/$ID/$ID-json.log
```

Wazuh collection configuration:

```xml
<localfile>
  <location>/var/lib/docker/containers/*/*-json.log</location>
  <log_format>json</log_format>
</localfile>
```

Restart agent:

```bash
sudo systemctl restart wazuh-agent
```

Validation:

```bash
sudo systemctl status wazuh-agent --no-pager
```

A key troubleshooting point was the correct Docker log path.

Incorrect:

```text
/var/lib/docker/containers/<ID>-json.log
```

Correct:

```text
/var/lib/docker/containers/<ID>/<ID>-json.log
```

### Evidence Placeholder

> **Insert Screenshot:** Docker JSON log output  
> **Suggested caption:** *Figure 13 — Raw Docker JSON logs for the Juice Shop container.*

### Evidence Placeholder

> **Insert Screenshot:** Wazuh Docker localfile configuration  
> **Suggested caption:** *Figure 14 — Ubuntu Wazuh agent configured to collect Docker JSON logs.*

---

# 12. Ubuntu Desktop Logging

The Ubuntu Wazuh agent was configured to collect journald directly.

Observed configuration:

```xml
<localfile>
  <log_format>journald</log_format>
  <location>journald</location>
</localfile>
```

Additional local sources included:

```text
/var/ossec/logs/active-responses.log
/var/log/dpkg.log
```

This provides operating-system telemetry without requiring all events to be duplicated into traditional syslog files.

### Evidence Placeholder

> **Insert Screenshot:** Ubuntu ossec.conf journald configuration  
> **Suggested caption:** *Figure 15 — Wazuh agent configured to collect Ubuntu journald telemetry.*

---

# 13. auditd Integration

Auditd was added to increase Linux visibility.

Installation:

```bash
sudo apt update
sudo apt install auditd audispd-plugins -y
```

Enable:

```bash
sudo systemctl enable --now auditd
```

Validation:

```bash
sudo systemctl status auditd --no-pager
sudo auditctl -s
sudo auditctl -l
```

Wazuh audit collection:

```xml
<localfile>
  <location>/var/log/audit/audit.log</location>
  <log_format>audit</log_format>
</localfile>
```

Recent events:

```bash
sudo ausearch -ts recent
```

### Evidence Placeholder

> **Insert Screenshot:** auditd status / audit rules  
> **Suggested caption:** *Figure 16 — auditd enabled on Ubuntu Desktop for Linux audit telemetry.*

### Evidence Placeholder

> **Insert Screenshot:** Wazuh audit log collection block  
> **Suggested caption:** *Figure 17 — Wazuh configured to collect `/var/log/audit/audit.log`.*

---

# 14. Windows Endpoint Networking

Confirmed internal IP:

```text
192.168.50.40
```

Interface validation:

```powershell
Get-NetAdapter
Get-NetIPAddress -AddressFamily IPv4
```

Wazuh connectivity:

```powershell
ping 192.168.50.20
```

Port checks:

```powershell
Test-NetConnection 192.168.50.20 -Port 1514
Test-NetConnection 192.168.50.20 -Port 1515
```

Successful communication:

```text
TcpTestSucceeded : True
```

### Evidence Placeholder

> **Insert Screenshot:** Windows network configuration  
> **Suggested caption:** *Figure 18 — Windows Endpoint configured as 192.168.50.40 on the wazuh-lab network.*

---

# 15. Windows Security Auditing

Advanced Windows auditing was enabled to increase endpoint visibility.

Configured categories included:

- Logon
- Logoff
- Account Lockout
- User Account Management
- Security Group Management
- Process Creation
- Process Termination
- Filtering Platform Connection

Example:

```powershell
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
```

Validation:

```powershell
auditpol /get /category:*
```

---

# 16. Event ID 4688 — Process Creation

Windows process creation auditing was enabled.

Group Policy path:

```text
Computer Configuration
  → Administrative Templates
    → System
      → Audit Process Creation
        → Include command line in process creation events
```

Local verification:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4688
} -MaxEvents 5 | Format-List TimeCreated,Id,Message
```

Wazuh successfully received and indexed process-creation telemetry from:

```text
agent.name: Windows-Endpoint
agent.ip: 192.168.50.40
```

Observed fields included:

```text
data.win.eventdata.newProcessId
data.win.eventdata.newProcessName
data.win.eventdata.parentProcessName
data.win.eventdata.processId
data.win.eventdata.subjectUserName
data.win.system.eventID
```

### Evidence Placeholder

> **Insert Screenshot:** Local Windows 4688 event  
> **Suggested caption:** *Figure 19 — Windows Security Event ID 4688 confirming process-creation auditing.*

### Evidence Placeholder

> **Insert Screenshot:** 4688 event in Wazuh  
> **Suggested caption:** *Figure 20 — Wazuh Threat Hunting event showing Windows process creation from Windows-Endpoint.*

---

# 17. PowerShell Module Logging — Event ID 4103

PowerShell Module Logging was enabled through:

```text
Computer Configuration
  → Administrative Templates
    → Windows Components
      → Windows PowerShell
        → Turn on Module Logging
```

Module list:

```text
*
```

Registry validation confirmed:

```text
EnableModuleLogging    REG_DWORD    0x1
```

Policy refresh:

```powershell
gpupdate /force
```

Test activity:

```powershell
Import-Module Microsoft.PowerShell.Management
Get-Process
Get-Service
Get-ChildItem C:\
```

Local verification:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-PowerShell/Operational'
    Id=4103
} -MaxEvents 5 | Format-List TimeCreated,Id,Message
```

Event ID `4103` was successfully generated locally.

A major lesson was that an event can exist locally without appearing in the Wazuh alert index if no qualifying alert rule is triggered.

### Evidence Placeholder

> **Insert Screenshot:** Local PowerShell Event ID 4103  
> **Suggested caption:** *Figure 21 — PowerShell Module Logging Event ID 4103 generated on the Windows endpoint.*

---

# 18. PowerShell Script Block Logging — Event ID 4104

Script Block Logging was enabled through:

```text
Computer Configuration
  → Administrative Templates
    → Windows Components
      → Windows PowerShell
        → Turn on PowerShell Script Block Logging
```

Wazuh collection:

```xml
<localfile>
  <location>Microsoft-Windows-PowerShell/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Local validation:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-PowerShell/Operational'
    Id=4104
} -MaxEvents 5 | Format-List TimeCreated,Id,Message
```

Wazuh successfully received Event ID `4104`.

Observed Wazuh fields included:

```text
data.win.system.channel:
Microsoft-Windows-PowerShell/Operational

data.win.system.eventID:
4104
```

The Wazuh alert also displayed:

```text
data.win.eventdata.scriptBlockText
```

Example content observed during the lab:

```text
Get-ChildItem C:\
```

This validated the complete PowerShell telemetry path from script execution to SIEM visibility.

### Evidence Placeholder

> **Insert Screenshot:** Local 4104 event  
> **Suggested caption:** *Figure 22 — PowerShell Script Block Logging Event ID 4104 generated locally.*

### Evidence Placeholder

> **Insert Screenshot:** 4104 in Wazuh  
> **Suggested caption:** *Figure 23 — Wazuh Threat Hunting showing PowerShell Event ID 4104 from Windows-Endpoint.*

### Evidence Placeholder

> **Insert Screenshot:** Expanded 4104 event showing scriptBlockText  
> **Suggested caption:** *Figure 24 — Expanded Wazuh 4104 event displaying the captured PowerShell `scriptBlockText`.*

---

# 19. Threat-Hunting Queries

Correct Wazuh Windows Event ID field:

```text
data.win.system.eventID
```

Windows endpoint:

```text
agent.name:"Windows-Endpoint"
```

PowerShell 4104:

```text
data.win.system.eventID:"4104"
```

Combined:

```text
agent.name:"Windows-Endpoint" AND data.win.system.eventID:"4104"
```

Process creation:

```text
agent.name:"Windows-Endpoint" AND data.win.system.eventID:"4688"
```

The project demonstrated why parsed-field filtering is more reliable than generic free-text searches.

---

# 20. Sysmon on Windows ARM64

The Windows guest architecture was verified with:

```powershell
$env:PROCESSOR_ARCHITECTURE
```

Result:

```text
ARM64
```

The downloaded Sysmon package contained:

```text
Sysmon.exe
Sysmon64.exe
Sysmon64a.exe
```

Initial use of:

```text
Sysmon64.exe
```

caused:

```text
The driver has been blocked from loading
```

and left a partially registered `Sysmon64` service.

The correct executable for Windows ARM64 was:

```text
Sysmon64a.exe
```

After stale service cleanup, installation used:

```powershell
cd C:\Tools\Sysmon
.\Sysmon64a.exe -accepteula -i .\sysmonconfig.xml
```

Service validation:

```powershell
Get-Service Sysmon*
```

Event validation:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    Id=1
} -MaxEvents 5 | Format-List TimeCreated,Id,Message
```

Sysmon Event ID `1` was successfully generated locally.

Wazuh collection:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

### Evidence Placeholder

> **Insert Screenshot:** Windows ARM64 architecture  
> **Suggested caption:** *Figure 25 — Windows endpoint confirming ARM64 architecture.*

### Evidence Placeholder

> **Insert Screenshot:** Sysmon service running  
> **Suggested caption:** *Figure 26 — ARM64 Sysmon installed and running successfully.*

### Evidence Placeholder

> **Insert Screenshot:** Sysmon Event ID 1  
> **Suggested caption:** *Figure 27 — Sysmon Event ID 1 showing process creation telemetry.*

---

# 21. Sysmon Investigation Value

Sysmon Event ID `1` provides:

```text
Image
CommandLine
ProcessId
ParentImage
ParentProcessId
User
Hashes
```

This complements Windows Security and PowerShell telemetry.

Correlation model:

```text
4688
   ↓
Windows process creation

Sysmon 1
   ↓
richer process / parent metadata

4104
   ↓
PowerShell script block

Wazuh
   ↓
alert / MITRE ATT&CK / investigation
```

---

# 22. Windows Defender Telemetry

The Windows Wazuh agent supports Defender Operational logging:

```xml
<localfile>
  <location>Microsoft-Windows-Windows Defender/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Local inspection:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Windows Defender/Operational" -MaxEvents 10 |
Select-Object TimeCreated,Id,Message
```

The lab does not require real malware to demonstrate Defender integration.

### Evidence Placeholder

> **Insert Screenshot:** Defender Operational event or Wazuh Defender event  
> **Suggested caption:** *Figure 28 — Windows Defender Operational telemetry available for Wazuh collection.*

---

# 23. Raw Event Archiving

An important Wazuh troubleshooting concept was the difference between raw received events and indexed alerts.

Temporary manager configuration:

```xml
<logall_json>yes</logall_json>
```

Restart:

```bash
sudo systemctl restart wazuh-manager
```

Raw events are written to:

```text
/var/ossec/logs/archives/archives.json
```

Example:

```bash
sudo grep '4103' /var/ossec/logs/archives/archives.json | tail -10
```

After troubleshooting:

```xml
<logall_json>no</logall_json>
```

should be restored to reduce unnecessary disk consumption.

---

# 24. MITRE ATT&CK Integration

Wazuh's MITRE ATT&CK module was successfully populated by real Windows endpoint detections.

## 24.1 MITRE Dashboard

The Dashboard showed:

- alert evolution,
- attacks by tactic,
- rule levels by attack,
- top tactics.

### Evidence Placeholder

> **Insert Screenshot:** MITRE Dashboard  
> **Suggested caption:** *Figure 29 — Wazuh MITRE ATT&CK Dashboard summarizing detected tactics and alert activity.*

## 24.2 MITRE Framework

The Framework view displayed mapped tactics and techniques.

Visible techniques included:

```text
T1112       Modify Registry
T1059       Command and Scripting Interpreter
T1059.001   PowerShell
T1105       Ingress Tool Transfer
T1087       Account Discovery
T1053       Scheduled Task/Job
T1485       Data Destruction
T1562.001   Disable or Modify Tools
T1543.003   Windows Service
T1057       Process Discovery
```

Visible tactic categories included:

```text
Defense Evasion
Impact
Command and Control
Execution
Discovery
Persistence
Privilege Escalation
Lateral Movement
Collection
```

### Evidence Placeholder

> **Insert Screenshot:** MITRE Framework  
> **Suggested caption:** *Figure 30 — Wazuh MITRE ATT&CK Framework showing mapped techniques generated by lab activity.*

## 24.3 MITRE Events

The Events view exposed:

```text
timestamp
agent.name
rule.mitre.id
rule.mitre.tactic
rule.description
rule.level
rule.id
```

This connected ATT&CK techniques directly to indexed endpoint events.

### Evidence Placeholder

> **Insert Screenshot:** MITRE Events  
> **Suggested caption:** *Figure 31 — Wazuh MITRE Events view linking real Windows endpoint detections to ATT&CK techniques.*

### Evidence Placeholder

> **Insert Screenshot:** Expanded ATT&CK event  
> **Suggested caption:** *Figure 32 — Expanded Wazuh event showing ATT&CK metadata, rule information, endpoint, and timestamp.*

---

# 25. Threat-Hunting Methodology

The structured investigation workflow used in the lab is:

```text
1. Identify the endpoint
2. Identify the timestamp
3. Identify the event source
4. Identify the Wazuh rule
5. Inspect process or script details
6. Inspect parent process
7. Identify the user
8. Search surrounding events
9. Correlate related telemetry
10. Review MITRE ATT&CK mapping
11. Classify activity
12. Preserve evidence
```

Example Windows PowerShell correlation:

```text
4104
   ↓
scriptBlockText
   ↓
process ID
   ↓
4688
   ↓
Sysmon Event 1
   ↓
parent process
   ↓
user
   ↓
MITRE ATT&CK
```

---

# 26. Incident Timeline Methodology

Every controlled scenario should begin with a timestamp.

Linux/Kali:

```bash
date -Is
```

Windows:

```powershell
Get-Date
```

Linux journal timestamps can be normalized with:

```bash
journalctl -o short-iso
```

Recommended reporting convention:

```text
All incident timestamps are normalized to America/Toronto local time.
```

The final timeline must contain actual observed timestamps only.

---

# 27. Final Incident Timeline

Use the table below and replace all placeholders with actual evidence from the final lab execution.

| Timestamp | Source | Event | Wazuh / Local Evidence | Rule / ATT&CK | Response |
|---|---|---|---|---|---|
| [INSERT] | Kali `192.168.50.10` | Initial controlled discovery activity | [INSERT] | [INSERT] | None |
| [INSERT] | Target endpoint | Endpoint telemetry generated | [INSERT] | [INSERT] | None |
| [INSERT] | Wazuh | Detection indexed | [INSERT] | [INSERT] | Investigation initiated |
| [INSERT] | Analyst | Related events correlated | [INSERT] | [INSERT] | Decision |
| [INSERT] | Endpoint / Wazuh | Response activity | [INSERT] | [INSERT] | Containment |
| [INSERT] | Analyst | Containment validated | [INSERT] | — | Confirmed |
| [INSERT] | Environment | Recovery / service restored | [INSERT] | — | Incident closed |

### Evidence Placeholder

> **Insert Screenshot:** Final incident timeline  
> **Suggested caption:** *Figure 33 — Final incident timeline correlating attacker activity, endpoint telemetry, Wazuh detection, investigation, and response.*

---

# 28. Evidence Matrix

Populate this table with your final screenshots.

| ID | Evidence | File / Screenshot |
|---|---|---|
| E01 | VirtualBox lab topology | [INSERT] |
| E02 | Wazuh agents active | [INSERT] |
| E03 | Kali `192.168.50.10` | [INSERT] |
| E04 | Wazuh Server `192.168.50.20` | [INSERT] |
| E05 | Windows Endpoint `192.168.50.40` | [INSERT] |
| E06 | Ubuntu Desktop network | [INSERT] |
| E07 | Juice Shop running in Docker | [INSERT] |
| E08 | Juice Shop accessible from Kali | [INSERT] |
| E09 | Windows 4688 local event | [INSERT] |
| E10 | Windows 4688 in Wazuh | [INSERT] |
| E11 | PowerShell 4103 local | [INSERT] |
| E12 | PowerShell 4104 local | [INSERT] |
| E13 | PowerShell 4104 in Wazuh | [INSERT] |
| E14 | Expanded 4104 `scriptBlockText` | [INSERT] |
| E15 | ARM64 Sysmon | [INSERT] |
| E16 | Sysmon Event ID 1 | [INSERT] |
| E17 | Docker JSON log | [INSERT] |
| E18 | auditd | [INSERT] |
| E19 | MITRE Dashboard | [INSERT] |
| E20 | MITRE Framework | [INSERT] |
| E21 | MITRE Events | [INSERT] |
| E22 | Final incident timeline | [INSERT] |

---

# 29. Validated Results

| Capability | Status |
|---|---|
| Wazuh server deployment | Validated |
| Wazuh Dashboard access | Validated |
| Kali internal IP | `192.168.50.10` |
| Wazuh internal IP | `192.168.50.20` |
| Windows internal IP | `192.168.50.40` |
| Juice Shop container | Validated |
| Juice Shop reachable from Kali | Validated |
| Windows Wazuh agent | Validated |
| Ubuntu Wazuh agent configuration | Validated |
| Windows Event ID 4688 locally | Validated |
| Windows Event ID 4688 in Wazuh | Validated |
| PowerShell 4103 locally | Validated |
| PowerShell 4104 locally | Validated |
| PowerShell 4104 in Wazuh | Validated |
| PowerShell scriptBlockText in Wazuh | Validated |
| Windows ARM64 Sysmon | Validated |
| Sysmon Event ID 1 locally | Validated |
| Docker JSON log path | Validated |
| journald Wazuh collection | Validated |
| MITRE Dashboard populated | Validated |
| MITRE Framework populated | Validated |
| MITRE Events populated | Validated |

---

# 30. Additional Detection and Response Scenarios

The following sections are intentionally retained for evidence-backed completion. Insert your screenshots and mark each scenario as completed only after validating it in the rebuilt lab.

## 30.1 File Integrity Monitoring

Controlled directory:

```text
/opt/wazuh-lab
```

Potential configuration:

```xml
<directories realtime="yes" whodata="yes" report_changes="yes">/opt/wazuh-lab</directories>
```

### Evidence Placeholder

> **Insert Screenshot:** FIM configuration  
> **Suggested caption:** *Figure 34 — Wazuh FIM configured for controlled monitoring of `/opt/wazuh-lab`.*

### Evidence Placeholder

> **Insert Screenshot:** FIM alert  
> **Suggested caption:** *Figure 35 — Wazuh FIM alert generated by a controlled file modification.*

---

## 30.2 Active Response

Use a controlled detection and response scenario only within the isolated lab.

Required final evidence:

- triggering alert,
- Active Response execution,
- endpoint firewall change,
- connectivity validation,
- automatic or manual recovery.

### Evidence Placeholder

> **Insert Screenshot:** Detection triggering Active Response  
> **Suggested caption:** *Figure 36 — Wazuh alert used to trigger a controlled Active Response.*

### Evidence Placeholder

> **Insert Screenshot:** Active Response log  
> **Suggested caption:** *Figure 37 — Endpoint Active Response log confirming execution.*

### Evidence Placeholder

> **Insert Screenshot:** Firewall containment  
> **Suggested caption:** *Figure 38 — Endpoint firewall state showing temporary containment of the test source.*

### Evidence Placeholder

> **Insert Screenshot:** Recovery  
> **Suggested caption:** *Figure 39 — Connectivity restored after containment timeout or rollback.*

---

## 30.3 Suricata IDS Integration

Potential source:

```text
/var/log/suricata/eve.json
```

Wazuh collection:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

### Evidence Placeholder

> **Insert Screenshot:** Suricata event  
> **Suggested caption:** *Figure 40 — Suricata network telemetry generated by controlled lab activity.*

### Evidence Placeholder

> **Insert Screenshot:** Suricata event in Wazuh  
> **Suggested caption:** *Figure 41 — Wazuh displaying Suricata IDS telemetry from the Ubuntu endpoint.*

---

## 30.4 Vulnerability Detection

Record actual Wazuh findings only.

Recommended fields:

```text
Endpoint
CVE
Package
Installed version
Severity
CVSS
Status
Remediation
```

### Evidence Placeholder

> **Insert Screenshot:** Wazuh vulnerability view  
> **Suggested caption:** *Figure 42 — Wazuh Vulnerability Detection view for a monitored endpoint.*

---

# 31. Major Troubleshooting Lessons

## 31.1 Collected Event vs Alert

A major lesson was:

```text
event generation
≠
event forwarding
≠
rule match
≠
alert indexing
```

Each layer requires independent validation.

## 31.2 Structured Field Queries

Correct:

```text
data.win.system.eventID:"4104"
```

Better than:

```text
"4104"
```

## 31.3 Verify Agent Service First

A major Windows issue was:

```text
WazuhSvc = Stopped
```

Always check:

```powershell
Get-Service *wazuh*
```

before modifying rules or manager settings.

## 31.4 ARM64 Architecture Matters

Windows:

```text
ARM64
```

Incorrect Sysmon:

```text
Sysmon64.exe
```

Correct:

```text
Sysmon64a.exe
```

## 31.5 Linux ARM VirtualBox Limitation

Kali ARM64 produced:

```text
Detected unsupported arm64 machine type
```

when attempting traditional Linux Guest Additions.

## 31.6 Kernel/Header Synchronization

Kali was synchronized to:

```text
7.1.5+kali1-arm64
```

with:

```text
/lib/modules/7.1.5+kali1-arm64/build
```

available.

## 31.7 Docker Log Path

Incorrect:

```text
/var/lib/docker/containers/<ID>-json.log
```

Correct:

```text
/var/lib/docker/containers/<ID>/<ID>-json.log
```

---

# 32. Security Boundaries

All security-testing activity is restricted to the isolated lab network:

```text
192.168.50.0/24
```

The project does not require:

- public target scanning,
- unauthorized access,
- real malware,
- destructive payloads,
- credential theft,
- production testing.

OWASP Juice Shop is used strictly as a local intentionally vulnerable training application.

---

# 33. Portfolio Relevance

This project demonstrates hands-on experience directly relevant to:

- SOC Analyst
- Cybersecurity Analyst
- Detection Engineer
- Incident Response Analyst
- SIEM Engineer
- Security Operations Engineer
- Blue Team Analyst

The lab demonstrates the ability to:

- deploy security monitoring infrastructure,
- configure endpoint telemetry,
- troubleshoot agents,
- investigate structured event fields,
- correlate Windows telemetry,
- monitor containers,
- interpret MITRE ATT&CK,
- reconstruct incident timelines,
- document technical findings,
- and validate detection pipelines.

---

# 34. Suggested GitHub Repository Structure

```text
Threat-Detection-Response-Wazuh/
│
├── README.md
│
├── docs/
│   ├── architecture.md
│   ├── installation.md
│   ├── windows-telemetry.md
│   ├── linux-telemetry.md
│   ├── docker-monitoring.md
│   ├── mitre-attack.md
│   ├── incident-timeline.md
│   └── troubleshooting.md
│
├── configs/
│   ├── windows-ossec-example.xml
│   ├── ubuntu-ossec-example.xml
│   ├── sysmonconfig.xml
│   └── local-rules-example.xml
│
├── evidence/
│   ├── wazuh/
│   ├── windows/
│   ├── linux/
│   ├── docker/
│   └── mitre/
│
└── diagrams/
    └── architecture.png
```

Do not upload:

- passwords,
- API keys,
- private keys,
- authentication tokens,
- sensitive personal information.

---

# 35. Recommended Screenshot Order

1. VirtualBox topology
2. Wazuh agents
3. Kali IP
4. Wazuh IP
5. Windows IP
6. Ubuntu IP
7. Juice Shop Docker container
8. Juice Shop in Kali
9. Windows 4688 local
10. Windows 4688 in Wazuh
11. PowerShell 4103
12. PowerShell 4104 local
13. PowerShell 4104 in Wazuh
14. Expanded `scriptBlockText`
15. Sysmon ARM64
16. Sysmon Event ID 1
17. Docker logs
18. auditd
19. MITRE Dashboard
20. MITRE Framework
21. MITRE Events
22. FIM
23. Active Response
24. Vulnerability Detection
25. Final incident timeline

---

# 36. Final Technical Architecture

```text
                         +-------------------------+
                         |       MacBook Host      |
                         |        Apple M5         |
                         |      VirtualBox         |
                         +-----------+-------------+
                                     |
                              192.168.50.0/24
                                     |
         +---------------------------+----------------------------+
         |                           |                            |
+--------v---------+        +--------v---------+         +--------v---------+
|      Kali        |        | Ubuntu Desktop   |         | Windows Endpoint |
| 192.168.50.10    |        | Docker Host      |         | 192.168.50.40    |
| Controlled tests |        | OWASP Juice Shop |         | Security 4688    |
| Service discovery|        | journald         |         | PowerShell       |
| HTTP validation  |        | auditd           |         | Sysmon           |
+------------------+        | Docker JSON logs |         | Defender         |
                            +--------+---------+         +--------+---------+
                                     |                            |
                                     +-------------+--------------+
                                                   |
                                          Wazuh Agent Telemetry
                                                   |
                                                   v
                                     +-----------------------------+
                                     |         Wazuh Server        |
                                     |        192.168.50.20        |
                                     | Manager / Indexer /         |
                                     | Dashboard / Filebeat        |
                                     +--------------+--------------+
                                                    |
                         +--------------------------+----------------------+
                         |                          |                      |
                         v                          v                      v
                  Threat Hunting              MITRE ATT&CK          Investigation
                  Event Search                Dashboard             Correlation
                  Rule Analysis               Framework             Timeline
                                              Events
```

---

# 37. Conclusion

The Threat Detection & Response with Wazuh project establishes a functional, multi-platform security-monitoring laboratory on Apple Silicon.

The environment integrates:

```text
Wazuh
Windows
PowerShell
Sysmon
Linux
journald
auditd
Docker
OWASP Juice Shop
Kali Linux
MITRE ATT&CK
```

The strongest validated telemetry chain is:

```text
PowerShell command
        ↓
Windows Event 4104
        ↓
Wazuh Windows Agent
        ↓
Wazuh Manager
        ↓
Indexed event / alert
        ↓
scriptBlockText
        ↓
MITRE ATT&CK
        ↓
SOC investigation
```

Windows Event ID `4688` provides native process-creation visibility, while Sysmon Event ID `1` provides richer process, parent-process, and command-line context.

The project demonstrates that effective detection engineering requires more than enabling logging. Every layer must be independently validated:

```text
Generation
   ↓
Collection
   ↓
Forwarding
   ↓
Parsing
   ↓
Rule evaluation
   ↓
Indexing
   ↓
Analyst visibility
```

The ARM64-specific troubleshooting performed during the project also demonstrates the importance of architecture awareness when building modern cybersecurity labs on Apple Silicon.

The final result is a reusable SOC environment capable of supporting continued threat hunting, detection engineering, container security monitoring, incident response, MITRE ATT&CK analysis, FIM, Active Response, IDS integration, and vulnerability-management exercises.

---

# 38. Current Project Status

## Completed / Validated

- Wazuh infrastructure
- isolated network
- Wazuh agents
- Juice Shop deployment
- Kali reachability
- Windows Security auditing
- Event ID `4688`
- PowerShell Module Logging
- PowerShell Script Block Logging
- Event ID `4104` in Wazuh
- script-block content in Wazuh
- ARM64 Sysmon
- Sysmon Event ID `1`
- Docker JSON log discovery
- Ubuntu journald collection
- MITRE Dashboard
- MITRE Framework
- MITRE Events
- incident timeline methodology

## Evidence to Insert / Finalize

- Ubuntu exact internal IP
- FIM evidence
- Active Response evidence
- Suricata evidence, if included
- Vulnerability Detection evidence
- final incident timeline
- final screenshots throughout this report

---

# 39. Security Engineering Principle

> A detection is not complete because an alert exists. It is complete when the telemetry source is understood, collection is verified, the event is explainable, the detection is reproducible, the investigation is evidence-based, and the response can be validated.
