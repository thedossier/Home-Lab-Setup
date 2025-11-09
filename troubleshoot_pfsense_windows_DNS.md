## 🧩 Fixing Windows VM Internet Connectivity in Proxmox (via pfSense)

### Overview
While setting up my Windows 10 VM in Proxmox and routing traffic through pfSense, the VM could successfully reach pfSense but had **no WAN/internet access**. After some troubleshooting, the root cause turned out to be a **DNS resolution issue** rather than a routing problem.

---

### 🧱 Setup Context
- **Hypervisor:** Proxmox VE  
- **Firewall/Router VM:** pfSense (managing LAN + WAN networks)  
- **Windows VM Network Adapter:** `VirtIO` using `vmbr1` (LAN bridge managed by pfSense)  
- **Goal:** Provide internet access to the Windows VM through pfSense’s NAT

---

### 🕵️‍♂️ Troubleshooting Steps

#### 1. Confirmed Network Reachability
- Verified Windows VM could **ping pfSense LAN IP** (e.g., `192.168.1.1`).
- Verified **gateway and IP assignment** were correct (`ipconfig` showed LAN IP 192.168.1.x, default gateway 192.168.1.1).

#### 2. Tested Internet Reachability
powershell
ping 8.8.8.8
ping google.com
✅ 8.8.8.8 worked
❌ google.com failed

This confirmed DNS resolution was the issue, not connectivity.

⚙️ Fix: DNS Resolver Issue
Option 1 (pfSense fix)

Log into pfSense → Services > DNS Resolver

Ensure it is enabled and set to listen on LAN interface

Apply changes

Option 2 (Windows fix that worked immediately)

Open an elevated Command Prompt

Run:ipconfig /flushdns
ipconfig /renew
Verify DNS is now resolving correctly.

✅ Internet access restored

🧠 Takeaway

Even if pfSense routing and NAT are configured correctly, cached DNS issues or resolver settings can block name resolution. Always test connectivity both by IP address and domain name — it quickly isolates whether the issue is network or DNS-related.
