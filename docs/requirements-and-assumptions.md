# Requirements and Assumptions

## Requirements

### Functional
- Internet access for residents, enterprise staff and guests, with automatic failover between three ISPs.
- Wired and Wi-Fi 6 access on every floor.
- Separate networks for residents (per floor), enterprise (AD-managed), guests, building systems (doors, elevators) and CCTV.
- A public web server in a DMZ; mail hosted externally (or pinned to one WAN).
- Central management of Wi-Fi, switches and servers; remote access for staff and vendors.

### Non-functional
- No single device failure takes the building offline (firewall, core, uplinks, ISPs).
- Door and elevator systems stay isolated and available on UPS/generator.
- Floor cabinets are reachable by residents, so they must be physically hardened.
- Documentation and configs are publishable: no secrets, no real public IPs or addresses.

## Assumptions (quantities follow the diagram)

| Area | Assumption |
|---|---|
| Floors | 1-2 enterprise, 3-8 residential, one floor switch each |
| Residents | 4 unit outlets and 2 APs per residence floor; unit count per floor to be confirmed |
| Enterprise | 20 outlets, 2 printers, 2 APs per floor; user count to be confirmed |
| Common areas | Lobby/parking/gym Wi-Fi comes from floor-1 APs (VLAN 60) |
| Building systems | 1 door controller and 1 elevator controller per floor |
| CCTV | 28 cameras: 8 each on floors 1-2, 2 each on floors 3-8 |
| PoE draw | 30 W per AP, 15 W per door controller, elevator controller and camera |
| Uplinks | 2x10G per floor, one link to each core switch |
| Addressing | Placeholder RFC 1918 (10.20.0.0/16); real addressing kept private |
| Vendor | Cisco IOS-style config syntax used as a baseline only |
| Mail | Hosted by a provider, or pinned to WAN 1 |

## Out of scope (for Rev 2)
BMS/HVAC, fire alarm, intercom, metering, structured-cabling drawings, rack elevations, detailed cost estimate.

## Open questions
1. Real unit counts and user counts per floor (outlets, APs, DHCP sizing)?
2. Is a second AP controller / NMS / jump host required?
3. Required RTO/RPO for the enterprise tenant and for building systems?
4. Do residents need per-unit isolation (PPSK or private VLANs) rather than per-floor?
5. Is remote video viewing required for residents or only for security staff?
6. Which firewall, switch and AP models fit the budget (inspected throughput and PoE budgets must be re-checked)?
