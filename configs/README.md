# Configs

Baseline device configurations for the network. Syntax is **Cisco IOS-style** for the switches; the firewall policy is
vendor-neutral. Adapt interface names and commands to the models you buy. All secrets are placeholders (`<REDACTED>`).

| Path | Device | Notes |
|---|---|---|
| `core/ds-stack.cfg` | Distribution stack DS-1 + DS-2 | L3: SVIs, ACLs, DHCP, LACP bundles to every floor, server blocks, firewall trunk |
| `access/FSW-F1.cfg` ... `FSW-F8.cfg` | Floor switches | F1-F2: 48-port PoE+ (enterprise). F3-F8: 24-port PoE+ (residence) |
| `firewalls/firewall-policy.md` | Firewall HA pair | Interfaces, rule base, VPN, CCTV and DMZ policy |
| `firewalls/wan-policy.md` | Firewall HA pair | Triple-ISP failover, load balancing, mail pinning |

## Conventions
- Hostnames: `DS-STACK`, `FSW-F<floor>`. Management IPs are in `tables/ip-address-plan.csv`.
- VLAN IDs and names match `tables/vlan-plan.csv`. Port assignments match `tables/port-map.csv`.
- Every trunk uses native VLAN 999 and an explicit allowed-VLAN list.
- Unused ports: shutdown, VLAN 999, `power inline never` on floor switches.
- Access ports: `portfast`, BPDU guard, port security. Residence outlets also use protected ports, DHCP snooping limits and storm control.
- SSH only, from the jump host (10.20.10.50). SNMPv3. Logs and NTP go to 10.20.10.30.

## Applying
1. Replace every `<REDACTED>` with a real secret from your password manager (never commit them).
2. Apply `core/` first, then `access/`, then the firewall policy.
3. Run the checks in `docs/operations-and-recovery.md` (verification section).
