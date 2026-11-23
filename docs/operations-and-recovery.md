# Operations and Recovery

## Monitoring (NMS, VLAN 10)
- SNMPv3 polling and syslog from every switch, firewall and AP controller; ping/port checks for servers and the NVR.
- Alerts: WAN gateway down, firewall failover, stack member loss, LACP member down, PoE budget above 80%, UPS on battery, cabinet tamper alarm, err-disabled ports, DHCP snooping violations.
- Dashboards: WAN usage per ISP, uplink utilisation per floor, PoE draw per switch.

## Backups
| What | How often | Where | Restore test |
|---|---|---|---|
| Switch and firewall configs | Nightly and after every change | NMS (encrypted), plus offline copy | Quarterly: restore to a spare |
| Server images / AD (2 DCs) | Nightly | Backup server + offline copy | Semi-annual full restore |
| NVR footage | Continuous | NVR storage (retention per policy) | Monthly spot check |
| Documentation and this repo | On change | Git | n/a |

## Change management
- Changes through a ticket with rollback plan; core and firewall changes in a maintenance window.
- Upgrade the standby firewall and the second stack member first; fail over, then upgrade the other.
- After every change update `tables/` and the diagram, bump the revision in `CHANGELOG.md`.

## Failure runbooks

| Event | Expected behaviour | Action |
|---|---|---|
| ISP down | Traffic moves to remaining WANs | Check monitor, open ticket with the ISP; if WAN 1 is down, confirm DNS/VPN failover |
| FW-A failure | FW-B becomes active | Replace/repair FW-A, resync config and state, re-test failover |
| DS-1 or DS-2 failure | All floors on the surviving member | Replace member, rejoin stack, verify all Po11-Po18 bundles |
| Floor uplink member failure | Floor stays up on the other link | Replace optic/cable; confirm LACP shows two members |
| Floor switch failure | Floor offline | Swap with the on-site spare, load `configs/access/FSW-F<n>.cfg`, check port security and PoE |
| PoE overload | Low-priority ports shed | Check budget, reduce load or upgrade the switch |
| Mains loss | UPS carries load, generator starts | Confirm ATS transfer; verify runtime and refuel plan |
| Cabinet tamper alarm | Alert to security/NMS | Inspect cabinet, check switch logs and port status |

## Spares on site
One 24-port PoE+ and one 48-port PoE+ floor switch (pre-configured), spare SFP+ optics/DAC, spare AP, spare PSU/fan modules where supported, and a console cable and USB stick with the latest configs.

## Power
Size UPS runtime for the core room and each floor rack including PoE load (see PoE estimates in `architecture.md`); test the generator and ATS monthly under load.

## Targets (assumptions, to be agreed)
RTO 1 hour for core and edge; 4 hours for a failed floor switch (spare on site); RPO 24 hours for configs and server data.

## Verification and test plan
Run after applying `configs/` and after major changes. Record pass/fail and date.

| # | Test | Expected |
|---|---|---|
| 1 | Residence PC (F3) to internet | Pass |
| 2 | Residence PC (F3) to residence PC (F4) | Blocked |
| 3 | Two PCs on the same residence floor | Blocked (protected ports) |
| 4 | Residence PC to enterprise servers | Blocked |
| 5 | Residence PC to DMZ web 80/443 | Pass; other ports blocked |
| 6 | Enterprise PC gets DHCP from DCs; reaches file/print | Pass |
| 7 | Enterprise PC to a door controller (VLAN 40) | Blocked |
| 8 | Guest client to internal networks | Blocked; internet only |
| 9 | Door/elevator controller to its ACS server | Pass |
| 10 | Door/elevator controller to internet | Blocked |
| 11 | Jump host SSH to switches | Pass |
| 12 | Any other host SSH to switches | Blocked |
| 13 | VPN user reaches anything but the jump host | Blocked |
| 14 | NVR to cameras | Pass |
| 15 | Camera/NVR to internet (except NTP to internal) | Blocked |
| 16 | Enterprise PC to NVR/cameras | Blocked |
| 17 | Disconnect WAN 1, then WAN 2, then WAN 3 (one at a time) | Failover works; restore resumes balancing |
| 18 | Power off FW-A | FW-B takes over, sessions survive |
| 19 | Power off DS-1 (or DS-2) | All floors, servers, firewall stay up |
| 20 | Unplug one floor uplink member | No loss beyond reconvergence (< 1-2 s) |
| 21 | Mains loss | UPS carries load, generator starts |
| 22 | Rogue DHCP server on a residence port | Dropped by DHCP snooping |
| 23 | Unknown device on a door-controller port | Port err-disabled |
| 24 | Plug into an unused floor-switch port | No link (shutdown) |
