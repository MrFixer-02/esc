# Changelog

---

## v1.2.0 — May 22, 2026

**VirusTotal v3 · Active Response · Custom Detection Rules**

- Migrated VirusTotal integration to v3 API (x-apikey header) — free tier v2 keys return empty responses
- Configured active response pipeline: EICAR → FIM → VirusTotal (62 engines) → auto-delete via remove-threat.sh
- Custom rules 100001 (T1003.008 — /etc/shadow modification) and 100002 (T1059 — executable in /tmp) verified firing at Level 10
- Total alerts: 4230+ · MITRE ATT&CK techniques detected: 10+

Worklog: [2026-05-22](worklogs/2026-05-22.md)

---

## v1.1.0 — May 18, 2026

**Lab Foundation · SSH Brute Force Detection · Case Study 001**

- Wazuh stack deployed on Apple Silicon — Indexer, Manager, Filebeat, Dashboard installed component by component via apt
- Agent enrolled and active — ubuntu-siem-target reporting to SIEM server
- SSH brute force simulation run — 799 alerts generated, 9 MITRE ATT&CK techniques detected
- Case study 001 (SSH Brute Force Detection) published with full evidence register

Case study: [001 — SSH Brute Force](case-studies/001-ssh-brute-force/case-study.md)

---

*Part of [esc - SOC Home Lab](https://github.com/MrFixer-02/esc) · Built by [deadlilac](https://github.com/MrFixer-02)*

## 2026-05-23 — VirusTotal free tier (4 req/min) is insufficient for realtime FIM on an active system. FIM generates burst events on eve
