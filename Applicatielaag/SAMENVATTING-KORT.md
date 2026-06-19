# Application Layer (Layer 7) — Cheat Sheet

The application layer is the topmost layer: here raw TCP/UDP connections are translated into concrete services (web, mail, files, P2P). Each protocol chooses its own style of commands, session management and (in)security, but almost always relies on **TCP** (reliable) or **UDP** (lightweight, no setup needed) for transport.

## OSI / TCP-IP / Tanenbaum stacks

| Layer | OSI | TCP/IP | Tanenbaum |
|---|---|---|---|
| 7 | Application | Application | Application |
| 6 | Presentation | ↑ | ↑ |
| 5 | Session | ↑ | ↑ |
| 4 | Transport | Transport | Transport |
| 3 | Network | Internet | Network |
| 2 | Data Link | Network Access | Data Link |
| 1 | Physical | ↑ | Physical |

- **Tricky cases**: DHCP = application layer (UDP broadcast); RTP = application layer (on top of UDP); OSPF = network layer (directly over IP, no TCP/UDP).
- OSI = reference model (7 layers, never fully implemented), TCP/IP = practical model (4 layers, successfully deployed).

## UDP applications: why not TCP?

| App | Why UDP |
|---|---|
| **DNS** | Small queries, stateless, anycast-compatible; TCP fallback for large responses |
| **DHCP** | Client has no IP yet → TCP handshake impossible; broadcast needed |
| **Wake-on-LAN** | Target host is powered off → cannot set up a TCP connection; magic packet = broadcast |

Common thread: TCP requires a working connection on both sides — that is precisely what does not yet exist for these apps.

## Remote login: Telnet vs SSH

- **Telnet** (port 23, TCP): sends commands and text in **plaintext**, no server authentication. Phases: connection management → optional negotiation → control → data transfer. Insecure, but still usable as a **debug tool** for text-based protocols over TCP (SMTP, HTTP).
- **SSH**: secure successor, three layers:
  - **Transport layer**: key exchange + server authentication (secure tunnel)
  - **Authentication layer**: user authentication (password or public key)
  - **Connection layer**: services on top (terminal, file transfer, TCP forwarding)
  - With **public-key authentication**, the client proves it has the private key without transmitting it.

## Email: architecture and protocols

Email is a **chain**: sender user agent → mail submission → message transfer agent → SMTP between mail servers → message transfer agent → final delivery → receiver user agent. Messages follow the **Internet Message Format** (headers: `To`, `Cc`, `Bcc`, `From`, `Received`, `Return-Path`, ...).

| | POP | IMAP | Webmail |
|---|---|---|---|
| Operation | Downloads mail locally, little sync | Keeps folders (inbox/drafts/...) synchronized | Browser = client, everything on server |
| Advantage | Simple | Good for multiple devices | Lightweight client, accessible anywhere |
| Disadvantage | Difficult across multiple devices | More overhead/complexity | Requires internet connection |

**SMTP** = command/response **relay protocol between mail servers** (not for mailbox sync!). Three phases: handshaking → transfer → closure. Flow: `HELO` → `MAIL FROM` → `RCPT TO` → `DATA` → message → `.` → `QUIT`. Because it is text-based, you can test it via Telnet.

## The web: URI, HTML, HTTP

- **URI** = identifier for a resource (can be `mailto:`, `ftp://`, `http://`, ...). A **URL** is the most common form, combining protocol + server + resource.
- Web users use names → **DNS** translates to IPs (see below).
- **HTML** structures text and provides meaning/formatting via tags + links.
- **HTTP** = request/response protocol, over TCP, traditionally port 80, **stateless** (no built-in session memory — context via cookies/sessions/app logic). Methods: `GET` (request document), `HEAD` (headers only), `POST` (send data), plus `PUT`, `OPTIONS`, `DELETE`. HTTP/1.x can also be tested via Telnet (human-readable requests).

### HTTP versions compared

| Version | Feature | Problem/advantage |
|---|---|---|
| **HTTP/1.0** | One TCP connection per object | Inefficient: extra round trips, each time new TCP startup + congestion control from small start |
| **HTTP/1.1** | **Persistent connections** + **pipelining** | Reusing connections reduces overhead; pipelining sends multiple requests without waiting for each response (works less well with dependencies between resources) |
| **HTTP/2** | Binary format, **framing**, **multiplexing**, prioritization — within one TCP connection (originated from SPDY) | Structurally solves the "fetching many separate objects" problem |

## FTP: separate control and data

FTP uses **two TCP ports**: port 21 for **control** (Telnet-like command channel, NVT), port 20 for **data**. This separates session management (persistent) from data transfer (per transfer), unlike HTTP where control+data run together. Commands: `USER`, `PASS`, `CWD`, `PASV`, `RETR`, `STOR`, `LIST`, `QUIT`.

## DHCP: automatic network configuration

**DHCP** (RFC 2131, **UDP**) automatically assigns hosts an IP address, subnet mask, gateway and DNS server as a **lease** (temporary, not permanent ownership). Core flow:

`DHCPDISCOVER` (client, broadcast from `0.0.0.0`) → `DHCPOFFER` (server) → `DHCPREQUEST` (client, broadcast) → `DHCPACK` (server)

- The **message type** is in an **option**, not as a fixed header field.
- **Transaction ID** links responses to requests.
- Lease is renewed at ~50% elapsed time; upon refusal a `DHCPNAK` follows.
- `DHCPRELEASE` = voluntarily returning an address.
- After assignment: **Duplicate Address Detection via ARP** — upon conflict the client sends `DHCPDECLINE`.
- **Why not TCP?** TCP requires a 3-way handshake with valid source and destination IP; during DISCOVER the client has neither. UDP allows broadcast from `0.0.0.0` to `255.255.255.255` without prior state.

## DNS: names to addresses

Previously one central `hosts.txt` file — did not scale. DNS is a **hierarchical, distributed** solution.

- Names are **not case-sensitive** (URL paths can be), and technically end with a dot (`www.google.com.`).
- **Hierarchy** from right to left: `nix.cs.kuleuven.ac.be` → country (`be`) → academic domain (`ac`) → organization (`kuleuven`) → department (`cs`) → host (`nix`). Compare to a postal address: broad → specific.
- **TLDs**: country codes (`.be`), generic (`.com`, `.org`), US-specific (`.edu`, `.gov`, `.mil`). Managed by **ICANN** (since 1998), delegated to **registrars** (e.g. Verisign for `.com`, DNS Belgium for `.be`).
- **Resource records**: Domain Name, **TTL**, Class, Type, Value. Most important types: `A` (IPv4), `NS` (name server), `MX` (mail server), `CNAME` (alias), `PTR` (reverse mapping).
- **Server hierarchy**: leaf (concrete records) → branch (mix of records and referrals) → root (knows TLD servers).
- **Zones** = non-overlapping pieces of the namespace; more zones = more load balancing but more management overhead.
- **Authoritative** (from the authorized zone administrator) vs **cached** (temporary copy).
- **Recursive** vs **iterative** lookup: a local resolver does recursive work for the client, and itself steps iteratively from root → TLD → authoritative server. **Root/TLD servers use anycast** for redundancy and performance.

| | Purely iterative | Purely recursive | Hybrid (practice) |
|---|---|---|---|
| Caching | Poor (no shared cache) | Good (local resolver caches for everyone) | Good |
| Load on root | High | Low | Low |
| Client latency | High (multiple round trips) | Low (1 round trip) | Low |
| Client complexity | High | Low | Low |

## Socket programming / client-server vs P2P

Classic applications (Telnet, SSH, mail, web, FTP, DHCP/DNS) follow the **client/server model**: client initiates, server listens on a well-known port, with an agreed protocol. In **P2P**, edge machines themselves provide resources (storage, bandwidth, CPU). Distinction:
- **P2P application** (application-centric): uses edge resources, may still have central components.
- **P2P network** (network-centric): fully decentralized, symmetric, without hierarchy.
An **overlay network** is a logical network on top of peers — 1 overlay hop can be multiple hops on the actual network layer, so minimize message passing.

## P2P systems

- **Napster**: central **indexing**, distributed file exchange → single point of failure/attack/lawsuit (was shut down).
- **Gnutella 0.4**: unstructured decentralized overlay, **flooding** with `PING`/`PONG` (peer discovery) and `QUERY`/`QUERYHIT` (search), TTL limits the flood. `PUSH` helps peers behind a firewall. File transfer itself via **HTTP**. Problem: **search horizon** (too low TTL = too few found, too high TTL = network flooded).
- **Gnutella 0.6**: **super-node** architecture — **ultrapeers** (strong nodes: discovery, routing, indices of leaves) and **leaf nodes** (connect to 1 ultrapeer, less work).
  - **Monitoring/spying**: as a regular peer you only see direct neighbors; as an ultrapeer also leaf queries. With **unlimited resources** an adversary can run many ultrapeers and thus observe a large portion of search traffic → scalability vs anonymity trade-off.
- **Chord (DHT)**: structured solution — consistent hashing (SHA-1) places nodes/files on a ring of 0..2^160-1, lookup in **O(log N)** hops via finger tables, guaranteed result (instead of TTL uncertainty with Gnutella). Hash function must be **uniformly distributed** (no hotspots) and **collision-free**; "virtual nodes" help with balance.
- **BitTorrent**: focused on **content distribution** (rather than discovery like Gnutella or routing like Chord). Discovery via web (torrent file) → **tracker** → **swarm**. File in **chunks** with **SHA-1 hash** (integrity + parallel downloading). **Rarest first**: spread rare chunks first to prevent bottlenecks. **Seeder** = peer with all chunks; regular peers are both down- and uploaders. **Tit-for-tat**: peers that don't contribute are **choked** (against free-riding). Newer versions use a **DHT (Kademlia)** instead of trackers.

## TOR: privacy and onion routing

**Onion routing**: message wrapped in multiple encryption layers, each router peels exactly 1 layer and knows only the previous/next hop — not the full route, origin, destination or content. Client = **Onion Proxy** (SOCKS interface, discovers routers via **directory server**, builds circuit). **Onion Routers** forward traffic or are **exit nodes**. **All links encrypted, except exit node → regular internet** (there traffic is plaintext if it's e.g. HTTP). Fixed **cell size** (control cells vs relay cells) makes traffic analysis harder. Circuit is built via `relay extend`; reverse path via **Circuit ID**. TOR primarily protects against **traffic analysis**, not against a global passive adversary (timing correlation remains possible) — more hops can even increase the chance of a compromised node.

**What does each node know?**

| | Source IP | Destination | Data | Previous hop | Next hop |
|---|---|---|---|---|---|
| **Entry** | yes | no | no | — | yes |
| **Middle** | no | no | no | yes | yes |
| **Exit** | no | yes | yes (plaintext!) | yes | — |

- **Minimum 3 nodes**: with 2 nodes, one node knows both source and destination → complete deanonymization.
- **Sybil attack**: attacker registers many fake nodes as TOR relays → increases chance that attacker controls both entry and exit → timing correlation.
- **Spying with limited resources**: exit nodes logging (sees destinations, not sources). With **unlimited resources** (state actor): global timing correlation between entry and exit.

---

## Common exam pitfalls / important to remember

- **SMTP is not a mailbox protocol**: it is a command/response **relay** protocol between servers. POP/IMAP are for client ↔ server (IMAP = sync, POP = local download).
- **DHCP must use UDP**: TCP requires a handshake with valid source/destination IPs, which the client does not yet have during DISCOVER. Core flow `DISCOVER → OFFER → REQUEST → ACK`, and **Duplicate Address Detection happens via ARP** (not via DHCP itself).
- **DNS names are not case-sensitive**, but URL paths can be. Know the meaning of `A`, `NS`, `MX`, `CNAME`. DNS lookup is a **hybrid**: recursive from the client/local resolver perspective, **iterative** higher in the hierarchy (root/TLD) — this hybrid model combines low client complexity with scalable, stateless root servers.
- **FTP uses 2 ports** (21 control, 20 data) — and **Active FTP fails behind NAT** because the server must set up an incoming connection to the client's private IP (NAT has no mapping for that). **Passive FTP (PASV)** solves this: the client initiates both connections (everything outbound).
- **P2P application ≠ P2P network**: Napster was a P2P app with central indexing, not a full P2P network. Know the difference between Gnutella 0.4 (flooding, search horizon problem), Gnutella 0.6 (ultrapeers/leaves) and Chord (DHT, O(log N), guaranteed but less anonymous/robust).
- **TOR**: at least **1 intermediate node** (3 nodes total) is needed so that no single node knows both source and destination. Only the link exit node → internet is unencrypted. More hops = more latency and potentially more chance of a compromised node (1 - 0.8^N), without protection against a global passive adversary.
