# Network Layer (Layer 3) — Short Summary

The **network layer** provides **end-to-end delivery**: getting a packet from source to destination across multiple routers, regardless of underlying technology. Two core tasks are central: **routing** (determining the path) and **forwarding** (actually forwarding the packet based on the routing table), plus everything around **IP addressing** that makes this possible.

## Connectionless vs connection-oriented

Routers operate using **store-and-forward**: a packet is fully received, checked and only then forwarded.

- **Connectionless (datagram)**: each packet carries the full destination address and is routed separately. Different packets from the same flow can take different routes. **IP is the prime example.**
- **Connection-oriented (virtual circuit)**: during setup a circuit is agreed upon; packets then carry only a **circuit/label identifier** (locally valid, can change per hop = **label switching**). **MPLS** is the example.

| | Virtual circuits | Datagrams |
|---|---|---|
| Advantage | simpler routing, smaller tables, QoS/resource reservation possible | no setup, faster start, more robust on router crash |
| Disadvantage | fixed path, setup overhead | no guarantees on order/path |

## Routing algorithms

A good algorithm is **correct, simple, robust, stable, fair and efficient**. The **optimality principle**: if J lies on the optimal path from I to K, then the optimal path from J to K also lies on that path. All optimal paths to one destination together form a **sink tree**.

**Dijkstra**: from the source node, repeatedly make the node with the lowest tentative cost permanent, then recalculate the neighbors. Basis of link-state routing.

| | Distance vector (Bellman-Ford) | Link state (e.g. OSPF) |
|---|---|---|
| What is shared? | own best estimates to neighbors | own link costs to neighbors (link-state packets), via **flooding** |
| Computation | `cost via neighbor = cost to neighbor + neighbor's estimate to destination` | each router reconstructs the entire graph and runs Dijkstra itself |
| Biggest problem | **count-to-infinity**: when a route disappears, a reasoning loop forms, distance rises slowly (3,4,5,6...) to "infinity" because routers only have local information | more computation, storage and control traffic |
| Convergence | bad news slow, good news fast | faster and more correct |

**Link-state in 5 steps**: (1) discover neighbors via **HELLO**, (2) determine link costs (bandwidth/delay), (3) create link-state packet (sender, **sequence number**, **age**, neighbors+costs), (4) flooding (to all links except incoming), (5) run Dijkstra. Flooding is limited via **TTL/hop counter**, **sequence numbers** (discard duplicates) and **age** (ignore old info).

**Hierarchical routing**: group regions so routers only know access points of other regions instead of all details. Gives smaller tables and better scalability, at the cost of sometimes non-optimal paths.

**Delivery models**: **unicast** (1-to-1), **broadcast** (to all), **multicast** (to group). Best broadcast in terms of number of packets = **spanning tree** (no loops, covers all routers), but requires global knowledge.

## Congestion control, QoS and internetworking

Congestion is addressed via a combination: avoidance (traffic-aware routing - works poorly due to **oscillation**), rejection (**admission control**), signaling and ultimately **dropping**. More buffers do not solve it (excessive wait times = instability).

Three throttling signals/mechanisms:

| Mechanism | Operation | Speed | Overhead |
|---|---|---|---|
| **Choke packets** | router sends separate control packet to sender | slow (full RTT) | high |
| **ECN** | router marks regular packet, receiver reports back to sender | slightly slower than choke (+1 RTT) | low |
| **RED** | router drops early and randomly with rising queue, hits faster senders relatively more often | fast | extra retransmissions |

As a load signal, **queuing delay** is an earlier/better signal than **packet loss** (which comes too late).

**QoS = 3 building blocks**: traffic shaping, packet scheduling, admission control.
- **Leaky bucket**: bucket with capacity **B** (burst tolerance) and drain rate **R** (long-term rate). B=0 → completely flat output.
- **Scheduling**: FIFO (simple, **tail drop**, not fair) → **Fair Queuing** (each flow gets its own queue, round-robin — but large-packet tricks can circumvent this) → **Weighted Fair Queuing** (weights per flow, e.g. real-time > background).

**Internetworking**: networks differ in addressing, MTU, routing, QoS, security. Two approaches: **translation** (direct conversion) vs **indirection** (common intermediate layer — this is **IP**). **Switches = layer 2 (frames)**, **routers = layer 3 (packets)**. **Tunneling** = encapsulating a packet of one type in another network type (e.g. IPv6-over-IPv4), counts as 1 hop for higher layers.

**Fragmentation**: needed due to different **MTUs**. Transparent (routers fragment and reassemble — extra router work) vs non-transparent (hosts must reassemble). Better: **Path MTU Discovery** — sender sends with **DF bit**, oversized packets are dropped + ICMP error message sent back, sender reduces packet size. PMTUD and regular fragmentation are **incompatible** (fragmentation hides the MTU problem).

## OSPF, BGP, mobility and ad-hoc

- **AS (Autonomous System)**: network under management of one organization, with **ASN**.
- **Intra-domain** (within an AS) → **interior gateway protocol** = **OSPF**.
- **Inter-domain** (between ASes) → **exterior gateway protocol** = **BGP**.

**OSPF**: link-state protocol (HELLO, flooding, Dijkstra), supports hierarchy via **areas** (**Area 0 = backbone**, stub areas hang off it) and **ECMP** (load balancing over paths with equal cost). OSPF messages travel as regular IP packets.

**BGP**: distance-vector-like but operates on **AS paths** instead of distances, and is driven by **policy** (which routes to accept/announce), not purely shortest path. Key concepts:
- **Peering**: ASes exchange traffic (often via **IXP**), free, and **not transitive** (A-B and B-C peering does not give A free transit to C).
- **Transit**: paying to have traffic carried further.
- **iBGP**: boundary routers within an AS share external routes; internal reachability via OSPF. Rule of thumb: OSPF chooses the path *within* the AS, BGP chooses *via which AS* you go outward.

**Mobility**: mobile host keeps a fixed **home address**; at a new location it gets a **care-of address**, reported to its **home agent**, which then tunnels traffic. This causes **triangle routing** (remote host → home agent → mobile host) and a **security risk** (registering a fake care-of address = redirecting traffic).

**Ad-hoc networks**: no fixed infrastructure, nodes = host + router simultaneously, **resource-/energy-constrained and unreliable**. **AODV** is **reactive/on-demand**: floods a **ROUTE_REQUEST** (with sequence number + TTL, TTL increases stepwise to limit floods) until a **ROUTE_REPLY** comes back along the reverse path. AODV addresses count-to-infinity via **destination sequence numbers**: higher sequence = fresher, older routes are discarded. Long timeouts remain a problem.

## IP, addressing, NAT, ICMP and ARP

**IPv4 header** (variable length, hence **IHL**): most important fields are **TTL** (decremented per hop, prevents infinite loops), **Protocol** (TCP/UDP/ICMP), **header checksum** (recalculated per hop because TTL changes), and **Identification + MF + Fragment Offset** (together responsible for reassembly of fragments). DF bit = Don't Fragment (see PMTUD). Max packet size = 64 KB.

**Addressing**: 32-bit IPv4 address = network prefix + host part, written in **CIDR** (e.g. `/24`) or subnet mask (e.g. `255.255.255.0`). Routers work with **prefixes**, not individual hosts → on overlap the **longest prefix match** wins. Classes (A/B/C) were too coarse-grained ("Three Bears Problem") and were replaced by **CIDR** (flexible prefix lengths, supernetting).

**DHCP** (application layer) distributes addresses automatically via **leases** (short = more efficient, long = less server load). **Private ranges**: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.

**NAT**: translates private addresses to one public address, uses **ports** as index in the translation table. Problems: breaks **end-to-end model**, is implicitly connection-oriented, works poorly with protocols that carry IP addresses in the payload (FTP, SIP, IPsec). **Traversal**: STUN (discover public address), TURN (relay), UPnP (automatic port mapping), connection reversal, ALG (application-level gateway).

| | IPv4 | IPv6 |
|---|---|---|
| Address length | 32 bit | 128 bit |
| Header | variable (min. 20 bytes, IHL field, options) | fixed **40 bytes**, extras in extension headers |
| Checksum | header checksum, recalculated per hop | **no** header checksum |
| Fragmentation | possible by routers | routers do **not** fragment → PMTUD mandatory; min. MTU **1280 bytes** |
| Address types | unicast/broadcast/multicast | **unicast/multicast/anycast** (no more broadcast) |
| Config | DHCP | SLAAC (stateless autoconfiguration) |

IPv6 notation: hex groups with `:`, drop leading zeros, and **`::` may only be used once** per address. Important ranges: **global unicast** (`2000::/3`, routable), **unique local** (`fc00::/7`, private networks), **link local** (`fe80::/10`, not routed, mandatory per interface).

### IPv6 SLAAC and privacy

**SLAAC** (Stateless Address Autoconfiguration): host generates its own IPv6 address from network prefix (via Router Advertisement) + **EUI-64** interface identifier (derived from MAC address: split MAC, insert `ff:fe`, flip 7th bit).

**Privacy problem**: MAC address is permanent → EUI-64 address is the same everywhere → user is trackable across networks + manufacturer (OUI) visible.

**Countermeasures**: RFC 4941 (random temporary addresses), RFC 7217 (stable but non-derivable per network), MAC randomization.

**SLAAC vs DHCPv6**: SLAAC = stateless, no server needed, scales better. DHCPv6 = stateful, more control (DNS, logging), but server = single point of failure.

### Subnetting: method

1. Prefix → subnet mask (`/25` = `255.255.255.128`)
2. Network address = IP AND subnet mask
3. Broadcast address = network address OR inverse mask (all host bits = 1)
4. First host = network address + 1, last host = broadcast - 1
5. Number of hosts = 2^(32-prefix) - 2

### IoT and IP stack (6LoWPAN)

IPv6 for IoT (802.15.4): **6LoWPAN** = adaptation layer that adapts IPv6 for constrained networks (header compression 40→few bytes, fragmentation with small MTU). IPv4 unsuitable: too few addresses, no SLAAC. Advantage of IP on IoT: end-to-end reachability, existing tools. Disadvantage: overhead for constrained devices.

**ICMP** = error reporting/diagnostic channel of IP: **destination unreachable** (including Type 3/Code 4 for PMTUD), **time exceeded** (TTL=0), **redirect**, **echo request/reply** (ping). **Traceroute** uses increasing TTL (1,2,3,...) so each router on the path sends back a "time exceeded".

**ARP**: translates IP address to MAC address **within the same subnet** via cache → broadcast request → unicast reply. For hosts outside the subnet, only the MAC address of the **default gateway** is cached, not the final destination's.

## Common exam pitfalls / important to remember

- **PMTUD and fragmentation are incompatible**: DF=1 + oversized packet → router drops and sends ICMP Type 3/Code 4 back (no silent fragmentation). A firewall that blocks ICMP creates a **PMTUD black hole**.
- **Count-to-infinity** is a fundamental problem of distance-vector/Bellman-Ford due to only local knowledge; **AODV** partially solves this with **destination sequence numbers** (higher = fresher), but long timeouts remain difficult.
- **Peering is not transitive** — and BGP revolves around **AS paths + policy**, not the shortest technical route (that is the domain of OSPF/intra-domain).
- For congestion notification: **choke packets** (slow, much overhead) vs **ECN** (fast, little overhead, marks existing packet) vs **RED** (early random drops, hits fast senders harder) — know the trade-offs.
- **ARP only works within the subnet**: a host outside your subnet is reached via your default gateway, and your ARP cache then only contains an entry for that gateway, not for the final destination.
- Calculation/notation classics: correctly convert CIDR notation and subnet mask (e.g. `/25` = `255.255.255.128`), **longest prefix match** with overlapping routes, and IPv6 `::` abbreviation may only be used **once**.
- **IPv6 SLAAC + EUI-64 = privacy problem**: MAC address in the IPv6 address enables tracking across networks. Recognizable by `ff:fe` in the middle of the interface ID.
- **NAT traversal**: incoming connections blocked → STUN/TURN/UPnP/connection reversal needed. FTP, SIP and P2P break behind NAT.
- **6LoWPAN**: adaptation layer for IPv6 on 802.15.4 (IoT) — header compression + fragmentation.
