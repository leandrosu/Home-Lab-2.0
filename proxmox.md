# Proxmox VE — Networking & VLAN Integration

## Host Network Interfaces
```
auto vmbr0
iface vmbr0 inet manual
    bridge_ports enp1s0
    bridge_stp off
    bridge_fd 0

auto vmbr10
iface vmbr10 inet static
    address 172.16.10.10/24
    gateway 172.16.10.1
    bridge_ports vmbr0.10
    bridge_stp off
    bridge_fd 0
```
> Each VM uses VLAN tag in NIC configuration.
