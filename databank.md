
# DATABANK — Debian File Server (Samba + Winbind)

  

## Overview

Debian-based file server joined to the fandangos.lab Active Directory domain.

  

## Network

- IP: 172.16.20.3

- Hostname: databank.fandangos.lab

- VLAN: 20 (Servers)

  

## Integration Steps

    sudo apt install -y samba winbind libnss-winbind libpam-winbind \
    
    krb5-user smbclient cifs-utils acl attr python3-dnspython
    
      
    
    net ads join -U Administrator
    
    systemctl restart smbd nmbd winbind
    
      
    
    Add to /etc/pam.d/common-session:
    
    session required pam_mkhomedir.so skel=/etc/skel/ umask=0022
    
      
    
    Test with:
    
    wbinfo -u | head
    
    getent passwd "FANDANGOS\\leandro.uehara"
