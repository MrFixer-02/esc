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

## Skills Being Built

| Skill | What It Looks Like in This Lab |
|---|---|
| SOC Analysis | Triaging alerts, reading rule firings, understanding what triggered and why |
| Incident Response | Building timelines, documenting findings, writing case studies from real detections |
| Threat Hunting | Going beyond alerts to look for patterns and anomalies in event data |
| Detection Engineering | Writing custom Wazuh rules to catch what built-in rules miss |
| Vulnerability Assessment | Using Nmap to enumerate services, identify exposure, validate hardening |

---

## Lab Status

| Metric | Current |
|---|---|
| Total alerts generated | 4230+ |
| MITRE ATT&CK techniques detected | 10+ |
| Case studies complete | 1 |
| Monitored endpoints | 1 |
| Custom detection rules written | 2 |

---

## What is Inside This Repo

```
esc/
├── case-studies/       One folder per investigation — report and evidence
├── worklogs/           Session logs — what was tested, found, and learned
├── detections/         Custom Wazuh rules — syntax, MITRE mapping, evidence
├── build-your-own/     Full step-by-step guide to replicate this lab
├── references/         Reference docs — credentials, lessons learned
└── assets/             Architecture diagrams, banners
```

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
