# 🧠 Home Lab 2.0

![Status](https://img.shields.io/badge/project-active-brightgreen) ![Lab Type](https://img.shields.io/badge/type-Home--lab-white) ![License](https://img.shields.io/badge/license-MIT-lightgrey) ![Platform](https://img.shields.io/badge/platform-Proxmox-orange) ![Focus](https://img.shields.io/badge/focus-Blue%20Team%20%2B%20OffSec-blue)


## 🔍 Overview
**Home Lab 2.0** is a personal enterprise-grade lab built for studying **networking, cybersecurity, virtualization, and system administration**.  
It combines **Virgin Media + MikroTik + Cisco + Proxmox** with VLAN segmentation, centralized authentication, and observability through **Wazuh**.

The goal is to replicate a **corporate infrastructure** at home — fully documented, automated, and safe for testing firewall rules, SIEM, IDS/IPS, and DevOps concepts.

---

## 🎯 Objectives
- Build a modular, secure, and scalable homelab for **networking + cybersecurity** studies.  
- Implement **pfSense**, **Windows Server 2022 (AD DS)**, **File Server**, **Wazuh**, **OpenVAS**, **Uptime Kuma**, and a **Web Server (Nginx + PHP + Java)**.  
- Centralize authentication, monitoring, and incident response using **Wazuh SIEM/EDR**.  
- Automate deployments through **GPOs** and scripting.  
- Replace MikroTik’s internal Wi-Fi with a **dedicated Ubiquiti AP** connected to the Cisco switch.  
- Maintain **external backups** for redundancy and disaster recovery.  

---

## 🧩 Network Topology

### Physical Infrastructure
- **Virgin Media Fibre Hub** → Internet uplink (no bridge mode).  
- **MikroTik hAP ax³ (RouterOS 7.16.x)** → Routing, VLAN trunking.  
  - `ether1` (2.5 GbE): Main desktop.  
  - `ether2`: WAN ← Virgin Hub.  
  - `ether3`: 802.1Q trunk → Cisco switch.  
  - **Wi-Fi disabled** (Ubiquiti AP used instead).  
- **Cisco Catalyst 2960G-24TC-L** → Managed switch with VLAN trunks for servers and AP.  
- **Beelink SER5 (Ryzen 5 5500U, 32 GB RAM, 500 GB NVMe)** → **Proxmox VE** hypervisor.  
- **Ubiquiti UniFi AP** → Connected to Cisco switch (trunk port, VLAN 40 – Wi-Fi/guest).  

---

### Logical VLAN Design
| VLAN | Network | Purpose | Devices |
|------|----------|----------|----------|
| 10 | 10.10.10.0/24 | Infrastructure / Admin | MikroTik, Cisco, Desktop |
| 20 | 192.168.20.0/24 | Servers / Backend | pfSense, Wazuh, AD, FileSrv, WebSrv |
| 30 | 172.16.30.0/24 | Pentest / Lab | OpenVAS, test machines |
| 40 | 192.168.40.0/24 | Wi-Fi / IoT / Guest | Ubiquiti AP, IoT devices |
| 50 | 192.168.50.0/24 | Workstations / Users | PCs, laptops, VMs |

> **Note:** The **Cisco switch** handles VLAN trunking for all connected devices (Proxmox, AP, Desktop).  
> The **MikroTik** routes between VLANs and acts as the Internet gateway.

---

### 🧭 Network Diagram
```
[Virgin Modem]
     |
     v
[MikroTik hAP ax3]───(2.5G)───[Desktop]
     |
   trunk
     |
[Cisco 2960G Switch]──[Ubiquiti AP]──(Wi-Fi VLAN40)
     |
[Beelink/Proxmox]──[VMs: pfSense, Wazuh, Win22, FileSrv, WebSrv, OpenVAS, Uptime]
```

---

## ⚙️ Virtual Machines & Services

| VM | Role | RAM | vCPU | Notes |
|----|------|------|------|-------|
| **pfSense** | Firewall, NAT, IDS (Suricata) | 2 GB | 2 | Gateway between VLANs |
| **Windows Server 2022** | AD DS, DNS, GPO | 4–6 GB | 2 | Automates Wazuh agent install via GPO |
| **File Server (Debian)** | Samba + External Backup | 4 GB | 2 | External drive mounted for rsync |
| **Web Server (Debian)** | Nginx + PHP-FPM + Tomcat | 3 GB | 2 | Lightweight stack for web apps |
| **Wazuh** | SIEM / EDR | 6–8 GB | 4 | Full stack (Manager + OpenSearch + Dashboard) |
| **OpenVAS** | Vulnerability Scanner | 4 GB | 2 | Isolated in VLAN 30 |
| **Uptime Kuma** | Monitoring | 1 GB | 1 | Tracks uptime for critical services |
| **Proxmox Host** | Hypervisor overhead | 2 GB | — | Reserved for base system |

🧮 **Total usage:** ~30 GB — leaving ~2 GB buffer → still stable and safe.

---

## 🌐 Web Server Configuration

**Stack Overview**
- **OS:** Debian / Ubuntu  
- **Web stack:** Nginx + PHP-FPM + Tomcat (for Java webapps)  
- **Optional DB:** MariaDB or PostgreSQL (on same or separate VM)  
- **Security:** Wazuh agent + ModSecurity (OWASP CRS) + fail2ban (optional)

**Example Reverse Proxy (Nginx)**
```nginx
# PHP app
server {
    listen 80;
    server_name lab.local;

    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
    }

    access_log /var/log/nginx/php_access.log;
    error_log /var/log/nginx/php_error.log;
}

# Java app
server {
    listen 8080;
    server_name java.lab.local;

    location / {
        proxy_pass http://127.0.0.1:8081; # Tomcat or Jetty internal port
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    access_log /var/log/nginx/java_access.log;
    error_log /var/log/nginx/java_error.log;
}
```

**Tomcat memory optimization (`/etc/default/tomcat9`):**
```bash
JAVA_OPTS="-Xms512m -Xmx1024m -XX:+UseG1GC"
```

**PHP-FPM pool tuning (`/etc/php/8.2/fpm/pool.d/www.conf`):**
```ini
pm = ondemand
pm.max_children = 8
pm.process_idle_timeout = 10s
memory_limit = 128M
```

---

## 🧠 Wazuh Agent Deployment via GPO
The Wazuh agent installation on Windows domain-joined machines is automated via **GPO**:

```powershell
$AgentPath = "\\srv-files\software\wazuh-agent.msi"
$ManagerIP = "192.168.20.10"

if (-not (Get-Service -Name "Wazuh" -ErrorAction SilentlyContinue)) {
    Start-Process "msiexec.exe" -ArgumentList "/i `"$AgentPath`" /qn WAZUH_MANAGER=$ManagerIP WAZUH_REGISTRATION_SERVER=$ManagerIP WAZUH_AGENT_GROUP=default"
}
```

📁 **GPO path:**  
`Computer Configuration > Policies > Windows Settings > Scripts (Startup)`

✅ Every workstation automatically installs and registers the Wazuh agent at logon.

---

## 💽 External Backup Strategy
Automated **rsync** job from the File Server to an external USB drive:

```bash
rsync -avh --delete /srv/samba/shares/ /mnt/backup/samba_$(date +%F)/
```

**Retention policy:**
- 7 daily backups  
- 4 weekly  
- 6 monthly  

Snapshots handled in **Proxmox** before upgrades.  
Optional upgrade: **Proxmox Backup Server (PBS)** for deduplication and long-term retention.

---

## 🧱 Hardware Overview

| Device | Model | Function |
|---------|--------|----------|
| Router | MikroTik hAP ax³ | Routing, VLANs, firewall |
| Switch | Cisco Catalyst 2960G-24TC-L | VLAN trunking, access |
| Hypervisor | Beelink SER5 (32 GB) | Proxmox VE host |
| Desktop | Ryzen 7 (2.5 GbE) | Admin / client |
| AP | Ubiquiti UniFi | Wi-Fi via Cisco trunk (VLAN 40) |
| ISP Modem | Virgin Fibre Hub | Internet gateway |

---

## 🔐 Security & Isolation
- VLAN30 isolated (pentest zone).  
- VLAN40 isolated via Cisco tagging + pfSense rules.  
- IDS/IPS (Suricata) on pfSense.  
- Wazuh agents on all hosts (including WebSrv).  
- Centralized authentication (AD DS).  
- External backups + snapshots.  
- Web Server hardened (ModSecurity + PHP-FPM restrictions).  
- Controlled scanning via OpenVAS.  

---

## ✅ Next Steps
- [ ] Disable MikroTik Wi-Fi, connect **Ubiquiti AP** to Cisco trunk (VLAN40).  
- [ ] Deploy **Web Server VM** and configure Nginx + PHP + Tomcat stack.  
- [ ] Integrate **File Server** with **AD DS** (Samba + GPO mappings).  
- [ ] Finish **Wazuh Agent GPO** rollout.  
- [ ] Automate **external backups** with rotation.  
- [ ] Finalize **Wazuh / OpenVAS / Uptime Kuma / WebSrv** setup.  
- [ ] Document everything in **Wiki.js** and track configs in **Gitea**.  

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
   - pfSense  
   - Windows Server 2022  
   - File Server  
   - Web Server  
   - Wazuh  
   - OpenVAS  
   - Uptime Kuma  
3. Configure VLANs on MikroTik & Cisco.  
4. Promote Win2022 to **AD DS**.  
5. Deploy **Wazuh agents**.  
6. Configure **Web Server (PHP + Tomcat)**.  
7. Set up external backups & monitor services via Uptime Kuma.  

---

**Author:** Leandro Stolfo Uehara  
**Project:** Homelab 2.0 — built for learning, testing, and breaking things safely 🔧💻🧩
