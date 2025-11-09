# pfSense Installation and Configuration on Proxmox

## Overview

This section documents the setup and configuration of **pfSense** running as a virtual machine within the Proxmox hypervisor. pfSense acts as the core firewall, router, and network segment controller for the home lab.  

The goal of this setup was to create a **secure and isolated virtualized environment** that simulates an enterprise network with internet routing, VLAN control, and segmentation for blue/red team lab work.

---

## Prerequisites

- **Proxmox installed and accessible**
- **At least one physical NIC** on the Proxmox host connected to a managed switch
- **Two virtual bridges** configured in Proxmox:
  - `vmbr0` – WAN bridge (external network / connected to router or ISP)
  - `vmbr1` – LAN bridge (internal network / connected to lab systems)
- pfSense ISO image uploaded to Proxmox local storage
- Sufficient system resources (1–2 vCPU, 2GB RAM minimum, 10GB storage)

---

## Step 1: Create the pfSense VM

1. In the **Proxmox Web UI**, click **Create VM**.
2. Assign a **VM ID** and **name** (e.g., `pfsense`).
3. Under **OS**, choose **Use CD/DVD disk image file (ISO)** and select your pfSense ISO.
4. For **System**:
   - BIOS: `SeaBIOS`
   - Machine: `i440fx`
   - SCSI Controller: `VirtIO SCSI`
5. For **Hard Disk**:
   - Bus/Device: `SCSI`
   - Disk Size: 10–20 GB
   - Storage: Local-LVM
6. For **CPU**: Assign 1–2 cores.
7. For **Memory**: Assign 2 GB (minimum).
8. For **Network**:
   - Add two adapters:
     - `vtnet0` → `vmbr0` (WAN)
     - `vtnet1` → `vmbr1` (LAN)

Click **Finish** to create the VM.

---

## Step 2: Install pfSense

1. Start the VM and open the **console**.
2. Follow the installation prompts:
   - Accept defaults for keymap.
   - Select **Install pfSense**.
   - Choose **ZFS** as the filesystem (recommended for stability and snapshot features).
   - Proceed with the automatic partitioning and installation.
3. Once installation completes, **reboot the VM**.

---

## Step 3: Assign Network Interfaces

Upon reboot, pfSense will prompt to assign network interfaces.

- It will detect:
  - `vtnet0` → WAN
  - `vtnet1` → LAN
- Assign them accordingly:
WAN = vtnet0
LAN = vtnet1

- You can skip VLAN configuration for now by answering **No** when prompted.

Once configured, pfSense will complete initialization and display:
WAN -> vtnet0 (IP via DHCP or your ISP/router)
LAN -> vtnet1 (Default 192.168.1.1/24)


---

## Step 4: Access the Web Interface

From any computer connected to the LAN bridge (`vmbr1` network):

1. Open a browser and navigate to:
IP ADDRESS REDACTED

2. The default credentials are:
Username: admin
Password: pfsense

3. Follow the setup wizard:
- Set hostname and domain
- Configure WAN (DHCP, static IP, or PPPoE)
- Configure LAN (default 192.168.1.1/24 or customize)
- Set DNS and time servers
- Change the admin password

---

## Step 5: Testing Connectivity

1. From the pfSense **Dashboard**, go to **Diagnostics → Ping**.
2. Try to ping `8.8.8.8` (Google DNS).
- If successful, WAN is working.
3. Next, connect a test VM (like Windows or Kali) to the LAN (`vmbr1`).
4. Check that it receives a DHCP IP (default range `192.168.1.100–200`).
5. Try to access the internet from that VM — you should have full connectivity.

---

## Step 6: Enable Management and Monitoring

To make pfSense more functional for the lab:

- **Enable SSH access** under `System → Advanced → Admin Access`.
- **Install pfBlockerNG** (optional) for DNS filtering and threat lists.
- **Enable NTP service** for time synchronization.
- **Take a snapshot** in Proxmox once pfSense is configured and stable.

---

## Troubleshooting Tips

- If pfSense shows no WAN IP:
- Check that your Proxmox host is connected to your main router or modem.
- Verify that `vmbr0` is bridged to the correct physical NIC.
- If LAN VMs have no internet:
- Ensure they are connected to the same virtual bridge as pfSense LAN (`vmbr1`).
- Confirm DHCP is enabled in pfSense under **Services → DHCP Server**.

---

## Summary

pfSense now serves as the **central router and firewall** for the lab.  
All Proxmox VMs using the LAN bridge are protected and routed through pfSense.  
This structure mimics a real-world enterprise network, where pfSense controls access, logging, and segmentation between internal systems and the external internet.

**Next Step:** Proceed to setting up your **Windows Server or Windows Workstation VM** on the LAN side for Active Directory, SOC monitoring, or other tools.
