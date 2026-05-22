# 🔐 Credentials & Security Guide

> Complete guide for managing credentials in your Wazuh SOC lab on ARM64 (Apple Silicon).

---

## Is Your Lab Safe to Use As-Is?

**Yes — completely.** Your entire lab runs inside UTM's NAT network (`192.168.64.0/24`). This is a private network that only exists inside your Mac. Nobody on the internet, nobody on your home WiFi, nobody anywhere can reach `192.168.64.2` or `192.168.64.4` except your Mac itself.

Even if someone reads the repo and knows the default credentials are `admin/admin` — those IPs don't exist outside your machine. Completely useless to anyone without physical access to your Mac.

> ⚠️ **One rule: Always keep UTM on Shared Network (NAT) mode.** If you switch to Bridged networking, your VMs become visible on your home network. Never use Bridged mode for a security lab.

---

## Default Credentials

| Service | Username | Password | Where Used |
|---|---|---|---|
| Wazuh Dashboard | `admin` | `admin` | `https://192.168.64.4` — web UI login |
| Wazuh Indexer | `admin` | `admin` | OpenSearch API on port 9200 |
| Wazuh API | `wazuh` | `wazuh` | REST API on port 55000 |
| Wazuh API (UI user) | `wazuh-wui` | (auto) | Internal dashboard communication |

> 💡 These are fine to keep for a local NAT lab. Nobody outside your Mac can use them. If you still want to change them — follow the process below.

---

## 🔑 Changing the Dashboard Password (Complete ARM64 Process)

> ⚡ **TRICKY STEP — ARM64 Bug Warning**
> The official `wazuh-passwords-tool.sh` has a known bug on ARM64 (Apple Silicon). Running it corrupts `internal_users.yml` with Java VM debug output mixed into the YAML file, breaking the entire security configuration. **Do NOT use it.** Use the OpenSearch API method below instead.

**If you already ran the tool and broke things — go to the Recovery section first.**

### Step 1 — Verify current credentials work

```bash
curl -k -u admin:admin https://192.168.64.4:9200
```
Should return JSON with `"cluster_name": "wazuh-cluster"`. If you get `Unauthorized` — go to Recovery section below.

### Step 2 — Unreserve the admin user

The admin user is write-protected by default. Unlock it first:
```bash
sudo nano /etc/wazuh-indexer/opensearch-security/internal_users.yml
```

Find the `admin:` section and change `reserved: true` to `reserved: false`:
```yaml
admin:
  hash: "$2a$12$..."
  reserved: false      ← change from true to false
  backend_roles:
  - "admin"
```
Save: `Ctrl+X` → `Y` → Enter

### Step 3 — Apply the config change

```bash
sudo /usr/share/wazuh-indexer/bin/indexer-security-init.sh
sleep 30
```

You should see `SUCC:` lines for each config file. If you see `ERR:` on `internal_users.yml` — the file is corrupted. Go to Recovery section.

### Step 4 — Change the password via OpenSearch API

```bash
curl -k -u admin:admin -X PUT \
  "https://192.168.64.4:9200/_plugins/_security/api/internalusers/admin" \
  -H "Content-Type: application/json" \
  -d '{"password": "YourNewPassword", "backend_roles": ["admin"]}'
```

✅ **Expected output:** `{"status":"OK","message":"'admin' updated."}`

If you get `FORBIDDEN` — you skipped Step 2. Go back and set `reserved: false` first.

### Step 5 — Update the dashboard config to match

```bash
sudo nano /etc/wazuh-dashboard/opensearch_dashboards.yml
```

Find the password line and update it:
```yaml
opensearch.password: "YourNewPassword"
```

Save and restart:
```bash
sudo systemctl restart wazuh-dashboard
sleep 60
```

### Step 6 — Restore reserved status

```bash
sudo nano /etc/wazuh-indexer/opensearch-security/internal_users.yml
```
Change `reserved: false` back to `reserved: true`. Then:
```bash
sudo /usr/share/wazuh-indexer/bin/indexer-security-init.sh
sleep 30
```

### Step 7 — Verify everything works

```bash
curl -k -u admin:YourNewPassword https://192.168.64.4:9200
```
Then login at `https://192.168.64.4` with your new password.

---

## 👤 Changing the Username

Renaming `admin` directly is not recommended — it is referenced in certificate CNs and multiple config files. The correct approach is to create a new admin user and disable the default one.

> 💡 **Why this works:** Wazuh recognizes admin access by **role**, not by username. A new user with the `admin` backend role has identical access to everything. Services don't care what the account is called — only what role it has.

### Step 1 — Find API credentials

The Wazuh API (port 55000) uses a separate user database from the dashboard. Install SQLite to inspect it:

```bash
sudo apt install sqlite3 -y
sudo sqlite3 /var/ossec/api/configuration/security/rbac.db "SELECT id, username FROM users;"
```

Output:
```
1|wazuh
2|wazuh-wui
```

Default API credentials: `wazuh:wazuh`

### Step 2 — Get an API token

```bash
TOKEN=$(curl -su 'wazuh:wazuh' -k -X GET "https://192.168.64.4:55000/security/user/authenticate?raw=true")
echo $TOKEN
```

Should return a long JWT token starting with `eyJ...`

### Step 3 — Create your new user in the Wazuh API

```bash
curl -k -X POST "https://192.168.64.4:55000/security/users" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"username": "yourusername", "password": "YourPassword@123"}'
```

### Step 4 — Give it admin role

```bash
curl -k -X POST "https://192.168.64.4:55000/security/roles/administrator/users" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"users": ["yourusername"]}'
```

### Step 5 — Create the same user in the Indexer

```bash
curl -k -u admin:admin -X PUT \
  "https://192.168.64.4:9200/_plugins/_security/api/internalusers/yourusername" \
  -H "Content-Type: application/json" \
  -d '{"password": "YourPassword@123", "backend_roles": ["admin"]}'
```

### Step 6 — Test before disabling admin

```bash
curl -k -u yourusername:YourPassword@123 https://192.168.64.4:9200
```

Also test dashboard login at `https://192.168.64.4`. Confirm full access before proceeding.

### Step 7 — Disable the default admin account

```bash
curl -k -u yourusername:YourPassword@123 -X PATCH \
  "https://192.168.64.4:9200/_plugins/_security/api/internalusers/admin" \
  -H "Content-Type: application/json" \
  -d '[{"op": "replace", "path": "/attributes/enabled", "value": "false"}]'
```

> ⚠️ Only do this after confirming your new user works. If you lock yourself out you will need to restore from backup.

---

## 🚑 Recovery — If the Password Tool Corrupted Your Config

### Step 1 — Find the backups

```bash
sudo ls /etc/wazuh-indexer/internalusers-backup/
```

### Step 2 — Verify the backup is clean

```bash
sudo cat /etc/wazuh-indexer/internalusers-backup/internal_users_TIMESTAMP.yml.bkp | head -20
```

Should start with clean YAML — `_meta:`, `type: "internalusers"` etc. Use the earliest backup (lowest timestamp).

### Step 3 — Restore

```bash
sudo cp /etc/wazuh-indexer/internalusers-backup/internal_users_TIMESTAMP.yml.bkp \
  /etc/wazuh-indexer/opensearch-security/internal_users.yml
```

### Step 4 — Re-initialize and restart

```bash
sudo /usr/share/wazuh-indexer/bin/indexer-security-init.sh
sleep 30
sudo systemctl restart wazuh-indexer
sleep 45
```

### Step 5 — Verify recovery

```bash
curl -k -u admin:admin https://192.168.64.4:9200
```

Should return JSON. You are back to `admin/admin`. Now use the API method above to change the password — not the tool.

---

## 📋 Complete Error Reference

| Error | Cause | Fix |
|---|---|---|
| `wazuh-passwords-tool.py: command not found` | Wrong file extension | Use `.sh` not `.py`. Full path: `/usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh` |
| `internal_users.yml not in OpenSearch Security 7 format` | Tool corrupted file on ARM64 | Restore from backup in `/etc/wazuh-indexer/internalusers-backup/` |
| `{"status":"FORBIDDEN","message":"Resource 'admin' is reserved."}` | Admin is write-protected | Set `reserved: false` in `internal_users.yml`, run security init, then retry |
| Dashboard "Invalid username or password" after change | Dashboard config has old password | Update `/etc/wazuh-dashboard/opensearch_dashboards.yml` and restart dashboard |
| `Unauthorized` on both old and new password | Config mismatch or services not restarted | Restart in order: indexer (45s) → manager → filebeat → dashboard (60s) |
| API returns `Unauthorized` with `admin:admin` | API uses different user system | API default is `wazuh:wazuh` — install sqlite3 and check `rbac.db` |
| `ModuleNotFoundError: No module named 'api'` | Wrong way to run API script | Don't run `wazuh_apid.py` directly — use curl against the API |
| Token returns `{"title": "Unauthorized"}` | Wrong API credentials | Default API user is `wazuh:wazuh` not `admin:admin` |
| New password works on indexer but not dashboard | Two separate auth systems | Update BOTH: OpenSearch API (port 9200) AND dashboard config file |

---

*Part of the [esc SOC Home Lab](https://github.com/MrFixer-02/esc) · Built by [Komal Kakarla](https://github.com/MrFixer-02)*
