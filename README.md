Home SIEM Lab: Threat Detection & Log Analysis

A self-hosted Security Information and Event Management (SIEM) lab built to practice log collection, threat detection, alert tuning, and incident visualization — using free, open-source tools on a MacBook. Includes two live endpoints: an Ubuntu Server and a Windows 11 (ARM64) machine, both actively reporting to a single Wazuh manager.
Overview

This project simulates a small enterprise security monitoring environment. It covers the full detection lifecycle: from raw log collection on endpoints, to correlation and alerting in a SIEM, to dashboards summarizing detected activity.

Goal: Understand how a SOC analyst uses a SIEM to detect and investigate suspicious behavior — brute-force login attempts, unauthorized account creation, and suspicious command execution.

```
┌─────────────────┐        ┌─────────────────┐
│  Ubuntu VM       │        │  Windows VM      │
│  (Wazuh Agent)   │        │  (Wazuh Agent)   │
└────────┬─────────┘        └────────┬─────────┘
         │        Encrypted log forwarding      │
         └───────────────┬───────────────────────┘
                          ▼
              ┌───────────────────────┐
              │   Wazuh Manager        │
              │   (Docker, on MacBook) │
              │   - Log ingestion      │
              │   - Rule correlation   │
              │   - Alerting engine    │
              └───────────┬────────────┘
                          ▼
              ┌───────────────────────┐
              │   Wazuh Dashboard      │
              │   (Kibana-based UI)    │
              └───────────────────────┘
```

Tools used:


Wazuh Manager + Dashboard (Docker on macOS)
UTM / VirtualBox for endpoint VMs
Ubuntu Server (Linux endpoint)
Windows 10/11 (Windows endpoint)


Setup Steps

1. Deploy the SIEM


Installed Docker Desktop on macOS.
Deployed Wazuh single-node stack via wazuh-docker repo.
Accessed dashboard at https://localhost:443.


2. Collect and Centralize Logs


Provisioned Ubuntu and Windows VMs via UTM.
Installed Wazuh agents on each, registered to the manager.
Verified agent connectivity status = Active.


3. Generate and Ingest Security Events

Simulated activity to produce log data:


Repeated failed SSH login attempts (brute-force simulation)
Local user account creation (useradd / Windows local user)
Suspicious command execution (reverse shell listener, unauthorized download)


4. Analyze Logs and Configure Alerts


Reviewed raw events under Threat Hunting.
Wrote a custom correlation rule in local_rules.xml to flag 5+ failed logins within 60 seconds.
Tuned rule severity levels to reduce false positives.


Example custom rule:

xml<rule id="100010" level="10">
  <if_matched_sid>5716</if_matched_sid>
  <same_source_ip />
  <description>Multiple failed SSH logins from same source - possible brute force</description>
  <mitre>
    <id>T1110</id>
  </mitre>
</rule>

5. Visualize and Report Findings


Built a dashboard tracking failed login trends, alert severity distribution, and top targeted accounts.
Captured screenshots of triggered alerts and dashboard views (see /screenshots).


Findings & Detections

| Simulated Activity | Detected? | Rule Triggered | Severity |
|---|---|---|---|
| SSH brute force | ✅ | Custom rule 100010 | High |
| New local account | ✅ | Wazuh default rule (T1136) | Medium |
| Reverse shell attempt | ✅ | Wazuh default rule | High |
| Sudo privilege escalation | ✅ | Wazuh default rule (T1548.003) | Medium |


Screenshots

### Wazuh Login
![Wazuh Login](screenshots/dashboard-login.jpeg)

### Dashboard Overview
![Dashboard Overview](screenshots/dashboard-overview.jpeg)

### Agent Status
![Agent Status](screenshots/agents-status.jpg)

### All Endpoints Connected
![All Agents](screenshots/agents-windows-status.jpg)

### Threat Hunting Overview
![Threat Hunting Overview](screenshots/threat-hunting-overview.jpg)

### Threat Hunting — Simulated Events
![Threat Hunting Events](screenshots/threat-hunting-events.jpg)


## Lessons Learned

Initially wrote a custom correlation rule (100010) keyed to Wazuh's default rule 5716 
("authentication failed"), but testing with a nonexistent username triggered rule 5710 
("invalid user") instead — a different failure path than expected. Diagnosed the mismatch 
by inspecting the manager's rule definitions directly inside the Docker container, corrected 
the `if_matched_sid` reference, and confirmed the rule fired correctly on retest (11 hits, 
level 10, MITRE T1110).

Also encountered a Wazuh agent/manager version mismatch (agent 4.14.6 vs manager 4.9.0) 
that silently rejected registration — resolved by pinning the agent install to the matching 
manager version.

Next Steps


Add Sysmon to Windows endpoint for deeper process-level visibility.
Integrate with a SOAR playbook for automated response.
Add MITRE ATT&CK mapping to all custom rules.


Disclaimer

This lab is for educational purposes only, built entirely in an isolated local environment. No production systems or real user data were involved.
