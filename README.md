# Enterprise Office Network (Cisco Packet Tracer)

A realistic 3‑tier office network built in Cisco Packet Tracer with:

- 7 VLANs (Users, Servers, Voice, Guest, Admin, CCTV, IoT)
- Central DHCP server with relay on the core switch
- Inter‑VLAN routing on a Layer 3 switch
- WAN connectivity to an ISP with NAT/PAT on the edge router
- Wired users, admin PC, network printer, and IP phones

## Topology

Packet Tracer file: **`office-network-topology.pkt`**

Open it with Cisco Packet Tracer 8.x to view the full design.

## IP Design

- **WAN:** `200.0.0.0/30` (ISP ↔ Edge), `8.8.8.8/32` (simulated Internet)
- **Edge–Core link:** `10.0.0.0/30`
- **LAN VLANs (private 192.168.0.0/16):**
  - VLAN 10 – Users: `192.168.10.0/24` (GW `192.168.10.1`)
  - VLAN 12 – Servers: `192.168.12.0/24` (GW `192.168.12.1`)
  - VLAN 20 – Voice: `192.168.20.0/24` (GW `192.168.20.1`)
  - VLAN 30 – Guest/WiFi: `192.168.30.0/24` (GW `192.168.30.1`)
  - VLAN 40 – Admin: `192.168.40.0/24` (GW `192.168.40.1`)
  - VLAN 60 – CCTV: `192.168.60.0/24` (GW `192.168.60.1`)
  - VLAN 61 – IoT: `192.168.61.0/24` (GW `192.168.61.1`)

DHCP pools for each VLAN are configured on **SRV‑MAIN (192.168.12.10)** and reached via `ip helper-address` on every SVI.

## Device Configurations

Raw configs are in the repository:

- `CORE-L3-SW.txt` – VLANs, SVIs, DHCP relay, default route
- `EDGE-ROUTER.txt` – NAT/PAT, static routes, loopback (TFTP)
- `ACCESS-SWITCH.txt` – trunk/access/voice ports
- `ISP-ROUTER.txt` – WAN link and return routes
- `SRV-MAIN-DHCP.txt` – DHCP pools and server IP setup

## How to Use

1. Download `office-network-topology.pkt`.
2. Open it in Cisco Packet Tracer.
3. Power on the devices and wait for DHCP.
4. From a user PC, test:
   - `ping 192.168.10.1` (gateway)
   - `ping 192.168.12.10` (server)
   - `ping 200.0.0.1` (ISP)
   - `ping 8.8.8.8` (Internet via NAT)

## Troubleshooting & Mistakes

During the build, a few intentional/real mistakes were fixed:

- Missing `ip helper-address` → DHCP did not work across VLANs.
- Missing routes on the edge router → could reach core but not ISP.
- No return routes on ISP → pings to 8.8.8.8 failed.
- No default route on the core → `tracert` stuck at gateway.

These are kept to show the troubleshooting process and learning.

Built as a CCNA‑level lab and portfolio project.

