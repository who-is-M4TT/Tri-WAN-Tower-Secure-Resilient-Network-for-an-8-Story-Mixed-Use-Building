# WAN Policy: Triple ISP

| WAN | Role | Carries |
|---|---|---|
| WAN 1 | Primary | Outbound mail (if self-hosted), VPN, DMZ inbound |
| WAN 2 | Active | Outbound user traffic (load balanced) |
| WAN 3 | Active | Outbound user traffic (load balanced) |

## Behaviour
- **Monitoring:** each WAN is probed against two public targets. Three consecutive failures mark it down; recovery needs a sustained pass to prevent flapping.
- **Load balancing:** WAN 2 and WAN 3 are weighted (start 1:1). Sessions are sticky per source so a user's public IP does not change mid-session.
- **Failure of WAN 2 or WAN 3:** the survivor and WAN 1 absorb the traffic.
- **Failure of WAN 1:** VPN and DMZ inbound move to WAN 2 (then WAN 3). Update DNS records with short TTLs (60-300 s) in advance.
- **All three down:** local services keep working; remote access is lost. Escalate to the ISPs.

## Physical and provider requirements
- Three different ISPs, three different building entry routes (A, B, C) and, where possible, different upstream paths.
- Modems or ONTs in **bridge mode** so the firewall holds the public addresses (no double NAT).
- Document each circuit (ID, SLA, contact) in a private location. Do not publish it here.

## Mail
Use a hosted mail provider (preferred), or pin the mail server to WAN 1 with matching SPF, DKIM, DMARC and reverse DNS.
Never load-balance mail across WANs.
