# Rule 100001 — FIM: /etc/shadow Modified

**Rule ID:** 100001  
**Level:** 10 (High)  
**Group:** syscheck  
**File:** `/var/ossec/etc/rules/local_rules.xml`  
**MITRE:** T1003.008 — OS Credential Dumping: /etc/passwd and /etc/shadow  
**First written:** 2026-05-20  
**Verified on rebuild:** 2026-05-22  

---

## Description

Fires when Wazuh FIM detects any modification to `/etc/shadow`. The shadow file stores hashed passwords for all system users — modification indicates possible credential access, credential dumping, or account manipulation.

---

## Rule Syntax

```xml
<rule id="100001" level="10">
  <if_group>syscheck</if_group>
  <match>/etc/shadow</match>
  <description>FIM: /etc/shadow modified - possible credential access or manipulation</description>
  <mitre>
    <id>T1003.008</id>
  </mitre>
</rule>
```

> **Note:** Wazuh 4.14 requires `if_group+match` syntax. The earlier `if_sid+field` approach silently failed in this version.

---

## MITRE ATT&CK

| Field | Value |
|---|---|
| Tactic | Credential Access |
| Technique | OS Credential Dumping |
| Sub-technique | T1003.008 — /etc/passwd and /etc/shadow |

---

## Prerequisites

`/etc` must be added to FIM realtime monitoring in `ossec.conf` **on the agent**:

```xml
<directories realtime="yes">/etc</directories>
```

This goes in `/var/ossec/etc/ossec.conf` on the monitored endpoint, not on the Wazuh server.

---

## Trigger Instructions

From a shell on the monitored agent:

```bash
sudo touch /etc/shadow
```

Any write to `/etc/shadow` fires the rule — the file does not need to actually change content. Normal system activity (package operations, password changes) will also trigger it.

---

## Evidence

From session 2026-05-22 (rebuild and re-verification):

| File | Description |
|---|---|
| [04-custom-rules-100001-100002-firing.png](../../worklogs/evidence/22.05.26/04-custom-rules-100001-100002-firing.png) | Rule 100001 firing at Level 10 in Wazuh dashboard |
| [05-fim-config-realtime.png](../../worklogs/evidence/22.05.26/05-fim-config-realtime.png) | FIM realtime config showing /etc monitored on agent |

**Observed alert text:** `FIM: /etc/shadow modified - possible credential access · T1003.008 · 2 hits`

---

## Lab Notes

- Rule fired 2 times in the 2026-05-22 session — once from a manual `touch /etc/shadow`, once from system activity writing to the file during package operations.
- Realtime FIM on `/etc` must be configured on the **agent** side. Configuring it only on the server has no effect.
- First written 2026-05-20 with `if_sid+field` syntax — rule was silently ignored. Fixed 2026-05-22 by switching to `if_group+match`.

---

*Part of [esc - SOC Home Lab](https://github.com/MrFixer-02/esc) · Built by [deadlilac](https://github.com/MrFixer-02)*
