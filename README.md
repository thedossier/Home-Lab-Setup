# 🧠 Home Lab Setup – IT Help Desk + SOC Environment

## Overview
This repository documents the full process of designing and building a professional-grade IT home lab.  
The lab was designed to simulate **enterprise-level IT and Security Operations** using free and open-source tools, built on real hardware to mimic production environments.

The goal of this project is twofold:
1. Build hands-on experience managing infrastructure for **Help Desk and IT Operations**.
2. Develop foundational skills for **Security Operations Center (SOC)** workflows.

This documentation includes every step of the build — from the Proxmox hypervisor installation to pfSense networking, Active Directory configuration, and virtual machine provisioning. Each section is written to help others replicate the process or understand the architecture for professional evaluation.

---

## 🧩 Lab Architecture

### Physical Hardware
| Device | Role | Notes |
|--------|------|-------|
| Dell OptiPlex 3050 | Proxmox Host | Primary hypervisor running all virtual machines |
| HP EliteDesk 800 G6 | Secondary Host / Future SOC node | To be integrated for expanded services |
| ZimaBlade | NAS and auxiliary node | Used for shared storage and backups |
| Nighthawk Tri-Band WiFi 6E Router | Core Network Device | Connects all hardware and manages VLANs |

### Virtual Infrastructure
| VM | Role | Description |
|----|------|-------------|
| pfSense | Firewall / Router | Manages internal segmentation and routing |
| Windows Server | Active Directory + DNS | Central authentication and domain management |
| Windows 10 | Help Desk Workstation | Simulated user environment for troubleshooting |
| Ubuntu Server | SOC Tools | Hosts monitoring and security analysis platforms |

---

## ⚙️ Key Features
- **Proxmox Virtualization:** Enterprise-grade hypervisor managing multiple networks and environments.
- **pfSense Firewall:** Provides realistic routing, VLAN control, and security segmentation.
- **Active Directory Integration:** Windows domain with centralized identity management.
- **SOC Tools Environment:** Linux-based monitoring stack for blue-team workflows.
- **Bridged + Internal Networking:** Dual-bridge design separating lab traffic from home LAN.

---

## 🧱 Project Objectives
- Recreate a small-scale **enterprise IT environment** using consumer-grade hardware.
- Demonstrate **network segmentation, authentication, and system administration** skills.
- Develop **incident response and security monitoring** experience.
- Document the process for future learners and professional review.

---

## 📂 Repository Structure
| File | Description |
|------|--------------|
| `Proxmox_Setup.md` | Installing and configuring Proxmox VE |
| `pfSense_Configuration.md` | Firewall and routing setup |
| `Windows_Install_no_drive_error.md` | Troubleshooting VirtIO driver issue |
| `Linux_VM_Configuration.md` | Setting up Linux VMs for SOC tools |
| `Networking_and_ISO_Management.md` | ISO management and bridge networking |
| `Troubleshooting_Guide.md` | Common issues and resolutions |
| `Future_Improvements.md` | Plans for scaling and professional expansion |

---

## 💡 Professional Value
This lab environment showcases technical competencies across:
- Virtualization (Proxmox VE)
- Network engineering (pfSense, VLANs, NAT)
- Windows and Linux system administration
- Troubleshooting and documentation
- Security operations readiness

It demonstrates both the **hands-on capability** to build complex systems and the **communication skills** to document them clearly for stakeholders and recruiters.

---

## 📜 License
This project and documentation are released under the MIT License for educational and professional use.

---

*Author: The Dossier*  
*Repository: [Home-Lab-Setup](https://github.com/thedossier/Home-Lab-Setup)*  
