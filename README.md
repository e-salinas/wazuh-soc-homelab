# Wazuh SIEM Lab — SSH Monitoring and File Integrity Investigation

I built this lab to practice collecting Linux security events, investigating Wazuh alerts, and documenting findings and response actions.

The project covers two controlled exercises: an SSH brute-force simulation and file-integrity monitoring of test files. It includes two incident reports, two response playbooks, and supporting screenshots.

The environment uses Wazuh on Ubuntu Server, monitored Ubuntu endpoints, and Kali Linux for the SSH exercise, running through UTM on two Apple Silicon Macs.

All activity was performed in my personal lab. Incident classifications and escalation examples are part of the simulation.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Lab Architecture](#lab-architecture)
- [Documented Lab Scenarios](#documented-lab-scenarios)
- [Incident Reports](#incident-reports)
- [Response Playbooks](#response-playbooks)
- [Tools & Technologies](#tools--technologies)
- [Screenshots](#screenshots)
- [Lessons Learned](#lessons-learned)

---

## Overview

This project simulates a real-world Security Operations Center environment where I detect, investigate, and respond to various cyber attacks. The goal is to demonstrate practical SOC analyst skills including:

- Real-time security event monitoring
- Alert triage and classification
- Incident investigation and documentation
- Threat containment and remediation
- Playbook development and execution

---

## Lab Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      SOC ANALYST HOMELAB                        │
│                    Network: 10.0.0.0/24                         │
└─────────────────────────────────────────────────────────────────┘

    ┌─────────────────────────────────────────────────────────┐
    │              MacBook (SIEM Server)                      │
    │  ┌───────────────────────────────────────────────────┐  │
    │  │            Ubuntu Server 24.04 VM (UTM)           │  │
    │  │                   10.0.0.65                       │  │
    │  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐  │  │
    │  │  │   Wazuh     │ │   Wazuh     │ │   Wazuh     │  │  │
    │  │  │   Manager   │ │   Indexer   │ │  Dashboard  │  │  │
    │  │  └─────────────┘ └─────────────┘ └─────────────┘  │  │
    │  └───────────────────────────────────────────────────┘  │
    └─────────────────────────────────────────────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │   Network: 10.0.0.0/24│
                    └───────────┬───────────┘
                                │
    ┌───────────────────────────────────────────────────────────┐
    │                    iMac (Attack Lab)                      │
    │                                                           │
    │  ┌─────────────────────────────────────────────────────┐  │
    │  │                 UTM Virtual Machines                │  │
    │  │                                                     │  │
    │  │  ┌─────────┐    ┌─────────────┐    ┌─────────────┐  │  │
    │  │  │  KALI   │    │   UBUNTU    │    │   UBUNTU    │  │  │
    │  │  │  LINUX  │    │   DESKTOP   │    │   SERVER    │  │  │
    │  │  │         │    │             │    │             │  │  │
    │  │  │ Attacker│───▶│   Target    │    │   Target    │  │  │
    │  │  │   VM    │    │ + Wazuh     │    │ + Wazuh     │  │  │
    │  │  │         │    │   Agent     │    │   Agent     │  │  │
    │  │  │10.0.0.34│    │ 10.0.0.18   │    │ 10.0.0.186  │  │  │
    │  │  └─────────┘    └─────────────┘    └─────────────┘  │  │
    │  │                                                     │  │
    │  └─────────────────────────────────────────────────────┘  │
    │                                                           │
    └───────────────────────────────────────────────────────────┘
```

### Components

| Component | Role | IP Address |
|-----------|------|------------|
| Wazuh Server | SIEM - Manager, Indexer, Dashboard | 10.0.0.65 |
| Ubuntu Desktop | Primary Target + Wazuh Agent | 10.0.0.18 |
| Ubuntu Server | Secondary Target + Wazuh Agent | 10.0.0.186 |
| Kali Linux | Attack Platform (Red Team) | 10.0.0.34 |

---

## Documented Lab Scenarios

| Scenario | What I practiced | Report |
|----------|------------------|--------|
| SSH brute-force simulation | Investigating authentication failures, identifying the source, and configuring a manual firewall block | [INC-001](incidents/INC-001-SSH-Brute-Force.md) |
| File-integrity monitoring | Investigating test-file creation and modification alerts and collecting file metadata and hashes | [INC-002](incidents/INC-002-Malware-Detection.md) |

---

## Incident Reports

Detailed documentation of each security incident including timeline, investigation steps, and escalation actions.

| Incident ID | Title | Severity | Classification | Link |
|-------------|-------|----------|----------------|------|
| INC-001 | SSH Brute Force Attack | 🔴 HIGH | True Positive | [View Report](incidents/INC-001-SSH-Brute-Force.md) |
| INC-002 | Test File Creation and Modification (FIM)) | 🟠 MEDIUM | Expected test activity | [View Report](incidents/INC-002-Malware-Detection.md) |

---

## Response Playbooks

Step-by-step Tier 1 procedures for triaging and escalating common security incidents.

| Playbook ID | Title | Threat Type | Link |
|-------------|-------|-------------|------|
| PB-001 | SSH Brute Force Detection & Triage | Credential Attack | [View Playbook](playbooks/PB-001-SSH-Brute-Force.md) |
| PB-002 | FIM / Suspicious File Response | File Integrity | [View Playbook](playbooks/PB-002-FIM-Malware-Detection.md) |

---

## Tools & Technologies

- **Wazuh 4.x** — Centralized security-event monitoring and file-integrity monitoring
- **Hydra** — Controlled SSH password-guessing simulation
- **UFW** — Manual firewall rule configuration on the monitored endpoint

### Infrastructure
- **UTM** - Virtualization platform (macOS)
- **Ubuntu Server 24.04** - SIEM host
- **Ubuntu Desktop 24.04** - Target endpoint
- **Kali Linux** - Attack platform

### Frameworks
- **MITRE ATT&CK** - Threat classification
- **NIST SP 800-61** - Incident response framework

---

## Screenshots

---

### Scenario 1: SSH Brute Force Attack

#### Attack Detection

![SSH Brute Force Alerts](screenshots/ssh-brute-force/01-alerts-overview.png)
*Wazuh detecting 94 authentication failure events from brute force attack*

#### Alert Details

![Alert Details](screenshots/ssh-brute-force/02-alert-details.png)
*Detailed view showing attacker IP, target system, and MITRE ATT&CK mapping (T1110)*

#### Attack Timeline

![Attack Timeline](screenshots/ssh-brute-force/03-attack-timeline.png)
*Timeline showing burst of authentication attempts during attack window*

#### Containment

![Firewall Block](screenshots/ssh-brute-force/04-containment-ufw.png)
*UFW firewall rule blocking attacker IP address*

---

### Scenario 2: File Integrity Monitoring

#### FIM Alert Overview

![FIM Alerts](screenshots/malware-detection/01-fim-alerts-overview.png)
*Wazuh FIM alerts generated during controlled test-file activity in /tmp.*

#### File Added Alert (Rule 554)

![File Added Details](screenshots/malware-detection/02-fim-alerts-added.png)
*Alert details showing file path and MD5/SHA256 hashes*

#### File Modified Alert (Rule 550)

![File Modified Details](screenshots/malware-detection/03-fim-modified-details.png)
*Rule 550 reporting changes to the test file's inode and modification time.*




## Lessons Learned

### Technical Insights
1. **Real-time detection is critical** - Wazuh detected the brute force attack within seconds, enabling rapid response
2. **Log correlation reveals patterns** - Multiple rule IDs (5758, 5710, 2502) together confirmed automated attack vs. user error
3. **Defense in depth works** - SSH rate limiting + SIEM detection + firewall blocking created multiple defensive layers
4. **FIM provides early warning** - File Integrity Monitoring detected suspicious files immediately upon creation, enabling rapid triage

### Process Improvements
1. **Document as you go** - Capturing timestamps during the incident made report writing much easier
2. **Playbooks save time** - Having a documented procedure reduced response time significantly
3. **Collect hashes immediately** - Gathering MD5/SHA256 hashes are used to check threat intelligence quickly

---

## Acknowledgments

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
