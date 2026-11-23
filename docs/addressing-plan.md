# Addressing Plan

All addresses are private placeholders (RFC 1918, 10.20.0.0/16). Machine-readable versions:
[`tables/vlan-plan.csv`](../tables/vlan-plan.csv), [`tables/ip-address-plan.csv`](../tables/ip-address-plan.csv),
[`tables/port-map.csv`](../tables/port-map.csv). Never publish real public IPs.

## Scheme
- Each VLAN has one /24: `10.20.<VLAN-ID>.0/24` (for example VLAN 25 is 10.20.25.0/24).
- Residence VLAN ID = 100 + floor number (floor 3 is VLAN 103 ... floor 8 is VLAN 108).
- The gateway is `.1` in every subnet; `.2-.9` are reserved for infrastructure; hosts and DHCP start at `.10`.
- Special subnets: `10.20.0.0/29` (VLAN 2, firewall transit) and `10.20.200.0/24` (DMZ).

## VLAN summary

| VLAN | Name | Subnet | Gateway | Zone |
|---|---|---|---|---|
| 2 | TRANSIT | 10.20.0.0/29 | Firewall VIP .1 | Core |
| 10 | MGMT | 10.20.10.0/24 | DS stack | Management |
| 20 | ENT-USERS | 10.20.20.0/24 | DS stack | Enterprise |
| 25 | ENT-SERVERS | 10.20.25.0/24 | DS stack | Enterprise |
| 30 | ENT-PRINTERS | 10.20.30.0/24 | DS stack | Enterprise |
| 40 | ACS-DOOR | 10.20.40.0/24 | none (not routed) | Building systems |
| 41 | ACS-ELEV | 10.20.41.0/24 | none (not routed) | Building systems |
| 50 | WIFI-ENT | 10.20.50.0/24 | DS stack | Enterprise |
| 60 | GUEST | 10.20.60.0/24 | DS stack | Guest |
| 70 | CCTV | 10.20.70.0/24 | Firewall | CCTV |
| 103-108 | RES-F3 ... RES-F8 | 10.20.103-108.0/24 | DS stack | Residence |
| 200 | DMZ | 10.20.200.0/24 | Firewall | DMZ |
| 999 | BLACKHOLE | n/a | n/a | Unused / native VLAN |

## DHCP
| Scope | Server | Range | Lease |
|---|---|---|---|
| Residence floors (103-108) | DS stack | .10-.254 | 7 days |
| Guest (60) | DS stack | .10-.254 | 8 hours |
| Enterprise users and Wi-Fi (20, 50) | DC-1 / DC-2 (helper addresses on DS) | .10-.254 | per AD policy |
| Cameras (70) | Static or reservations | .101-.128 | n/a |

## Key hosts
Floor switches: `10.20.10.21`-`.28` (floor 1 to 8). Jump host `10.20.10.50`. NMS/syslog/NTP `10.20.10.30`. AP controller `10.20.10.20`.
DC-1/DC-2 `10.20.25.10`/`.11`, file `.12`, print `.13`, RADIUS `.14`. ACS servers `10.20.40.10` and `10.20.41.10`. NVR `10.20.70.10`.
Cameras `10.20.70.101-128` (8 per enterprise floor, 2 per residence floor). Web server `10.20.200.10`.

## Growth
Each /24 holds about 240 hosts per VLAN; if a residence floor needs more, move it to a /23 and update ACLs. Keep spare VLAN IDs (110-199) for new residence or tenant networks.
