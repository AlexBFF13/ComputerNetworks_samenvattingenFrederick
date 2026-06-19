# Computer Networks — Open-Book Reference Guide

> G0Q43A · KU Leuven · Exam June 23, 2026
> Optimized for quick lookup under time pressure. Focus: architectural logic, cross-layer reasoning, design choices.

---

## Table of Contents

**1. Application Layer**
- 1.1 DNS: hybrid resolution architecture
- 1.2 DHCP: bootstrapping without IP
- 1.3 FTP: NAT incompatibility and passive mode
- 1.4 Gnutella 0.4 vs 0.6: architecture, espionage, transport
- 1.5 TOR: knowledge distribution, minimum hops, surveillance
- 1.6 HTTP 1.0 vs 1.1: persistent connections
- 1.7 P2P chat app: overlay design choice
- 1.8 NAT: application layer problems and traversal

**2. Transport Layer**
- 2.1 RTP vs TCP: sequencing, download vs stream, congestion
- 2.2 Sliding Windows: formulas, satellite, LoRa
- 2.3 TCP Flow Control vs Go-Back-N vs Selective Repeat
- 2.4 ACK Clock: self-regulating pacing
- 2.5 Nagle + Delayed ACKs: interaction problem
- 2.6 TCP Tahoe vs Reno: loss response
- 2.7 QUIC: design and difference with TCP
- 2.8 Congestion Notification: Choke/ECN/RED
- 2.9 TCP Connection Management: handshake, teardown, timers

**3. Network Layer**
- 3.1 IPv4 Fragmentation vs Path MTU Discovery
- 3.2 IPv6 SLAAC: privacy via EUI-64
- 3.3 IPv6: no header checksum — why?
- 3.4 IoT and the IP Stack (6LoWPAN)
- 3.5 AODV vs OSPF: reactive vs proactive + count-to-infinity
- 3.6 OSPF vs BGP: intra-AS vs inter-AS
- 3.7 Subnetting: method + routing tables + ARP
- 3.8 PDU analysis in multi-hop routing
- 3.9 NAT problems + traversal techniques
- 3.10 Addressing per layer: IP vs MAC vs port
- 3.11 Zigbee/BLE mesh routing to IPv6 gateway

**4. Data Link & Physical Layer**
- 4.1 CSMA/CA: Hidden/Exposed Terminal + RTS/CTS + NAV
- 4.2 BMAC vs TSMP: preamble/sampling trade-off + fusion
- 4.3 Hub vs Switch: architecture, collision domains, privacy
- 4.4 Collision Domains and Cable Length (CSMA/CD)
- 4.5 802.11 Packet Loss as TCP Congestion Signal
- 4.6 LoRa: limitations, ALOHA analysis, MAC improvements
- 4.7 Pure ALOHA vs Slotted ALOHA
- 4.8 Framing: byte-stuffing vs bit-stuffing
- 4.9 Ethernet Frame Padding + Minimum Frame
- 4.10 Stop-and-Wait: when good/bad
- 4.11 Null MAC: problems under load
- 4.12 802.11 Power Saving (two strategies)
- 4.13 Cross-layer: MAC protocol choice with AODV

---

## Quick Reference

**PDU terminology per layer (use correctly in answers):**

| Layer | PDU name | Example |
|---|---|---|
| Application (L7) | Message / Data | HTTP request, DNS query |
| Transport (L4) | **Segment** (TCP) / **Datagram** (UDP) | TCP segment, UDP datagram |
| Network (L3) | **Packet** | IP packet |
| Data Link (L2) | **Frame** | Ethernet frame, 802.11 frame |
| Physical (L1) | Bits / Symbols | — |

**Cross-layer decision table (open engineering scenarios):**

| Network requirement | L3 (Network) | L4 (Transport) | L2 (Data Link) |
|---|---|---|---|
| Dynamic / ad-hoc / mobile | AODV (reactive) | UDP | CSMA/CA or BMAC (asynchronous) |
| Stable IoT mesh | IPv6 via 6LoWPAN | UDP + CoAP | TSMP (TDMA, energy-optimal) |
| Live multimedia / streaming | IP (best-effort) | RTP over UDP | 802.11 / Switched Ethernet |
| Reliable bulk transfer | OSPF (intra-AS) / BGP (inter-AS) | TCP (flow + congestion control) | Switched Ethernet (full-duplex) |
| Sparse long-range sensing | IPv6 via 6LoWPAN | UDP + Stop-and-Wait | LoRa (ALOHA / Class B) |

---

## 1. Application Layer

### 1.1 DNS: Hybrid Resolution Architecture

**Why combine recursive + iterative?**

DNS uses neither model in pure form — it combines them so that each component does the task suited to its position. The table shows why both extremes don't scale:

| Aspect | Purely Iterative | Purely Recursive | Hybrid (actual) |
|---|---|---|---|
| Client complexity | High — client chases referrals itself | Low — one question, one answer | Low |
| Load on root servers | High — every client starts at root | Exploding — server must resolve everything | Low — only stateless referrals |
| Caching | No shared cache | Server caches, but not scalable | Local resolver caches for entire region |
| Scalability | Poor | Poor | Good |

The hybrid model works because each component does what it's best at:
- **Client** → recursive query to local resolver (simplicity)
- **Local resolver** → iterative steps toward root/TLD (shared cache absorbs load)
- **Root servers** → stateless referrals (scalable, anycast)

**Why DNS over UDP?** DNS queries are small, short, one-shot. TCP's three-way handshake per lookup (and per iterative hop) multiplies overhead. UDP enables anycast: multiple physical servers share one IP for redundancy. If the answer is too large: fallback to TCP.

---

### 1.2 DHCP: Bootstrapping Without IP

**Flow: DISCOVER → OFFER → REQUEST → ACK**

DHCP has a chicken-and-egg problem: the client needs an IP to communicate, but must communicate to get an IP. That's why every step is broadcast — the client can't do unicast yet:

| Phase | Direction | Source IP | Destination IP | Why broadcast? |
|---|---|---|---|---|
| DISCOVER | Client → all | 0.0.0.0 | 255.255.255.255 | Client knows neither its own IP nor server IP |
| OFFER | Server → all/client | Server IP | 255.255.255.255 | Client has no IP yet to receive unicast |
| REQUEST | Client → all | 0.0.0.0 | 255.255.255.255 | Also informs rejected servers |
| ACK | Server → client | Server IP | Client IP or broadcast | Confirms lease |

After ACK: client performs Duplicate Address Detection via gratuitous ARP. On conflict: DHCPDECLINE → restart.

**Why TCP is fundamentally impossible:** TCP requires a three-way handshake with valid source IP and destination IP. During DISCOVER, the client has neither. TCP is unicast and connection-oriented; DHCP needs broadcast. UDP allows sending from 0.0.0.0 to 255.255.255.255 without prior state.

**Lease time trade-off (Hughes focus):** short leases → efficient reuse of scarce addresses, more server load. Long leases → less overhead, but addresses remain blocked by inactive hosts.

---

### 1.3 FTP: NAT Incompatibility

**Why two TCP connections?** Control (port 21) stays persistent for the entire session; data (port 20) is set up and torn down per file transfer. This separates commands from data transfer — an abort command isn't blocked by a buffer-full data stream.

**Active FTP fails behind NAT:** the core problem is that FTP embeds IP addresses in the application payload (the PORT command), but NAT only rewrites headers — not payloads. This causes the server to send data to an unreachable private address:
```
Client (private 192.168.1.10) → NAT → Server (public 203.0.113.5)
1. Client → Server:21 (control) ✓ outbound, NAT creates mapping
2. Client sends PORT 192.168.1.10,p  (private address in payload!)
3. Server → 192.168.1.10:p (inbound) ✗ NAT has no mapping
   → packet dropped → data fails
```
Core problem: NAT only maintains state for outbound connections. FTP embeds IP addresses in the payload; NAT doesn't rewrite those.

**Passive FTP (PASV) solution:** server opens a listening port, client initiates data connection outbound → NAT lets this through. Both connections (control + data) are now outbound from the client.

---

### 1.4 Gnutella: Architecture and Espionage

**0.4 vs 0.6 core difference:**

Gnutella evolved from a completely flat network (0.4) to a hierarchical model (0.6) with "superpeers" (ultrapeers). This improves scalability but concentrates knowledge — and thus espionage risk — at those ultrapeers:

| | 0.4 (flat) | 0.6 (hierarchical) |
|---|---|---|
| Topology | All peers equal | Ultrapeers + leaf nodes |
| Search traffic | Flooding via all peers | Concentrated on ultrapeers |
| Scalability | Poor (O(N) messages) | Better |
| Anonymity | Higher (only neighbors know you) | **Lower** — ultrapeer sees everything from its leaves |

**Espionage scenario (open-book favorite):**

*Limited resources (1 node):*
- **0.4 as regular peer:** you only see QUERY/QUERYHIT passing through your direct neighbors. Limited visibility.
- **0.6 as leaf node:** you only see your own traffic. Minimal visibility.
- **0.6 as ultrapeer:** you see all QUERYs from your leaf nodes + neighboring ultrapeers. You know: search terms, IP addresses, file lists, activity patterns.

*Unlimited resources:*
- **0.4:** deploy hundreds of nodes with high degree → see a large portion of flooding traffic
- **0.6:** become ultrapeer at multiple locations → controlling 10% of ultrapeers = significant portion of traffic visible

**Hughes point:** 0.6 buys performance but loses anonymity — copyright surveillance becomes trivial.

**Transport choice:** Discovery (PING/PONG/QUERY) → UDP fits: small, connectionless, flooding-friendly. File transfer → TCP/HTTP: reliable, ordered, complete file required.

**Gnutella on Chord DHT (cross-layer overlay question):**

Chord replaces Gnutella's flooding with a structured DHT with O(log N) lookup via consistent hashing on a circular ring. The idea is that each resource gets a hash that determines which node is responsible — searching becomes deterministic instead of blind forwarding:

| Aspect | Gnutella (flooding) | Gnutella on Chord |
|---|---|---|
| Search overhead | O(N) messages | O(log N) directed hops |
| Search guarantee | No (TTL horizon) | Yes, if the file exists |
| Privacy | Higher (broadly distributed query) | Lower (deterministic path, traceable) |
| Keyword search | Yes (substring matching) | **No** — only exact hash lookups |

**Hash function requirements:** (1) uniform distribution over keyspace (otherwise: hotspots on overloaded nodes), (2) collision-free (otherwise: conflicting resources on one node). Virtual nodes resolve unequal load with imperfect distribution.

**Exam pitfall:** Chord does not support keyword/substring search — only exact file hashes. For text-based search you must keep flooding or build an additional index layer.

---

### 1.5 TOR: Knowledge Distribution and Surveillance

**What does each node know?**

TOR's security revolves around distributing knowledge: no single node in the circuit knows both the sender and the destination. Each layer of encryption is removed by exactly one node (onion model):

| Node | Source IP | Destination | Data | Previous hop | Next hop |
|---|---|---|---|---|---|
| Entry | **Yes** | No | Encrypted (2 layers) | Client | Intermediate |
| Intermediate | No | No | Encrypted (1 layer) | Entry | Exit |
| Exit | No | **Yes** | Plaintext (if no end-to-end encryption) | Intermediate | Server |

**Minimum: 3 nodes (entry + 1 intermediate + exit).** With 2 nodes (entry = exit or direct entry→exit): one node knows both source and destination → complete deanonymization. The intermediate node breaks this direct link.

**Why NOT 10 hops:**
1. Latency: each hop ≈ 100 ms → 10 hops = 1s extra
2. Paradoxically less secure: probability of compromised node = 1 − (1−p)^N. At p=20%: N=3 → 49%, N=10 → 89%
3. Global timing correlation (state actor) is not defeated by extra hops

**Surveillance with limited vs unlimited resources:**
- **Limited:** run exit nodes, log plaintext traffic + traffic pattern analysis. You see content but rarely the sender.
- **Unlimited (state actor):** global timing correlation — observe entry and exit, correlate packet timing/volume end-to-end.

Key takeaway: TOR protects against a local adversary, not against someone who simultaneously observes both ends.

---

### 1.6 HTTP 1.0 vs 1.1

HTTP/1.0 opens a **new TCP connection per object** (handshake + slow-start cost each time). HTTP/1.1 uses **persistent connections** (reuse of one TCP connection) and **pipelining** (send next request without waiting for response).

The advantage is greatest for pages with many embedded objects (modern web pages).

**Cross-layer insight (Hughes favorite):** HTTP/1.0 is "connectionless/stateless" at the application layer but runs over connection-oriented TCP. An application protocol can be connectionless while its transport is connection-oriented.

---

### 1.7 P2P Chat App: Overlay Design Choice

A P2P chat app has multiple functions (login, search, chatting, file sharing) that each have different requirements. The key is to choose the overlay per phase that best matches the traffic pattern:

| Phase | Best overlay | Why |
|---|---|---|
| Login / presence (who is online) | Napster-style central or Gnutella 0.6 superpeers | Reliable, queryable directory needed; flooding is wasteful for presence |
| Peer discovery / contact lookup | Gnutella 0.6 | Superpeers route lookups efficiently; leaves stay lightweight |
| 1-on-1 chat | Direct connection (like Gnutella HTTP transfer) | Minimal latency, no overhead |
| Large file sharing to group | BitTorrent (swarming, chunked) | Scalable for popular content |

Practical side issue: NAT traversal via PUSH/relay mechanism.

---

### 1.8 NAT: Problems at Application Level

NAT operates at layer 3/4 (rewrites IP/port in headers) but does not inspect the application payload. Protocols that carry their own addresses in the payload therefore break behind NAT, and incoming connections are blocked because no mapping exists.

**Two traversal techniques:**
1. **Client-initiated connections** — FTP passive mode, Gnutella PUSH
2. **Relays/helpers** — STUN/TURN (discover public mapping or relay via server), ALG (NAT rewrites known payloads), UPnP port mapping

---

## 2. Transport Layer

### 2.1 RTP vs TCP

TCP and RTP appear superficially similar (both number data), but their design goals differ fundamentally: TCP guarantees everything arrives, RTP accepts loss but demands correct timing:

| Property | TCP | RTP |
|---|---|---|
| Sequence numbers | 32-bit, counts **bytes** | 16-bit, counts **packets** |
| Numbering purpose | Ordering, retransmission, SACK | Loss detection, reordering for playout buffer |
| Timing | Implicit via RTT estimation | Explicit 32-bit timestamp (media units) |
| Retransmission | Always | Never — a late frame is worthless |
| Congestion control | Yes (AIMD) | No — runs on UDP |
| Multicast | No | Yes |

**Downloading video (not streaming):** TCP is the correct choice. A download requires all bytes — a missing GOP makes the file undecodable. RTP's loss tolerance is a feature for live media, not for stored content.

**High-volume RTP congestion problem:**
- **TCP starvation:** TCP halves on loss (AIMD); RTP/UDP sends constantly → TCP structurally gets less bandwidth
- **Congestion collapse:** RTP has no feedback loop → continuously sends useless packets into full queues
- **No ACK clock:** TCP's self-clocking smooths injection based on bottleneck; RTP lacks this → burst behavior

---

### 2.2 Sliding Windows: Formulas and Application

The variable `a` is the ratio between propagation delay and transmission time — the larger `a`, the more time the line sits "empty" while waiting for an ACK. This determines whether a sliding window is needed:

**Core formulas:**
```
a = propagation_delay(one-way) / transmission_time = (RTT/2) / T_trans
T_trans = (packet_size × 8) / link_speed

Stop-and-Wait:    η = 1 / (1 + 2a)
Go-Back-N:        η = min(W / (1 + 2a), 1)     requires: W ≥ 1 + 2a for η = 100%
Selective Repeat: η = min(W / (1 + 2a), 1)     requires: W ≥ 1 + 2a for η = 100%
```

**When Stop-and-Wait is efficient:** when a ≈ 0, i.e., bandwidth-delay product < one packet size.

| Medium | a-value | S&W efficiency | Justification |
|---|---|---|---|
| GEO satellite (RTT 2600 ms, 10 Mbps, 1700 B) | ~956 | 0.052% | Catastrophic — window of 1913 packets needed |
| LoRa SF12 (RTT 134 μs, 250 bps, 50 B) | ~0.00004 | 99.99% | T_trans >> T_prop → S&W optimal |
| Serial 9600 bps over 1 m cable | ~0 | ~100% | T_trans = 1.42 s vs T_prop = 5 ns |

**Window constraint (Selective Repeat):** window size ≤ ½ of the sequence number range. With larger windows, retransmissions after ACK loss cannot be distinguished from new packets with the same number.

---

### 2.3 TCP Flow Control vs Go-Back-N vs Selective Repeat

TCP combines elements of both textbook mechanisms but is not a pure implementation of either Go-Back-N or Selective Repeat. The three differ mainly in how they respond to loss and what the receiver does with out-of-order data:

| Property | TCP | Go-Back-N | Selective Repeat |
|---|---|---|---|
| Window unit | Bytes (dynamic) | Packets (fixed N) | Packets (fixed N) |
| Window advertisement | Receiver advertises free buffer (WIN field) | Protocol parameter | Protocol parameter |
| Receiver buffers out-of-order | Yes (+ SACK) | No — discards | Yes |
| Retransmission on loss | Only gaps (with SACK) | Lost packet **+ all following** | Only lost packet |
| ACK type | Cumulative + optional SACK | Purely cumulative | Individual per packet |

**Window probe:** if the receiver advertises WIN=0 (buffer full), the window-update ACK can be lost → deadlock. The sender periodically sends a tiny probe to force the receiver to re-advertise its window.

**usable window = min(rwnd, cwnd)** — flow control protects the end host, congestion control protects the network.

---

### 2.4 ACK Clock

The ACK clock is TCP's most elegant self-regulating mechanism: the network itself sets the pace, without the sender needing to know the bottleneck capacity.

```
Sender (1 Gbps) ──► Bottleneck (1 Mbps) ──► Receiver
```

1. Sender bursts packets → bottleneck spaces them out in time
2. Receiver sends ACKs with the same spacing as the bottleneck
3. Sender sends new packets at the rhythm of incoming ACKs → no faster than the bottleneck

Result: self-regulating pace-matching without explicit feedback about network topology. This is TCP's "self-clocking".

---

### 2.5 Nagle + Delayed ACKs: Interaction Problem

**Delayed ACKs:** receiver waits ≤200 ms for return data for piggybacking before sending ACK.
**Nagle:** as long as there is an unacknowledged segment in flight, buffer small data until a full segment OR until an ACK comes back.

**The problem:** both mechanisms are individually logical, but together they create a standoff — each waits for the other:
```
Client sends small request → Nagle: "wait for ACK"
Server receives request → Delayed ACK: "wait 200ms for return data"
→ Systematic 200ms latency per small interaction
```

No permanent deadlock (timer expires), but unacceptable for interactive apps (SSH, games).

**Solutions:** TCP_NODELAY (disable Nagle), TCP_QUICKACK (disable Delayed ACKs). QUIC/HTTP2 avoid this structurally.

**Tinygram syndrome:** 1 byte data → 41 bytes header overhead (IP 20 + TCP 20 + 1 byte). Nagle solves this by buffering small data.

**Silly Window Syndrome (receiver side):** app reads 1 byte at a time → receiver keeps advertising tiny windows → sender sends tiny segments. Clark's algorithm: receiver delays window update until it can accept a full MSS or half its buffer.

---

### 2.6 TCP Tahoe vs Reno

The difference lies exclusively in how they respond to 3 duplicate ACKs (= probably one lost packet). Tahoe reacts as aggressively as with a timeout, while Reno recognizes that 3 dup ACKs means the network is still partially functioning:

| Phase | Tahoe | Reno |
|---|---|---|
| Slow start | cwnd doubles per RTT until ssthresh | Identical |
| Congestion avoidance | cwnd +1 MSS per RTT (linear) | Identical |
| Loss (timeout) | cwnd → 1, ssthresh = ½ previous cwnd, restart slow start | Identical |
| Loss (3 dup ACKs) | Same as timeout: cwnd → 1 | **Fast Recovery:** cwnd → ½, stay in congestion avoidance |

Reno recovers much faster from packet loss by not fully resetting.

**SACK (Selective ACK):** receiver specifies which byte ranges have already been received → sender retransmits only gaps. Backwards-compatible via TCP Options.

**ECN vs RED deployment:** ECN requires support on routers + both endpoints. RED runs only on routers → actually deployed. Hughes point: RED is what's actually used.

---

### 2.7 QUIC

QUIC is a modern transport protocol on top of **UDP in user space**. It solves three structural TCP problems: slow connection setup (handshake + TLS separate), head-of-line blocking (one lost packet blocks all streams), and ossification (TCP is in the kernel, changes take years):

| Property | TCP | QUIC |
|---|---|---|
| Encryption | Optional (TLS separate) | **Mandatory** (TLS built-in) |
| Connection setup | 1-3 RTT (TCP + TLS) | **0-1 RTT** |
| Head-of-line blocking | Yes — one lost packet blocks everything | **No** — independent streams |
| Connection migration | No — bound to IP/port | **Yes** — survives WiFi→4G switch |
| Implementation | OS kernel | User space → faster updates |

Why UDP underneath: middleboxes/NAT understand UDP; user space avoids OS kernel ossification (TCP changes require OS updates everywhere).

**QUIC does have congestion control** (typically CUBIC), conceptually comparable to Reno/Tahoe but per-connection and decoupled from kernel TCP.

QUIC is **not** an RTP replacement: no unreliable mode for loss-tolerant real-time media.

---

### 2.8 Congestion Notification

The core problem is: how do you tell the sender the network is filling up? There are three approaches, each with a different trade-off between overhead and deployability:

| Mechanism | How it works | Overhead | Speed | Deployment |
|---|---|---|---|---|
| **Choke Packets** | Router sends separate packet to sender | High — extra packets in congested network | Fast (1 RTT) | Sender only |
| **ECN** | Router marks bit in existing packet; receiver echoes via ECE | None — reuses existing packets | Medium (~1 RTT via receiver) | Routers + both endpoints |
| **RED** | Router drops packets probabilistically before queue is full | Negative — useless retransmissions | Fast — TCP interprets loss directly | **Routers only** → actually deployed |

RED: drop probability increases with average queue length. Heavy senders have more packets in the queue → are hit proportionally more often → fair.

---

### 2.9 TCP Connection Management and Timers

**Three-way Handshake (setup):**

The three steps are needed because both sides must communicate and confirm each other's initial sequence number (ISN). With only two steps, the server wouldn't know whether the client received its SYN-ACK:

```text
Client              Server
  |--- SYN (seq=x) --->|       Client chooses initial sequence number
  |<-- SYN-ACK (seq=y, |       Server chooses own ISN, confirms x+1
  |    ack=x+1) -------|
  |--- ACK (ack=y+1) ->|       Connection open, data can ride in this segment
```

Why 3 steps: both sides must know and confirm each other's ISN. 2 steps is insufficient — the server then doesn't know if the client received its SYN-ACK (half-open connection).

**Connection Teardown (4-way):**

```text
A --- FIN ---> B       A wants to close (half-close)
A <-- ACK --- B        B confirms, can still send data
A <-- FIN --- B        B done, also wants to close
A --- ACK ---> B       Fully closed
```

A enters TIME_WAIT (2×MSL) to catch late duplicates.

**TCP Timers:**

TCP has multiple timers to handle situations where packets or ACKs are lost — without these timers the connection could end up in a deadlock:

| Timer | Function | Why needed |
|---|---|---|
| **Retransmission (RTO)** | Retransmit on missing ACK | Adaptive: RTO = smoothed RTT + 4×variance. Too short → unnecessary retransmits; too long → slow recovery |
| **Persistence** | Window probe at WIN=0 | Prevents deadlock if window-update ACK is lost (see 2.3) |
| **Keep-alive** | Detects dead connections | Optional; periodically sends probe if no data for a long time — prevents half-open connections |
| **TIME_WAIT (2×MSL)** | After close: wait until all segments from old connection have expired | Prevents a new connection on the same port from accepting old segments |

---

## 3 — Network Layer

### 3.1 IPv4 Fragmentation vs Path MTU Discovery

When a packet is larger than the MTU of the next link, the router must choose: silently split it (fragmentation) or warn the sender so it sends smaller packets (PMTUD). The DF bit in the IP header determines which strategy applies:

| DF bit | Router behavior with oversized packet | Feedback to sender |
|---|---|---|
| DF = 0 | Router silently splits packet | None — sender never learns path MTU |
| DF = 1 (PMTUD) | Router drops packet, sends ICMP Type 3 Code 4 | Yes — with bottleneck MTU |

They are **incompatible**: fragmentation hides the MTU mismatch, PMTUD reveals it. You cannot simultaneously fragment and discover the MTU.

**PMTUD Black Hole:** firewalls that block ICMP → sender never receives MTU info → connection "hangs" (SYN-ACK succeeds, data too large, no error message). Solution: RFC 4821 (Packetization Layer PMTUD — gradually tries smaller segments via TCP without ICMP dependency).

**Fragmentation vs segmentation (exam pitfall):**

- Fragmentation (network layer): router splits oversized packet → reassembly at **destination**, not at next router
- Segmentation (transport layer): TCP divides byte stream into segments <= MSS **before** sending

**IPv6:** fragmentation by routers is **forbidden**; PMTUD is mandatory. This simplifies routers (no reassembly state needed) and makes forwarding faster.

---

### 3.2 IPv6 SLAAC and EUI-64 Privacy

SLAAC lets a host give itself an IPv6 address without a server, by embedding the MAC address in the address. The problem: this coupling makes the address permanently trackable across networks.

SLAAC derives the interface ID from the MAC address via EUI-64: MAC `00:1A:2B:3C:4D:5E` → insert `FFFE`, flip U/L bit → `021A:2BFF:FE3C:4D5E`.

**Privacy problems:**

1. **Permanent tracking:** MAC is hardware-permanent → same interface ID on every network → trackable across locations
2. **Address predictability:** knowledge of MAC → IPv6 address predictable on any network
3. **OUI leakage:** first 24 bits of MAC = manufacturer → device type revealed

**Countermeasures:** RFC 7217 (opaque, stable addresses per network — not derivable), RFC 4941 (temporary privacy addresses — rotate periodically).

**SLAAC vs DHCPv6:** SLAAC leaks more (host derives address from hardware, no central control). DHCPv6 can randomize, keeps logs, fits in a managed network. SLAAC suits IoT/always-on devices that must autoconfigure without a server.

**Recognition:** an address with `ff:fe` in the middle is the EUI-64 fingerprint → probably SLAAC.

---

### 3.3 IPv6: No Header Checksum — Why?

**Reasoning (non-trivial):** error detection already happens at two other layers:

- **Link layer:** Ethernet FCS, 802.15.4 CRC — detects bit errors per hop
- **Transport layer:** TCP/UDP checksum — end-to-end integrity check

An IPv6 header checksum would be **redundant**. But the crucial argument is **performance**: a router decrements the hop limit for every packet. With a header checksum, recalculation would be needed at **every hop** → slows forwarding on high-speed routers processing millions of packets per second. IPv4 has this problem: the header checksum must be recalculated per hop because of the TTL field.

---

### 3.4 IoT and the IP Stack (6LoWPAN)

IEEE 802.15.4 max frame: **127 bytes**. This makes standard IP headers problematic — an IPv6 header alone consumes almost a third of the frame. The question is: IPv4 or IPv6 for IoT, and how do you make it workable?

| Barrier | IPv4 | IPv6 |
|---|---|---|
| Header | 20-60 bytes | 40 bytes fixed (1/3 of frame!) |
| Address space | 32-bit, exhausted, requires NAT | 128-bit, sufficient |
| Autoconfiguration | DHCP (server required) | SLAAC (serverless) |
| Fragmentation | By routers (expensive) | Forbidden by routers; min MTU 1280 bytes |
| End-to-end | NAT breaks direct M2M | No NAT needed |

**Winner: IPv6**, but not unmodified. **6LoWPAN** is the adaptation layer:

- Compresses 40-byte IPv6 header to **2-3 bytes** (link-local prefix and EUI-64 interface ID are derivable → don't send them)
- Handles fragmentation across multiple 802.15.4 frames (own fragmentation header, not IPv6)
- SLAAC enables serverless mesh networks

**Why not just IPv4 with a smaller header?** NAT breaks machine-to-machine communication, DHCP requires infrastructure, and the address space problem makes IPv4 unsustainable for billions of IoT devices.

---

### 3.5 AODV vs OSPF + Count-to-Infinity

OSPF and AODV represent two opposite philosophies: OSPF precomputes all routes (ready when needed, but expensive overhead), while AODV only searches for a route when data actually needs to be sent (economical, but higher initial latency):

| | OSPF (proactive, link-state) | AODV (reactive, distance-vector) |
|---|---|---|
| Routes | Continuously maintained, complete topology | On-demand, only during active communication |
| Overhead with no traffic | High — constant flooding of link-state info | **None** — no control traffic |
| Memory per router | O(N) — complete topology view | Minimal — only active routes |
| Suited for | Stable networks (campus, enterprise) | Dynamic ad-hoc (mobile, sensor mesh) |

**Count-to-Infinity (Distance Vector):**
"Good news travels fast, bad news travels slowly." On link failure, a neighbor node advertises the old route → another node adopts it + 1 → loop → distances slowly rise toward infinity.

Mitigations: split horizon, poison reverse, maximum metric (RIP: 16), hold-down timers — but none of these fully solve it.

**AODV's solution:** destination sequence numbers. Route with **higher** sequence number = fresher. On failure: destination increments sequence number, sends RERR. Nodes with stale routes (lower number) reject them **immediately** → no count-to-infinity loop possible.

---

### 3.6 OSPF vs BGP: Intra-AS vs Inter-AS

The Internet is divided into Autonomous Systems (ASes), each with its own management. Within an AS you want the fastest route (OSPF), but between ASes political and commercial interests come into play — BGP chooses routes based on policy, not just distance:

| | OSPF | BGP |
|---|---|---|
| Scope | Within an AS | Between ASes (ISPs) |
| Algorithm | Link-state (Dijkstra) | Path-vector with policy |
| Knowledge per router | Complete area topology | Border routers: reachability + policy |
| Scale | Not for country-scale | Designed for global Internet |
| Routing criterion | Naive shortest-path (cost) | Policy: costs, bandwidth, which ASes to avoid |
| Transport | Directly on IP (protocol 89) | **TCP** (port 179) |

**Why OSPF is connectionless:** floods link-state advertisements to all neighbors, broadcast-style. A TCP session per neighbor would be heavy and pointless for flooding. (Technically: OSPF runs directly on IP protocol 89, not literally on UDP — but it is connectionless in spirit.)

**Why BGP over TCP:** inter-AS sessions are **long-lived** (days/weeks), must be reliable (no lost routing updates between ISPs). TCP provides reliability + ordered delivery + flow control. A lost BGP update can make an entire AS unreachable.

**Peering vs Transit (exam favorite):** peering is the free exchange of traffic between two ASes, typically via an IXP. Peering is **not transitive**: if A peers with B and B peers with C, A cannot reach C for free via B. For that, transit (paid) is needed.

---

### 3.7 Subnetting: Method + Routing Tables + ARP

Subnetting divides an IP range into smaller pieces. The CIDR notation `/n` indicates how many bits form the network portion — the rest are host bits. The method below works systematically for any subnet division:

**Method (works for any mask):**
```
1. CIDR /n  -->  first n bits = network, rest = host
2. Network address    = IP AND mask
3. Broadcast          = network with all host bits = 1
4. First host         = network + 1
5. Last host          = broadcast - 1
6. #usable hosts      = 2^(32-n) - 2
7. Same subnet?       --> (IP_1 AND mask) == (IP_2 AND mask)
8. Private (RFC 1918): 10/8, 172.16/12, 192.168/16
```

**Example network topology (192.168.22.0/24):**

| Subnet | CIDR | Usable range |
|---|---|---|
| LAN A | 192.168.22.0/25 | .1 - .126 (126 hosts) |
| Inter-router | 192.168.22.128/30 | .129 - .130 (2 hosts) |
| LAN B | 192.168.22.192/26 | .193 - .254 (62 hosts) |

**Routing Table Router 1:**

| Destination | Mask | Gateway | Interface |
|---|---|---|---|
| 192.168.22.0 | /25 | directly connected | eth0 |
| 192.168.22.128 | /30 | directly connected | eth1 |
| 192.168.22.192 | /26 | 192.168.22.130 | eth1 |

**ARP Cache after HTTP request PC-A (LAN A) to Webserver (LAN B):**

| Host | Learns via ARP |
|---|---|
| PC-A | Gateway R1 MAC (only!) |
| R1 | PC-A MAC + R2 MAC |
| R2 | R1 MAC + Webserver MAC |
| Webserver | R2 MAC (only!) |

ARP works **only within the same subnet** — PC-A never knows the MAC of the webserver, only that of its gateway.

---

### 3.8 PDU Analysis in Multi-hop Routing

**Core principle — what changes per hop and what doesn't:**

In multi-hop routing it's essential to know which addresses remain constant end-to-end (IP, ports) and which change per link (MAC). This is a common exam question where you must fill in the PDU fields per hop:

```
PC-A ---[R1]---[R2]--- Webserver
```

| Field | Scope | Changes per hop? |
|---|---|---|
| Src/Dst IP (NSAP) | End-to-end | **No** (except with NAT) |
| Src/Dst MAC | Per link | **Yes** — each hop has new src/dst MAC via ARP |
| Src/Dst Port (TSAP) | End-to-end | **No** (except with NAT) |
| TTL/Hop Limit | Per hop | **Yes** — decrements per router |

**After NAT:** private IP/port is rewritten to public IP/port at the NAT boundary. The application payload is **not** rewritten.

**Common mistakes:**

- Default gateway = IP address of the router interface on **your** subnet, NOT the switch (switches are L2, have no IP forwarding)
- Broadcasts do **not** cross routers — a router is by definition a broadcast domain boundary
- A switch learns MAC addresses but does not route — frames to unknown MACs are flooded

---

### 3.9 NAT Problems + Traversal Techniques

NAT rewrites IP/port at the boundary but **not** addresses in application payloads, and blocks incoming connections (no mapping in the NAT table).

**Protocols that break:**

- FTP active mode: client sends `PORT 192.168.1.10,p` (private IP in payload) → server tries inbound connection → NAT blocks
- SIP/VoIP: IP address in SDP payload → peer calls private address → unreachable

**Two categories of traversal techniques:**

| Category | Techniques | Principle |
|---|---|---|
| Client-initiated | FTP PASV, Gnutella PUSH | Reverse the connection direction so everything is outbound |
| Relay/helper | STUN, TURN, ALG, UPnP | External server discovers/relays mapping, or NAT rewrites payload |

**Can a plain router with a public IP replace NAT?** No — sharing a public IP across multiple hosts requires address translation. Without NAT functionality, a router can only serve one host per public IP.

---

### 3.10 Addressing per Layer — Why All Three Are Needed

Each layer has its own address type because each layer solves a different problem. IP routes between networks, MAC delivers on the local link, and port numbers distinguish applications on the same host:

| Layer | Address type | Scope | Function |
|---|---|---|---|
| Network (L3) | IP address (NSAP) | Logical, end-to-end | **Routing** between networks |
| Data Link (L2) | MAC address | Physical, next-hop | **Delivery** on the local wire/link |
| Transport (L4) | Port number (TSAP) | End-to-end | **Demultiplexing** to the correct application |

**Why not just IP?** IP addresses are logical and don't change per hop, but the link layer knows nothing about IP. Ethernet/WiFi hardware delivers frames based on MAC. ARP translates IP to MAC **per subnet**. Without MAC, every NIC would have to process every frame at the IP level → inefficient.

**Why not just MAC?** MAC addresses are flat (no hierarchy) → not routable. IP addresses are hierarchical (prefix = network) → aggregatable in routing tables. Without IP, every router would need an entry for every device on the Internet.

**Port numbers:** without ports, a host cannot distinguish whether an incoming packet is for the web server (80), SSH (22) or DNS (53). Ports are the demultiplexing mechanism of the transport layer.

---

### 3.11 Zigbee/BLE Mesh Routing to IPv6 Gateway

**Scenario:** constrained devices in a mesh need to send sensor data to an IPv6 gateway (sink).

**Best fit routing: AODV (reactive)** — low overhead, on-demand, no control traffic during silence. But standard AODV was designed for ad-hoc laptops, not for energy-constrained sensors.

**Necessary modifications:**

- **Energy-aware metric:** choose routes via nodes with more battery, not purely shortest-path
- **Fewer HELLOs:** reduce beaconing frequency → longer sleep periods
- **Longer route lifetimes:** avoid unnecessary route discovery with stable topology

**Gateway as sink — RPL (DODAG):**
For traffic that predominantly flows toward a gateway, RPL (Routing Protocol for Low-Power and Lossy Networks) is more suitable than AODV. RPL builds a Destination-Oriented Directed Acyclic Graph (DODAG) with the gateway as root. Advantages: structurally directed toward the sink, possibility for data aggregation along the way, and explicit support for 6LoWPAN.

**Cross-layer MAC choice:** AODV/RPL fits with contention-based MACs (CSMA/CA, BMAC) — not with TDMA/TSMP that assume a fixed schedule (see 4.8).

---

## 4 — Data Link & Physical Layer

### 4.1 CSMA/CA: Hidden/Exposed Terminal + RTS/CTS + NAV

CSMA/CA listens locally to the channel before transmitting, but in wireless networks that's not enough — two nodes that can't hear each other can transmit simultaneously and cause a collision at an intermediate receiver.

**Hidden Terminal:**
```
A ←→ B(AP) ←→ C     (A and C can't hear each other)
A: "channel free" → transmits | C: "channel free" → transmits → collision at B
```
**Why carrier sense fails:** A and C listen locally, but their ranges don't overlap. The channel appears free to both, while B receives both signals simultaneously and can't decode either.

**Exposed Terminal:**
```
A ←→ B ←→ C ←→ D    (B transmits to A; C wants to send to D)
C hears B → waits unnecessarily, while C→D wouldn't cause a collision
```
**Why this is wasteful:** C's transmission to D wouldn't disturb B's reception (A and D are in opposite directions), but carrier sense prohibits transmitting because the channel "sounds" busy.

**CSMA solves neither** — it only listens locally. WiFi also can't reliably detect collisions while transmitting (own signal dominates) → that's why Collision **Avoidance** instead of Detection.

**NAV (Network Allocation Vector):** virtual carrier sense. Node reads Duration field in 802.11 header → sets timer → doesn't transmit until timer = 0. Works even if you can't hear the data signal — purely based on overheard control frames.

**RTS/CTS solves hidden terminal:**
```
1. A → B: RTS (Duration = data + CTS + ACK)
2. B → all: CTS (Duration = data + ACK) ← C hears this!
3. C sets NAV → stays silent
4. A → B: DATA (collision-free)
5. B → A: ACK
```
C may not have heard A's RTS (hidden), but does hear B's CTS → knows the channel is busy.

**RTS/CTS does NOT solve exposed terminal** — actually worsens it: C hears B's RTS and unnecessarily sets NAV, causing C to wait even longer than with carrier sense alone.

**When to use RTS/CTS:** only for large frames. The overhead of 2 extra frames per transmission is only justified when the cost of a collision on a large data frame exceeds the RTS/CTS overhead. Default RTS threshold = 2347 bytes (often disabled).

---

### 4.2 BMAC vs TSMP: Preamble/Sampling Trade-off + Fusion

BMAC and TSMP are two opposite strategies for saving energy on sensor networks. BMAC lets nodes sleep and wakes them with a long preamble (good for sporadic traffic), while TSMP gives nodes exact time slots to be awake (good for predictable traffic):

| Feature | BMAC (LPL) | TSMP (TDMA + channel hopping) |
|---|---|---|
| Traffic | Unpredictable, event-driven | Predictable, periodic |
| Topology | Dynamic, nodes come/go | Stable, known participants |
| Synchronization | Not required | Required (clock master) |
| Latency | Low (transmit immediately) | Higher (wait for assigned slot) |
| Energy (low traffic) | Good (radio mostly off) | Excellent (slot = precisely planned wake) |
| Energy (high traffic) | Poor (long preamble per packet) | Good (no preamble overhead) |
| Scalability | Good (decentralized) | Limited (central slot assignment) |
| Largest energy cost | Preamble transmission | (Re)joining the network |

**Preamble/Sampling trade-off (core formula):**
```
Preamble_length ≥ Sample_Interval
```
**Why:** the receiver periodically samples the channel briefly. The preamble must last at least as long as the sample interval so the receiver is guaranteed to detect activity during a sample moment.

| Configuration | Sample interval | Preamble | Result |
|---|---|---|---|
| **High traffic** | Short | Short | Low per-packet overhead, more sampling energy |
| **Low traffic** | Long | Long | Maximum sleep time, expensive but rare transmissions |

**BMAC+TSMP fusion design:** use TDMA slots as backbone for steady/periodic traffic (camera feeds, temperature sensors). Reserve an LPL window for event-driven/unexpected messages and new nodes. New nodes join via BMAC (no schedule needed), and switch to TDMA after synchronization. Synchronized slots prevent both regimes from colliding.

**Exam pitfall:** TSMP offers not only TDMA but also **channel hopping** (spreads interference risk across frequencies) and **redundancy** (spatial: different neighbor, temporal: different time). BMAC doesn't offer QoS differentiation — TSMP can allocate more slots to a camera than to a temperature sensor.

---

### 4.3 Hub vs Switch: Architecture, Collision Domains, Privacy

A hub is a "dumb" device that sends everything to everyone (L1 repeater), while a switch intelligently forwards frames based on MAC addresses (L2). This difference has direct consequences for collisions, bandwidth, and privacy:

| Aspect | Hub | Switch |
|---|---|---|
| Operation | Physical repeater, copies signal to all ports | Learns MAC addresses (MAC table), forwards per port |
| Collision domain | One shared domain (all ports) | Separate domain per port |
| Duplex | Half-duplex (CSMA/CD required) | Full-duplex possible (no collisions) |
| Bandwidth | Shared across all ports | Dedicated per port |
| Privacy | Everyone sees all traffic | Only your own unicast traffic |
| Cost/complexity | Cheap, no logic | More expensive, forwarding logic |

**When is a hub better than a switch?** Network sniffing, protocol analysis, lab education — you want all hosts to see all traffic via promiscuous mode. With a switch this is only possible with **port mirroring** (SPAN), which requires configuration.

**Privacy comparison across media:**

| Medium | Eavesdropping ease | Protection |
|---|---|---|
| Hub Ethernet | Trivial: promiscuous mode on any port | Physical access control |
| Switched Ethernet | Difficult, but MAC flooding/spoofing can force switch into hub mode | Port security, 802.1X |
| 802.11 WiFi | Inherently sniffable (radio waves for everyone) | Entirely dependent on encryption (WPA2/3) |

---

### 4.4 Collision Domains and Cable Length (CSMA/CD)

**Core requirement:** if a frame has already been fully transmitted before the collision signal returns, the sender never notices the collision. That's why there's a minimum frame size that depends on cable length and link speed:

CSMA/CD only works if the sender detects a collision **while still transmitting**:
```
T_transmission ≥ 2 × T_propagation
```
The minimum frame size (64 bytes = 512 bits) is chosen to satisfy this requirement.

**Calculation example 10 Mbps:** T_tx = 512 bits / 10 Mbps = 51.2 μs → max cable = 51.2 μs × 200 m/μs / 2 ≈ **2560 m**.
**100 Mbps with hub:** T_tx = 512 bits / 100 Mbps = 5.12 μs → max cable ≈ **256 m** (10x stricter!).

| Configuration | CSMA/CD | Cable limitation |
|---|---|---|
| Classic 10 Mbps, hub | Yes | ~2500 m (collision timing) |
| Fast 100 Mbps, hub | Yes | ~256 m (**stricter**) |
| Switched, half-duplex | Yes (per port) | Per port |
| Switched, **full-duplex** | **No** | **Only attenuation** (~100 m Cat5e) |

**Why full-duplex removes the limit:** transmitting and receiving on separate wire pairs → collisions physically impossible → CSMA/CD disabled → the collision-driven cable limit disappears. The only remaining limit is signal attenuation (~100 m for Cat5e).

**Gigabit Ethernet:** at 1 Gbps, T_tx = 5.12 ns (unworkably short). Solution: **carrier extension** artificially extends short frames to 512 bytes (4096 bits) + **frame bursting** for efficiency. At 10G: full-duplex only, no hubs, CSMA/CD unnecessary.

---

### 4.5 802.11 Packet Loss as TCP Congestion Signal

**Core problem:** TCP was designed for wired networks where packet loss almost always means congestion. On WiFi that assumption doesn't hold — most losses are radio errors, not full queues. TCP interprets **all** packet loss as congestion → halves cwnd (AIMD). On WiFi, most losses are due to:

- Radio interference (other networks, microwaves, Bluetooth)
- Hidden terminal collisions
- Signal weakness/multipath fading
- Exhausted MAC-layer retransmissions (802.11 internally retries up to 7x)

None of these is congestion. TCP "punishes" itself unnecessarily → throughput drops dramatically on a non-congested network.

**Solutions:**

| Approach | Layer | How it works |
|---|---|---|
| **ECN** | Network | Explicit congestion signal, no confusion with link errors |
| **Data-link ARQ** | Data link | 802.11 ACKs + retransmission mask link loss locally |
| **TCP Westwood** | Transport | Estimates available bandwidth via ACK timing instead of loss |

**Cross-layer insight:** data-link retransmission (hop-by-hop, low latency) and transport retransmission (end-to-end, higher latency) are complementary. Local recovery on a lossy wireless hop prevents TCP from misinterpreting the loss as congestion. This is why 802.11 provides **acknowledged connectionless** service (as opposed to Ethernet's unacknowledged connectionless).

---

### 4.6 LoRa: Limitations, ALOHA Analysis, MAC Improvements

LoRa is designed for long range with minimal power, but pays for this with extremely low data rates. The combination of long airtime and regulatory duty-cycle restrictions makes MAC design crucial here.

**Characteristics:** range ~20 km, data rate 250 bps (SF12) – 50 kbps (SF7), T_prop ≈ 67 μs (negligible compared to airtime).

**1. Duty Cycle (regulatory):** ISM 868 MHz, max 1% duty cycle per channel.
At SF12, 50 bytes → T_airtime ≈ 1.6 s → wait time = 1.6 / 0.01 = **160 s**. Effective throughput ≈ **2.5 bps**.

**2. Collision analysis (Pure ALOHA):**
```
P(collision) = 1 - e^(-2 * lambda * T_airtime)
```
100 nodes, each 1 pkt/min → lambda = 100/60 ≈ 1.67 pkt/s, T_airtime = 1.6 s → P ≈ **99.5%** collision. LoRa fundamentally doesn't scale with Pure ALOHA at high node density.

**3. Sliding Window choice:** a = T_prop / T_frame = 67 μs / 1600 ms ≈ 0 → Stop-and-Wait efficiency ≈ 99.99%. S&W is optimal for LoRa (no benefit from sliding window).

**4. Hidden terminal:** nodes 35+ km apart can't hear each other but share the same gateway → carrier sense (CSMA) impossible at this scale → star topology with central gateway + ALOHA-like access is inevitable.

**MAC improvements to mitigate ALOHA problems:**

| Technique | What it solves |
|---|---|
| TDMA/slotting (Class B) | Avoids blind collisions through time slots |
| CSMA/Listen-Before-Talk | Carrier sense where possible (short range) |
| Channel hopping | Combats narrowband interference |
| Adaptive Data Rate | Shortens airtime where signal is strong enough → smaller collision window |
| Duty-cycle coordination | Spreads transmission moments across gateways |
| ACKs + retransmission | Reliability for critical data |

| Scenario | Recommended protocol | Justification |
|---|---|---|
| Single node, light traffic | Stop-and-Wait | Simplicity, efficiency ≈ 100% |
| Dense (>50 nodes) | TDMA (Class B) | Avoids collisions |
| Mobile nodes | ALOHA (Class A, best effort) | No synchronization possible |
| Critical data | Confirmed (Class A + ACK) | Retransmission on loss |

---

### 4.7 Pure ALOHA vs Slotted ALOHA

Pure ALOHA lets everyone transmit whenever they want — simple but inefficient. Slotted ALOHA forces all senders to start at fixed time boundaries, halving the time window in which a collision can occur:

| Feature | Pure ALOHA | Slotted ALOHA |
|---|---|---|
| When to transmit | At any arbitrary moment | Only at slot boundaries |
| Vulnerable period | 2 × frame time | 1 × frame time |
| Max channel efficiency | 1/(2e) ≈ **18%** | 1/e ≈ **37%** |
| Synchronization needed | No | Yes (global clock) |

**Why slotting doubles throughput:** in Pure ALOHA, a frame can collide with any frame that starts in the 2T period around the own frame. By forcing all frames to start at slot boundaries, a frame can only collide with frames in the **same** slot → vulnerable period halves → double maximum throughput.

**Trade-off:** slotting requires time synchronization. In networks where synchronization is difficult or expensive (large distances, no central clock), Pure ALOHA may be the only option.

**Can Pure ALOHA work on 802.11?** Physically possible (same radio), but a poor choice: 802.11 deliberately uses CSMA/CA with collision avoidance, backoff, and RTS/CTS. ALOHA's blind transmissions waste a shared radio channel, ignore hidden terminals, and provide no ACK mechanism.

---

### 4.8 Framing: Byte-Stuffing vs Bit-Stuffing

**Problem:** the receiver receives a continuous bit stream and must know where a frame begins and ends. A special flag pattern marks those boundaries, but if that same pattern happens to appear in the data, the receiver sees a false frame end. Both methods solve this by making the flag pattern impossible to appear in the data:

**Byte-stuffing:** flag byte marks boundaries. For every flag or ESC in the data: insert an ESC byte before the relevant byte. Simple to implement, but expensive when the flag value occurs frequently.

**Bit-stuffing:** flag = `01111110`. Rule: after 5 consecutive 1-bits in data, automatically insert a 0. Receiver removes every 0 after 5 ones. The flag pattern (six consecutive ones) can thus never accidentally appear in the payload.

**Calculation example — byte 0xFF (11111111):**

| Method | Result | Overhead |
|---|---|---|
| Bit-stuffing | 11111**0**111 = 9 bits | +1 bit |
| Byte-stuffing | ESC + 0xFF = 16 bits | +8 bits (entire ESC byte) |

Bit-stuffing is **8x more efficient** here. The more flag-like patterns in the data, the greater the advantage of bit-stuffing.

**Alternative — byte count:** length field at the start specifies frame length. Compact, but fatally weak: if the length field gets corrupted, the receiver loses synchronization completely and a checksum won't save it (you no longer know what it covers).

---

### 4.9 Ethernet Frame Padding + Minimum Frame

**Why 64 bytes minimum:** CSMA/CD requires that the sender is still transmitting when the collision signal returns from the farthest point on the network. At 10 Mbps, max ~2500 m cable:
```
T_propagation (round trip) ≈ 51.2 us
T_transmission (64B = 512 bits at 10 Mbps) = 51.2 us  -->  exactly equal
```
If the payload is shorter than needed, the frame is **padded** with fill bytes up to 64 bytes (including header + CRC). Without padding, a short frame would already be fully sent before the collision returns → collision undetectable.

**Gigabit Ethernet:** at 1 Gbps, 64 bytes would only take 0.512 μs → far too short. Solution: **carrier extension** effectively increases the minimum to 512 bytes. **Frame bursting** allows sending multiple short frames back-to-back within a single carrier session, so the carrier extension overhead is shared.

---

### 4.10 Stop-and-Wait: When Good/Bad

**Throughput:** efficiency ≈ 1 / (1 + 2a), where a = T_propagation / T_frame.

Stop-and-Wait is efficient when **bandwidth-delay product < frame size** (a ≈ 0).

| Scenario | RTT | Frame | Throughput | Verdict |
| --- | --- | --- | --- | --- |
| Short Ethernet (100m) | ~10 μs | 1500 B | ~1.2 Gbps | **Good** (a ≈ 0) |
| LoRa (SF12, 20 km) | ~134 μs | 1600 ms airtime | ~2.5 bps | **Good** (a ≈ 0) |
| Satellite (GEO) | ~540 ms | 1000 B | ~15 kbps | **Bad** (a >> 1) |
| Trans-Atlantic | ~80 ms | 1500 B | ~150 kbps | **Bad** |

**Why bad with high RTT:** the sender waits idle for an ACK while the pipeline is empty. The link is unused most of the time. Solution: **sliding window** (Go-Back-N or Selective Repeat) fills the pipeline.

**Method for exam questions:** (1) estimate RTT of the medium, (2) calculate a = T_prop / T_frame, (3) if a << 1: S&W suffices, if a >> 1: sliding window needed.

---

### 4.11 Null MAC: Problems Under Load

**Definition:** Null MAC is the "do nothing" protocol — no carrier sense, no backoff, no retransmission. It's the theoretical lower bound against which you compare other MAC protocols to show why each mechanism is needed:

| Situation | Problem |
|---|---|
| High traffic | Many collisions, no exponential backoff to spread load, no recovery → **goodput collapses** to zero |
| Densely populated (city center) | Many nodes, much interference, no mechanism to wait → near-zero usable throughput |
| Light traffic, few nodes | Only scenario where Null MAC is workable — collisions are rare |

**Why this is an exam topic:** it illustrates that MAC protocols are not optional. Each mechanism (carrier sense, backoff, retransmission) solves a specific problem. Null MAC = worst-case baseline to compare other protocols against.

---

### 4.12 802.11 Power Saving (Two Strategies)

WiFi clients that always listen consume a lot of power. 802.11 offers two power-saving strategies that differ in who determines the wake schedule — the access point or the client itself:

| Strategy | Mechanism | Use case |
|---|---|---|
| **Beacon + TIM / PS-Poll** | AP periodically sends beacon with Traffic Indication Map. Sleeping client wakes only for beacons, checks if it's flagged, sends PS-Poll to retrieve buffered data | Battery-powered device that's mostly idle (sensor, phone in standby) |
| **APSD (Automatic Power Save Delivery)** | Client wakes only when it wants to send (fixed intervals). AP delivers buffered downlink data in the same wake window | Periodic reporting (VoIP, telemetry) — predictable timing |

**Core difference:** with Beacon+TIM, the **AP** determines when the client can retrieve (beacon interval). With APSD, the **client** determines the schedule — more efficient for predictable applications.

**Cross-layer link:** compare with BMAC (client-side sampling) vs TSMP (network-controlled schedule). The same trade-off: who controls the wake schedule, and how predictable is the traffic?

---

### 4.13 Cross-layer: MAC Protocol Choice with AODV

AODV is reactive/on-demand: routes are only discovered when needed (RREQ flooding). Topology is dynamic — nodes come and go. This requires a MAC that **can transmit at any moment without a pre-configured schedule**. The MAC protocol choice must match this dynamic character — a scheduled MAC clashes with AODV's philosophy:

| MAC | Fits AODV? | Why |
|---|---|---|
| **CSMA/CA** | **Good** | Transmits when needed, no schedule, tolerates nodes that come/go |
| **BMAC** | **Good** | Same + energy savings via LPL; asynchronous, no central coordination |
| **TSMP/TDMA** | **Poor** | Requires pre-established schedule + time synchronization → clashes with AODV's dynamic route discovery |
| **Pure ALOHA** | **Poor** | No carrier sense, no backoff → high collision rate during RREQ flooding |

**Core reasoning:** scheduled MACs (TSMP/TDMA) assume a stable topology with known participants. AODV assumes the opposite. A topology change with TDMA forces expensive rescheduling, while with CSMA/CA or BMAC a new node can simply start transmitting.

**Why Pure ALOHA is also poor:** AODV's route discovery uses flooding (RREQ to all neighbors). Without carrier sense or backoff, this leads to massive collisions — precisely when the network needs communication most.
