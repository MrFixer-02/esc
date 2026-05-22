# Detections

Custom detection rules written and verified in the esc SOC lab.

Rules live in `/var/ossec/etc/rules/local_rules.xml` on the Wazuh server. All rules use `if_group+match` syntax — required in Wazuh 4.14+. The earlier `if_sid+field` approach silently fails in this version.

---

## Index

| Rule ID | Name | MITRE | Level | Verified |
|---|---|---|---|---|
| [100001](rules/100001-etc-shadow-modified.md) | FIM: /etc/shadow modified | T1003.008 | 10 | 2026-05-22 |
| [100002](rules/100002-executable-in-tmp.md) | FIM: Executable dropped in /tmp | T1059 | 10 | 2026-05-22 |

---

*Part of [esc - SOC Home Lab](https://github.com/MrFixer-02/esc) · Built by [deadlilac](https://github.com/MrFixer-02)*
