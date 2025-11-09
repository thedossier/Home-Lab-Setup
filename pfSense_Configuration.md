# 🛡️ pfSense Configuration Guide

## Overview
This document provides a complete walkthrough for installing and configuring **pfSense** within a **Proxmox** environment.  
pfSense serves as the **firewall, router, and gateway** for the home lab — enabling network segmentation, security policy testing, and realistic enterprise routing scenarios.

This setup creates a foundation for future IT Help Desk and SOC simulations by managing both external (WAN) and internal (LAN) traffic through virtual bridges in Proxmox.

---

## 🧩 System Overview

| Component | Role | Bridge | Description |
|------------|------|---------|--------------|
| `vtnet0` | WAN Interface | `vmbr0` | Connects to home router / internet |
| `vtnet1` | LAN Interface | `vmbr1` | Internal lab network for all virtual machines |

---

## 🧱 Step 1 – Create pfSense VM

1. In the **Proxmox Web UI**, click **Create VM**.
2. Configure:
   - **Node:** your Proxmox host
   - **VM ID:** auto or custom (e.g. `100`)
   - **Name:** `pfSense`
3. **OS Tab:**
   - Use ISO image: select `pfSense.iso`
   - Guest OS: *Other*
4. **System Tab:**
   - BIOS: *OVMF (UEFI)* or *SeaBIOS* (default is fine)
   - Machine: *q35*
   - SCSI Controller: *VirtIO SCSI*
5. **Hard Disk:**
   - Bus/Device: *VirtIO Block*
   - Disk size: *20 GB*
6. **CPU & Memory:**
   - 2 Cores / 4 GB RAM minimum
7. **Network Devices:**
   - **Adapter 1:** `vmbr0` (WAN)
   - **Adapter 2:** `vmbr1` (LAN)
   - Both using *VirtIO (paravirtualized)*

Click **Finish** and do not start yet.

---

## ⚙️ Step 2 – Install pfSense

1. Start the VM and open the **Console** in Proxmox.
2. Choose **Install pfSense (default)**.
3. Accept defaults for keymap and proceed with installation.
4. When prompted for filesystem:
   - Choose **UFS** for simplicity and performance.
   - *(ZFS is optional but unnecessary for this lab.)*
5. Let the installer finish, then **reboot** when prompted.
6. Remove the ISO (in Proxmox → Hardware → CD/DVD → Unmount).

---

## 🧭 Step 3 – Assign Interfaces

Upon reboot, pfSense will prompt:
Should VLANs be set up now? [y/n]
- **Answer:** `n` (no VLANs at this stage)

Next, it will ask to assign interfaces:
Enter the WAN interface name: vtnet0
Enter the LAN interface name: vtnet1

Confirm assignments:
WAN -> vtnet0
LAN -> vtnet1
Press **Enter** to proceed.

---

## 🌐 Step 4 – LAN Configuration

pfSense will assign a default LAN IP of `192.168.1.1/24`.  
However, this often conflicts with home routers (like a Netgear at `192.168.1.1`).  
To prevent overlap, we assign a **custom lab network**.

1. From the pfSense console menu, select **Option 2: Set interface IP address**.
2. Choose the **LAN interface (vtnet1)**.
3. Assign a new IP: 10.10.10.1
4. Subnet mask: 24
Press **Enter** for no upstream gateway.
When prompted to enable DHCP, select: y

- Range: `10.10.10.100` to `10.10.10.200`
-  Decline IPv6 configuration for now.

Your internal lab subnet is now **10.10.10.0/24**.

---

## 🌍 Step 5 – Access the Web Interface

From your **laptop** (connected to the same switch or via Proxmox LAN bridge):

1. Assign your laptop a static IP in the same range (e.g. `10.10.10.20`).
2. Open your browser and go to: 10.10.10.1
3. Log in with:
- **Username:** admin  
- **Password:** pfsense
4. Follow the **pfSense Setup Wizard**:
- Hostname: `pfsense`
- Domain: `lab.local`
- Primary DNS: `1.1.1.1` or `8.8.8.8`
- Timezone: your local
- WAN: set to DHCP
- LAN: verify `10.10.10.1/24`

After applying settings, pfSense will reload and you’ll have full web access.

---

## 🧱 Step 6 – Proxmox Bridge Verification

Return to the **Proxmox web interface** and verify:
- `vmbr0` is tied to your physical NIC (internet-facing)
- `vmbr1` has no ports assigned and is used for internal routing

Example `/etc/network/interfaces` (Proxmox host):
auto lo
iface lo inet loopback

auto eno1
iface eno1 inet manual

auto vmbr0
iface vmbr0 inet static
address 192.168.1.50/24
gateway 192.168.1.1
bridge_ports eno1
bridge_stp off
bridge_fd 0

auto vmbr1
iface vmbr1 inet manual
bridge_ports none
bridge_stp off
bridge_fd 0


---

## 🚦 Step 7 – Testing Connectivity

1. From the pfSense **Diagnostics → Ping** menu, ping:
   - `8.8.8.8` (internet)
   - `10.10.10.1` (LAN interface)
2. From your laptop or another VM connected to `vmbr1`, test connectivity to:
   - pfSense Web UI
   - The internet (to confirm NAT routing works)

If successful, pfSense is now routing traffic between your **LAN** and **WAN**.

---

## 🧰 Step 8 – Next Steps

With pfSense configured:
- Add Windows Server 2019 or 2022 next to create **Active Directory**.
- Connect it to the **LAN bridge (vmbr1)**.
- pfSense will serve DHCP and DNS for the domain environment if desired.

Follow the next documentation:
📘 [`Windows_Server_Active_Directory.md`](./Windows_Server_Active_Directory.md)

---

## ✅ Key Skills Demonstrated

- Virtual network segmentation and isolation
- Firewall and routing fundamentals
- DHCP and IP management
- Secure access and management of virtual infrastructure

---

*Author: The Dossier*  
*Project: [Home-Lab-Setup](https://github.com/thedossier/Home-Lab-Setup)*  







