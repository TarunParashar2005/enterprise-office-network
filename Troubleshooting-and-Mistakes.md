# Troubleshooting & Mistakes

This lab was not built perfectly in one shot.  
Here are the main issues, how they were fixed, and what was learned.

---

## 1. DHCP not working for any PCs

**Symptom**

- All user PCs stayed on `0.0.0.0`.
- `ipconfig` on PCs showed no IP, no gateway.

**Cause**

- DHCP pools were created on SRV-MAIN (192.168.12.10).
- But the core SVIs had no `ip helper-address`.
- DHCP broadcasts from VLAN 10 never reached the server in VLAN 12.

**Fix (Core switch)**

interface Vlan10
ip helper-address 192.168.12.10
!
interface Vlan12
ip helper-address 192.168.12.10
!
interface Vlan20
ip helper-address 192.168.12.10
!
interface Vlan30
ip helper-address 192.168.12.10
!
interface Vlan40
ip helper-address 192.168.12.10
!
interface Vlan60
ip helper-address 192.168.12.10
!
interface Vlan61
ip helper-address 192.168.12.10


**Lesson**

DHCP is broadcast‑based. In multi‑VLAN networks, every SVI that needs DHCP must have a helper address pointing to the DHCP server.

---

## 2. Can ping gateway but not edge / ISP

**Symptom**

- From a PC, `ping 192.168.10.1` worked.
- `ping 10.0.0.1` (EDGE inside) and `ping 200.0.0.1` (ISP) failed.

**Cause**

- EDGE router had no static routes for the internal 192.168.x.x networks.
- It only knew 10.0.0.0/30 and 200.0.0.0/30.

**Fix (Edge router)**

ip route 192.168.10.0 255.255.255.0 10.0.0.2
ip route 192.168.12.0 255.255.255.0 10.0.0.2
ip route 192.168.20.0 255.255.255.0 10.0.0.2
ip route 192.168.30.0 255.255.255.0 10.0.0.2
ip route 192.168.40.0 255.255.255.0 10.0.0.2
ip route 192.168.60.0 255.255.255.0 10.0.0.2
ip route 192.168.61.0 255.255.255.0 10.0.0.2


**Lesson**

Routing must be symmetric. The edge must know how to reach every internal subnet, not only the core.

---

## 3. Ping 8.8.8.8 fails even after NAT

**Symptom**

- NAT was configured on the edge router.
- `ping 200.0.0.1` from a PC worked.
- `ping 8.8.8.8` still timed out.

**Cause**

- ISP router did not have return routes for 192.168.x.x networks.
- It only knew 200.0.0.0/30.

**Fix (ISP router)**

ip route 192.168.10.0 255.255.255.0 200.0.0.2
ip route 192.168.12.0 255.255.255.0 200.0.0.2
ip route 192.168.20.0 255.255.255.0 200.0.0.2
ip route 192.168.30.0 255.255.255.0 200.0.0.2
ip route 192.168.40.0 255.255.255.0 200.0.0.2
ip route 192.168.60.0 255.255.255.0 200.0.0.2
ip route 192.168.61.0 255.255.255.0 200.0.0.2


**Lesson**

Every router in the path must know the way back. NAT does not replace routing.

---

## 4. Tracert stuck at 192.168.10.1

**Symptom**

- `tracert 8.8.8.8` from a PC showed only one hop:
  - 1: 192.168.10.1 (core gateway)
- No further hops.

**Cause**

- Core L3 switch had no default route towards the edge router.

**Fix (Core switch)**
ip route 0.0.0.0 0.0.0.0 10.0.0.1


**Lesson**

The core needs a default route pointing to the edge so unknown destinations go to the WAN.

---

## 5. Using wireless router as a full router

**Symptom**

- WRT300N was initially planned as a router with its own DHCP and NAT.
- This conflicted with the central design and made troubleshooting harder.

**Fix**

- Decided to use WRT300N only as an access point on its VLAN (30) and leave routing/NAT to the core and edge.

**Lesson**

Keep the design simple: one routing domain in the LAN, one NAT point at the edge.

---

These mistakes are part of the learning process and are kept here on purpose so others can follow the troubleshooting steps.
