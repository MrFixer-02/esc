<!-- Opening Banner -->
<p align="center">
  <img src="esc_banner.gif" alt="esc — SOC Home Lab" width="100%"/>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Status-Active_Development-EF9F27?style=flat-square"/>
  <img src="https://img.shields.io/badge/Phase_A-Complete-1D9E75?style=flat-square"/>
  <img src="https://img.shields.io/badge/Platform-Apple_Silicon-000000?style=flat-square&logo=apple&logoColor=white"/>
  <img src="https://img.shields.io/badge/Architecture-ARM64-7B2FBE?style=flat-square"/>
  <img src="https://img.shields.io/badge/License-MIT-4C1D95?style=flat-square"/>
</p>

> 🔄 **This lab is under active development.** New tools, phases, and features are being added regularly. Star or watch this repo to follow along as it grows.

---

## 🧠 What is `esc`?

**`esc`** is a self-hosted, professional-grade SOC (Security Operations Center) home lab built from scratch on Apple Silicon. It simulates the defensive security infrastructure used by real security teams — monitoring endpoints, detecting threats, and generating alerts in real time.

`esc` is the name we gave our own lab. You can call yours anything you like — the steps work the same regardless.

This is not a cloud-based or pre-configured lab. Every component is installed manually, configured properly, and documented completely. That is the point — understanding how it works, not just following instructions blindly.

### What is a SOC Lab?

A SOC lab is a controlled environment where you practice defensive security skills — the same skills used by security analysts in companies, banks, hospitals, and governments. It lets you:

- Monitor systems for suspicious activity in real time
- Detect attacks as they happen and understand what triggered them
- See both sides — what the attacker does and what the defender sees
- Practice with real tools used in the industry every day

### Who is this for?

- Cybersecurity students building practical skills beyond theory
- Career switchers wanting hands-on experience before their first job
- Developers who want to understand the security side of infrastructure
- Anyone curious about how real security monitoring actually works

### The Apple Silicon / macOS Context

Most SOC lab tutorials assume Windows or Intel Linux. This lab is built entirely on **macOS with Apple Silicon (ARM64)**. That means different ISO files, different package types, and a few ARM64-specific troubleshooting steps that Intel users never encounter.

If you are on an M-series Mac, this guide has you covered completely.

> 💡 **Windows or Intel Mac users:** The lab design is identical. Replace UTM with VirtualBox or VMware. Select `amd64` packages instead of `aarch64` wherever mentioned. Everything else — Wazuh, networking, detection — works exactly the same way.

---

## 🛠️ The Stack

### Why Wazuh and Not Other SIEMs?

There are several SIEM options available. Here is why we chose Wazuh:

| SIEM | Reason We Didn't Choose It |
|---|---|
| Splunk | Free tier limited to 500MB/day ingestion — fills up fast in a lab |
| Elastic Security | Heavy resource usage, significantly steeper learning curve |
| Microsoft Sentinel | Cloud-only, requires Azure subscription |
| Wazuh Cloud | 14-day trial expires — lab dies before you finish building it |
| **Wazuh Self-hosted** | ✅ Free forever · Full control · ARM64 compatible · Industry-standard |

Wazuh also ships with 3,000+ built-in detection rules, MITRE ATT&CK mapping, file integrity monitoring, vulnerability detection, and compliance reporting — all out of the box, no configuration required.

### Why Self-Hosted and Not Cloud?

- The cloud trial expires in 14 days — not enough time to learn and build
- Self-hosting teaches you the full architecture, not just the dashboard UI
- Your data stays on your machine — no third party involved
- It is completely free with no limits

### Recommended Specs

| Component | Minimum | Recommended |
|---|---|---|
| **RAM** | 16 GB | 24 GB+ |
| **Storage** | 100 GB free | 200 GB+ |
| **CPU** | 4 cores | 8 cores |
| **OS** | macOS with Apple Silicon | macOS Ventura or later |

> ⚠️ **Why so much RAM?** Each VM needs its own allocation. Wazuh server ~4 GB, Ubuntu target ~4 GB, future Kali VM ~4 GB, macOS itself ~6 GB. You need 16-24 GB minimum for a comfortable multi-VM setup.

### Technologies & Versions

> ⚠️ **Version compatibility:** The versions below are tested and verified to work together. If you want to use newer versions, upgrade all related components together — especially Wazuh and Filebeat, which must be version-matched. Mismatched versions cause silent connection failures. Always check [Wazuh's compatibility matrix](https://documentation.wazuh.com/current/getting-started/components/index.html) before upgrading.

| Tool | Version Used | Purpose | Download |
|---|---|---|---|
| ![Homebrew](https://img.shields.io/badge/Homebrew-Latest-FBB040?style=flat-square&logo=homebrew&logoColor=black) | Latest | macOS package manager | [brew.sh](https://brew.sh) |
| ![UTM](https://img.shields.io/badge/UTM-4.x-555555?style=flat-square) | 4.x | Hypervisor — runs all VMs | [mac.getutm.app](https://mac.getutm.app) |
| ![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04_LTS-E95420?style=flat-square&logo=ubuntu&logoColor=white) | 22.04.5 LTS | Wazuh server OS — must be 22.04 | [ubuntu.com](https://ubuntu.com/download/server/arm) |
| ![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420?style=flat-square&logo=ubuntu&logoColor=white) | 24.04 LTS | Target VM OS | [ubuntu.com](https://ubuntu.com/download/server/arm) |
| ![Wazuh](https://img.shields.io/badge/Wazuh-4.7.5-7B2FBE?style=flat-square) | 4.7.5 | SIEM — full stack | [packages.wazuh.com](https://packages.wazuh.com/4.x/apt/) |
| ![Filebeat](https://img.shields.io/badge/Filebeat-7.10.2-005571?style=flat-square&logo=elastic&logoColor=white) | 7.10.2 | Log shipper — must match Wazuh | via apt |
| ![Nmap](https://img.shields.io/badge/Nmap-Latest-4A90D9?style=flat-square) | Latest | Network scanner | `brew install nmap` |
| ![Git](https://img.shields.io/badge/Git-Latest-F05032?style=flat-square&logo=git&logoColor=white) | Latest | Version control | `brew install git` |

**Mac/ARM64 specific:** UTM, Homebrew, ARM64 ISO files, `aarch64` agent package
**Universal (works anywhere):** Wazuh 4.7.5, Filebeat 7.10.2, Nmap, Git

### VM Inventory

| VM | OS | IP | RAM | Disk | Role |
|---|---|---|---|---|---|
| `wazuh-server` | Ubuntu 22.04 ARM64 | 192.168.64.4 | 4 GB | 50 GB | SIEM brain |
| `Ubuntu-Target-SIEM` | Ubuntu 24.04 ARM64 | 192.168.64.2 | 4 GB | 32 GB | Monitored endpoint |
| Kali Linux | Kali ARM64 | TBD | 4 GB | 40 GB | Attacker VM *(coming soon)* |

### Network

All VMs run on UTM Shared Network NAT mode — subnet `192.168.64.0/24`. VMs talk to each other and reach the internet, but are isolated from your home network.

---

## 🔧 Building Your SOC Lab

> ✅ **Tested on:** MacBook Pro M5 · UTM 4.x · macOS Sonoma · May 2026
>
> These steps are verified to work on this setup. If you are on a different M-series Mac (M1/M2/M3/M4) the steps are identical. Different UTM versions may have slightly different UI but the same core functionality.

### Before You Start

This guide installs Wazuh **component by component** using apt, not the all-in-one installer script. The official `wazuh-install.sh` rejects ARM64 systems with a 64-bit compatibility error despite ARM64 being fully 64-bit. We skip it entirely and install each component directly — this also gives you a better understanding of the architecture.

### Tricky Step Legend

| Tag | Meaning |
|---|---|
| ⚡ **TRICKY STEP** | High chance of error — read the full step before running anything |
| 💡 **NOTE** | Important context that affects the next steps |
| ✅ **VERIFY** | Run this check before continuing — do not skip |

---

### Part 0 — Homebrew (Start Here)

**Homebrew** is macOS's package manager — think of it as the App Store for terminal tools. One command installs almost anything. You will use it throughout this lab and beyond.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Install essential tools:
```bash
brew install nmap wget git
brew install --cask utm
```

> 💡 Every tool on your Mac gets installed through Homebrew. Everything inside the Linux VMs gets installed through apt. Two package managers, two environments — keep them separate.

---

### Part 1 — Ubuntu Target VM

**1.1 — Download Ubuntu 24.04 ARM64 ISO**
```bash
cd ~/Downloads
wget https://cdimage.ubuntu.com/releases/24.04/release/ubuntu-24.04-live-server-arm64.iso
```

✅ **VERIFY** — File should be ~2 GB:
```bash
ls -lh ~/Downloads/ubuntu-24.04-live-server-arm64.iso
```

**1.2 — Create VM in UTM**

Open UTM → `+` → **Virtualize** → **Linux**

| Setting | Value |
|---|---|
| Boot ISO | ubuntu-24.04-live-server-arm64.iso |
| RAM | 4096 MB · CPU: 2 cores · Storage: 32 GB |
| Network | Shared Network (NAT) |
| Name | `Ubuntu-Target-SIEM` |

⚡ **TRICKY STEP** — Always select **Virtualize**, not Emulate. Emulate is 10-20x slower.

**1.3 — Install Ubuntu Server**

- Language: English · Keyboard: English (US)
- Type: **Ubuntu Server** (not minimized)
- Network: leave as default
- Storage: Use entire disk — **uncheck LVM**
- ⚡ **TRICKY STEP** — On the SSH screen, check **Install OpenSSH server**. Without this you cannot SSH in from Mac Terminal.
- Snaps: skip everything

⚡ **TRICKY STEP — Post-install reboot loop**
If the VM boots back into the installer: shut down → UTM VM Settings → Drives → remove the ISO entry → reboot.

**1.4 — SSH in from Mac**
```bash
ip addr show          # inside VM — note the 192.168.64.X address
ssh yourusername@192.168.64.2   # from Mac Terminal
```

✅ **VERIFY** — Prompt shows `yourusername@Ubuntu-Target-SIEM:~$`

---

### Part 2 — Wazuh Server VM

💡 **NOTE** — The Wazuh server must run **Ubuntu 22.04 LTS**, not 24.04. Using 24.04 causes package dependency failures.

**2.1 — Download Ubuntu 22.04 ARM64 ISO**
```bash
wget https://cdimage.ubuntu.com/releases/22.04/release/ubuntu-22.04.5-live-server-arm64.iso
```

✅ **VERIFY** — File should be ~1.9 GB.

**2.2 — Create VM in UTM**

| Setting | Value |
|---|---|
| Boot ISO | ubuntu-22.04.5-live-server-arm64.iso |
| RAM | 4096 MB · CPU: 2 cores · Storage: **50 GB** |
| Network | Shared Network (NAT) · Name: `wazuh-server` |

**2.3 — Install Ubuntu and SSH in**

Same process as Part 1. Enable SSH. Then from Mac Terminal:
```bash
ssh yourusername@192.168.64.4
```

All remaining commands run inside this SSH session.

---

### Part 3 — Install Wazuh

💡 **NOTE** — Running `wazuh-install.sh` on ARM64 returns: `ERROR: Incompatible system. This script must be run on a 64-bit system.` This is a bug in the script's architecture check. We install each component via apt instead.

**3.1 — Update system and add Wazuh repository**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl gnupg2 -y

curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring \
  --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && \
  sudo chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | \
  sudo tee /etc/apt/sources.list.d/wazuh.list

sudo apt update
```

**3.2 — Install Wazuh Indexer**
```bash
sudo apt install wazuh-indexer -y
```

**3.3 — Generate SSL Certificates**

⚡ **TRICKY STEP** — chmod files individually FIRST, then lock the directory. Locking first blocks your own access to the files inside.

```bash
sudo curl -sO https://packages.wazuh.com/4.7/wazuh-certs-tool.sh
sudo curl -sO https://packages.wazuh.com/4.7/config.yml

sudo sed -i 's/<indexer-node-ip>/192.168.64.4/g' config.yml
sudo sed -i 's/<wazuh-manager-ip>/192.168.64.4/g' config.yml
sudo sed -i 's/<dashboard-node-ip>/192.168.64.4/g' config.yml

sudo bash ./wazuh-certs-tool.sh -A
sudo chmod 755 ~/wazuh-certificates/
sudo chmod 644 ~/wazuh-certificates/*.pem
```

**3.4 — Deploy Indexer certificates**
```bash
sudo mkdir /etc/wazuh-indexer/certs
sudo cp ~/wazuh-certificates/node-1.pem /etc/wazuh-indexer/certs/indexer.pem
sudo cp ~/wazuh-certificates/node-1-key.pem /etc/wazuh-indexer/certs/indexer-key.pem
sudo cp ~/wazuh-certificates/admin.pem /etc/wazuh-indexer/certs/admin.pem
sudo cp ~/wazuh-certificates/admin-key.pem /etc/wazuh-indexer/certs/admin-key.pem
sudo cp ~/wazuh-certificates/root-ca.pem /etc/wazuh-indexer/certs/root-ca.pem
sudo chmod 400 /etc/wazuh-indexer/certs/*.pem
sudo chmod 500 /etc/wazuh-indexer/certs
sudo chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/certs
```

**3.5 — Start Indexer and initialize security**
```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-indexer
sudo systemctl start wazuh-indexer
sleep 30
sudo /usr/share/wazuh-indexer/bin/indexer-security-init.sh
sleep 30
sudo systemctl restart wazuh-indexer
sleep 30
```

✅ **VERIFY:**
```bash
curl -k -u admin:admin https://192.168.64.4:9200
```
Should return JSON with `"cluster_name": "wazuh-cluster"`. If you see `OpenSearch Security not initialized` — wait 30 more seconds and retry.

**3.6 — Install Wazuh Manager**
```bash
sudo apt install wazuh-manager -y
sudo systemctl daemon-reload
sudo systemctl enable wazuh-manager
sudo systemctl start wazuh-manager
```

✅ **VERIFY:** `sudo systemctl status wazuh-manager --no-pager` → `Active: active (running)`

**3.7 — Install and configure Filebeat**

⚡ **TRICKY STEP** — The default Filebeat template uses `${username}` and `${password}` placeholders that must be replaced manually. It also references SSL cert filenames that don't match what the cert tool generates — both need fixing before Filebeat will start.

```bash
sudo apt install filebeat -y
sudo curl -so /etc/filebeat/filebeat.yml \
  https://packages.wazuh.com/4.7/tpl/wazuh/filebeat/filebeat.yml
sudo sed -i 's|hosts:.*|hosts: ["192.168.64.4:9200"]|' /etc/filebeat/filebeat.yml
sudo nano /etc/filebeat/filebeat.yml
```

Find `output.elasticsearch:` and make it look exactly like this:
```yaml
output.elasticsearch:
  username: "admin"
  password: "admin"
  hosts: ["192.168.64.4:9200"]
  protocol: https
  ssl.certificate_authorities:
    - /etc/filebeat/certs/root-ca.pem
  ssl.certificate: "/etc/filebeat/certs/wazuh-manager.pem"
  ssl.key: "/etc/filebeat/certs/wazuh-manager-key.pem"
```

Remove any `${username}` or `${password}` lines. Save: `Ctrl+X` → `Y` → Enter.

```bash
sudo curl -so /tmp/wazuh-filebeat.tar.gz \
  https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.4.tar.gz
sudo tar -xvz -f /tmp/wazuh-filebeat.tar.gz -C /usr/share/filebeat/module

sudo curl -so /etc/filebeat/wazuh-template.json \
  https://raw.githubusercontent.com/wazuh/wazuh/v4.7.5/extensions/elasticsearch/7.x/wazuh-template.json
sudo chmod go+r /etc/filebeat/wazuh-template.json

sudo mkdir /etc/filebeat/certs && sudo chmod 755 /etc/filebeat/certs
sudo cp ~/wazuh-certificates/root-ca.pem /etc/filebeat/certs/root-ca.pem
sudo cp ~/wazuh-certificates/wazuh-1.pem /etc/filebeat/certs/wazuh-manager.pem
sudo cp ~/wazuh-certificates/wazuh-1-key.pem /etc/filebeat/certs/wazuh-manager-key.pem
sudo chmod 400 /etc/filebeat/certs/*.pem
sudo chmod 500 /etc/filebeat/certs

sudo systemctl daemon-reload
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

✅ **VERIFY:** `sudo systemctl status filebeat --no-pager` → `Active: active (running)`

**3.8 — Install Wazuh Dashboard**

⚡ **TRICKY STEP** — The dashboard expects `dashboard.pem` but the cert tool generates `wazuh-dashboard.pem`. Fix with symlinks.

```bash
sudo apt install wazuh-dashboard -y

sudo mkdir /etc/wazuh-dashboard/certs && sudo chmod 755 /etc/wazuh-dashboard/certs
sudo cp ~/wazuh-certificates/root-ca.pem /etc/wazuh-dashboard/certs/root-ca.pem
sudo cp ~/wazuh-certificates/dashboard.pem /etc/wazuh-dashboard/certs/wazuh-dashboard.pem
sudo cp ~/wazuh-certificates/dashboard-key.pem /etc/wazuh-dashboard/certs/wazuh-dashboard-key.pem
sudo chmod 400 /etc/wazuh-dashboard/certs/*.pem
sudo ln -s /etc/wazuh-dashboard/certs/wazuh-dashboard.pem /etc/wazuh-dashboard/certs/dashboard.pem
sudo ln -s /etc/wazuh-dashboard/certs/wazuh-dashboard-key.pem /etc/wazuh-dashboard/certs/dashboard-key.pem
sudo chmod 500 /etc/wazuh-dashboard/certs
sudo chown -R wazuh-dashboard:wazuh-dashboard /etc/wazuh-dashboard/certs

sudo systemctl daemon-reload
sudo systemctl enable wazuh-dashboard
sudo systemctl start wazuh-dashboard
```

Wait 60-90 seconds. Then open `https://192.168.64.4` in your Mac browser.

Browser will show "Not Secure" — normal for self-signed certificates. Click Advanced → Proceed.

Login: `admin` / `admin` ← change this after first login.

---

### Part 4 — Enroll the Agent

**4.1 — Get install command from dashboard**

Dashboard → ☰ → **Endpoints** → **Deploy new agent**

- OS: Linux · Package: **DEB aarch64** ← do NOT pick amd64
- Server address: `192.168.64.4` · Agent name: `ubuntu-target`

Copy the generated command.

**4.2 — Install on the target VM**
```bash
ssh yourusername@192.168.64.2
# paste the generated command
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

✅ **VERIFY** — Dashboard → Endpoints → `ubuntu-target` shows green **Active** within 60 seconds.

---

## 🚨 Alerts & Detection

### Trigger Your First Alert

From Mac Terminal:
```bash
ssh wronguser@192.168.64.2
```
Wrong password 3 times → `Ctrl+C`. Repeat once more.

### What You Will See

Dashboard → **Threat Hunting → Events**:

| Rule ID | Level | Description |
|---|---|---|
| `5710` | 5 | sshd: Attempt to login using a non-existent user |
| `5503` | 5 | PAM: User login failed |
| `2502` | 10 | User missed the password more than one time |

Wazuh automatically maps these to **MITRE ATT&CK T1110 — Brute Force**.

### What Wazuh Detects Out of the Box

- Authentication events — failed logins, sudo, new users
- File integrity — changes to `/etc/passwd`, `/etc/shadow`, critical files
- Process monitoring — suspicious processes, privilege escalation
- Log analysis — syslog, auth.log, application logs
- Vulnerability detection — CVEs in installed packages
- Rootkit detection — common rootkit signatures

---

## 🔭 Make It Yours

### Popular Additions

**Attack capability:** [Kali Linux](https://www.kali.org/get-kali/#kali-virtual-machines) · [Metasploit](https://www.metasploit.com/) · [Burp Suite](https://portswigger.net/burp/communitydownload)

**More detection:** [Suricata](https://suricata.io/) · [Zeek](https://zeek.org/) · [Wireshark](https://www.wireshark.org/)

**More SIEMs:** [Splunk Free](https://www.splunk.com/en_us/download.html) · [Elastic Security](https://www.elastic.co/security) · [Microsoft Sentinel](https://azure.microsoft.com/en-us/products/microsoft-sentinel)

**Vulnerable targets:** [DVWA](https://github.com/digininja/DVWA) · [Metasploitable](https://sourceforge.net/projects/metasploitable/) · [VulnHub](https://www.vulnhub.com/)

**Incident response:** [TheHive](https://thehive-project.org/) · [Shuffle](https://shuffler.io/) · [OpenCTI](https://www.opencti.io/)

⭐ **Star** · 👁️ **Watch** · 🍴 **Fork** — to follow updates or build your own version.

---

## 📋 Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| `ERROR: Incompatible system. Must be 64-bit` | Wazuh installer rejects ARM64 | Install components via apt — skip the installer script |
| `OpenSearch Security not initialized` | Indexer not fully warmed up | Wait 30s, restart indexer, re-run security init |
| `missing field: output.elasticsearch.username` | Filebeat config placeholders | Replace `${username}` and `${password}` in filebeat.yml |
| `ENOENT: dashboard-key.pem not found` | Cert filename mismatch | Symlink `dashboard.pem → wazuh-dashboard.pem` |
| `chmod: cannot access certs/*` | Directory locked before files chmod'd | chmod files first, then chmod the directory |
| VM boots back into installer | ISO still attached in UTM | Remove ISO from VM Settings → Drives |
| Agent shows `Never Connected` | Agent cannot reach manager | Check Manager is running, verify IP in `/var/ossec/etc/ossec.conf` |

---

## 🐛 Found an Issue?

If a step didn't work for you — open a GitHub Issue. Include your Mac model, macOS version, and UTM version. Reports from different setups help make this guide better for everyone.

👉 [Open an Issue](https://github.com/MrFixer-02/esc/issues)

---

<!-- Closing Banner -->
<p align="center">
  <img src="esc_closing_banner.gif" alt="The defense is built. Now watch it work." width="100%"/>
</p>

<p align="center">
  <sub>
    Built by <a href="https://github.com/MrFixer-02">Komal Kakarla</a> ·
    <a href="https://www.linkedin.com/in/kk117">LinkedIn</a> ·
    MIT License
  </sub>
</p>
