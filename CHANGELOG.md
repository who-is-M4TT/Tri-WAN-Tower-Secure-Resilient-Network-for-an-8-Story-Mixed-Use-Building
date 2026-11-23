# Changelog

All notable changes to this design. Format follows Keep a Changelog; versions are design revisions.

## [Unreleased]
### Planned
- Lab simulation and recorded test results
- Separate L1 / L2 / L3 drawings and rack elevations
- Per-unit isolation for residents (PPSK or private VLANs)
- Intercom, fire alarm, HVAC/BMS and metering networks
- Device models and cost estimate

## [2.0] - 2026-11-22
### Added
- Third ISP (triple WAN) with diverse entry routes and bridge-mode modems; WAN policy and mail pinning
- HA firewall pair, stacked distribution pair (DS-1 + DS-2), UPS and generator
- PoE+ floor switch on every floor (8 total), 2x10G LACP uplinks dual-homed to the core
- Management block (jump host, NMS/syslog/NTP/backup), MFA VPN, guest Wi-Fi (VLAN 60)
- Separate server roles (2x DC, file, print, RADIUS/NPS) on the core at 10G
- Repository restructure: docs, configs (firewalls/core/access), data tables, zone view diagram
### Changed
- CCTV relabelled as a logically isolated zone (VLAN 70); cameras now on floor switches (28 in total)
- "AC" renamed ACS; doors (VLAN 40) and elevators (VLAN 41) separated and not routed
- Floor build per diagram: 2 APs per floor, 1 door and 1 elevator controller per floor, 4 unit outlets per residence floor
- Residence isolation hardened (protected ports, DHCP snooping, ARP inspection, storm control)
### Removed
- Unmanaged PoE switch and the single-switch residence/enterprise access design

## [1.0] - 2026-11-22
### Added
- Initial concept: dual-WAN firewall/router, one distribution switch, one residence and one enterprise access switch, unmanaged PoE CCTV switch, DMZ, per-floor VLANs for residences
