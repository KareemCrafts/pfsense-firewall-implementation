# 🔒 pfSense Firewall Implementation

![pfSense](https://img.shields.io/badge/pfSense-2.8.1-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

> A complete hands-on implementation of pfSense as a network firewall — from installation to live traffic blocking

---

## 📡 Network Architecture

\```
[Windows PC] ──→ [pfSense Firewall] ──→ [Home Router] ──→ [Internet]
 192.168.1.8        192.168.1.7           192.168.1.1
\```

---

## 📸 Step-by-Step Walkthrough

---

### ✅ Step 1 — pfSense Console: Initial Boot

![Step 1](screenshots/01-pfsense-console-boot.png)

After booting the pfSense VM, the console menu appears confirming the system is live.

| Setting | Value |
|--------|-------|
| WAN Interface | `em0` |
| Assigned IP | `192.168.1.7/24` |

> This confirms pfSense booted successfully. The IP `192.168.1.7` is what we use to access the web interface and what client machines use as their gateway.

---

### ✅ Step 2 — Web Interface Login

![Step 2](screenshots/02-web-login-page.png)

Accessed the pfSense web GUI by navigating to `https://192.168.1.7` in a browser.

| Field | Value |
|-------|-------|
| Username | `admin` |
| Password | `pfsense` |

> The web interface lets us configure everything visually — firewall rules, aliases, gateways — without needing the command line.

---

### ✅ Step 3 — Installation Wizard

![Step 3](screenshots/03-installation-wizard.png)

Selected **Install** from the wizard to permanently install pfSense onto the VM's virtual hard drive.

> After installation, pfSense boots automatically from disk without the ISO — making it a persistent firewall on the network.

---

### ✅ Step 4 — General System Configuration

![Step 4](screenshots/04-general-info-config.png)

Configured core system identity settings in Setup Wizard Step 2 of 9.

| Setting | Value |
|--------|-------|
| Hostname | `KareemsFirewall` |
| Domain | `localdomain` |
| Primary DNS | `8.8.8.8` |
| Secondary DNS | `8.8.8.8` |
| Override DNS | ✅ Enabled |

> The hostname identifies this device on the network. DNS servers allow pfSense to resolve domain names — critical for rule enforcement and system updates.

---

### ✅ Step 5 — WAN Interface Configuration

![Step 5](screenshots/05-wan-interface-config.png)

Configured the WAN interface — pfSense's connection to the internet through the home router.

| Setting | Value |
|--------|-------|
| Configuration Type | Static |
| IP Address | `192.168.1.7` |
| Subnet Mask | `255.255.255.0 (/24)` |
| Upstream Gateway | `192.168.1.1` |

> A static IP ensures pfSense always has the same address. The gateway `192.168.1.1` tells pfSense where to forward internet-bound traffic.

---

### ✅ Step 6 — pfSense Dashboard

![Step 6](screenshots/06-dashboard.png)

The main dashboard loaded after setup — confirming everything is configured and running.

| Info | Value |
|------|-------|
| System Name | `KareemsFirewall.localdomain` |
| pfSense Version | `2.8.1-RELEASE` |
| Uptime | 11 Minutes 49 Seconds |
| DNS Servers | `127.0.0.1`, `192.168.1.1`, `8.8.8.8` |

---

### ✅ Step 7 — System Resource Metrics

![Step 7](screenshots/07-system-metrics.png)

Reviewed the lower dashboard panel showing live system performance.

| Metric | Value |
|--------|-------|
| CPU Usage | 12% |
| Memory Usage | 26% of 968 MiB |
| SWAP Usage | 0% of 1024 MiB |
| Disk Usage | 7% of 12G |

> Low resource usage confirms pfSense is healthy and has plenty of headroom to inspect and filter traffic without performance issues.

---

### ✅ Step 8 — Firewall Menu Navigation

![Step 8](screenshots/08-firewall-menu.png)

Opened the **Firewall** dropdown to explore available configuration options.

| Option | Purpose |
|--------|---------|
| Aliases | Shortcuts for IPs/networks |
| NAT | Network Address Translation |
| Rules | Create and manage firewall rules |
| Schedules | Time-based rule activation |
| Traffic Shaper | Bandwidth management |

---

### ✅ Step 9 — Creating a Firewall Alias

![Step 9](screenshots/09-alias-config.png)

Created an alias called `RedHuntDOTNet` to represent a website by a friendly name instead of a raw IP.

| Setting | Value |
|--------|-------|
| Name | `RedHuntDOTNet` |
| Type | Host(s) |
| IP Address | `3.11.197.46` |

> Aliases act as reusable labels. If an IP changes, you update the alias once and every rule using it updates automatically.

---

### ✅ Step 10 — Allow-All Rule Configuration

![Step 10](screenshots/10-allow-all-rule.png)

Created a Pass rule that allows all traffic not caught by rules above it.

| Setting | Value |
|--------|-------|
| Action | Pass |
| Interface | WAN |
| Protocol | Any |
| Source | Any |
| Destination | Any |

> ⚠️ **Critical:** This rule MUST sit below all block rules. If placed above them, it allows everything before block rules are ever checked — making them completely useless.

---

### ✅ Step 11 — Final Firewall Rules Configuration

![Step 11](screenshots/11-final-rules.png)

The completed rule set on the WAN interface in correct order.

| Priority | Rule | Action |
|----------|------|--------|
| 1 | Anti-Lockout Rule | ✅ Allow (System Default) |
| 2 | Block bogon networks | 🚫 Block (System Default) |
| 3 | Block Facebook | 🚫 Block (Custom) |
| 4 | Allow All | ✅ Pass (Custom) |

> pfSense reads rules **top to bottom** and stops at the first match. Traffic to Facebook hits Rule 3 and gets blocked before it ever reaches Rule 4.

---

### ✅ Step 12 — Block Facebook Rule Details

![Step 12](screenshots/12-block-facebook-rule.png)

Full configuration of the custom rule that silently drops all traffic to Facebook's IP.

| Setting | Value |
|--------|-------|
| Action | Block |
| Interface | WAN |
| Protocol | Any |
| Source | Any |
| Destination | `102.132.97.35` |
| Description | `Block FaceBook` |

> **Block vs Reject:** Block silently drops packets with no response. Reject sends an error back, potentially revealing the firewall's presence. Block is the more secure choice.

---

### ✅ Step 13 — Gateway Configuration

![Step 13](screenshots/13-gateway-config.png)

Verified the gateway settings that control how pfSense routes traffic to the internet.

| Gateway | Address | Status |
|---------|---------|--------|
| WAN_DHCP | `192.168.1.1` | 🟢 Online |
| WAN_DHCP6 | `fe80::1%em0` | 🟢 Online |
| WANGW | `192.168.1.1` | 🟢 Online |

---

### ✅ Step 14 — Windows Client Network Settings

![Step 14](screenshots/14-windows-network-settings.png)

Changed the Windows PC's network settings so all traffic routes through pfSense instead of directly to the router.

| Setting | Before | After |
|--------|--------|-------|
| Default Gateway | `192.168.1.1` (Router) | `192.168.1.7` (pfSense) |
| Preferred DNS | `192.168.1.1` | `192.168.1.7` |

> ⚠️ **This is the most critical step.** Without pointing the gateway to pfSense, all traffic bypasses the firewall entirely — making every rule completely useless.

---

### ✅ Step 15 — Testing: Facebook Successfully Blocked

![Step 15](screenshots/15-facebook-blocked-test.png)

Attempted to visit `facebook.com` — browser returned a timeout error confirming the block rule works.

\```
ERR_CONNECTION_TIMED_OUT — www.facebook.com took too long to respond
\```

| Site | Expected | Result |
|------|----------|--------|
| `facebook.com` | 🚫 Blocked | ✅ ERR_CONNECTION_TIMED_OUT |
| `google.com` | ✅ Allowed | ✅ Loads successfully |
| `youtube.com` | ✅ Allowed | ✅ Loads successfully |

> The timeout confirms packets are being **silently dropped** — not rejected. The Allow-All rule is correctly positioned below the block rule.

---

## 💡 Key Lessons Learned

**Rule order is everything** — a misplaced Allow-All above a block rule silently breaks your entire security policy with no error message.

**The gateway change activates the firewall** — pfSense does nothing for a client until that client's default gateway points to `192.168.1.7`.

**DNS must go through the firewall** — if DNS queries bypass pfSense, domain-based rules can be circumvented entirely.

**Block vs Reject matters** — Block is stealthier and reveals nothing to attackers. Reject tells them a firewall exists.

**Aliases = scalable management** — one IP change updates every rule at once instead of editing them one by one.

---

## 🛠️ Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| pfSense | 2.8.1-RELEASE | Firewall OS |
| VirtualBox | — | Virtualization |
| Windows 10/11 | — | Client machine |
| Google DNS | 8.8.8.8 | DNS resolution |

---

## 📜 Certification

**Security Blue Team Level 1 (BTL1)**
Domain: Network Security / Firewall Administration
Completed: February 2026

---

*Made by Kareem — BTL1 Cybersecurity Student*
