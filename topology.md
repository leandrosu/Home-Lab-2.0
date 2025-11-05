# Network Topology & VLAN Design

## Physical Setup
- Virgin Media Hub → Internet uplink  
- MikroTik hAP ax³ → VLANs + DHCP Relay  
- Cisco Catalyst 2960G → VLAN trunks + access ports  
- Beelink SER5 (Proxmox VE) → Host for all VMs  
- Desktop (Win11) → VLAN30 client  
- Ubiquiti AP → Connected via Cisco trunk (VLAN40)

## Logical VLANs
| VLAN | Name | Subnet | Gateway | Usage |
|------|------|---------|----------|--------|
| 10 | Infra | 172.16.10.0/24 | 172.16.10.1 | Proxmox management |
| 20 | Servers | 172.16.20.0/24 | 172.16.20.1 | AD, FileSrv, SIEM |
| 30 | Clients | 192.168.30.0/24 | 192.168.30.1 | Workstations |
| 40 | Wi-Fi | 10.10.40.0/24 | 10.10.40.1 | Ubiquiti / IoT |
| 50 | Mgmt | 10.10.50.0/24 | 10.10.50.1 | Reserved |

## Trunks
- MikroTik ↔ Cisco: tagged 10,20,30,40,50  
- Cisco ↔ Proxmox: trunked for VM VLAN tagging  
- Desktop port (Gi0/2): access VLAN30
