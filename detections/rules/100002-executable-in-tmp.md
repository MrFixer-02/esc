# Rule 100002 — FIM: Executable Dropped in /tmp

**Rule ID:** 100002  
**Level:** 10 (High)  
**Group:** syscheck  
**File:** `/var/ossec/etc/rules/local_rules.xml`  
**MITRE:** T1059 — Command and Scripting Interpreter  
**First written:** 2026-05-20  
**Verified on rebuild:** 2026-05-22  

---

## Description

Fires when Wazuh FIM detects a file added to `/tmp`. Executables and payloads dropped in `/tmp` are a common indicator of malware staging — attackers use the world-writable `/tmp` directory to land files before execution or exfiltration.

---

## Rule Syntax

```xml
<rule id="100002" level="10">
  <if_group>syscheck</if_group>
  <match>/tmp</match>
  <description>FIM: Executable file dropped in /tmp - possible malware staging</description>
  <mitre>
    <id>T1059</id>
  </mitre>
</rule>
```

> **Note:** Wazuh 4.14 requires `if_group+match` syntax. The earlier `if_sid+field` approach silently failed in this version.

---

## MITRE ATT&CK

| Field | Value |
|---|---|
| Tactic | Execution |
| Technique | Command and Scripting Interpreter |
| ID | T1059 |

---

## Prerequisites

`/tmp` must be monitored by FIM in `ossec.conf` **on the agent**:

```xml
<directories realtime="yes">/tmp</directories>
```

This goes in `/var/ossec/etc/ossec.conf` on the monitored endpoint, not on the Wazuh server.

---

## Trigger Instructions

Drop any file into `/tmp` on the monitored agent:

```bash
cp /bin/ls /tmp/test-exec
chmod +x /tmp/test-exec
```

The EICAR standard test file was used in the lab — it also triggers the VirusTotal integration:

```bash
echo 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*' > /tmp/eicar-test.txt
```

When VirusTotal integration is active, the full chain fires: FIM → rule 100002 (Level 10) → VirusTotal hash lookup → rule 87105 (Level 12) → active response auto-delete.

---

## Evidence

From session 2026-05-22 (rebuild and re-verification):

| File | Description |
|---|---|
| [04-custom-rules-100001-100002-firing.png](../../worklogs/evidence/22.05.26/04-custom-rules-100001-100002-firing.png) | Rules 100001 and 100002 both firing at Level 10 |
| [03-virustotal-alert-62-engines.png](../../worklogs/evidence/22.05.26/03-virustotal-alert-62-engines.png) | VirusTotal enrichment — 62 engines detected EICAR |
| [05-fim-config-realtime.png](../../worklogs/evidence/22.05.26/05-fim-config-realtime.png) | FIM realtime config showing /tmp monitored on agent |

**Observed alert text:** `FIM: Executable file dropped in /tmp - possible malware staging · T1059 · 1 hit`

---

## Lab Notes

- Rule fired 1 time in the 2026-05-22 session — triggered by EICAR test file dropped at `/tmp/eicar-test.txt`.
- Rule 100002 fires before VirusTotal returns a result — it is a staging indicator independent of hash reputation.
- Free tier VirusTotal rate limit (4 req/min) caused rule 87101 errors during rapid testing but did not prevent rule 100002 from firing.
- First written 2026-05-20 with `if_sid+field` syntax — rule was silently ignored. Fixed 2026-05-22 by switching to `if_group+match`.

---

*Part of [esc - SOC Home Lab](https://github.com/MrFixer-02/esc) · Built by [deadlilac](https://github.com/MrFixer-02)*
