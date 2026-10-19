# 📘 Playbook: PB-001

## SSH Brute Force Detection & Triage

| Field | Value |
|-------|-------|
| **Playbook ID** | PB-001 |
| **Version** | 1.0 |
| **Last Updated** | December 6, 2025 |
| **Author** | Esteban |
| **Threat Type** | Suspected SSH password guessing |
| **Severity** | 🔴 HIGH |
| **MITRE ATT&CK** | [T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/) |

---

## Trigger

Execute this playbook when you see these alerts in Wazuh SIEM:

| Rule ID | Description | Level |
|---------|-------------|-------|
| 5758 | Maximum authentication attempts exceeded | 8 |
| 5710 | Attempt to login using a non-existent user | 5 |
| 5720 | Multiple authentication failures | 10 |
| 2502 | User missed the password more than one time | 10 |

---

## Step 1: Alert Triage

Review the activity before assigning a classification:

- Identify the affected endpoint and define the investigation time window.
- Review the source IP, attempted usernames, frequency, and pattern of failures.
- Check whether the source and activity match an approved test, administrative task, or reported login problem.
- Look for related successful authentication events and other suspicious activity.
- Record the dashboard filters and time window used for any event counts.

High volume can support an investigation, but there is no universal alert-count threshold that proves an attack. Low-volume activity can also be suspicious.

---

## Step 2: Gather Key Information

Click on an alert and document these fields:

| Field | Where to Find | Your Notes |
|-------|---------------|------------|
| **Source IP** | `data.srcip` | _______________ |
| **Target System** | `agent.name` | _______________ |
| **Target Username** | `data.dstuser` | _______________ |
| **Timestamp** | `timestamp` | _______________ |
| **Rule ID** | `rule.id` | _______________ |
| **Alert Count** | Total hits | _______________ |

**Take screenshots of:**
- Alert overview (showing total count)
- Alert details (expanded view with source IP)

---

## Step 3: Review Related Authentication Activity

Set an explicit investigation time window and filter by the affected endpoint and source IP.

Example filter for the original lab:
```text
agent.name: "ubuntu-desktop" AND data.srcip: "10.0.0.34"
```

Review the event descriptions and underlying logs for failed and successful authentication. Rule 5715 can help locate SSH authentication-success events where that rule is used, but do not rely on a single rule ID as a complete search.

- Record the endpoint, source, time window, filters, and findings.
- Correlate any successful login with the username and surrounding activity.
- Investigate suspicious successful authentication promptly.
- If no success is found, report “No successful authentication observed within the reviewed scope.”
- If logging coverage is uncertain, document the gap.
  
---

## Step 4: Classification Decision

Use the available evidence and operational context:

| Finding | Action |
|---------|--------|
| Activity matches an authorized lab test | Document the detected behavior and identify it as a controlled simulation |
| Failures match a verified user or application problem | Document the explanation and coordinate troubleshooting |
| Pattern supports unauthorized password guessing | Document the evidence and follow the applicable escalation and containment procedure |
| Successful login is associated with suspicious activity | Prioritize investigation and escalate according to the response procedure |
| Evidence is insufficient | Keep the finding unresolved, document the gaps, and gather more context |

An authentication failure can be accurately detected without representing malicious activity. Do not classify or block a source based only on alert volume.

---

## Step 5: Documentation

Before escalating or closing, ensure you have:

- [ ] Screenshots of alerts saved
- [ ] Source IP documented
- [ ] Target system identified
- [ ] Alert count recorded
- [ ] Successful login check completed
- [ ] Classification decision documented
- [ ] Investigation time window and filters recorded
- [ ] Logging gaps or unresolved questions documented
- [ ] Any containment action and its authorization recorded
- [ ] Configuration evidence distinguished from a test of its effect

---

## Containment and Verification

For a personal lab, confirm the source IP belongs to the intended simulation system before changing firewall rules.

In an organizational environment, follow the approved response procedure and confirm authorization before blocking a source. Consider shared addresses and legitimate access that could be affected.

After an approved block:
- Confirm the intended rule is active.
- Test a new connection from the controlled source when authorized.
- Check that required legitimate access still works.
- Document the observed result and any limitations.
  
---

## Quick Reference

### Wazuh Filters

```
# SSH brute force alerts
rule.id: (5758 OR 5710 OR 5720 OR 2502)

# Filter by source IP
data.srcip: 10.0.0.34

# Filter by target system
agent.name: ubuntu-desktop

# Example SSH success rule; use with endpoint and time-window filters
rule.id: 5715
```

### Key Fields

| Field | Description |
|-------|-------------|
| `data.srcip` | Attacker IP address |
| `agent.name` | Target system name |
| `data.dstuser` | Username being targeted |
| `rule.id` | Detection rule that fired |
| `rule.description` | What the rule detected |
| `timestamp` | When the event occurred |

---

## References

- [MITRE ATT&CK T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/)
- [Wazuh Documentation](https://documentation.wazuh.com/)

---

## Revision History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Dec 6, 2025 | Initial playbook |

---

*This playbook reflects Tier 1 SOC Analyst responsibilities for SSH brute force detection and triage.*
