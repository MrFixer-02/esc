# Changelog

---

## 2026-05-22

**Wazuh rebuild · VirusTotal v3 API migration · Custom rules verified**

- Rebuilt Wazuh stack from scratch — component-by-component install, ARM64 all-in-one dashboard timeout workaround
- Expanded LVM from 23.5G to full 47G
- Re-enrolled ubuntu-siem-target agent ID 002, Active v4.14.5
- Migrated VirusTotal integration from deprecated v2 API to v3 (x-apikey header) — free tier v2 keys return empty responses
- Fixed custom rule syntax — `if_sid+field` silently ignored in Wazuh 4.14, switched to `if_group+match`
- Fixed agent-side FIM realtime config for `/etc` and `/tmp` on ubuntu-siem-target
- Verified full detection chain: EICAR → FIM → VirusTotal 62 engines → rule 87105 Level 12
- Verified custom rules 100001 (T1003.008) and 100002 (T1059) both firing at Level 10

Worklog: [2026-05-22](worklogs/2026-05-22.md) · Evidence: [22.05.26](worklogs/evidence/22.05.26/)

---

## 2026-05-20

**VirusTotal integration · FIM active response pipeline · Custom detection rules**

- Configured VirusTotal integration in `ossec.conf` — FIM events trigger hash lookup via free tier API
- Created `remove-threat.sh` active response script — auto-deletes files confirmed malicious by VirusTotal
- Tested full pipeline end-to-end with EICAR standard test file
- Added `/etc` to realtime FIM monitoring scope
- Wrote custom rules 100001 (`/etc/shadow` modification, T1003.008) and 100002 (executable in `/tmp`, T1059)
- Both rules verified firing against live system events

Worklog: [2026-05-20](worklogs/2026-05-20.md) · Evidence: [20.05.26](worklogs/evidence/20.05.26/)

---

## 2026-05-18

**Phase A complete**

- Wazuh stack deployment, agent enrollment, attack simulations, and case study 001 documentation shipped
- Lab fully operational on Apple Silicon — all ARM64-specific issues documented and resolved
- Case study 001 (SSH Brute Force Detection) published with full evidence register

Case study: [001 — SSH Brute Force](case-studies/001-ssh-brute-force/case-study.md)

---

## 2026-05-15

**Lab initialized**

- Created Ubuntu target VM (Ubuntu 24.04 ARM64) and Wazuh server VM (Ubuntu 22.04 ARM64) in UTM
- Installed Wazuh Indexer, Manager, Filebeat, and Dashboard manually via apt — official installer rejects ARM64
- Enrolled ubuntu-target as first Wazuh agent (DEB aarch64 package)
- Dashboard accessible, agent Active, first alert (rule 5710) fired

Worklog: [2026-05-15](worklogs/2026-05-15.md)

---

*Part of [esc - SOC Home Lab](https://github.com/MrFixer-02/esc) · Built by [deadlilac](https://github.com/MrFixer-02)*
