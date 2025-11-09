# Windows Virtual Machine Installation and VirtIO Driver Fix on Proxmox

## Overview

This documentation explains how to install **Windows** as a virtual machine on Proxmox, connect it to the lab network via pfSense, and resolve a common installation issue: **the “No Drives Found” error** caused by missing VirtIO storage drivers.

This process replicates a realistic enterprise environment where Windows servers or workstations operate within a virtualized, network-segmented lab for monitoring, SOC, or red team development.

---

## Objectives

- Create a Windows virtual machine in Proxmox
- Properly connect it to pfSense’s LAN network
- Install required VirtIO drivers for disk and network detection
- Document a troubleshooting fix for the “no drives found” issue

---

## Prerequisites

- **Proxmox host** already running with `vmbr1` connected to pfSense LAN  
- **pfSense VM** installed and functioning
- **Windows ISO image** (Windows 10, 11, or Server)
- **VirtIO drivers ISO** downloaded from the [Fedora VirtIO repository](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/)
- Sufficient VM resources (2–4 GB RAM, 2 vCPU minimum, 32 GB+ disk)

---

## Step 1: Upload the ISOs to Proxmox

1. Navigate to **Proxmox Web UI → Datacenter → Storage → local (or local-lvm)**.
2. Click **Upload** and select both:
   - The Windows ISO file (e.g., `Win11.iso`)
   - The VirtIO ISO file (e.g., `virtio-win-0.1.xx.iso`)

---

## Step 2: Create the Windows VM

1. In the Proxmox sidebar, click **Create VM**.
2. Configure the following:

   **General:**
   - Name: `Windows-VM`
   - Start at boot: Enabled

   **OS:**
   - Use CD/DVD: Select your Windows ISO

   **System:**
   - BIOS: `OVMF (UEFI)`
   - Machine: `q35`
   - Tick “Add TPM” if using Windows 11

   **Hard Disk:**
   - Bus/Device: `VirtIO SCSI`
   - Storage: `local-lvm`
   - Disk Size: `64 GB`

   **CPU:**
   - Cores: `2`

   **Memory:**
   - RAM: `4096 MB`
   - Ballooning Device: Enabled

   **Network:**
   - Bridge: `vmbr1` (LAN network connected to pfSense)
   - Model: `VirtIO (paravirtualized)`

3. Complete the wizard and **create the VM**.

---

## Step 3: Mount the VirtIO Driver ISO

Before installation begins:

1. Select the new Windows VM in Proxmox.
2. Go to **Hardware → Add → CD/DVD Drive**.
3. Choose **Use ISO Image** and select the `virtio-win.iso` file.
4. Set **Device Type** to `IDE` or `SATA` (not SCSI).
5. Click **Add**.

Now, the VM has two virtual drives:
- Windows installer ISO
- VirtIO driver ISO

---

## Step 4: Start the VM and Begin Installation

1. Start the VM and open the **Console**.
2. The Windows Setup wizard will appear.
3. Proceed through language and keyboard setup until reaching the **“Where do you want to install Windows?”** screen.

---

## Step 5: Fix the “No Drives Found” Error

If you see the message:
We couldn’t find any drives. To get a storage driver, click Load Driver.


This means the **VirtIO storage controller** is not recognized by Windows yet.

### To fix this:

1. Click **Load driver** → **Browse**.
2. Navigate to:
virtio-win → vioscsi → [Windows version] → amd64
3. Select the driver and click **Next**.
4. Windows Setup will load the driver and detect the virtual disk.
5. Select the newly visible drive and click **Next** to continue installation.

---

## Step 6: Network Driver Installation

Once Windows finishes installing and boots for the first time:

1. Log in and open **Device Manager**.
2. You’ll notice a **network adapter with a yellow exclamation mark**.
3. Insert the VirtIO ISO again if needed, then open: virtio-win → NetKVM → [Windows version] → amd64
4. Right-click the device → **Update driver** → **Browse my computer** → select the above folder.
5. The VirtIO network driver will install, enabling internet access.

---

## Step 7: Verify Connectivity and Integration

- Open Command Prompt and run:
ipconfig
ping 192.168.1.1
ping 8.8.8.8

- Confirm that:
- The Windows VM has an IP in the pfSense LAN range.
- It can reach pfSense (gateway) and external internet (if routing is enabled).

---

## Step 8: Snapshot and Backup

Once the system is running and updated:
1. In Proxmox, click the VM → **Snapshots → Take Snapshot**.
2. Name it something like “Base Windows Install”.
3. This ensures you have a clean, ready-to-clone image for future experiments.

---

## Troubleshooting Recap

### Issue:
Windows setup didn’t detect the virtual hard disk.

### Cause:
VirtIO storage drivers not natively included in the Windows installation ISO.

### Solution:
Manually load the appropriate VirtIO drivers during setup from the `virtio-win.iso` file.

### Result:
Windows successfully recognized the disk and completed installation.

---

## Summary

This guide demonstrated how to:
- Set up a Windows virtual machine in Proxmox
- Attach and load VirtIO drivers for performance and compatibility
- Integrate the VM into a pfSense-managed virtual LAN
- Resolve one of the most common Proxmox + Windows installation issues

---

## Supporting Documentation
See the dedicated troubleshooting report:
[`Windows_Install_no_drive_error.md`](./Windows_Install_no_drive_error.md)

---

## Next Step
You can now move on to creating **Active Directory**, **Wazuh**, or **ELK Stack** VMs for SOC simulation, or add Kali Linux for penetration testing workflows.
