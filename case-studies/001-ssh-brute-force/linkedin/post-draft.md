# LinkedIn Draft — Case 001 SSH Brute Force

---

Ran an SSH brute force simulation in my Wazuh lab and documented what actually happened on the detection side.

The interesting part wasn't the attack. It was watching Wazuh escalate automatically.

Individual failed logins fired at Level 5 — low priority, informational.
After a few rounds, rule 2502 fired at Level 10 — brute force pattern detected.

No custom configuration. No tuning. The built-in correlation logic just worked.

A few things I noted:
- Username enumeration (rule 5710) and authentication failure (rule 5503) are separate signals — together they tell a cleaner story
- Root SSH is disabled on Ubuntu by default — all root attempts fail before reaching auth
- The Level 10 escalation is what a real analyst would actually act on, not the individual Level 5 events

Full case study with evidence, MITRE mapping, and analyst response notes:
github.com/MrFixer-02/esc

Next: documenting the port scan that came before this — building the full reconnaissance → attack chain.

#cybersecurity #soc #wazuh #blueteam #homelab #applesilicon
