# IP Addressing and VLAN Design

## Overview

The enterprise uses a structured IPv4 VLSM addressing plan combined with IPv6 dual-stack addressing.

Each site has its own summarizable IPv4 and IPv6 address space:

| Site | IPv4 Address Space | IPv6 Address Space |
|---|---|---|
| Headquarters | `10.10.0.0/22` | `2001:DB8:10::/48` |
| Melaka | `172.20.0.0/23` | `2001:DB8:20::/48` |
| Kuching | `192.168.50.0/23` | `2001:DB8:30::/48` |

VLSM was used to allocate subnet sizes according to the expected number of devices in each VLAN instead of assigning the same subnet size everywhere.

For IPv6, each VLAN receives its own `/64` prefix.

---

# Headquarters

Headquarters contains the largest number of VLANs because it hosts most departments, centralized services, wireless networks, and the DMZ.

## HQ VLAN Plan

| VLAN | Purpose | IPv4 Subnet | Default Gateway / HSRP VIP | IPv6 Prefix |
|---:|---|---|---|---|
| 10 | Network Management | `10.10.3.64/27` | `10.10.3.65` | `2001:DB8:10:10::/64` |
| 20 | IT | `10.10.2.0/26` | `10.10.2.1` | `2001:DB8:10:20::/64` |
| 30 | Human Resources | `10.10.2.64/26` | `10.10.2.65` | `2001:DB8:10:30::/64` |
| 40 | Finance | `10.10.2.128/26` | `10.10.2.129` | `2001:DB8:10:40::/64` |
| 50 | Sales / Marketing | `10.10.1.128/25` | `10.10.1.129` | `2001:DB8:10:50::/64` |
| 60 | Management | `10.10.3.0/27` | `10.10.3.1` | `2001:DB8:10:60::/64` |
| 80 | Internal Servers | `10.10.3.32/27` | `10.10.3.33` | `2001:DB8:10:80::/64` |
| 110 | Staff Wi-Fi | `10.10.0.0/24` | `10.10.0.1` | `2001:DB8:10:110::/64` |
| 120 | Guest Wi-Fi | `10.10.1.0/25` | `10.10.1.1` | `2001:DB8:10:120::/64` |
| 130 | IoT Wi-Fi | `10.10.2.192/26` | `10.10.2.193` | `2001:DB8:10:130::/64` |
| 200 | DMZ Services | `10.10.3.96/28` | `10.10.3.97` | `2001:DB8:10:200::/64` |
| 201 | DMZ Management | `10.10.3.112/29` | `10.10.3.113` | `2001:DB8:10:201::/64` |

Two additional VLANs are used throughout the switching design:

- **VLAN 99** — Native VLAN
- **VLAN 100** — Blackhole VLAN for unused access ports

These VLANs are not used as normal end-user networks.

---

## HQ Centralized Servers

The internal server segment is located in VLAN 80.

| Service | IPv4 Address |
|---|---|
| DHCP / Internal DNS | `10.10.3.40` |
| AAA / RADIUS | `10.10.3.41` |
| Syslog / NTP | `10.10.3.42` |
| Intranet | `10.10.3.43` |
| IoT Registration Server | `10.10.3.44` |

The default gateway for the server VLAN is:

`10.10.3.33`

---

## DMZ Servers

Public-facing services are hosted in VLAN 200.

| Server | Private IPv4 Address | Public IPv4 Address |
|---|---|---|
| Web | `10.10.3.100` | `198.51.100.10` |
| DNS | `10.10.3.101` | `198.51.100.11` |
| FTP | `10.10.3.102` | `198.51.100.12` |
| Mail | `10.10.3.103` | `198.51.100.13` |

The DMZ service gateway is:

`10.10.3.97`

The dedicated DMZ management subnet uses:

`10.10.3.112/29`

with gateway:

`10.10.3.113`

---

# Melaka Branch

Melaka uses HSRP between two branch routers, so each user VLAN has a shared virtual default gateway.

## Melaka VLAN Plan

| VLAN | Purpose | IPv4 Subnet | HSRP VIP | IPv6 Prefix |
|---:|---|---|---|---|
| 10 | Network Management | `172.20.1.160/28` | `172.20.1.161` | `2001:DB8:20:10::/64` |
| 20 | Operations | `172.20.0.128/26` | `172.20.0.129` | `2001:DB8:20:20::/64` |
| 30 | Sales | `172.20.1.0/26` | `172.20.1.1` | `2001:DB8:20:30::/64` |
| 40 | IT | `172.20.1.96/27` | `172.20.1.97` | `2001:DB8:20:40::/64` |
| 50 | Procurement | `172.20.1.64/27` | `172.20.1.65` | `2001:DB8:20:50::/64` |
| 110 | Staff Wi-Fi | `172.20.0.0/25` | `172.20.0.1` | `2001:DB8:20:110::/64` |
| 120 | Guest Wi-Fi | `172.20.0.192/26` | `172.20.0.193` | `2001:DB8:20:120::/64` |
| 130 | IoT Wi-Fi | `172.20.1.128/27` | `172.20.1.129` | `2001:DB8:20:130::/64` |

Melaka also uses:

- VLAN 99 — Native VLAN
- VLAN 100 — Blackhole VLAN

---

# Kuching Branch

Kuching uses Router-on-a-Stick with a single branch router.

Unlike HQ and Melaka, the VLAN gateways are therefore hosted directly on router subinterfaces rather than using IPv4 HSRP.

## Kuching VLAN Plan

| VLAN | Purpose | IPv4 Subnet | Default Gateway | IPv6 Prefix |
|---:|---|---|---|---|
| 10 | Network Management | `192.168.51.64/28` | `192.168.51.65` | `2001:DB8:30:10::/64` |
| 20 | Logistics | `192.168.50.64/26` | `192.168.50.65` | `2001:DB8:30:20::/64` |
| 30 | Operations | `192.168.50.224/27` | `192.168.50.225` | `2001:DB8:30:30::/64` |
| 40 | Support | `192.168.51.0/27` | `192.168.51.1` | `2001:DB8:30:40::/64` |
| 110 | Staff Wi-Fi | `192.168.50.0/26` | `192.168.50.1` | `2001:DB8:30:110::/64` |
| 120 | Guest Wi-Fi | `192.168.50.192/27` | `192.168.50.193` | `2001:DB8:30:120::/64` |
| 130 | IoT Wi-Fi | `192.168.50.128/26` | `192.168.50.129` | `2001:DB8:30:130::/64` |
| 150 | Voice | `192.168.51.32/27` | `192.168.51.33` | `2001:DB8:30:150::/64` |

The Voice VLAN gateway also acts as the local Cisco CME address and DHCP Option 150 destination:

`192.168.51.33`

---

# Corporate WAN Addressing

Point-to-point IPv4 networks are used between the main WAN routers.

| Link | Network | Endpoint A | Endpoint B |
|---|---|---|---|
| CORE ↔ HQ | `10.255.0.0/30` | CORE `10.255.0.1` | HQ `10.255.0.2` |
| CORE ↔ Melaka | `10.255.0.4/30` | CORE `10.255.0.5` | MEL `10.255.0.6` |
| CORE ↔ Kuching | `10.255.0.8/30` | CORE `10.255.0.9` | KCH `10.255.0.10` |
| Melaka ↔ Kuching Backup | `10.255.0.20/30` | MEL `10.255.0.21` | KCH `10.255.0.22` |

The Melaka–Kuching link is configured as a higher-cost OSPF path and is primarily used as backup connectivity.

IPv6 WAN links use prefixes from:

`2001:DB8:FF::/48`

with individual `/64` prefixes assigned to point-to-point segments.

---

# Internet Edge Addressing

The enterprise Internet edge uses separate transit, ISP, public service, and simulated Internet networks.

| Purpose | Network |
|---|---|
| CORE ↔ IPS | `10.255.1.0/30` |
| IPS ↔ Internet Edge Router | `10.255.1.4/30` |
| Enterprise Edge ↔ ISP | `198.51.100.0/30` |
| Enterprise Public NAT Block | `198.51.100.8/29` |
| Simulated Internet Server Network | `203.0.113.0/24` |
| External Security Test Network | `203.0.114.0/24` |

Key addresses include:

| Device / Function | Address |
|---|---|
| Enterprise Internet Edge | `198.51.100.1` |
| ISP Router | `198.51.100.2` |
| PAT Public Address | `198.51.100.9` |
| Public Web | `198.51.100.10` |
| Public DNS | `198.51.100.11` |
| Public FTP | `198.51.100.12` |
| Public Mail | `198.51.100.13` |
| Internet Server | `203.0.113.10` |
| External Security Test Host | `203.0.114.20` |

---

# Addressing Design Summary

The addressing plan was designed to provide:

- Clear separation between sites.
- Route summarization by location.
- Efficient IPv4 address usage through VLSM.
- One IPv6 `/64` per VLAN.
- Consistent VLAN numbering where practical.
- Dedicated management, service, wireless, IoT, voice, and DMZ networks.
- Separate public and private addressing at the Internet edge.

The use of different private IPv4 ranges for each site also makes the origin of traffic easier to identify during routing, NAT, ACL, VPN, Syslog, and troubleshooting tests.

---

## Related Documentation

- [Network Architecture](01-architecture.md)
- [Layer 2 Design](03-layer2-design.md)
- [Routing and Redundancy](04-routing-redundancy.md)
- [IPv6 Design](05-ipv6.md)
