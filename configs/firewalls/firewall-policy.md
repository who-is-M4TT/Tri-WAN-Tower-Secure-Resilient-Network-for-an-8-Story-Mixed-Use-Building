# Firewall HA Pair: Policy (vendor-neutral)

Maps to pfSense/OPNsense, FortiGate, Palo Alto, MikroTik or similar. Default deny. FW-A is active, FW-B is standby, with a
dedicated state-sync link and a shared virtual IP (VRRP/CARP) on every interface.

## Interfaces

| Interface | Address (VIP) | Connects to | Notes |
|---|---|---|---|
| if-wan1 | `<REDACTED>` | ISP-1 modem (bridge mode) | Primary: mail, VPN, DMZ inbound |
| if-wan2 | `<REDACTED>` | ISP-2 modem (bridge mode) | User traffic |
| if-wan3 | `<REDACTED>` | ISP-3 modem (bridge mode) | User traffic |
| if-lan (VLAN 2) | 10.20.0.1/29 (FW-A .2, FW-B .3, DS-STACK .4) | DS-1 / DS-2 (10G, one each) | Transit to all internal VLANs |
| if-cctv (VLAN 70) | 10.20.70.1/24 | Same 10G trunk as if-lan | Logically isolated CCTV zone |
| if-dmz (VLAN 200) | 10.20.200.1/24 | Web server (2.5G) | Dedicated port |
| if-sync | n/a | FW-A to FW-B | State and config sync |

## Rule base (top to bottom)

| # | From | To | Service | Action |
|---|---|---|---|---|
| 1 | Jump host (10.20.10.50) | Internal nets, if-cctv, firewall GUI | SSH, HTTPS, RTSP | Allow |
| 2 | Admin / vendor VPN pool | Jump host only | RDP, SSH | Allow (MFA required) |
| 3 | Any VPN pool | Anything else | Any | **Deny, log** |
| 4 | CCTV (VLAN 70) | NTP 10.20.10.30 | NTP | Allow |
| 5 | CCTV (VLAN 70) | Any other | Any | **Deny, log** (no internet) |
| 6 | NVR (10.20.70.10) | Cameras (10.20.70.101-128) | RTSP, ONVIF, HTTPS | Allow |
| 7 | Any | CCTV (VLAN 70) | Any | **Deny, log** (except rules 1 and 6) |
| 8 | Residence, enterprise, guest | DMZ web server | 80, 443 | Allow |
| 9 | DMZ | Internal nets | Any | **Deny, log** |
| 10 | DMZ | WAN | 53, 80, 443, 123 | Allow (updates) |
| 11 | WAN | DMZ web server | 80, 443 | Allow (port forward, WAN 1) |
| 12 | Guest (VLAN 60) | Internet | 53, 80, 443 | Allow, rate-limited |
| 13 | Internal 10.20.0.0/16 except VLAN 40/41/70 | WAN | Any | Allow (NAT, IPS on) |
| 14 | ACS VLAN 40 / 41 | Anything | Any | **Deny, log** (ACS is not routed) |
| 15 | Any | Any | Any | **Deny, log** |

## Optional hardening
Route enterprise-to-server flows (VLAN 20/50 to VLAN 25) through the firewall for stateful rules and logging. This costs firewall
throughput, so keep the DS ACLs as a backstop.

## Remote access
- One VPN service (WireGuard or IPsec) with MFA; separate pools for staff admins and vendors.
- After login a session can reach the jump host only. All device and ACS access starts from the jump host and is logged.
- Vendor accounts are time-boxed and disabled when the support case closes.

## Sizing and operations
- Size for **inspected throughput** (IPS + VPN + NAT on), not the raw routing figure.
- Nightly encrypted config backup to the NMS. Never commit exports to this repository.
