Home SIEM Lab: Threat Detection & Log Analysis

A self-hosted Security Information and Event Management (SIEM) lab built to practice log collection, threat detection, alert tuning, and incident visualization — using free, open-source tools on a MacBook.

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

Simulated ActivityDetected?Rule TriggeredSeveritySSH brute force✅Custom rule 100010HighNew local account✅Wazuh default ruleMediumReverse shell attempt✅Wazuh default ruleHigh

Screenshots

### Wazuh Login
![Wazuh Login](screenshots/dashboard-login.jpeg)

### Dashboard Overview
![Dashboard Overview](screenshots/dashboard-overview.jpeg)





Lessons Learned


Notes on false positives encountered and how thresholds were adjusted.
Notes on any detection gaps (e.g., Windows Event Log visibility without Sysmon).


Next Steps


Add Sysmon to Windows endpoint for deeper process-level visibility.
Integrate with a SOAR playbook for automated response.
Add MITRE ATT&CK mapping to all custom rules.


Disclaimer

This lab is for educational purposes only, built entirely in an isolated local environment. No production systems or real user data were involved.
