# Cloud-Based SOC Home Lab: Multi-OS Threat Detection with Wazuh

A hands-on SOC home lab built across DigitalOcean and AWS to demonstrate end-to-end detection engineering, attack simulation, and MITRE ATT&CK–aligned threat detection using Wazuh 4.9.2.

---

## Key Outcomes

- Deployed a fully functional Wazuh 4.9.2 SIEM in the cloud
- Onboarded Linux and Windows endpoints across two cloud providers
- Generated 1,000+ correlated SSH authentication-failure alerts during the brute-force simulation, demonstrating reliable detection and alert triage capability
- Mapped detections to MITRE ATT&CK T1110.001 (Password Guessing)
- Validated alert visibility through Wazuh dashboards, event tables, and rule-level analysis

---

## Overview

This project simulates a small enterprise security monitoring environment using Wazuh 4.9.2 as the SIEM platform.

The goal of the lab was to develop practical blue-team and SOC analyst skills, including:

- SIEM deployment and configuration
- Endpoint log collection
- Detection validation
- MITRE ATT&CK mapping
- Multi-cloud infrastructure management
- Troubleshooting real-world logging and monitoring issues

The environment intentionally spans multiple cloud providers and operating systems to better reflect the heterogeneous environments commonly monitored by enterprise SOC teams.

---

## Tech Stack

- **SIEM:** Wazuh 4.9.2
- **Cloud Providers:** DigitalOcean, AWS EC2
- **Operating Systems:** Ubuntu 24.04, Windows Server 2022
- **Log Sources:** Syslog, PAM authentication logs, Windows Event Logs
- **Frameworks:** MITRE ATT&CK
- **Tools:** SSH, auditpol, Wazuh Agent, Wazuh Dashboard

---

## Architecture

| Component | Platform | OS | Role |
|---|---|---|---|
| Wazuh Server | DigitalOcean | Ubuntu 24.04 | SIEM (Indexer, Manager, Dashboard) |
| Linux Target | DigitalOcean | Ubuntu 24.04 | Monitored endpoint with Wazuh Agent |
| Windows Target | AWS EC2 | Windows Server 2022 | Monitored endpoint with Wazuh Agent |

**Architecture diagram:** `diagrams/architecture.png`

*Diagram to be added: attacker → monitored endpoints → Wazuh server data flow.*

```
Internet
    |
    v
[Attack Simulation]
        |
        v
[Ubuntu 24.04 Target] ----> [Wazuh Server]
                                 ^
                                 |
                    [Windows Server 2022 Target]
```

### Screenshots

The following screenshots will be added as the project progresses:

- `screenshots/dashboard-overview.png` — Wazuh dashboard summary
- `screenshots/alerts-evolution.png` — Alert volume during brute-force simulation
- `screenshots/top-tactics.png` — MITRE ATT&CK tactics visualization
- `screenshots/event-detail.png` — Event-level investigation and rule details

---

### Why Multi-Cloud?

The lab was deliberately built across DigitalOcean and AWS to practice adapting to real-world infrastructure constraints. An earlier deployment attempt in Azure was abandoned after free-tier regional quota limitations prevented VM provisioning, requiring a redesign of the environment.

---

## Detections Implemented

### SSH Brute-Force Attack (Linux)

**MITRE ATT&CK:** [T1110.001 – Password Guessing](https://attack.mitre.org/techniques/T1110/001/)

**Tactics:** Credential Access, Lateral Movement

**Detection & Verification Process**

1. Generated repeated failed SSH authentication attempts against the Ubuntu target
2. Verified raw failure events directly in `/var/log/auth.log` on the endpoint
3. Confirmed the Wazuh agent was forwarding logs to the manager (agent status: Active)
4. Cross-checked the Wazuh dashboard to confirm alerts appeared for the same time window as the simulated attack
5. Reviewed individual events in the Events table to confirm correct rule classification and MITRE technique tagging

**Wazuh Rules Triggered**

| Rule ID | Description |
|---|---|
| 5710 | sshd: Attempt to login using a non-existent user |
| 5760 | sshd: authentication failed |
| 5503 | PAM: User login failed |
| 2502 | syslog: User missed the password more than one time |

**Results**

- Generated and triaged 1,000+ correlated SSH authentication-failure alerts during the brute-force simulation
- Alerts were correctly classified as SSH authentication failures
- Events were successfully correlated with MITRE ATT&CK T1110.001 – Password Guessing

---

### Windows Failed Logon Detection (In Progress)

**MITRE ATT&CK:** T1110 – Brute Force

> **Note on naming:** the Windows endpoint is referred to as "Windows Target" in the architecture table above, and appears as `windows-target` (its registered Wazuh agent name) in dashboard screenshots and event data throughout this README. Same machine, two labels.

**Current Status: Under active investigation**

- Enabled Windows audit logging for logon success/failure (`auditpol`)
- Simulated failed authentication attempts locally on the Windows target
- Filtered the Wazuh Events dashboard to `agent.name: windows-target` — only 3 total events returned, none corresponding to a failed-logon (Event ID 4625) alert; events seen so far are agent-connectivity notices, not authentication failures
- **Root cause hypothesis:** the Windows agent's `ossec.conf` may be missing the Security eventchannel `<localfile>` block required to forward Windows Security log events to the manager

**Next Steps**

- Restart the Wazuh agent service (`WazuhSvc`) and re-test
- Inspect and, if needed, correct the Security eventchannel configuration in `ossec.conf`
- Confirm Event ID 4625 is generated locally on the Windows target
- Re-run the simulated attack and confirm the alert reaches the Wazuh dashboard

---

## Skills Demonstrated

- SIEM deployment and configuration (Wazuh Indexer, Manager, Dashboard)
- Linux and Windows Wazuh agent deployment
- Log source configuration (`ossec.conf`, syslog, Windows Event Logs)
- Attack simulation and detection validation
- MITRE ATT&CK framework mapping
- Multi-cloud infrastructure management (DigitalOcean and AWS)
- Service troubleshooting and log collection debugging
- Security event analysis and alert correlation

---

## Challenges & Troubleshooting

### Azure Free-Tier Quota Limitation

- Initial deployment attempted in Azure
- VM creation blocked by regional quota restrictions in multiple regions
- Re-architected the environment using AWS EC2 for the Windows endpoint

### Windows Event Log Collection Issue

**Symptoms:** Windows Security logs (failed logon events) are not appearing in the Wazuh dashboard, despite the agent showing as Active.

**Troubleshooting steps taken so far:**

- Confirmed the Windows agent is installed and reporting Active status
- Enabled audit logging for logon success/failure via `auditpol`
- Filtered Wazuh events by agent name to isolate Windows-specific activity
- Identified that only non-authentication events (agent connectivity) are currently reaching the manager

**Status:** Actively debugging — likely a missing or misconfigured Security eventchannel entry in `ossec.conf`. This is documented here deliberately, as working through infrastructure and log-collection gaps like this is a core part of real SOC/detection engineering work.

---

## Quick Start

**Deploy Wazuh Server**

```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

**Install Ubuntu Wazuh Agent**

```bash
curl -sO https://packages.wazuh.com/4.x/wazuh-agent_4.9.2-1_amd64.deb
sudo dpkg -i wazuh-agent_4.9.2-1_amd64.deb
```

**Start the Agent**

```bash
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

---

## The Story (Non-Technical Summary)

I built a small-scale version of the kind of monitoring system used by a real Security Operations Center (SOC).

A central Wazuh server acted as the security monitoring platform, collecting logs from both a Linux and a Windows cloud-hosted endpoint. I simulated a realistic attack by repeatedly attempting to log in with incorrect credentials against the Linux endpoint.

The objective was to verify that the system could:

1. Capture the failed login events
2. Forward them to the SIEM
3. Generate security alerts
4. Classify the activity correctly
5. Map the alert to the appropriate MITRE ATT&CK technique

The lab successfully detected the simulated brute-force activity on Linux and generated alerts mapped to MITRE ATT&CK T1110.001 — the same technique real-world defenders use to classify brute-force login attempts. The Windows side of the lab is still being debugged, and that troubleshooting process is documented above as part of the project.

---

## Roadmap

- [ ] Complete and tune Windows failed-logon detection
- [ ] Create a custom Wazuh detection rule (e.g., threshold-based brute-force correlation)
- [ ] Implement automated response to block IPs after repeated failed SSH logins
- [ ] Add memory forensics with Volatility
- [ ] Build a SOC investigation playbook for the brute-force scenario
- [ ] Publish a full LinkedIn technical write-up with screenshots and detection analysis

---

## Repository Structure

```
.
├── README.md
├── screenshots/
├── diagrams/
├── detection-notes/
├── wazuh-config/
└── attack-simulation/
```

---

## Future Improvements

- Dashboard screenshots and architecture diagrams
- Custom correlation rules and threshold tuning
- Automated active response workflows
- Additional attack simulations (web attacks, privilege escalation, persistence)
- Detection coverage mapping across multiple MITRE ATT&CK tactics

---

## Author

**Anastasia Kwarteng**

Aspiring SOC Analyst / Blue Team Analyst with a focus on:

- SIEM engineering
- Threat detection
- Log analysis
- Incident investigation
- Cloud security monitoring

---

## License

This project is for educational and portfolio purposes and is intended to demonstrate practical SOC and detection engineering skills in a controlled lab environment.

