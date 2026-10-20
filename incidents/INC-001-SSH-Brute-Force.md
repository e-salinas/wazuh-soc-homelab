# 🚨 Incident Report: INC-001

## SSH Brute Force Attack

| Field | Value |
|-------|-------|
| **Incident ID** | INC-001 |
| **Severity** | High — lab-assigned investigation priority |
| **Classification** | True positive — controlled SSH password-guessing simulation |
| **Status** | Investigation documented; manual firewall deny rule configured |
| **Analyst** | Esteban Salinas — lab author |
| **MITRE ATT&CK** | [T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/) |

---

## Executive Summary

I generated a controlled SSH password-guessing simulation from the Kali Linux VM (10.0.0.34) against the ubuntu-desktop endpoint (10.0.0.18). I investigated the resulting Wazuh authentication alerts, identified the source IP and attempted username, and configured a manual UFW deny rule on the target. The retained screenshots document the authentication alerts and active firewall rule.

---

## Attack Details

| Field | Value |
|-------|-------|
| **Source IP** | 10.0.0.34 (Internal - Kali Linux VM) |
| **Target System** | ubuntu-desktop (10.0.0.18) |
| **Target Service** | SSH (Port 22) |
| **Target Username** | admin |
| **Attack Tool** | Hydra v9.5 (Password Cracking Tool) |
| **Observed Event Counts** | 90 events in the authentication-failure-filtered view; 94 total hits in a separate captured view |
| **Authentication Evidence** | Captured alerts show authentication failures, including attempts using a nonexistent username |

These counts reflect the filters and time windows shown in each screenshot. They are dashboard event counts, not a verified count of password guesses or unique login attempts.

---

## Investigation and Response Sequence

1. Generated controlled SSH password-guessing activity from the Kali VM.
2. Reviewed authentication-failure alerts in Wazuh.
3. Identified the source IP, target endpoint, and attempted username.
4. Reviewed the triggered rules and authentication activity.
5. Configured a UFW deny rule for the source IP and confirmed the rule was active.

---

## Detection Details

### Wazuh Rules Triggered

| Rule ID | Description | Level | Significance |
|---------|-------------|-------|--------------|
| 5758 | Maximum authentication attempts exceeded | 8 | Authentication-attempt limit reached; review with surrounding events |
| 5710 | Attempt to login using a non-existent user | 5 | Attempted authentication using a nonexistent username |
| 2502 | User missed the password more than one time | 10 | Repeated failed authentication attempts |

### MITRE ATT&CK Mapping

- **Tactic:** Credential Access
- **Technique:** T1110 - Brute Force
- **Sub-technique:** T1110.001 - Password Guessing

### Evidence Screenshots

#### Alert Overview
![Alert Overview](../screenshots/ssh-brute-force/01-alerts-overview.png)
*Filtered Wazuh dashboard showing 90 authentication-failure events.*

#### Alert Details
![Alert Details](../screenshots/ssh-brute-force/02-alert-details.png)
*Wazuh event list showing authentication-related rules, the monitored endpoint, and the simulation source IP where available.*

#### Event Distribution
![Timeline](../screenshots/ssh-brute-force/03-attack-timeline.png)
*A separate Wazuh view showing 94 total hits and a concentrated burst of events.*

---

## Response Actions Taken

### 1. Detection & Triage
- Identified multiple authentication failure alerts in Wazuh SIEM dashboard
- Filtered events by agent (`ubuntu-desktop`) and rule IDs to isolate the incident
- Noted the spike in authentication failures indicating automated attack

### 2. Investigation
- Analyzed alert details to identify source IP (10.0.0.34)
- Confirmed target system (ubuntu-desktop) and service (SSH)
- Identified attack pattern: rapid successive login attempts with common passwords
- Reviewed the captured authentication failures, including rule 5710 for a nonexistent username
  
### 3. Classification
- Classified the alerts as true positives for the controlled password-guessing activity
- Correlated the source IP and authentication events with the known lab simulation
- Mapped the simulated behavior to MITRE ATT&CK T1110.001 — Password Guessing
- Distinguished detection of the activity from proof of successful access or verified prevention

### 4. Containment
Implemented firewall block on target system:

```bash
sudo ufw enable
sudo ufw deny from 10.0.0.34
sudo ufw status
```

#### Containment Evidence
![UFW Block](../screenshots/ssh-brute-force/04-containment-ufw.png)
*UFW status showing an active deny rule for the simulation source IP.*

### 5. Verification
- Confirmed UFW was active using `ufw status`
- Confirmed a deny rule for the simulation source IP, `10.0.0.34`, appeared in the firewall configuration
- The retained evidence demonstrates rule configuration; a post-block connection test is not included
  
---

## Analysis of Observed Results

### Simulation Activity
Hydra generated controlled SSH password-guessing activity against the Ubuntu endpoint.

### Detection
Wazuh recorded authentication-related events, including attempts using a nonexistent username and maximum authentication attempts being exceeded. I reviewed the source IP, target endpoint, and rule descriptions to investigate the activity.

### Authentication Outcome
The captured alerts demonstrate failed authentication attempts. They do not establish that password strength caused the failures or that SSH applied a persistent source-IP block.

### Manual Response
I configured a UFW deny rule for the Kali VM's source IP and confirmed that the firewall was active and the rule was present.

---

## Completed Response and Suggested Improvements

### Completed in This Exercise
- Investigated the captured Wazuh authentication alerts.
- Identified the simulation source IP and monitored endpoint.
- Configured a manual UFW deny rule and confirmed it was active.

### Suggested Improvements — Not Implemented in This Exercise
- Test a new SSH connection from the blocked source to verify the firewall's effect.
- Review SSH configuration and consider key-based authentication with password authentication disabled.
- Evaluate automated blocking through fail2ban or Wazuh Active Response.
- Define a repeatable search for successful and failed authentication events within a documented investigation window.

These are follow-up recommendations, not requirements for completing the documented scope of this lab.

---

## Affected Assets

| Asset | Role | Observed Result |
|-------|------|-----------------|
| ubuntu-desktop (10.0.0.18) | Monitored SSH endpoint | Authentication-failure alerts recorded |
| Kali Linux (10.0.0.34) | Simulation source | Source IP identified in captured alerts |
| Username `admin` | Attempted login name | Captured rule 5710 events report a nonexistent user |
| UFW on ubuntu-desktop | Manual response control | Active deny rule documented |

---

## Related Documentation

- **Response Playbook:** [PB-001 - SSH Brute Force Response](../playbooks/PB-001-SSH-Brute-Force.md)
- **MITRE ATT&CK:** [T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/)

---

## Sign-off

| Role | Name | Date |
|------|------|------|
| **Investigating Analyst** | Esteban Salinas | December 6, 2025 |
| **Report Completed** | December 6, 2025 | |

---

*This incident report was created as part of a SOC Analyst homelab project to demonstrate incident response capabilities.*
