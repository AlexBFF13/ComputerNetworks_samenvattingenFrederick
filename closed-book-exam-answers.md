# Computer Networks — Closed Book Exam Answers

> **Purpose:** Essential answers to the most frequently asked closed-book exam topics, organized by layer.
> **Sources:** All answers based exclusively on the course summaries (application, transport, network, data link layers).
> **Exam periods analyzed:** 2013–2025 (14 exam moments).

---

## Table of Contents

- [1. Application Layer](#1-application-layer--closed-book)
  - [1.1 Gnutella 0.4 / 0.6](#11-gnutella-04--06--message-types-routing-ttl-and-04-vs-06)
  - [1.2 DNS — Recursive vs Iterative, Why UDP, Split Design](#12-dns--recursive-vs-iterative-why-udp-split-design)
  - [1.3 DHCP — How It Works, Why UDP, Lease/Renew/Decline](#13-dhcp--how-it-works-why-udp-leaserenewdecline)
  - [1.4 OSI / TCP-IP / Tanenbaum Stacks](#14-osi--tcp-ip--tanenbaum-stacks--draw-and-place-protocols)
  - [1.5 TOR — Knowledge Distribution and Minimal Hops](#15-tor--knowledge-distribution-and-minimal-hops)
  - [1.6 HTTP 1.0 vs 1.1](#16-http-10-vs-11--persistent-connections-and-pipelining)
  - [1.7 FTP — Control/Data Connection and Passive Mode](#17-ftp--controldata-connection-and-passive-mode)
  - [1.8 UDP Applications — Why No TCP](#18-udp-applications--why-no-tcp)
- [2. Transport Layer](#2-transport-layer--closed-book)
  - [2.1 TCP Flow Control — Window, Deadlock, Probes](#21-tcp-flow-control--sliding-window-window-advertisement-window-probe-deadlock)
  - [2.2 Silly Window Syndrome + Tinygram + Nagle](#22-silly-window-syndrome--tinygram--nagles-algorithm-delayed-acks-interaction)
  - [2.3 RTP — Features, Header Fields, Multicast](#23-rtp--features-difference-with-tcp-header-fields-multicast)
  - [2.4 TCP Congestion Control — Tahoe/Reno, AIMD](#24-tcp-congestion-control--tahoereno-slow-start-aimd-threshold)
  - [2.5 ECN vs RED](#25-ecn-vs-red--how-ecn-works-advantagesdisadvantages-vs-red)
  - [2.6 QUIC — Design, Difference with TCP](#26-quic--design-difference-with-tcp)
  - [2.7 TCP Header — All Fields](#27-tcp-header--all-fields-explained)
  - [2.8 Sliding Window Protocols — Comparison](#28-sliding-window-protocols--comparison-sequence-number-limitations)
  - [2.9 Retransmission — Why in Transport Layer](#29-retransmission--why-in-the-transport-layer)
- [3. Network Layer](#3-network-layer--closed-book)
  - [3.1 AODV — Reactive vs Proactive](#31-aodv--how-it-works-route_request--route_reply-reactive-vs-proactive)
  - [3.2 Congestion Notifications — RED vs ECN vs Choke](#32-congestion-notifications--red-vs-ecn-vs-choke-packets)
  - [3.3 IP Addressing / Subnetting](#33-ip-addressing--subnetting--broadcast-address-cidr-host-range)
  - [3.4 Distance Vector / Count-to-Infinity](#34-distance-vector-routing--count-to-infinity-problem)
  - [3.5 BGP vs OSPF](#35-bgp-vs-ospf--intradomain-vs-interdomain)
  - [3.6 Round Robin Fair Queuing](#36-round-robin-fair-queuing--how-it-works-abuse)
- [4. Data Link & Physical Layer](#4-data-link--physical-layer--closed-book)
  - [4.1 B-MAC vs TSMP](#41-b-mac-vs-tsmp--when-to-choose-which-lpl-preamble-configuration)
  - [4.2 Hidden / Exposed Terminal Problem](#42-hidden--exposed-terminal-problem--mac-solutions-csmaca-rtscts)
  - [4.3 Pure ALOHA / Slotted ALOHA](#43-pure-aloha--slotted-aloha)
  - [4.4 Switches vs Hubs](#44-switches-vs-hubs)
  - [4.5 Ethernet — Cable Length, Padding, CSMA/CD](#45-ethernet--cable-length-limit-padding-csmacd)
  - [4.6 802.11 Power Saving](#46-80211-power-saving--beacon-frames-apsd-two-strategies)
  - [4.7 MAC Protocols with AODV](#47-mac-protocols-with-aodv--which-work-which-dont)
  - [4.8 Channel Hopping](#48-channel-hopping--advantages)
  - [4.9 Stop-and-Wait Analysis](#49-stop-and-wait-analysis--which-medium-goodbad-throughput-calculation)

---

## 1. Application Layer — Closed Book

---

### 1.1 Gnutella 0.4 / 0.6 — Message Types, Routing, TTL, and 0.4 vs 0.6

Gnutella is a decentralized P2P overlay network built on top of TCP/IP. Unlike Napster (which used a central index server), Gnutella distributes discovery across all participating peers. This removes the single point of failure but introduces significant overhead.

**Gnutella 0.4 message types**

Every peer in the flat overlay participates in forwarding these messages:

| Message | Purpose | Direction | Details |
|---------|---------|-----------|---------|
| `PING` | Peer discovery | Broadcast (flooded) | Sent to find active peers in the overlay |
| `PONG` | Reply to PING | Back along the PING path | Contains IP address, port, and metadata of the responding peer |
| `QUERY` | Resource search | Broadcast (flooded) | Contains a search string; each peer checks locally for matches |
| `QUERYHIT` | Reply to QUERY | Back along the QUERY path | Contains address, port, and file info so the requester can download |
| `PUSH` | Firewall bypass | Via overlay | Asks a firewalled peer to initiate the data connection itself (outbound from behind the firewall) |

**Routing and TTL**

Messages are flooded through the overlay: each peer forwards them to all its neighbors. To prevent unlimited flooding, every message carries a **TTL (Time to Live)**. Each peer decrements the TTL by 1 and drops the message when it reaches 0.

This creates the **search horizon problem**: a TTL that is too low means content may exist but never be found; a TTL that is too high floods the network with unmanageable traffic. There is no good middle ground at large scale.

**File transfer** happens outside the overlay: after a QUERYHIT, the requester opens a direct **HTTP** connection to the peer holding the file. If that peer is behind a firewall, the `PUSH` mechanism is used so the firewalled peer initiates the connection itself.

**Gnutella 0.6 — the super-node architecture**

Gnutella 0.6 introduces a two-tier hierarchy to reduce flooding overhead:

| Role | Description | Requirements |
|------|-------------|--------------|
| **Ultrapeer** | Handles peer discovery, routing, and maintains indices of its leaf nodes' files | No firewall, sufficient bandwidth, uptime, RAM, and CPU |
| **Leaf node** | Connects to one ultrapeer; does minimal discovery work itself | Any device |

Search queries are concentrated among ultrapeers rather than flooded through every single node. Ultrapeers act as proxies for their leaves.

**0.4 vs 0.6 comparison**

| Aspect | Gnutella 0.4 (flat) | Gnutella 0.6 (hierarchical) |
|--------|---------------------|----------------------------|
| Topology | All peers equal | Ultrapeers + leaf nodes |
| Search traffic | Flooding through **all** peers | Concentrated on ultrapeers |
| Scalability | Poor (O(N) messages) | Better |
| Anonymity | Higher (only direct neighbors see your queries) | **Lower** — an ultrapeer sees the search terms, IP addresses, and file lists of all its leaf nodes |

The key trade-off: 0.6 buys **performance** at the cost of **anonymity**. A malicious ultrapeer can observe everything its leaf nodes do. With unlimited resources, an adversary could run many ultrapeers and monitor a significant fraction of all search traffic.

---

### 1.2 DNS — Recursive vs Iterative, Why UDP, Split Design

DNS (Domain Name System) translates human-readable names into IP addresses. It replaced the old centralized `hosts.txt` file, which could not scale because it became a single bottleneck requiring constant coordination.

**Hierarchical structure**

DNS names encode their hierarchy from right to left. For example, `nix.cs.kuleuven.ac.be` reads as: country (`be`) → academic domain (`ac`) → organization (`kuleuven`) → department (`cs`) → host (`nix`). This hierarchy enables delegation: each zone administrator manages their own subtree independently.

The server hierarchy consists of:
- **Root servers** — know the TLD name servers (`.com`, `.be`, etc.)
- **TLD servers** — know the authoritative servers for domains under their TLD
- **Authoritative servers** — hold the actual resource records for a zone

**Important resource record types**

| Type | Meaning |
|------|---------|
| `A` | Maps a name to an IPv4 address |
| `NS` | Identifies the authoritative name server for a domain |
| `MX` | Identifies the mail server for a domain |
| `CNAME` | Canonical name (alias pointing to another name) |

Each record has a **TTL** (Time to Live) that controls how long it can be cached.

**Recursive vs iterative lookup**

- **Recursive**: the client asks a question and expects a complete answer. The queried server does all the work to find the final result.
- **Iterative**: the client asks a question and may receive a partial answer — a referral to another server that might know more. The client (or its resolver) then follows up with that next server.

**The hybrid model (split design) — how DNS actually works**

In practice, DNS uses a combination of both:

1. The **client** sends a **recursive** query to its **local resolver**: "Give me the answer; I do not want to chase referrals myself."
2. The **local resolver** performs **iterative** steps toward root → TLD → authoritative servers: each of those servers returns a referral rather than doing the full lookup.
3. **Root/TLD servers** give only stateless referrals — they never recursively resolve the entire query themselves.

This hybrid is efficient because the local resolver acts as a **shared cache** for its entire region. Once one user looks up `www.google.com`, the answer is cached and subsequent users in the same organization get the answer without hitting root or TLD servers again.

| Aspect | Purely iterative | Purely recursive | Hybrid (practice) |
|--------|-----------------|------------------|-------------------|
| Client complexity | High — client does all steps | Low — one question, one answer | Low |
| Load on root servers | High — every client starts at root | Exploding — root must resolve everything | Low — only stateless referrals |
| Caching | No shared cache | Server caches, but not scalable | Local resolver caches for entire region |
| Scalability | Poor | Poor | Good |

**Why DNS uses UDP (not TCP)**

1. **Queries are small and short**: a typical DNS query fits in a single UDP datagram (< 512 bytes). TCP's three-way handshake would triple the latency for a simple name resolution.
2. **Each iterative step would need a separate TCP connection**: a local resolver often performs 2-4 iterative steps (root → TLD → authoritative). With TCP, that means 2-4 separate handshakes.
3. **Anycast compatibility**: multiple physical root servers share the same IP address. The closest one responds. TCP's connection state would break if packets from the same connection went to different physical servers.
4. **Statelessness**: DNS servers hold no per-client connection state, making them much more scalable — especially root and TLD servers handling millions of queries per second.

**Exception**: DNS falls back to TCP for responses larger than 512 bytes (e.g., with DNSSEC), and zone transfers between servers always use TCP because they require reliability and transfer large volumes of data.

---

### 1.3 DHCP — How It Works, Why UDP, Lease/Renew/Decline

DHCP (Dynamic Host Configuration Protocol, RFC 2131) automatically assigns IP addresses, subnet masks, gateways, and DNS server addresses to hosts. Without DHCP, every device would need manual configuration — unworkable at scale, especially for mobile devices.

**Why DHCP uses UDP and not TCP**

TCP is **fundamentally impossible** for DHCP, not merely inconvenient:

- TCP requires a three-way handshake with a **valid source IP** and a **valid destination IP**. During DHCPDISCOVER, the client has **neither**: it does not yet have its own IP (that is exactly what it is trying to obtain), and it does not know the DHCP server's IP.
- TCP is unicast and connection-oriented — it cannot broadcast. DHCP needs to broadcast from `0.0.0.0` to `255.255.255.255`.
- UDP allows sending from `0.0.0.0` to `255.255.255.255` without any prior state.

**The DHCP flow (DORA)**

The four-step exchange, with broadcast logic explained:

| Step | Direction | Source IP | Destination IP | Why broadcast? |
|------|-----------|-----------|----------------|----------------|
| **DISCOVER** | Client → all | `0.0.0.0` | `255.255.255.255` | Client knows neither its own IP nor the server's IP |
| **OFFER** | Server → all | Server IP | `255.255.255.255` | Client has no IP yet to receive unicast |
| **REQUEST** | Client → all | `0.0.0.0` | `255.255.255.255` | Also informs **rejected servers** that their offer was not chosen, so they can free the reserved address |
| **ACK** | Server → client | Server IP | Client IP or broadcast | Confirms the lease |

After receiving the ACK, the client has a usable IP address.

**Key details**

- The DHCP **message type** (DISCOVER, OFFER, REQUEST, ACK, etc.) is carried in an **option field**, not as a fixed header field. The **Transaction ID** links responses to requests.
- A DHCP address is a **lease** (temporary reservation), not permanent ownership.

**Lease trade-offs**

| Lease duration | Advantage | Disadvantage |
|----------------|-----------|--------------|
| Short (minutes) | Efficient address reuse; addresses freed quickly when hosts leave | More server load from frequent renewals |
| Long (days/weeks) | Less renewal overhead | Addresses stay blocked by hosts that are no longer active |

- **Renewal**: happens when approximately **50%** of the lease time has elapsed. If the server refuses, it sends a `DHCPNAK` and the client must release the address.
- **Release**: a host can voluntarily send `DHCPRELEASE` when it no longer needs the address.
- **Duplicate Address Detection**: after receiving an ACK, the client uses **ARP** to check if the assigned address is already in use on the network. If a conflict is detected, the client sends a `DHCPDECLINE`.

---

### 1.4 OSI / TCP-IP / Tanenbaum Stacks — Draw and Place Protocols

Three protocol stack models are used in the course. The OSI model is a **reference model** (7 layers, never fully implemented as a protocol stack). The TCP/IP model is the **practical model** that powers the internet. The Tanenbaum model is a **didactic hybrid** (5 layers) that takes OSI's clear lower-layer separation but merges the top three OSI layers.

**Side-by-side comparison**

| Layer | OSI (7 layers) | TCP/IP (4 layers) | Tanenbaum (5 layers) |
|-------|---------------|-------------------|---------------------|
| 7 | Application | Application | Application |
| 6 | Presentation | (merged into Application) | (merged into Application) |
| 5 | Session | (merged into Application) | (merged into Application) |
| 4 | Transport | Transport | Transport |
| 3 | Network | Internet | Network |
| 2 | Data Link | Network Access (Host-to-network) | Data Link |
| 1 | Physical | (merged into Network Access) | Physical |

**Protocol placement per layer (common exam cases)**

| Protocol | Layer | Why |
|----------|-------|-----|
| HTTP, SMTP, DNS, DHCP, FTP, SSH | Application | Provide services to end users/applications |
| TCP, UDP | Transport | End-to-end reliability and multiplexing |
| IP, ICMP | Network / Internet | Routing and logical addressing |
| OSPF, BGP | Network | Routing protocols (OSPF runs directly on IP, protocol 89 — not on TCP/UDP) |
| Ethernet, 802.11, PPP | Data Link / Network Access | Hop-to-hop delivery and framing |
| **RTP** | Application (in Tanenbaum) | Runs on top of UDP in user space; adds transport-like features but is not part of the transport layer itself |
| **DHCP** | Application | An application protocol that exchanges configuration data, even though it influences the network layer (IP assignment) and uses UDP |
| **ARP** | Between Network and Data Link | In the TCP/IP model, it is part of the Network Access layer |

**Key exam point**: the answer to "which layer?" depends on which model you use. Always state the model.

---

### 1.5 TOR — Knowledge Distribution and Minimal Hops

TOR (The Onion Router) provides anonymity through **onion routing**: a message is wrapped in multiple layers of encryption, and each intermediate router peels off exactly one layer. This way, each router only learns who gave it the message and who to forward it to — never the full route, the true origin, the final destination, or the full content.

**What each node knows (key exam table)**

| Node | Source IP | Destination | Data content | Previous hop | Next hop |
|------|-----------|-------------|-------------|--------------|----------|
| **Entry node** | **Yes** | No | Encrypted (2 layers remain) | Client | Intermediate |
| **Intermediate node** | No | No | Encrypted (1 layer remains) | Entry | Exit |
| **Exit node** | No | **Yes** | **Plaintext** (if no end-to-end encryption) | Intermediate | Server |

The critical property: **no single node knows both the source and the destination**.

**Why minimally 3 nodes (entry + intermediate + exit)?**

With only 2 nodes, one node would know both the source IP and the destination, enabling complete **deanonymization**. The intermediate node breaks this link: the entry knows the source but not the destination; the exit knows the destination but not the source.

**Why NOT 10 hops?**

1. **Latency**: each hop adds delay, making TOR unusably slow.
2. **Paradoxically less secure**: the probability that at least one node is compromised increases with the number of nodes. With a 20% chance per node: 3 nodes give a 49% chance of at least 1 bad node; 10 nodes give 89%.
3. **Global timing correlation** (by a state actor who can observe both ends of the communication) is not defeated by extra intermediate hops.

---

### 1.6 HTTP 1.0 vs 1.1 — Persistent Connections and Pipelining

HTTP is a **stateless request/response protocol** running on TCP (traditionally port 80).

**Cross-layer insight (exam favorite)**: HTTP/1.0 is connectionless/stateless at the application layer, but runs over connection-oriented TCP. An application protocol can exhibit connectionless behavior while its transport protocol is connection-oriented.

**HTTP 1.0 — one connection per object**

A page with 1 HTML file and 10 images needs **11 TCP connections**. Each connection costs: TCP three-way handshake (1.5 RTT) + HTTP request/response + TCP teardown. TCP's **slow start** begins from a small congestion window every time, further reducing throughput.

**HTTP 1.1 — two key improvements**

| Feature | How it works | Benefit |
|---------|-------------|---------|
| **Persistent connections** | The TCP connection stays open after the first request/response; subsequent objects are fetched over the **same connection** | Only one TCP handshake for multiple objects; congestion window does not reset; fewer sockets and less state on the server |
| **Pipelining** | Multiple requests are sent without waiting for each response to complete | Reduces total latency by overlapping requests |

The benefit of HTTP 1.1 is **greatest for pages with many small embedded objects**.

**Pipelining limitations**: **Head-of-line blocking** (server must respond in order), browser may not know which objects it needs until HTML is parsed, some servers/proxies do not support pipelining correctly.

**HTTP/2** (from SPDY) adds **binary framing and multiplexing** within a single TCP connection, plus stream prioritization.

---

### 1.7 FTP — Control/Data Connection and Passive Mode

FTP uses **two separate TCP connections**:

| Connection | Port | Purpose | Lifetime |
|------------|------|---------|----------|
| **Control** | 21 | Session management (commands like `USER`, `PASS`, `CWD`, `PASV`, `RETR`, `QUIT`) | Persistent for the entire session |
| **Data** | 20 | Actual file transfer | Opened per transfer, then closed |

The separation ensures that an **abort command** sent on the control channel is not blocked by a full data buffer.

**Active FTP vs Passive FTP (NAT problem)**

In **active mode**, the client sends a `PORT` command with its private IP. The server initiates an inbound data connection, which **fails behind NAT** because: (1) NAT only tracks outbound connections, and (2) FTP embeds IP addresses in the application payload, which NAT does not inspect.

In **passive mode (PASV)**, the client initiates both connections (outbound). NAT handles this without issues.

---

### 1.8 UDP Applications — Why No TCP

| Application | Why UDP, not TCP |
|-------------|-----------------|
| **DNS** | Queries are small (< 512 bytes) and one-shot. TCP's handshake would triple latency. UDP enables **anycast**. DNS servers are **stateless**. Exception: TCP for large responses and zone transfers. |
| **DHCP** | TCP is **fundamentally impossible**: requires valid source/destination IP for handshake. During DISCOVER, the client has neither. DHCP requires **broadcast**. |
| **Wake-on-LAN** | Target machine is **powered off** — cannot perform TCP handshake. Magic packet must arrive as broadcast. |

**The common thread**: UDP is chosen when (1) TCP setup is impossible (no valid IP, machine is off), (2) speed matters more than reliability (real-time media), (3) messages are small and one-shot, or (4) broadcast is required.

---

## 2. Transport Layer — Closed Book

---

### 2.1 TCP Flow Control — Sliding Window, Window Advertisement, Window Probe, Deadlock

TCP flow control prevents a fast sender from overwhelming a slow receiver. The receiver communicates how much buffer space it has left via the **Window Size** field in every ACK it sends. This value is called the **advertised window (rwnd)**.

**How it works:**

1. The receiver maintains a buffer (e.g. 4096 bytes). As the application reads data from that buffer, space frees up.
2. With every ACK, the receiver announces: `WIN = buffer size - (received but not yet read data)`.
3. The sender may never have more unacknowledged data in flight than the receiver's advertised window.
4. The actual sending limit is always `min(rwnd, cwnd) - data already in flight`, where **cwnd** is the congestion window.

**The deadlock problem:**

| Step | What happens |
|---|---|
| 1 | Receiver sends `ACK, WIN = 0` (buffer full) |
| 2 | Sender stops sending and waits for a window update |
| 3 | Application on receiver side reads data; receiver sends a window update with `WIN > 0` |
| 4 | **That window update is lost in the network** |
| 5 | Sender waits for an update that never arrives; receiver thinks it already sent one |

Result: **deadlock** — both sides wait forever.

**Window probes as the solution:**

- When the sender receives `WIN = 0`, it starts a **persistence timer**.
- When the timer expires, the sender sends a **window probe**: a tiny segment (1 byte).
- The receiver must respond with an ACK containing the current window size.
- If `WIN` is still 0, the timer restarts with **exponential backoff**.
- If `WIN > 0`, normal transmission resumes.

Two exceptions allow sending even when `WIN = 0`: **urgent data** (URG flag) and **window probes**.

---

### 2.2 Silly Window Syndrome + Tinygram — Nagle's Algorithm, Delayed ACKs, Interaction

These four mechanisms address the problem of sending tiny, inefficient segments. A 1-byte payload with 20-byte IP + 20-byte TCP headers means less than 2.5% efficiency.

**Overview of problems and solutions:**

| Problem | Cause | Side | Solution |
|---|---|---|---|
| Tinygram syndrome | Sender sends very small segments | Sender | **Nagle's algorithm** |
| Silly Window Syndrome | Receiver advertises very small windows | Receiver | **Clarke's algorithm** |
| Too many bare ACKs | Receiver ACKs every segment immediately | Receiver | **Delayed ACKs** |
| Nagle + Delayed ACK interaction | Both sides wait on each other | Both | **TCP_NODELAY** or **TCP_QUICKACK** |

**Nagle's algorithm (RFC 896):**
- If no unacknowledged segment is outstanding: send data immediately, even if small.
- If there is an unacknowledged segment: buffer new small data until either a full MSS is collected OR the outstanding ACK arrives.
- Effect: never more than one small segment in flight at a time.

**Delayed ACKs:**
- The receiver waits up to **200 ms** before sending an ACK, hoping to piggyback it on return data.
- If no return data arrives within 200 ms, a bare ACK is sent anyway.

**Clarke's algorithm (against SWS):**
- The receiver delays window updates until either at least one **MSS** fits in the buffer OR the buffer is at least **half empty**.

**The dangerous Nagle + Delayed ACK interaction:**

1. Client sends small segment 1 → Nagle says: "there is now an unacknowledged segment, buffer further data."
2. Client has more data → Nagle says: "wait for ACK of segment 1."
3. Server receives segment 1 but waits for more data → Delayed ACK says: "wait up to 200 ms for piggyback opportunity."
4. **Impasse**: client waits for ACK (Nagle); server waits 200 ms for return data (delayed ACK).

This is **not a permanent deadlock** — the delayed ACK timer expires after 200 ms and the ACK is sent. But the systematic **200 ms latency per interaction** is unacceptable for interactive applications.

**Solutions:** `TCP_NODELAY` (disable Nagle), `TCP_QUICKACK` (disable delayed ACKs), single write() call, or use QUIC/HTTP2.

---

### 2.3 RTP — Features, Difference with TCP, Header Fields, Multicast

RTP (Real-time Transport Protocol, RFC 3550) is designed for audio, video, and other real-time media streams. Its core philosophy: **being too late is often worse than being lost**.

**RTP vs TCP:**

| Aspect | RTP (over UDP) | TCP |
|---|---|---|
| Goal | Play media correctly in time | Deliver all bytes reliably in order |
| Retransmission | No (too-late data is useless) | Yes (every byte must arrive) |
| Flow control | No (source determines tempo) | Yes (window advertisement) |
| Congestion control | No (constant bitrate, multicast) | Yes (AIMD, cwnd) |
| Multicast | Yes (core design feature) | No (unicast only) |
| Loss tolerance | Tolerant (interpolation possible) | Intolerant |

**Why no flow/congestion control?** The send rate is determined by the media source (camera, microphone), not by the receiver. Slowing the source would produce silence or frozen video. Additionally, RTP often uses multicast where each receiver sits on a different path with different congestion levels, making per-receiver congestion control impractical.

**RTP header fields:**

| Field | Purpose |
|---|---|
| Version | RTP version |
| P (Padding) | Whether padding bytes are present |
| X (Extension) | Whether an extension header follows |
| CC (CSRC Count) | Number of contributing sources listed |
| M (Marker) | Profile-specific marker, e.g. start of a video frame |
| Type | Encoding algorithm used (payload type) |
| Sequence Number | Incrementing number to detect loss and reorder |
| Timestamp | When the first byte of this media unit was generated |
| SSRC | Synchronization Source — identifies the stream |
| CSRC | Contributing Sources — used when data from multiple sources is mixed |

The **sequence number** answers "which packet comes after which?" while the **timestamp** answers "when should this media be played?" Both are needed because ordering alone is insufficient for real-time playback.

**Risk:** Because RTP/UDP has no congestion control while TCP does (AIMD), they compete unfairly on shared links. This causes **TCP starvation** and can lead to **congestion collapse** at high loads.

---

### 2.4 TCP Congestion Control — Tahoe/Reno, Slow Start, AIMD, Threshold

Congestion control protects the **network** (vs. flow control which protects the **receiver**). TCP uses **packet loss** as an implicit congestion signal. The effective sending limit is always `min(rwnd, cwnd)`.

**Two phases of TCP congestion control:**

| Phase | Growth | When active |
|---|---|---|
| **Slow start** | Exponential (cwnd doubles per RTT) | cwnd < ssthresh |
| **Congestion avoidance** | Linear (+1 MSS per RTT, the "AI" in AIMD) | cwnd >= ssthresh |

**Slow start in detail:**
1. Start with cwnd = 1 MSS.
2. For every ACK received: cwnd += 1 MSS.
3. Since a window of N segments generates N ACKs, cwnd effectively doubles per RTT.
4. Continue until cwnd >= ssthresh, then switch to congestion avoidance.

**Tahoe vs Reno:**

| Event | TCP Tahoe (1988) | TCP Reno (1990) |
|---|---|---|
| **Timeout** | ssthresh = cwnd/2, cwnd = 1, restart slow start | Identical to Tahoe |
| **3 duplicate ACKs** | ssthresh = cwnd/2, cwnd = 1, restart slow start | ssthresh = cwnd/2, cwnd = cwnd/2, **fast retransmit** the lost segment, continue with **congestion avoidance** (no slow start) |

The key difference: upon 3 duplicate ACKs, **Tahoe falls back to cwnd = 1** and restarts slow start, while **Reno halves cwnd and continues** with congestion avoidance (fast recovery).

**Why 3 duplicate ACKs?** Receiving 3 duplicate ACKs (4 identical ACK numbers total) strongly suggests one specific segment was lost but later segments are still arriving — the network is still functioning. A timeout means no feedback at all, indicating more severe congestion.

**Typical Tahoe/Reno diagram (cwnd in MSS over time in RTTs):**

```
cwnd
(MSS)
  |
32|                                    *
  |                                 *
16|              *               *         <-- Reno: fast recovery
  |           *     *         *
 8|        *          *    *
  |     *               *
 4|  *                   (3 dup ACKs: Reno halves to 8, Tahoe drops to 1)
  | *
 2|*
 1|*
  +---------------------------------------------------> time (RTTs)
     ^ slow start    ^ congestion     ^ loss     ^ recovery
                       avoidance
```

**SACK (Selective Acknowledgments):** With cumulative ACKs alone, the sender only knows "everything up to here is fine." SACK lets the receiver report which higher byte-ranges have been received, enabling targeted retransmission when **multiple packets are lost** in one window.

---

### 2.5 ECN vs RED — How ECN Works, Advantages/Disadvantages vs RED

Both mechanisms signal congestion to the sender, but in fundamentally different ways.

**RED (Random Early Detection):**
- Runs on **routers only**.
- Measures average queue length; when it exceeds a threshold, starts dropping packets with **increasing probability**.
- Heavy senders have more packets in the queue and are hit proportionally more often (fairer than tail-drop).
- **Broadly deployed** because it only requires router changes.

**ECN (Explicit Congestion Notification):**
- Both endpoints negotiate ECN support during the TCP handshake.
- Sender marks packets as ECN-capable (ECT bit in IP header).
- A congested router sets the **CE bit** instead of dropping the packet.
- Receiver sees CE and sends back an ACK with the **ECE flag**.
- Sender receives ECE, reduces cwnd, and responds with the **CWR flag**.

**Comparison:**

| Aspect | RED | ECN |
|---|---|---|
| Mechanism | Router drops packets probabilistically | Router marks packet; receiver echoes to sender |
| Packet loss | Yes (intentional) | No (marking instead of drop) |
| Retransmission overhead | Yes (lost packets must be retransmitted) | None (packet is delivered) |
| Deployment requirement | **Routers only** | Routers + **both** endpoints |
| Actually deployed? | **Yes, broadly** | Limited |
| Best for short flows? | No (random drop can disproportionately hurt small transfers) | Yes (no data loss) |

**Key exam point (professor emphasis):** RED is broadly deployed because it only needs router changes. ECN is theoretically superior but requires end-to-end support, making practical deployment much harder. This is a textbook example of a theoretically better solution losing to a pragmatic one due to deployment barriers.

---

### 2.6 QUIC — Design, Difference with TCP

QUIC (RFC 9000, ~30% of internet traffic) is a modern transport protocol built in **userspace on top of UDP** with integrated security, stream multiplexing, and connection migration.

**Core QUIC vs TCP differences:**

| Aspect | TCP | QUIC |
|---|---|---|
| Encryption | Optional, separate (TLS on top) | Mandatory, TLS 1.3 built in |
| Connection setup | 2-3 RTT (TCP handshake + TLS) | **1 RTT** new / **0 RTT** resumed |
| Data model | Single ordered byte-stream | Frame-based with **multiple independent streams** |
| Head-of-line blocking | Yes (one lost segment blocks everything) | No (only the affected stream waits) |
| Connection migration | No (tied to IP/port 4-tuple) | Yes (uses **Connection IDs**) |
| Implementation | OS kernel | User space (faster updates) |
| Sequence numbers | 32-bit (can wrap) | 62-bit (practically never wraps) |
| ACKs | Cumulative (+ optional SACK) | Range-based (always, richer info) |

**Stream multiplexing:** QUIC supports multiple independent streams within one connection. If a packet in stream 1 is lost, streams 2 and 3 continue unaffected. This eliminates **head-of-line blocking at the connection level**.

**0-RTT connection resumption:** If the client has a stored session ticket from a previous connection, it can send application data in the very first packet. **Security risk:** 0-RTT data is vulnerable to **replay attacks**, so it is only safe for **idempotent operations** (e.g. GET requests, not bank transfers).

**Connection migration:** When a client switches from WiFi to mobile data, the IP/port changes but the Connection ID stays the same. The server validates the new path via **PATH_CHALLENGE / PATH_RESPONSE** frames. The **congestion control state is reset** upon migration.

**Setup latency comparison:**

| Combination | RTTs before first application data |
|---|---|
| TCP + TLS 1.2 | 3 RTT |
| TCP + TLS 1.3 | 2 RTT |
| QUIC (new connection) | **1 RTT** |
| QUIC (resumed) | **0 RTT** |

---

### 2.7 TCP Header — All Fields Explained

The TCP header is minimally **20 bytes** (without options) and maximally **60 bytes** (with options).

| Field | Size | Function |
|---|---|---|
| **Source Port** | 16 bits | Identifies the sending process (TSAP) |
| **Destination Port** | 16 bits | Identifies the receiving process (TSAP) |
| **Sequence Number** | 32 bits | Byte number of the **first byte** in this segment |
| **Acknowledgment Number** | 32 bits | Next byte the receiver **expects** (cumulative: ACK=2048 means "received all up to byte 2047") |
| **Header Length (Data Offset)** | 4 bits | Header length in 32-bit words (min 5 = 20 bytes, max 15 = 60 bytes) |
| **Reserved** | 3 bits | Must be 0, reserved for future use |
| **Flags** | 9 bits | Control bits: URG, ACK, PSH, RST, SYN, FIN, ECE, CWR, NS |
| **Window Size** | 16 bits | Bytes the receiver can still accept (flow control) |
| **Checksum** | 16 bits | Mandatory error detection over header + data + IP pseudo-header |
| **Urgent Pointer** | 16 bits | Offset to end of urgent data (valid only when URG=1) |
| **Options** | Variable | Extensions: MSS, Window Scale, SACK, Timestamps (TLV format) |

**Flag meanings:**

| Flag | Meaning |
|---|---|
| **SYN** | Synchronize — initiates connection setup (3-way handshake) |
| **ACK** | Acknowledgment number field is valid |
| **FIN** | Finish — initiates graceful connection close |
| **RST** | Reset — abruptly terminates the connection |
| **PSH** | Push — deliver data to application immediately |
| **URG** | Urgent pointer is valid; data before the pointer gets priority |
| **ECE** | ECN-Echo — receiver echoes router's congestion marking |
| **CWR** | Congestion Window Reduced — sender confirms it lowered cwnd |
| **NS** | ECN-nonce concealment protection |

**Important options:** MSS (max segment size, SYN only), Window Scale (enables windows up to ~1 GB, SYN only), SACK Permitted/blocks, Timestamps (RTT measurement and PAWS).

---

### 2.8 Sliding Window Protocols — Comparison, Sequence Number Limitations

The three sliding window protocols form a spectrum from simplicity to efficiency.

| Property | Stop-and-Wait | Go-Back-N | Selective Repeat |
|---|---|---|---|
| Sender window | 1 | W | W |
| Receiver window | 1 | 1 | W |
| Receiver buffering | No | No | Yes (out-of-order) |
| ACK type | Per segment | Cumulative | Individual per segment |
| On loss | Retransmit that one segment | Retransmit **all** from lost segment onward | Retransmit **only** the lost segment |
| Seq. number requirement | 2 (0 and 1) | W + 1 | 2W (window <= half of seq. space) |
| Efficiency | Low when a >> 0 | High if no loss, poor if much loss | Always high |
| Complexity | Minimal | Medium | High |

**Efficiency formulas:**

```
a = (RTT / 2) / T_trans        where T_trans = (packet_size * 8) / link_speed

Stop-and-Wait:         η = 1 / (1 + 2a)
Go-Back-N:             η = min(W / (1 + 2a), 1)
Selective Repeat:      η = min(W / (1 + 2a), 1)
```

For 100% efficiency you need W >= 1 + 2a. Stop-and-wait is only acceptable when a ≈ 0.

**Sequence number limitations (frequently asked):**

- **Go-Back-N** needs at least **W + 1** sequence numbers. The receiver has window 1. If all W ACKs are lost and the sender retransmits everything, the receiver must distinguish new segments from retransmissions.
- **Selective Repeat** needs at least **2W** sequence numbers (window at most half the sequence space). The receiver now also has a window of W and buffers out-of-order. This stricter requirement comes from having a receiver window > 1.

---

### 2.9 Retransmission — Why in the Transport Layer

**The end-to-end argument:** functions that require end-to-end correctness must be implemented at the endpoints.

| Layer | Type | Protects against | Limitation |
|---|---|---|---|
| **Data link** | Hop-by-hop | Frame loss between directly connected nodes | Does **not** protect against errors **inside routers** or losses due to congestion |
| **Transport** | End-to-end | **All** forms of loss across the entire path | Slower reaction (higher RTT), retransmission loads the full path |
| **Application** | Application-specific | Whatever the app chooses | Duplicates transport-layer functionality; no standardization |

Data-link retransmission is useful as a **complement** but can never be the sole protection because errors can also originate **within routers**.

---

## 3. Network Layer — Closed Book

---

### 3.1 AODV — How It Works (ROUTE_REQUEST / ROUTE_REPLY), Reactive vs Proactive

AODV (Ad-hoc On-Demand Distance Vector) is a routing protocol for ad-hoc networks where every node acts as both host and router. The key design choice: AODV is **reactive (on-demand)** — it only discovers routes when a node actually needs to send data, rather than continuously maintaining a full routing table.

**Why reactive?** Ad-hoc networks have resource-constrained, energy-constrained, and mobile nodes. A proactive protocol like OSPF would flood link-state updates at every topology change, wasting energy and bandwidth on routes that may never be used.

**Route Discovery process:**

| Step | What happens | Why |
|------|-------------|-----|
| 1 | Source needs a route to a destination not in its routing table | Triggers on-demand discovery |
| 2 | Source broadcasts a ROUTE_REQUEST with a **sequence number** and **TTL** | Flooding ensures the request reaches the destination even in an unknown topology |
| 3 | TTL starts small, then increases stepwise if no reply comes | Avoids expensive network-wide floods when the destination is nearby |
| 4 | Intermediate nodes record reverse path | Builds temporary reverse paths back to the source |
| 5 | Destination (or node with valid route) sends ROUTE_REPLY back along the recorded path | Hop count updated at each step; intermediate nodes also learn the route |
| 6 | Data flows along discovered route | Route stays active as long as it is used |

**Route Maintenance:** When a node detects a failed neighbor, it removes dependent routes, warns affected neighbors, and those nodes can trigger a new ROUTE_REQUEST if needed.

**Reactive vs Proactive — when to use which:**

| Criterion | AODV (reactive) | OSPF (proactive) |
|-----------|-----------------|-------------------|
| Topology changes | Frequent (mobile nodes) | Rare (stable infrastructure) |
| Traffic volume | Low/sporadic — no control traffic when nobody sends data | High/continuous — routes are always ready |
| Resources | Constrained (IoT, sensors) — stores only active routes | Sufficient — can maintain full topology database |
| Latency on first packet | Higher (must discover route first) | Zero (route already computed) |
| Energy | Less control traffic = longer battery life | Continuous HELLOs and flooding = higher energy |

**Core exam reasoning:** Much topology change + little traffic = AODV. Stable topology + much traffic = OSPF.

---

### 3.2 Congestion Notifications — RED vs ECN vs Choke Packets

The network layer must provide signals that tell hosts to slow down. Three mechanisms exist, each with different trade-offs.

| Property | Choke Packets | ECN | RED |
|----------|---------------|-----|-----|
| **How it works** | Router sends a separate control packet back to the sender | Router sets a bit in a regular data packet; receiver echoes back to sender | Router probabilistically drops packets before queue is full |
| **Signal type** | Explicit, separate packet | Explicit, marking in existing packet | Implicit via packet loss |
| **Extra overhead** | High: additional packets in already congested network | None: reuses existing packets | Negative: causes retransmissions |
| **Reaction speed** | Fast (~1 RTT, directly to sender) | Medium (~1 RTT, via receiver) | Fast: TCP interprets loss directly |
| **Deployment requirements** | Only the sender must react | Routers + both endpoints | **Only routers** |
| **Fairness** | Not inherently fair | Not inherently fair | Fair: heavy senders hit proportionally more |
| **Deployed in practice** | Historical, rarely used | Limited | **Yes — most widely deployed** |

**Why RED is the most deployed:** RED requires **no changes to endpoints**. Routers can implement it autonomously. ECN requires both routers and both communicating hosts to support it.

**Why RED's random dropping is fair:** A sender that pushes more packets into the network occupies a larger share of the queue. Random drops hit aggressive senders with higher probability, without the router needing to track individual flows.

---

### 3.3 IP Addressing / Subnetting — Broadcast Address, CIDR, Host Range

An IPv4 address is 32 bits, split into a **network prefix** and a **host part**. CIDR notation (e.g., `192.168.5.0/25`) means the first 25 bits are the network part. CIDR replaced the wasteful classful system (A/B/C), enabling flexible prefix lengths and **supernetting** to shrink routing tables.

**Subnetting calculation method:**

Given an IP address and CIDR prefix `/n`:

| Step | Formula | Example: `192.168.5.130/25` |
|------|---------|----------------------------|
| Subnet mask | First n bits = 1, rest = 0 | `255.255.255.128` |
| Network address | IP AND subnet mask | `192.168.5.128` |
| Broadcast address | Network address with all host bits set to 1 | `192.168.5.255` |
| First usable host | Network address + 1 | `192.168.5.129` |
| Last usable host | Broadcast address - 1 | `192.168.5.254` |
| Number of usable hosts | 2^(32-n) - 2 | 2^7 - 2 = **126** |

The network address (all host bits 0) identifies the subnet itself. The broadcast address (all host bits 1) sends to all hosts in that subnet. Neither can be assigned to a host, hence -2.

**Same-subnet test:** Two hosts are on the same subnet if and only if `(IP_1 AND mask) == (IP_2 AND mask)`. Same subnet → communicate directly via ARP. Different subnet → send to the **default gateway**.

---

### 3.4 Distance Vector Routing / Count-to-Infinity Problem

**How Distance Vector works (Bellman-Ford):**

Each router maintains a table with, for every destination, the estimated distance and the best next hop. Routers periodically exchange these tables with direct neighbors. Upon receiving a neighbor's table, a router recalculates:

`cost to destination via neighbor = cost to neighbor + neighbor's estimate to destination`

The router picks the neighbor yielding the lowest total cost.

**The Count-to-Infinity problem:**

When a destination becomes unreachable, routers get trapped in circular reasoning. Consider three routers A—B—C where A is directly connected to destination X. When A—X fails:

1. A sets its cost to X to infinity
2. B still advertises "to X, cost 2" (stale information)
3. A thinks: "via B costs 3" and accepts this
4. B updates to 4 based on A's new cost 3
5. Costs bounce: 3, 4, 5, 6... slowly rising toward infinity

**Why it happens:** No router knows whether it is itself already part of the proposed path. Each router only sees local information and cannot detect circular routes.

**Mitigations (none fully solve it):**

| Technique | How it works | Fully solves it? |
|-----------|-------------|-----------------|
| **Split horizon** | Do not advertise a route back to the neighbor you learned it from | No — fails with loops of 3+ nodes |
| **Poison reverse** | Advertise the route back with cost = infinity | Better, but still fails with larger loops |
| **Maximum metric** | Define a maximum (e.g., RIP uses 16 = unreachable) | Limits duration, does not prevent loops |

**How AODV solves count-to-infinity:** AODV uses **destination sequence numbers**. Each route carries a sequence number incremented by the destination. A higher sequence number is always fresher. When a link fails, the detecting node sends a RERR with an incremented sequence number. Nodes with stale sequence numbers discard their old route immediately — no loop can form.

---

### 3.5 BGP vs OSPF — Intradomain vs Interdomain

Routing inside an organization has a different goal than routing between organizations. Inside an AS, you want technically optimal paths. Between AS'es, you need to enforce **business policy**.

| Property | OSPF | BGP |
|----------|------|-----|
| **Scope** | Intra-domain (within one AS) | Inter-domain (between AS'es) |
| **Algorithm** | Link-state (flood link costs → each router runs Dijkstra) | Path-vector (AS-paths, not simple distances) |
| **Routing criterion** | Technical shortest path | **Policy**: which AS'es to avoid, which transit to prefer |
| **Knowledge per router** | Full topology of its area | Only reachability + AS-path |
| **Transport** | Directly on IP (protocol 89), **connectionless** | **TCP** (port 179), **connection-oriented** |
| **Convergence** | Fast (direct flooding) | Slower (policy filters) |

**Why OSPF uses connectionless transport:** OSPF floods link-state advertisements to all neighbors. TCP sessions per neighbor would be heavy and pointless for flooding.

**Why BGP uses TCP:** Inter-AS sessions are long-lived (days to weeks). They must be reliable — a lost BGP update can make an entire AS unreachable. TCP provides reliable, ordered delivery.

**Key BGP concepts:**
- **AS-path:** Lists every AS a route traverses, enabling policy decisions
- **Peering:** Two AS'es exchange traffic, typically without payment. **Peering is not transitive.**
- **Transit:** An AS pays another to carry its traffic to the rest of the internet
- **iBGP:** Boundary routers share external routes internally. OSPF handles internal reachability of those boundary routers.

---

### 3.6 Round Robin Fair Queuing — How It Works, Abuse

**The problem:** With FIFO scheduling, aggressive senders fill the queue and other flows experience delay or loss. FIFO is simple but unfair.

**How Round Robin Fair Queuing works:**

Each flow gets its own queue. The router cycles through all queues in a round-robin pattern, pulling one packet from each non-empty queue per round. Empty queues are skipped, redistributing bandwidth to active flows.

This ensures each flow gets an equal share of bandwidth, regardless of how aggressively any single flow sends.

**How to abuse Round Robin:**

- **Large-packet trick:** If the scheduler takes one packet per turn, a flow sending **1500-byte packets** wins 15x more bytes per round than one sending 100-byte packets.
- **Multiple flows:** A user opens several connections to the same destination, occupying multiple queues and getting proportionally more bandwidth.

**Solution — byte-level fairness:** Track how many **bytes** each flow may send per round, neutralizing the large-packet trick.

**Weighted Fair Queuing (WFQ):** Each flow gets a weight determining how many bytes it may send per round, allowing prioritization while maintaining controlled fairness.

---

## 4. Data Link & Physical Layer — Closed Book

---

### 4.1 B-MAC vs TSMP — When to Choose Which, LPL, Preamble Configuration

B-MAC and TSMP represent two fundamentally different philosophies for energy-efficient MAC in wireless sensor networks: **asynchronous and decentralized** vs **synchronized and coordinated**.

**B-MAC (Berkeley MAC) — Low Power Listening (LPL)**

- Nodes keep their radio off most of the time. They briefly turn it on at regular intervals to **sample** the channel for activity.
- A sender transmits a **long preamble** before the actual data. This preamble must last at least as long as the receiver's sample interval, guaranteeing the receiver detects it during one of its periodic checks.
- Core constraint: `Preamble_length >= Sample_Interval`
- Fully **decentralized** — no coordinator or schedule needed.

**TSMP (Time Synchronized Mesh Protocol) — TDMA**

- Uses a **central manager** to assign precise **time slots** (TDMA). Nodes know exactly when to send and listen.
- Requires **tight time synchronization** across all nodes.
- Also uses **frequency hopping** to spread interference risk.
- Biggest energy cost: **(re)joining** the network.

**Preamble/sample-interval configuration for B-MAC:**

The trade-off is between sampling energy (listening often) and preamble energy (long transmissions).

| Traffic pattern | Sample interval | Preamble length | Why |
|---|---|---|---|
| **High data rate** (frequent packets) | **Short** | **Short** | Radio samples often but each preamble is short. Frequent traffic amortizes the sampling cost. |
| **Low data rate** (rare packets) | **Long** | **Long** | Radio sleeps as long as possible. Each transmission is expensive but transmissions are rare. |

**When to choose which:**

| Criterion | B-MAC | TSMP |
|---|---|---|
| Traffic pattern | **Unpredictable / bursty / event-driven** | **Predictable / periodic** |
| Topology | Dynamic, nodes come and go | Stable, known participants |
| Synchronization | Not needed | Required (central manager) |
| Scalability | Good (decentralized) | Limited (manager = bottleneck) |
| Energy efficiency | Good but wastes some energy on preambles/sampling | Best possible (radio on only at scheduled moments) |
| QoS | No guarantees | Can reserve bandwidth per node |
| Latency for sporadic traffic | Low (send anytime with preamble) | Higher (must wait for assigned slot) |

---

### 4.2 Hidden / Exposed Terminal Problem — MAC Solutions (CSMA/CA, RTS/CTS)

**Hidden terminal problem:** Two senders (A and C) cannot hear each other but both transmit to the same receiver (B). Each thinks the channel is free, yet their transmissions collide at B.

**Exposed terminal problem:** A node (C) hears a nearby sender (B) and concludes the channel is busy, even though C's transmission to a different receiver (D) would cause no interference. This leads to unnecessary self-censorship.

**RTS/CTS (used in 802.11, based on MACA):**

```
1. A -> B: RTS (includes Duration field)
2. B -> all: CTS (includes Duration field) -- C hears this!
3. C sets its NAV timer and stays silent
4. A -> B: DATA (collision-free)
5. B -> A: ACK
```

Even if C could not hear A's RTS (hidden terminal), C **does** hear B's CTS. The Duration field tells C how long the channel will be busy (NAV = Network Allocation Vector = virtual carrier sense).

RTS/CTS does **not** solve the exposed terminal problem. It has **overhead** (two extra control frames) and is only worthwhile for large data frames.

| Problem | Solved by CSMA? | Solved by RTS/CTS? |
|---|---|---|
| Hidden terminal | No (local listening is insufficient) | **Yes** (CTS reaches hidden nodes) |
| Exposed terminal | No (causes unnecessary silence) | **No** |

---

### 4.3 Pure ALOHA / Slotted ALOHA

**Pure ALOHA:** A node transmits **whenever it wants**. If two transmissions overlap, both are lost. After a collision, each node waits a **random** time before retransmitting. Vulnerable period = 2T. Maximum utilization: **S_max = 1/(2e) ≈ 18.4%**.

**Slotted ALOHA:** Time is divided into discrete **slots**. Nodes may only begin transmitting at the start of a slot. Vulnerable period = 1T. Maximum utilization: **S_max = 1/e ≈ 36.8%**. Requires **synchronization**.

| Property | Pure ALOHA | Slotted ALOHA |
|---|---|---|
| Vulnerable period | 2T | 1T |
| Max throughput | ~18% | ~37% |
| Synchronization needed | No | Yes |

**Can Pure ALOHA work on 802.11?** Technically yes, but it is a **terrible choice**. 802.11 deliberately uses CSMA/CA with collision avoidance, backoff, and RTS/CTS. Pure ALOHA's blind transmissions would massively waste the channel, ignore the hidden terminal problem, and cause catastrophic collisions during RREQ flooding (e.g., with AODV).

---

### 4.4 Switches vs Hubs

| Aspect | Hub | Switch |
|---|---|---|
| Operation | Physical repeater: copies signals to **all** ports | Learns MAC addresses, forwards only to **correct port** |
| Collision domain | One shared domain (all ports) | Separate domain **per port** |
| Duplex | Half-duplex only (CSMA/CD required) | **Full-duplex** possible |
| Bandwidth | Shared across all ports | **Dedicated** per port |
| Privacy | Everyone sees all traffic | Only your own unicast traffic |

**When is a hub actually better than a switch?**

- **Network sniffing / protocol analysis:** A hub broadcasts everything, so tools like Wireshark see all traffic without configuration. On a switch, you need port mirroring (SPAN).
- **Lab / teaching:** To demonstrate collisions and CSMA/CD behavior.

---

### 4.5 Ethernet — Cable Length Limit, Padding, CSMA/CD

Classic Ethernet uses **1-persistent CSMA/CD**: listen first, wait if busy, transmit immediately when free. On collision, use **binary exponential backoff**.

**Why a minimum frame size (64 bytes)?** The sender must still be transmitting when the collision signal from the farthest point returns. Requirement: `T_transmission >= 2 × T_propagation`. If payload + headers < 64 bytes, **padding** is added.

**Cable length limits across Ethernet variants:**

| Configuration | CSMA/CD? | Cable limit | Determined by |
|---|---|---|---|
| Classic 10 Mbps with hub | Yes | ~2500 m | Collision timing |
| Fast 100 Mbps with hub | Yes | ~256 m (10x stricter) | Collision timing |
| Switched, **full-duplex** | **No** | ~100 m (Cat5e) | **Signal attenuation only** |
| 10 Gigabit | **No** (full-duplex only) | Fiber: km's, copper: ~100 m | Attenuation |

**Why full-duplex eliminates the collision limit:** Sending and receiving use **separate wire pairs**. Collisions are physically impossible, CSMA/CD is disabled, and the only remaining limit is signal attenuation.

**Why higher speeds make CSMA/CD harder:** At higher bit rates, the same 512-bit minimum frame transmits in less time, so the maximum cable length shrinks proportionally. At 10 Gbps, CSMA/CD was abandoned entirely — only full-duplex point-to-point links.

---

### 4.6 802.11 Power Saving — Beacon Frames, APSD, Two Strategies

**Strategy 1: Beacon + TIM / PS-Poll (AP-controlled)**

1. Client enters power save mode and tells the AP.
2. AP **buffers** all frames for that client.
3. Client wakes only at **beacon intervals** (typically 100 ms).
4. Each beacon contains a **Traffic Indication Map (TIM)** — a bitmap showing which sleeping clients have buffered data.
5. If TIM shows data for this client, the client sends a **PS-Poll** to retrieve it.
6. Between beacons, the client sleeps.

Best for: **idle battery-powered devices** with unpredictable traffic.

**Strategy 2: APSD (Automatic Power Save Delivery, client-controlled)**

1. The **client** decides when to wake up based on its own application rhythm.
2. Client wakes only when it **wants to send data**.
3. AP delivers buffered downlink data in the same wake window.

Best for: **periodic applications** with predictable timing (VoIP, telemetry).

| Aspect | Beacon + TIM / PS-Poll | APSD |
|---|---|---|
| Who determines wake schedule | **AP** (beacon interval) | **Client** (application rhythm) |
| Best suited for | Idle devices, unpredictable traffic | Periodic applications |

---

### 4.7 MAC Protocols with AODV — Which Work, Which Don't

AODV is a reactive, on-demand routing protocol that discovers routes via RREQ flooding and assumes a dynamic topology. This requires a MAC that can **transmit at any time without a pre-configured schedule**.

| MAC Protocol | Compatible with AODV? | Why |
|---|---|---|
| **CSMA/CA** | **Good** | Asynchronous, decentralized, transmits when needed |
| **B-MAC** | **Good** | Asynchronous, decentralized, energy-saving via LPL |
| **TSMP / TDMA** | **Bad** | Requires pre-established schedule; assumes stable topology. Topology changes force expensive rescheduling. |
| **Pure ALOHA** | **Bad** | No carrier sense, no backoff. AODV's RREQ flooding causes massive collisions without collision avoidance. |

---

### 4.8 Channel Hopping — Advantages

Channel hopping (as used in TSMP) means all synchronized nodes hop through a known sequence of frequency channels instead of staying on a single fixed channel.

- **Interference resilience:** If an external source disrupts a specific channel, only a few timeslots are affected.
- **Multipath fading mitigation:** Signal strength varies per frequency. Hopping avoids being stuck on a "bad" frequency.
- **Security:** Without knowledge of the hopping sequence, eavesdropping and jamming are harder.
- **Fair spectrum distribution:** All nodes share good and bad channels equally over time.

---

### 4.9 Stop-and-Wait Analysis — Which Medium Good/Bad, Throughput Calculation

Stop-and-wait efficiency: `η = 1 / (1 + 2a)` where `a = T_propagation / T_transmission`.

Stop-and-wait is efficient when the **bandwidth-delay product is smaller than the frame size** (a close to 0).

**When stop-and-wait works well (a ≈ 0):**

| Medium | Why |
|---|---|
| Short Ethernet (100 m) | T_propagation negligibly small vs T_transmission |
| LoRa SF12 (20 km, 250 bps, 50 bytes) | T_transmission ≈ 1600 ms, T_propagation ≈ 67 μs |
| Serial 9600 bps over short cable | T_transmission >> T_propagation |

**When stop-and-wait is bad (a >> 1):**

| Medium | Why |
|---|---|
| GEO satellite (RTT ~540 ms, 10 Mbps) | Sender finishes frame in microseconds but waits hundreds of ms for ACK |
| Trans-Atlantic cable (RTT ~80 ms, high bandwidth) | High bandwidth-delay product means sender is idle most of the time |

**Exam methodology:** Calculate `a = T_propagation / T_transmission`. If `a << 1`: stop-and-wait is fine. If `a >> 1`: a sliding window protocol is needed.
