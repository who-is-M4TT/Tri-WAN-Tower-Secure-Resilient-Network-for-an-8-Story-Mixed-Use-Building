# Security

## Trust zones
Internet (untrusted), DMZ, Guest, Residence (per floor), Enterprise (users, servers, printers), Management, Building systems (ACS), CCTV.
The firewall is the boundary to the internet, DMZ and CCTV; the distribution stack enforces coarse ACLs between internal VLANs.

## Access matrix (rows = source, columns = destination)

| From \ To | Internet | MGMT | ENT users | ENT servers | Printers | ACS (40/41) | Residence | Guest | DMZ | CCTV |
|---|---|---|---|---|---|---|---|---|---|---|
| **Residence floor** | Yes | No | No | No | No | No | **No** (other floors and units) | No | 80/443 | No |
| **Enterprise users / Wi-Fi** | Yes | No | Yes | AD, file, print, RADIUS | Print ports | No | No | No | 80/443 | No |
| **Enterprise servers** | Updates only | No | Replies | Yes | Yes | No | No | No | No | No |
| **Guest** | Yes (rate-limited) | No | No | No | No | No | No | Isolated | 80/443 | No |
| **ACS** | **No** | No | No | No | No | Intra-VLAN | No | No | No | No |
| **MGMT (jump host)** | Yes | Yes | Yes | Yes | Yes | Yes (admin) | Yes | Yes | Yes | Yes (admin/viewing) |
| **DMZ** | Updates only | No | No | No | No | No | No | No | n/a | No |
| **CCTV** | **No** (NTP internal only) | NTP only | No | No | No | No | No | No | No | Intra only |

## Controls

### Network
- Default deny between segments; only documented flows are allowed.
- IPS, NAT and logging on the firewall; per-rule logging for denies on CCTV, DMZ and ACS.
- CCTV: deny-all inbound, NTP only outbound, remote viewing only via VPN and jump host, time-boxed rules for camera firmware updates.
- ACS: no SVI, no gateway; admin via jump host; vendor support via MFA VPN to the jump host.
- Guests: internet only, rate-limited, client isolation.

### Access layer
- Port security on all access ports (sticky MAC on door, elevator and camera ports; MAC limits on outlets).
- Residence outlets: protected ports, DHCP snooping with rate limits, dynamic ARP inspection, storm control.
- Unused ports shut down in VLAN 999; native VLAN 999 on all trunks.
- BPDU guard on access ports; rogue-switch test is part of the verification plan.

### Wi-Fi and identity
- Enterprise Wi-Fi uses 802.1X against RADIUS/NPS; two DCs; roles separated (DC, file, print, RADIUS).
- Residence Wi-Fi: client isolation, one SSID per floor VLAN. Per-unit PPSK recommended later.

### Management
- SSH only, from the jump host; SNMPv3; AAA with unique admin accounts and MFA on the VPN.
- Logs to the NMS/syslog; NTP from one internal source.
- Nightly encrypted config backups; copies kept offline. Secrets live in a password manager, never in this repo.

### Physical
- Floor cabinets sit between the elevator and entrance door and are reachable by residents: locked, ventilated, tamper-alarmed, keys controlled.
- Core room access restricted and logged; UPS and generator protect the core, firewalls and floor switches.

## Publishing checklist (this repo is public)
- [ ] Only RFC 1918 or documentation address ranges; no ISP-assigned public IPs
- [ ] No ISP names, account numbers, circuit IDs or contract details
- [ ] No real building name, street address or tenant names
- [ ] No passwords, SNMP communities, pre-shared keys, VPN configs or tokens (`<REDACTED>` only)
- [ ] No serial numbers, MAC addresses or licence keys
- [ ] No camera placements or door-system details that reveal security gaps
- [ ] Diagram contains no real hostnames or domain names
- [ ] Written permission from the building owner or client to publish
- [ ] Git history checked: secrets removed from earlier commits, not just the latest
