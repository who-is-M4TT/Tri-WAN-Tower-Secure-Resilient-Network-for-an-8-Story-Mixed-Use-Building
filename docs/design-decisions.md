# Design Decisions

Short records of the main choices, with rationale and trade-offs.

| ID | Decision | Rationale | Trade-off / follow-up |
|---|---|---|---|
| D-01 | **Three ISPs**, diverse entries, bridge-mode modems | Survives a provider or route failure; firewall owns public IPs (no double NAT) | Extra recurring cost; three circuits to manage |
| D-02 | **WAN 1 pinned** for mail/VPN/DMZ; WAN 2+3 balanced for users | Keeps SPF/PTR consistent and gives users the most capacity | DNS TTLs must be short to fail WAN 1 services over |
| D-03 | **HA firewall pair** (active/standby, state sync) | Removes the firewall as a single point of failure | Two licences, stricter change control |
| D-04 | **Stacked/MLAG distribution** (DS-1 + DS-2) | Survives a core switch failure | Stack software must be upgraded carefully; stack cables are a dependency |
| D-05 | **PoE+ switch on every floor**, 2x10G dual-homed uplinks | Powers APs/controllers/cameras locally; fibre/10G avoids the 1G bottleneck | 16 optics for floors; more devices to patch |
| D-06 | **Servers and NVR on the core at 10G** | Enterprise-to-server flows no longer cross a 2x1G trunk twice | Core ports consumed by server blocks |
| D-07 | **Roles separated** (2x DC, file, print, RADIUS/NPS) | Fault and security isolation, second DC for AD resilience | More servers/VMs to run |
| D-08 | **VLAN per residence floor** with protected ports, snooping, client isolation | Floor-level segmentation with simple operations | Not per-unit; PPSK or private VLANs are a future option |
| D-09 | **CCTV is a logically isolated zone** (VLAN 70, firewall gateway, deny-all inbound, NTP only, VPN viewing) | Keeps time sync, firmware updates and remote viewing manageable | Not air-gapped; relies on correct firewall rules |
| D-10 | **Cameras on managed floor switches** (no unmanaged PoE switch) | Port security, monitoring and remote PoE control; shorter cable runs | Camera PoE counts toward floor budgets |
| D-11 | **ACS is not routed**; doors (VLAN 40) and elevators (VLAN 41) separated | Building systems cannot be reached from user networks; vendors get only what they need | Admin and vendor access must go through the jump host |
| D-12 | **Routing and ACLs at the core**, firewall as gateway only for CCTV/DMZ/WAN | Fast inter-VLAN traffic at 10G | Stateless ACLs; stateful option documented for server flows |
| D-13 | **Hosted mail** (or pinned to WAN 1) | Avoids mail reputation problems from multi-WAN NAT | Dependency on a provider |
| D-14 | **Jump host + MFA VPN** is the only admin path | One audited entry point for devices, ACS and CCTV | Jump host must be hardened and highly available |
| D-15 | **Locked, ventilated, tamper-alarmed cabinets**, port security, unused ports shut | Cabinets are in a corridor reachable by residents; ACS terminates there | Cabinet cost, key management |

## Design review log (Rev 1 to Rev 2)

| Area | Review point | Outcome |
|---|---|---|
| Physical | Switch per floor, quantities, PoE+ budgets, cabinet security | Done (D-05, D-15) |
| Redundancy | HA firewall, stacked core, UPS/generator, ISP diversity, bridge mode | Done (D-01, D-03, D-04) |
| Bandwidth | 10G uplinks, 2.5G AP ports, servers off the 2x1G trunk | Done (D-05, D-06) |
| Security | CCTV labelling, managed camera switching, residence isolation, VLAN 40 admin path, mail, AD roles, RADIUS | Done (D-07 to D-14) |
| Security | Per-unit PPSK/private VLANs | Open |
| Security | Stateful enterprise-to-server rules via firewall | Optional (documented) |
| Missing | Management/monitoring, guest Wi-Fi, common areas | Done |
| Missing | Intercom, fire alarm, HVAC/BMS, meters | Open |
| Presentation | ACS naming, bundled port-channel lines, title block, VLAN/interface IDs | Done |
| Presentation | Separate L1/L2/L3 drawings, port labels at both ends on the drawing, device models | Open (port maps are in `tables/port-map.csv`) |
