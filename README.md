<h1 align="center">esc — SOC Home Lab</h1>

<p align="center">
  <img src="https://img.shields.io/badge/Status-Active_Development-EF9F27?style=flat-square"/>
  <img src="https://img.shields.io/badge/Phase_A-Complete-1D9E75?style=flat-square"/>
  <img src="https://img.shields.io/badge/Platform-Apple_Silicon-000000?style=flat-square&logo=apple&logoColor=white"/>
  <img src="https://img.shields.io/badge/Architecture-ARM64-7B2FBE?style=flat-square"/>
  <img src="https://img.shields.io/badge/License-MIT-4C1D95?style=flat-square"/>
</p>

> 🔄 **This lab is under active development.** New phases, detections, and case studies added regularly. Star or watch to follow along.

---

## What is `esc`?

**`esc`** is my personal SOC (Security Operations Center) home lab — built from scratch on Apple Silicon, documented completely, and grown one phase at a time.

This is not a tutorial repo. It is a working lab I use to build real defensive security skills — deploying infrastructure, simulating attacks, analyzing detections, and writing case studies. Everything gets documented. Every meaningful finding becomes a case study.

---

## What I Am Building

A self-hosted security operations environment that mirrors what real security teams use — running entirely on my Mac using virtual machines, isolated from the internet, no cloud dependencies.

```
Phase A — Foundation      Wazuh SIEM · monitored endpoint · first detections     ✅ Complete
Phase B — Attacker VM     Kali Linux · real attack tools · network detection      ⏳ Next
Phase C and beyond        Web apps · network IDS · more SIEMs · detection eng.    📌 Planned
```

Each phase builds on the previous one. The goal is a lab that covers the full attack-detection chain end to end.

---

## What is Inside This Repo

```
esc/
├── case-studies/       One folder per investigation — report, evidence, detections
├── worklogs/           Daily session logs — what was tested, found, and learned
├── build-your-own/     Full step-by-step guide to replicate this lab
├── docs/               Reference docs — credentials, lessons learned, templates
└── assets/             Architecture diagrams, banners
```

---

## Phase A — What I Found

Full Wazuh SIEM deployed on Apple Silicon. One monitored endpoint enrolled. Seven attack types simulated manually.

**799 alerts · 9 MITRE ATT&CK techniques · 0 custom rules written**

Key finding: defense evasion failed completely. Attempted to delete auth logs after attacks — Wazuh had already shipped everything to the SIEM server. Log deletion had zero effect.

👉 [Phase A Case Study](case-studies/001-ssh-brute-force/case-study.md)

---

## Case Studies

| # | Title | MITRE Techniques | Status |
|---|---|---|---|
| [001](case-studies/001-ssh-brute-force/) | SSH Brute Force Detection | T1110 · T1110.001 | ✅ Complete |
| 002 | Port Scan Reconnaissance | T1046 | 📌 Planned |
| 003 | Privilege Escalation | T1548 | 📌 Planned |

---

## Want to Build Your Own Lab?

Full step-by-step guide — Parts 0 through 4, ARM64-specific issues documented, every error and fix included.

👉 [Build Your Own SOC Lab](build-your-own/README.md)

---

## Follow the Progress

- ⭐ **Star** this repo to bookmark it
- 👁️ **Watch** → All Activity for updates
- 💬 **Discussions** — questions, your own results, suggestions

<!-- Closing Banner -->
<p align="center">
  <img src="assets/esc_closing_banner.gif" alt="The defense is built. Now watch it work." width="100%"/>
</p>

<p align="center">
  <sub>Built by <a href="https://github.com/MrFixer-02">deadlilac</a> · <a href="https://www.linkedin.com/in/kk117">LinkedIn</a> · MIT License</sub>
</p>
