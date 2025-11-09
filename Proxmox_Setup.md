# ⚙️ Proxmox Setup and Configuration

## Overview
This document provides a complete walkthrough of installing and configuring **Proxmox Virtual Environment (VE)** on a dedicated host.  
The goal is to establish a reliable and modular hypervisor that will serve as the foundation of the home lab — supporting IT Help Desk and Security Operations (SOC) simulations.

This guide covers:
- Preparing installation media
- Installing Proxmox VE
- Configuring networking bridges
- Uploading ISO images
- Setting up storage and system optimizations
- Preparing for virtual machine (VM) deployments

---

## 🖥️ Hardware Used
| Device | Model | Role |
|--------|--------|------|
| Dell OptiPlex 3050 | Intel i5 / 16 GB RAM / 512 GB SSD | Primary Proxmox Host |
| HP EliteDesk 800 G6 | Intel i7 / 32 GB RAM / 1 TB SSD | Secondary node (future expansion) |
| ZimaBlade | ARM-based mini server | NAS and shared lab storage |

---

## 🧩 Step 1 – Preparing Installation Media

### Requirements
- USB flash drive (8 GB minimum)
- Proxmox VE ISO (downloaded from [proxmox.com/downloads](https://www.proxmox.com/en/downloads))
- ISO burning tool (Rufus or Balena Etcher)

### Procedure
1. Download the latest **Proxmox VE ISO**.
2. Use **Rufus** to write the ISO to your USB drive:
   - Boot selection: *Proxmox ISO*
   - Partition scheme: *GPT*
   - Target system: *UEFI*
3. Insert the USB into the Dell OptiPlex and boot to BIOS (`F12` or `Del`).
4. Ensure **UEFI boot** is enabled and select the USB device.

---

## ⚡ Step 2 – Installing Proxmox VE

1. Choose **“Install Proxmox VE (Graphical)”** at the boot menu.
2. Accept the license agreement.
3. Select your primary disk (SSD) as the installation target.
4. When prompted for a **filesystem**, choose:
   - **ext4** → Recommended for simplicity and stability
   - *(ZFS is optional for advanced setups or redundancy)*
5. Configure:
   - **Country / Timezone / Keyboard layout**
   - **Root password** (secure)
   - **Email address** (for alerts)
6. Network configuration:
   - Hostname: `proxmox.local` (or similar)
   - Management interface: your Ethernet NIC
   - IP address: static (example: `192.168.1.50`)
   - Gateway: your router (e.g., `192.168.1.1`)

Once installation completes, remove the USB and reboot.

---

## 🌐 Step 3 – Accessing the Web Interface

On a separate device (e.g., your laptop):

1. Open a browser and go to: IP ADDRESS CREATED
2. Log in with:
- **Username:** root
- **Password:** the one set during installation
3. You’ll now see the **Proxmox web dashboard**, where all VM and network management is done.

---

## 🔧 Step 4 – Creating Network Bridges

Proxmox uses **Linux bridges** to connect virtual machines to physical and virtual networks.

### In this lab:
| Bridge | Role | Description |
|---------|------|-------------|
| `vmbr0` | LAN Bridge | Connects to the physical network (via host NIC) |
| `vmbr1` | Internal Bridge | Isolated lab traffic managed by pfSense |

### Configuration Steps
1. Go to **Datacenter → Node → Network**.
2. Edit the existing bridge (`vmbr0`) to confirm it’s attached to your NIC.
3. Create a new bridge:
- Name: `vmbr1`
- Leave bridge port empty
- Check **“Autostart”**
- Apply changes and reboot

After reboot, both bridges (`vmbr0` and `vmbr1`) will be active:
- `vmbr0` → LAN/Internet connection  
- `vmbr1` → Internal pfSense-managed network

---

## 📦 Step 5 – Uploading ISO Images

To create new VMs, ISO files for operating systems are required.

### Recommended ISOs
| OS | Purpose |
|----|----------|
| pfSense | Firewall and routing |
| Windows Server 2019 / 2022 | Active Directory and DNS |
| Windows 10 / 11 | Help Desk client |
| Ubuntu Server | SOC and monitoring stack |
| Kali Linux | Red Team and analysis tools |

### Upload Procedure
1. In the Proxmox web UI, go to **Datacenter → Node → Local (Storage)**.
2. Select **“ISO Images” → Upload**.
3. Upload each `.iso` file from your local system.

---

## 🧱 Step 6 – Storage Configuration

If you plan to add multiple drives or a NAS:
- Add the device under **Datacenter → Storage → Add → Directory or NFS.**
- Mount ZimaBlade (NAS) via NFS or SMB for backups and shared images.
- Label storage clearly, e.g.:
- `local` → OS and VM disks
- `backup` → Snapshots / ISO / templates

---

## 🧠 Step 7 – Preparing for pfSense and Active Directory

Before installing pfSense:
- Confirm both bridges (`vmbr0` and `vmbr1`) exist.
- Verify that the host has outbound internet access.
- pfSense will act as a **virtual router**, so all other VMs (e.g., Windows, Ubuntu) will connect through it using `vmbr1`.

---

## 🧩 Next Steps
After completing this setup:
1. Proceed to [`pfSense_Configuration.md`](./pfSense_Configuration.md) to build the network backbone.
2. Once networking is operational, install Windows Server for Active Directory integration.

---

## 🧰 Key Skills Demonstrated
- Hypervisor installation and configuration
- Virtual networking design
- Storage and ISO management
- Foundation for enterprise-grade lab environments

---

*Author: The Dossier*  
*Project: [Home-Lab-Setup](https://github.com/thedossier/Home-Lab-Setup)*  

