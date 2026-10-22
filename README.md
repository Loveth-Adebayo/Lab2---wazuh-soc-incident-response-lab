
# Wazuh SOC: Threat Hunting & Incident Response Lab

## Project Overview
This repository documents a simulated Security Operations Center (SOC) Level 2 investigation. The project demonstrates proactive threat hunting, deep-dive incident analysis, and automated threat containment using the Wazuh SIEM platform. 

The scenario involves investigating a suspected credential compromise on a Windows endpoint, correlating multiple security events to reconstruct an attack timeline, and configuring an automated firewall block to contain the threat.

## Architecture
The lab was built using VirtualBox with the following architecture:
- **Wazuh Manager, Indexer & Dashboard:** Deployed on an Ubuntu Server Virtual Machine.
- **Wazuh Agent:** Installed on a Windows 10 Host Machine (Agent Name: DESKTOP-ADDFGV1).
- **Network:** Both machines communicated via a Bridged Adapter setup.

## Key Phases Implemented

### Phase 1: Proactive Threat Hunting
- Enabled Windows Audit Policies for Process Creation and PowerShell Script Block Logging.
- Investigated Windows Security Event IDs 4624 (Successful Logon) and 4625 (Failed Logon) to identify brute-force patterns.
- Hunted for process creation events (Event ID 4688) using Wazuh queries to identify suspicious command-line activity.

### Phase 2: Deep-Dive Incident Analysis
- Correlated failed logins with successful logins and subsequent PowerShell execution.
- Investigated PowerShell Script Block Logging (Event ID 4104).
- Simulated a malicious payload download (`Invoke-WebRequest`) and captured the resulting File Integrity Monitoring (FIM) alert.

### Phase 3: Active Response & Containment
- Configured Wazuh Active Response on the Manager.
- Implemented the `firewall-drop` command to automatically block an attacking IP address for 600 seconds (10 minutes).
- Verified successful containment by analyzing the `active-responses.log`.

## Key Evidence & Findings
- **IOCs Identified:** Malicious domain (`malicious-domain.com`), payload file (`payload.exe`), and the attacking IP address.
- **MITRE ATT&CK Mapping:** 
  - T1110: Brute Force
  - T1059.001: PowerShell
  - T1105: Ingress Tool Transfer
- **Containment:** Automated IP block executed successfully via Wazuh Active Response.

## Configuration Snippets

**Wazuh Agent Configuration (Windows) - Enabling PowerShell Logging:**
```xml
<localfile>
  <location>Microsoft-Windows-PowerShell/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

**Wazuh Manager Configuration (Ubuntu) - Active Response:**
```xml
<command>
  <name>firewall-drop</name>
  <executable>firewall-drop</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>

<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5710,60122</rules_id> 
  <timeout>600</timeout>
</active-response>
```

## Technical Challenges Resolved
- **Network Disconnects:** Resolved VirtualBox Bridged Adapter dropping IP addresses by manually refreshing DHCP leases and toggling the virtual network cable.
- **Missing Event Logs:** Enabled Windows Audit Policies (`auditpol`) and registry keys to force the OS to generate Event IDs 4688 and 4104 for Wazuh to ingest.
- **RAM Constraints:** Managed limited host RAM by configuring systemd timeouts (`TimeoutStartSec=300`) to prevent the Wazuh Indexer from failing during startup.

## Author
**Loveth Adebayo**
*Cybersecurity Analyst | SOC Lab Project*
