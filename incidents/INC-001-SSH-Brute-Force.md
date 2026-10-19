# 🚨 Incident Report: INC-001

## SSH Brute Force Attack

| Field | Value |
|-------|-------|
| **Incident ID** | INC-001 |
| **Severity** | 🔴 **HIGH** |
| **Classification** | True Positive - Confirmed Attack |
| **Status** | ✅ CONTAINED |
| **Analyst** | Esteban (SOC Analyst) |
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
| **Total Alerts Generated** | 90 security events |
| **Authentication Result** | ✅ **FAILED** - No credentials compromised |

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
| 5758 | Maximum authentication attempts exceeded | 8 | Indicates brute force attack pattern |
| 5710 | Attempt to login using a non-existent user | 5 | Attacker testing invalid usernames |
| 2502 | User missed the password more than one time | 10 | Repeated failed authentication attempts |

### MITRE ATT&CK Mapping

- **Tactic:** Credential Access
- **Technique:** T1110 - Brute Force
- **Sub-technique:** T1110.001 - Password Guessing

### Evidence Screenshots

#### Alert Overview
<!-- Add screenshot of Wazuh dashboard showing 94 alerts -->
![Alert Overview](../screenshots/ssh-brute-force/01-alerts-overview.png)
*Wazuh Security Events showing 94 authentication failure alerts*

#### Alert Details
<!-- Add screenshot of expanded alert showing source IP and rule details -->
![Alert Details](../screenshots/ssh-brute-force/02-alert-details.png)
*Detailed alert view showing attacker IP (10.0.0.34), rule.mitre.id (T1110), and full log entry*

#### Attack Timeline
<!-- Add screenshot showing the histogram/timeline of events -->
![Timeline](../screenshots/ssh-brute-force/03-attack-timeline.png)
*Event timeline showing concentration of attacks between 15:49-15:50*

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
- Verified no successful authentication occurred

### 3. Classification
- Classified as **TRUE POSITIVE** - confirmed malicious activity
- Attack matched MITRE ATT&CK technique T1110 (Brute Force)
- Volume and velocity of attempts indicated automated tool usage

### 4. Containment
Implemented firewall block on target system:

```bash
sudo ufw enable
sudo ufw deny from 10.0.0.34
sudo ufw status
```

#### Containment Evidence
<!-- Add screenshot of UFW status showing the deny rule -->
![UFW Block](../screenshots/ssh-brute-force/04-containment-ufw.png)
*UFW firewall rule blocking attacker IP*

### 5. Verification
- Confirmed firewall rule active via `ufw status`
- Verified subsequent connection attempts from attacker IP were blocked
- Continued monitoring for any related suspicious activity

---

## Root Cause Analysis

### Attack Vector
The attacker utilized Hydra, an automated password cracking tool, to perform a dictionary attack against the SSH service. The attack targeted the "admin" username with a wordlist of common passwords.

### Why Detection Succeeded
1. **Wazuh agent** was properly configured on the target system
2. **SSH logging** was enabled and forwarded to SIEM
3. **Correlation rules** detected the pattern of multiple failed authentications
4. **Real-time alerting** enabled immediate analyst notification

### Why Attack Failed
1. **Strong password policy** - Target account password was not in attacker's wordlist
2. **SSH rate limiting** - Built-in SSH protection kicked in after multiple failures
3. **Rapid containment** - Firewall block prevented continued attempts

---

## Recommendations

### Immediate Actions
1. ✅ **Completed** - Block attacker IP via firewall
2. ⬜ Review all systems for similar attack patterns from same source
3. ⬜ Verify no unauthorized access occurred on any system

### Short-term Improvements
1. **Implement fail2ban** - Automatically block IPs after failed attempts
2. **Disable password authentication** - Use SSH key-based authentication only
3. **Configure Wazuh Active Response** - Automate IP blocking on detection

### Long-term Recommendations
1. **Network segmentation** - Limit lateral movement potential
2. **Remove unnecessary accounts** - Audit and disable unused usernames like "admin"
3. **Implement MFA** - Add multi-factor authentication for SSH access
4. **Regular security assessments** - Conduct periodic penetration testing

---

## Affected Assets

| Asset | Impact | Status |
|-------|--------|--------|
| ubuntu-desktop (10.0.0.18) | Targeted - No compromise | ✅ Secured |
| SSH Service | Attack target | ✅ Operational |
| Admin account | Targeted username | ✅ Not compromised |

---

## Related Documentation

- **Response Playbook:** [PB-001 - SSH Brute Force Response](../playbooks/PB-001-SSH-Brute-Force.md)
- **MITRE ATT&CK:** [T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/)

---

## Sign-off

| Role | Name | Date |
|------|------|------|
| **Investigating Analyst** | Steve | December 6, 2025 |
| **Report Completed** | December 6, 2025 | |

---

*This incident report was created as part of a SOC Analyst homelab project to demonstrate incident response capabilities.*
