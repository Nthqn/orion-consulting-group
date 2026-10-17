# ORION Consulting Group · Multi-site infrastructure

Design and deployment of the IT infrastructure of a 40-person consulting firm spread across two buildings: segmented network, site-to-site VPN, Active Directory replicated across both sites, file services with DFS, a public website in a DMZ, monitoring and asset management.

Project carried out for the BTS SIO SISR degree (2025–2026). The full technical report (50 pages, in French) is in [`docs/`](docs/DRT.md).

![Infrastructure diagram](diagrams/infrastructure.png)

## At a glance

| | |
|---|---|
| **Scope** | 9 virtual machines on VMware ESXi, two sites |
| **Network** | 8 VLANs (802.1Q), VLSM addressing in /28, inter-VLAN routing on pfSense |
| **Security** | Two pfSense firewalls with least-privilege rules, isolated DMZ, IPsec site-to-site tunnel |
| **Identity** | Active Directory on two domain controllers (Windows Server 2025), integrated DNS, AGDLP groups |
| **Files** | DFS Namespaces and DFS Replication between sites, NTFS permissions, drive mapping by GPO |
| **Operations** | Zabbix 7.0 monitoring, GLPI asset management with LDAPS, central admin portal |

## Virtual machines

| VM | OS | Role | Site |
|---|---|---|:-:|
| ORION | pfSense 2.7.2 | Firewall, router, IPsec VPN, DHCP | 1 |
| RIGEL | Windows Server 2025 | Primary AD DS, DNS, file server | 1 |
| MEISSA | Ubuntu 24.04 | Nginx web server (DMZ) | 1 |
| BELLATRIX | Ubuntu 24.04 | GLPI, admin portal | 1 |
| PC-CLIENT-01 | Windows 11 | User workstation | 1 |
| LYRA | pfSense 2.7.2 | Firewall, router, IPsec VPN, DHCP | 2 |
| VEGA | Windows Server 2025 | Secondary AD DS, DNS, file replication | 2 |
| SULAFAT | Ubuntu 24.04 | Zabbix monitoring | 2 |
| PC-CLIENT-02 | Windows 11 | User workstation | 2 |

## Key design decisions

- **Star topology around two pfSense firewalls.** Each one is the default gateway for its site, so every flow between VLANs goes through a filtering point.
- **Segmentation by department.** Management, HR, accounting, IT, development and Wi-Fi each get their own VLAN, which limits lateral movement after an incident.
- **IPsec site-to-site tunnel.** Traffic between the two sites is encrypted and never crosses the host network in clear text. See the [VPN diagram](diagrams/vpn-ipsec.png).
- **Resilience on the secondary site.** A second domain controller and DFS-R keep the directory, DNS and shared files available if the main site goes down.
- **Public website in a DMZ.** MEISSA is reachable from the WAN through NAT but has no access to the internal network.

## Repository contents

```
docs/
  DRT.md                          Full technical report (French)
  DRT-ORION-Consulting-Group.pdf  Same report, PDF version
  images/                         Screenshots used in the report
  addressing-plan.xlsx            IP addressing plan and VLAN layout
  rbac-matrix.pdf                 Access matrix between VLANs, DMZ and servers
diagrams/
  infrastructure.png / .svg       Overall architecture
  vpn-ipsec.png                   Site-to-site tunnel
  network-sketch.pkt              First network sketch (Cisco Packet Tracer)
```

## Possible improvements

Listed in the report's conclusion: automated workstation deployment (WDS), HTTPS on the public website, Zabbix alerting, a backup strategy for AD, pfSense, GLPI and Zabbix, GPO hardening, and full 802.1Q trunking on client workstations.

## Related procedures

Step-by-step procedures for several building blocks of this project are published on [nathan-lallier.com](https://nathan-lallier.com/procedures) (in French): VLAN segmentation on pfSense, Active Directory domain controller, containerised Zabbix, GLPI.

---

© Nathan LALLIER · [nathan-lallier.com](https://nathan-lallier.com/en/projects/orion-consulting-group)
