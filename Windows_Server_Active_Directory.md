# 🧱 Windows Server Setup and Active Directory Deployment

## Overview
This document provides a detailed walkthrough for deploying **Windows Server 2019/2022** inside **Proxmox VE**, resolving common installation issues (like the *no drive detected* error), and configuring it as an **Active Directory Domain Controller** for the home lab environment.

This is a critical component in simulating real enterprise networks, allowing integration between IT Help Desk, endpoint management, and Security Operations workflows.

---

## 🧩 System Overview

| Component | Role | Network | Description |
|------------|------|----------|--------------|
| `pfSense` | Firewall / Router | `vmbr0` (WAN), `vmbr1` (LAN) | Routes traffic and isolates the lab |
| `Windows Server` | Domain Controller (AD, DNS, DHCP optional) | `vmbr1` | Manages authentication and directory services |
| `Windows 10` | Endpoint / User Machine | `vmbr1` | Joins to domain and simulates user environment |

---

## 🧱 Step 1 – Create Windows Server VM in Proxmox

1. In the **Proxmox Web UI**, click **Create VM**.
2. **General Tab:**
   - Node: your Proxmox host
   - Name: `Windows-Server`
3. **OS Tab:**
   - ISO Image: `Windows Server 2019` or `Windows Server 2022`
   - Guest OS: `Microsoft Windows`
4. **System Tab:**
   - BIOS: *OVMF (UEFI)*  
   - Machine: *q35*  
   - Display: *Default*  
   - Tick **QEMU Guest Agent**
5. **Hard Disk:**
   - Bus/Device: *VirtIO SCSI (single)*
   - Disk Size: *60 GB*
6. **CPU & Memory:**
   - 2–4 Cores / 8 GB RAM
7. **Network Device:**
   - Bridge: `vmbr1` (connects to internal lab)
   - Model: *VirtIO (paravirtualized)*

✅ **Before starting**, attach a **second CD-ROM** to the VM and mount the **VirtIO driver ISO**:
   - In Proxmox: **Hardware → Add → CD/DVD Drive → Storage:** `local` → ISO Image: `virtio-win.iso`

---

## ⚙️ Step 2 – Windows Installation (Fixing “No Drive Detected” Error)

When you first boot into the Windows Server installer, you’ll likely encounter:
> “We couldn’t find any drives. To get a storage driver, click Load driver.”

### Solution:
1. Click **Load driver**.
2. Navigate to the VirtIO CD:
E:\viostor\w10\amd64
3. Select the driver and click **Next**.
4. The drive should now appear — continue installation as normal.

🧠 **Pro tip:**  
This issue occurs because Windows lacks the VirtIO driver needed for Proxmox’s virtual storage controller.

---

## 🔐 Step 3 – Initial Configuration

After installation completes:

1. Log in as **Administrator**.
2. Set a secure password.
3. Open **Server Manager** → **Local Server**:
- Rename the computer: `LAB-DC01`
- Disable IE Enhanced Security Configuration (optional for lab)
- Assign a static IP address matching the pfSense LAN subnet:
  - IP: `10.10.10.2`
  - Subnet: `255.255.255.0`
  - Gateway: `10.10.10.1`
  - DNS: `10.10.10.2` (self-reference)

Reboot when prompted.

---

## 🧭 Step 4 – Install Active Directory Domain Services (AD DS)

1. In **Server Manager**, click **Add Roles and Features**.
2. Select **Role-based installation**.
3. Under **Server Roles**, check:
- `Active Directory Domain Services`
- `DNS Server`
4. Proceed through the wizard and **Install**.

When complete, click the **flag icon** in Server Manager → **Promote this server to a domain controller**.

---

## 🏛️ Step 5 – Promote to Domain Controller

1. Select **Add a new forest**.
2. Enter: lab.local
3. Set a **Directory Services Restore Mode (DSRM)** password.
4. Continue through defaults.
5. Reboot when prompted.

---

## 🧩 Step 6 – Verify Domain Services

After reboot:

1. Log in as: lab\Administrator
2. Open **Active Directory Users and Computers (ADUC)**.
3. Verify domain: lab.local
4. You can now create organizational units (OUs), user accounts, and groups.

Example structure:
OU=IT
OU=Users
OU=Workstations
OU=Servers


---

## 💻 Step 7 – Add Windows 10 Endpoint to Domain

1. Deploy a new **Windows 10 VM** in Proxmox:
   - Attach to `vmbr1`
   - Assign static IP (e.g. `10.10.10.20`)
   - Gateway: `10.10.10.1`
   - DNS: `10.10.10.2` (DC IP)
2. Log in → open **System Properties → Domain Join**.
3. Enter: lab.local
4. Provide credentials: lab\Administrator
5. Reboot when prompted.

You’ve now successfully created a **domain environment** with:
- pfSense routing
- Windows Server as AD + DNS
- Windows 10 as a joined workstation

---

## 🧠 Step 8 – Useful Administrative Commands

| Task | Command |
|------|----------|
| List all domain computers | `Get-ADComputer -Filter *` |
| List all domain users | `Get-ADUser -Filter *` |
| Verify replication | `repadmin /replsummary` |
| Test DNS resolution | `nslookup lab.local` |
| Force Group Policy Update | `gpupdate /force` |

---

## 🧰 Step 9 – Integrate with SOC or Help Desk Scenarios

With AD configured:
- Deploy **Sysmon** on endpoints and forward logs to a future **SIEM VM**.
- Create **help desk tickets** for account resets or GPO issues.
- Test **Group Policy** enforcement, logon scripts, and user permissions.

This blends **IT support fundamentals** with **security operations** workflows.

---

## ✅ Skills Demonstrated

- Virtual machine provisioning in enterprise-style topology  
- Troubleshooting VirtIO driver installation  
- Domain controller and DNS configuration  
- Network IP management and integration with pfSense  
- Windows domain join and authentication setup  
- Foundational SOC and Help Desk practices  

---

*Author: The Dossier*  
*Project: [Home-Lab-Setup](https://github.com/thedossier/Home-Lab-Setup)*  

