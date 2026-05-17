# 001 — SSH Brute Force Detection

> Repeated failed SSH login attempts detected by Wazuh, automatically escalated from individual failures to a brute force pattern at Level 10.

**Date:** May 16, 2026
**Lab:** esc - SOC Home Lab
**Tools:** Wazuh 4.7.5 · Ubuntu 24.04 target · Mac Terminal
**From worklog:** [2026-05-16](../../worklogs/2026-05-16.md)

---

## Summary

Multiple failed SSH login attempts were simulated from the analyst workstation against the monitored endpoint using a non-existent username and wrong passwords. Wazuh detected the individual failures immediately and automatically escalated to a brute force pattern alert (Level 10) after repeated attempts — without any custom rule configuration.

---

## Environment

- **SIEM:** Wazuh 4.7.5 — self-hosted on Apple Silicon
- **SIEM Server:** Ubuntu 22.04 LTS ARM64
- **Target VM:** Ubuntu 24.04 LTS ARM64
- **Network:** UTM Shared Network NAT — fully isolated from internet

---

## Scenario

Simulated an attacker repeatedly attempting SSH logins using a non-existent username and wrong passwords — the most common attack against internet-facing SSH servers. Two rounds of attempts were made: one targeting a non-existent user, one targeting root (disabled on Ubuntu by default).

---

## Objectives

- Detect failed SSH login attempts in Wazuh
- Observe automatic severity escalation on brute force pattern
- Verify MITRE ATT&CK auto-mapping
- Document analyst response for each alert type

---

## Simulation Steps

```bash
ssh wronguser@[target-ip]    # wrong password 5x → Ctrl+C, repeated twice
ssh root@[target-ip]         # root SSH disabled on Ubuntu — always fails
```

---

## Detection

| Rule ID | Severity | Count | Description |
|---|---|---|---|
| 5710 | Level 5 | 8 | sshd: Attempt to login using non-existent user |
| 5503 | Level 5 | 7 | PAM: User login failed |
| 2502 | Level 10 | 5 | User missed the password more than one time |
| 5712 | Level 10 | 1 | sshd: Brute force trying to get access — non-existent user |

**Screenshot:** [01-alert-overview](./evidence/screenshots/01-alert-overview.png)

---

## Investigation Timeline

| Time | Observation |
|---|---|
| 15:00 | First SSH attempt — wronguser@[target-ip] |
| 15:00 | Rule 5710 fired — non-existent user detected |
| 15:01 | Rule 5503 fired — PAM login failed |
| 15:02 | Rule 2502 fired — Level 10 — brute force pattern detected |
| 15:03 | Second round — root@[target-ip] |
| 15:03 | Rule 5712 fired — brute force confirmed |

**Screenshot:** [02-events-list](./evidence/screenshots/04-events-list.png)

---

## Key Evidence

- [E-01 — Alert overview](./evidence/screenshots/01-alert-overview.png) — Wazuh dashboard showing Level 10 escalation
- [E-02 — Threat hunting dashboard](./evidence/screenshots/02-threat-hunting-dashboard.png) — 799 alerts overview
- [E-03 — MITRE wheel](./evidence/screenshots/03-mitre-wheel.png) — T1110 and T1110.001 mapped automatically
- [E-04 — Events list](./evidence/screenshots/04-events-list.png) — Individual rule firings with timestamps
- [E-05 — Events fullscreen](./evidence/screenshots/05-events-fullscreen.png) — 799 hits, 54 pages
- [E-06 — Agent active](./evidence/screenshots/06-agent-active.png) — ubuntu-target confirmed Active

See [evidence-index.md](./evidence/evidence-index.md) for full evidence register.

---

## Findings

- Rule 2502 at Level 10 confirms Wazuh's automatic correlation — individual Level 5 failures were combined into a higher-severity brute force alert without any configuration
- Root SSH is disabled by default on Ubuntu — all root attempts fail immediately at the OS level before reaching authentication
- Both username enumeration (5710) and authentication failure (5503) are logged separately, giving the analyst two independent signals

---

## What a Real Analyst Would Do

Rule 2502 at Level 10 demands immediate attention. A real analyst checks three things: **(1)** did any login succeed after the failures — if yes, confirmed breach, escalate to IR immediately. **(2)** What is the source IP and does it appear elsewhere in logs — check for lateral movement or reconnaissance. **(3)** Is this a known IP or a new one — flag either way and block at firewall. If only failures with no success, the source IP gets blocked and placed on a watchlist. The analyst documents the timeline and keeps the case open for 24 hours watching for follow-up activity.

---

## Response / Remediation

- Source IP blocked at firewall (simulated — lab environment)
- Confirmed no successful logins followed the failed attempts
- Recommended hardening: SSH key-based auth only, disable password auth, deploy fail2ban

---

## MITRE ATT&CK Mapping

| Technique | ID | Why It Applies |
|---|---|---|
| Brute Force | T1110 | Multiple failed login attempts in rapid succession |
| Password Guessing | T1110.001 | Systematic wrong password attempts against known usernames |

---

## Lessons Learned

- Wazuh's built-in correlation is powerful — Level 5 individual events automatically escalate to Level 10 pattern alerts with zero configuration
- Username enumeration and authentication failure are separate signals — treat them as a combined indicator, not two isolated events
- Root SSH being disabled is a default Ubuntu hardening measure — worth verifying on any new server deployment

---

*Part of [esc - SOC Home Lab](https://github.com/MrFixer-02/esc) · Built by [deadlilac](https://github.com/MrFixer-02)*
