# 🧠 Home Lab 2.0

![Status](https://img.shields.io/badge/project-active-brightgreen) ![Lab Type](https://img.shields.io/badge/type-Home--lab-white) ![License](https://img.shields.io/badge/license-MIT-lightgrey) ![Platform](https://img.shields.io/badge/platform-Proxmox-orange) ![Focus](https://img.shields.io/badge/focus-Blue%20Team%20%2B%20OffSec-blue)

  
**Location:** Dublin, Ireland  
**Home Lab 2.0** is a personal enterprise-grade lab built for studying **networking, cybersecurity, virtualization, and system administration**.  
It combines **Virgin Media + MikroTik + Cisco + Proxmox** with VLAN segmentation, centralized authentication, and observability through **Wazuh**.

The goal is to replicate a **corporate infrastructure** at home — fully documented, automated, and safe for testing firewall rules, SIEM, IDS/IPS, and DevOps concepts.


---

## 🔍 Overview
Home Lab 2.0 is an enterprise-grade personal lab focused on VLANs, routing, centralized authentication, and full observability.  
It combines:

- **Virgin Media + MikroTik + Cisco** → routing, VLAN trunking, switching  
- **Proxmox VE** → hypervisor for virtualization  
- **Windows Server 2022 (KRANG)** → AD DS, DNS, DHCP  
- **Ubuntu Server (DATABANK)** → Samba/Winbind integration with AD  
- **Web Server (Debian)** → Nginx + PHP-FPM + Tomcat ( Lightweight stack for web apps )
- **Wazuh** → SIEM / EDR, Full stack (Manager + OpenSearch + Dashboard) 
- **OpenVAS** → Vulnerability Scanner (Isolated in VLAN 30) 
- **Uptime Kuma** → Monitoring ( Tracks uptime for critical services )
- **Proxmox Host** → Hypervisor overhead ( Reserved for base system )

---

## 🧭 Network Topology (V3 — Nov 2025)

```
[Virgin Media Hub]
   |
[MikroTik hAP ax³]
   VLAN10: 172.16.10.1/24  (Infra)
   VLAN20: 172.16.20.1/24  (Servers)
   VLAN30: 192.168.30.1/24 (Clients)
   DHCP Relay → 172.16.20.2 (KRANG)
   |
   +-- trunk (10,20,30,40,50)
       |
       |
[Cisco Catalyst 2960G]
   Gi0/1 trunk ↔ MikroTik
   Gi0/3 trunk ↔ Proxmox
   Gi0/2 access VLAN30 (Desktop)
       |
[Proxmox VE Host]
   vmbr10 (172.16.10.10/24)
   vmbr0 (trunk → enp1s0)
   |
   ├── KRANG (WinServer22) – AD/DNS/DHCP
   ├── WAZUH (Debian)
   └── DATABANK (Ubuntu) – Samba + Winbind
```
---

## 🔹 VLAN Table

| VLAN | Name | Subnet | Gateway | Purpose |
|------|------|---------|----------|----------|
| 10 | Infra | 172.16.10.0/24 | 172.16.10.1 | Management, Proxmox |
| 20 | Servers | 172.16.20.0/24 | 172.16.20.1 | AD, FileSrv, SIEM |
| 30 | Clients | 192.168.30.0/24 | 192.168.30.1 | Workstations |
| 40 | Wi-Fi | 10.10.40.0/24 | 10.10.40.1 | Ubiquiti AP |
| 50 | Mgmt | 10.10.50.0/24 | 10.10.50.1 | Reserved |

---

## 🖥️ Virtual Machines

| VM | Function | VLAN/IP | CPU | RAM | Status |
|----|-----------|---------|-----|------|--------|
| KRANG | Windows Server 2022 (AD DS, DNS, DHCP) | VLAN20 / 172.16.20.2 | 2 | 6GB | ✅ Running |
| DATABANK | Ubuntu File Server (Samba/Winbind) | VLAN20 / 172.16.20.3 | 2 | 4GB | ✅ Joined to AD |
| Wazuh | SIEM/EDR | VLAN20 / 172.16.20.4 | 4 | 8GB | ✅ Running |
| Uptime Kuma | Monitoring | VLAN10 / 172.16.10.20 | 1 | 1GB | 🔜 Configuring |
| Grafana+Prometheus | Observability | VLAN20 / 172.16.20.10 | 2 | 3GB | 🔜 Planned |
| Guacamole | Remote Access Portal | VLAN10 / 172.16.10.40 | 2 | 2GB | 🔜 Planned |
| OpenVAS | Vulnerability Scanner | VLAN30 / 192.168.30.10 | 2 | 4GB | 🔜 Planned |

---

## 📚 Documentation Index

- `docs/topology.md`
- `docs/windows-server.md`
- `docs/databank.md`
- `docs/proxmox.md`

---


## 💡 Why This Lab Exists
A fully documented and self-hosted environment to strengthen skills in:
- Networking (VLANs, routing, DHCP)  
- Security (SIEM, IDS/IPS, hardening, incident response)  
- Virtualization (Proxmox, snapshots, containers)  
- System Administration (Linux + Windows)  
- Automation and Troubleshooting  

**Built to learn, break, fix, and repeat — safely.**

---

## 🚀 Getting Started
1. Install **Proxmox VE** on the Beelink SER5.  
2. Create VMs in order:    
   - ~~Windows Server 2022~~
   - ~~File Server~~  
   - Web Server  
   - ~~Wazuh~~  
   - OpenVAS  
   - Uptime Kuma  
3. ~~Configure VLANs on MikroTik & Cisco~~  
4. ~~Promote Win2022 to **AD DS**~~  
5. ~~Deploy **Wazuh agents**~~
6. Configure **Web Server (PHP + Tomcat)**.  
7. Set up external backups & monitor services via Uptime Kuma.  

More coming...
---

**Author:** Leandro Stolfo Uehara  
**Project:** Homelab 2.0 — built for learning, testing, and breaking things safely 🔧💻🧩
