# Transport Layer (Layer 4) — Short Summary

The transport layer provides **end-to-end delivery** between processes on different hosts: data is packaged into **segments** and forms the boundary between the application world and the network. It tries to be efficient, reliable and cost-manageable at the same time.

## Connectionless vs connection-oriented

- **Connectionless**: fast, simple, little overhead, but limited error control and no flow control (much responsibility falls on the application).
- **Connection-oriented**: more overhead, but much built-in support, with three phases: **establishment → data transfer → release**.

## Addressing: TSAP vs NSAP

- **NSAP** = network layer endpoint (IP address), **TSAP** = transport layer endpoint (port). One IP address can have many TSAPs/connections (analogy: one phone number, multiple extensions).
- **Berkeley sockets**: server does `SOCKET → BIND → LISTEN → ACCEPT → SEND/RECEIVE → CLOSE`; client does `SOCKET → CONNECT → SEND/RECEIVE → CLOSE`.
- **ICP (Initial Connection Protocol)**: a process server listens on a well-known TSAP and only starts the actual server upon an incoming connection (e.g. `inetd`) — saves resources.

## Connection management: fundamental problems

- Packets can arrive **out-of-order**, delayed or **duplicated** — an old, delayed packet can reappear later (classic example: double-executed bank transfer).
- **Bounded packet lifetime**: sequence numbers are not reused too quickly + real-time clock after a crash, so that old sequence numbers are not accidentally reused.
- **Two generals problem**: even with ACK exchange you can never know with absolute certainty that both sides have the same knowledge of the connection state — **there is no perfect solution**.

## Error control

- **End-to-end checksum** is needed even if lower layers already do checksums, because errors can also arise **inside a faulty router** (hop-by-hop is not enough).
- Basic logic: sender sends segment → receiver ACKs → no ACK within timeout → retransmission. Sender must keep unACKed data.

### Retransmission: where does it belong?

**End-to-end argument**: retransmission in the transport layer is the only one that guarantees end-to-end correctness. Data link layer retransmission (hop-by-hop) improves performance on bad links but guarantees nothing end-to-end. Application layer retransmission gives the app maximum control but duplicates transport layer functionality.

## Flow control: sliding window protocols

Flow control = how much data may be "in flight" so that a **slow receiver** is not overwhelmed (vs. congestion control = protection of the **network**).

| Protocol | Window size | Out-of-order | Retransmission on loss |
|---|---|---|---|
| **Stop-and-wait** | 1 | n/a | only that segment |
| **Go-Back-N** | sender: W, receiver: 1 | discarded | segment k **and all following** in window |
| **Selective Repeat** | sender: W, receiver: W | buffered | only the missing/damaged segment |

- **Stop-and-wait**: data/ACK alternate between 0 and 1, sender starts timer; **piggybacking** = sending ACK along with own data. Inefficient at high RTT (high a).
- **Go-Back-N**: cumulative ACKs (ACK for n implies everything ≤ n is good). On timeout: everything from the lost segment onward is resent → can cause much **unnecessary retransmission**.
- **Selective Repeat**: more efficient, but requires buffers on **both sides** and NACKs. **Important rule**: window size ≤ **half of the sequence number space**, otherwise the receiver confuses duplicates with new segments.

### Efficiency formulas (sliding window)

- **a = (RTT/2) / T_trans** (propagation delay relative to transmission time).
- Stop-and-wait: **η = 1 / (1 + 2a)** — with satellite (RTT=2600ms, a≈956) you get η ≈ 0.05%, virtually unusable.
- Go-Back-N / Selective Repeat: for η ≈ 100% you need **W ≥ 1 + 2a** (in the same example: W ≥ ~1913 packets). Otherwise η ≈ W / (1 + 2a).
- Stop-and-wait is only good when **a ≈ 0**, i.e. bandwidth-delay product < 1 packet (slow links over short distance, LoRa, RS-232).

## Congestion control

- Goals: utilize bandwidth, avoid congestion, be fair, react quickly.
- **Goodput** (useful, arrived data) ≠ **throughput** — under overload, losses/retransmissions can cause goodput to drop → **congestion collapse**.
- **Kleinrock's power = load / delay**: more load is not always better if delay increases disproportionately.
- **Min-max fairness**: you cannot give flow A more bandwidth without disadvantaging flow B.
- **AIMD** (Additive Increase, Multiplicative Decrease): cautiously increase linearly, aggressively decrease multiplicatively upon congestion — easy to push the network into congestion, harder to get out.

## UDP

UDP (RFC 768) is **connectionless** and lightweight:

- **Does not provide**: flow control, ordering, congestion control.
- **Does provide**: ports (TSAP addressing) and checksum (calculated over UDP header + IP **pseudo-header**).
- Useful for: low overhead, no connection setup, **anycast/broadcast/multicast** (DNS, DHCP).

## TCP: reliable byte stream

TCP provides a **reliable, end-to-end byte stream** (not a message service — message boundaries are lost). Connection = **full duplex**, between exactly two sockets, identified by (src IP, src port, dst IP, dst port, protocol).

- **Sequence number** = sequence number of the **first byte** in the segment; **ACK number** = next expected byte (cumulative — ACK=2048 means "everything up to and including byte 2047 received").
- Important flags: **SYN/FIN** (setup/teardown), **ACK**, **RST** (abrupt reset), **PSH** (deliver quickly), **URG**, **ECE/CWR** (ECN).
- **Window size** field = core of flow control: how many bytes the receiver can still accept.

### 3-way handshake

```
client → SYN, SEQ=x          (SYN=1, ACK=0)
server → SYN, ACK, SEQ=y, ACK=x+1   (SYN=1, ACK=1)
client → ACK, ACK=y+1        (SYN=0, ACK=1)
```

Three steps are needed so both sides know the other is reachable, know the sequence numbers, and both directions are ready.

### Closing

Two simplex directions are closed separately via **FIN**. Then **TIME_WAIT** (≈ 2× max packet lifetime) so old segments can "die out" — prevents old packets from ending up in a new connection with the same socket combination. TCP is a **state machine with 11 states** (CLOSED, LISTEN, SYN_SENT, ESTABLISHED, TIME_WAIT, ...).

### Sliding window & flow control

- ACK and buffer allocation are decoupled: `ACK=4096, WIN=0` means "everything up to 4095 received, but no buffer available" → sender may not send anything new.
- **Window probes**: if advertised window is 0 and the update notification is lost, a window probe prevents a **deadlock**.

### Tinygram syndrome / Nagle / Silly Window Syndrome

- **Delayed ACKs**: receiver waits (up to ~200ms) to piggyback the ACK.
- **Nagle's algorithm**: sender buffers small data as long as there is an unacknowledged segment in transit — bad for interactive apps (games, SSH); disable via **TCP_NODELAY**.
- **Silly Window Syndrome / Clarke's algorithm**: receiver waits with window updates until at least one MSS fits or buffer is half empty. Nagle (sender) and Clarke (receiver) are complementary.
- **Interaction**: Nagle + delayed ACK can systematically add ~200ms latency for small interactions (no real deadlock, but structural delay). Solution: `TCP_NODELAY` and/or `TCP_QUICKACK`.

### Timers

| Timer | Purpose |
|---|---|
| **RTO** (retransmission timeout) | when to retransmit if no ACK; based on **SRTT** + **RTTVAR** |
| **Persistence timer** | sends window probe when WIN=0 |
| **Keep-alive timer** | checks if peer is still alive (optional) |
| **TIME_WAIT timer** | lets old segments die out after closing |

### Congestion control in TCP

- Effective send limit = **min(receiver window, congestion window)** — receiver window protects against a slow receiver, congestion window against a full network.
- TCP uses **packet loss** as an implicit congestion signal (works well on wired networks, less so on wireless).
- **ACK clock**: new data is sent at the rate ACKs come back, and that rate is determined by the **bottleneck** — self-regulating, prevents bursts.
- **Slow start**: congestion window grows **exponentially** (doubles per RTT) until **slow-start threshold**, then **additive increase** (AIMD). After congestion, threshold ≈ half of current congestion window.
- **Tahoe**: fell back hard on loss. **Reno**: added **fast recovery** via duplicate ACKs — retransmit quickly, fall back to around the new threshold, continue with additive increase (works well mainly with **one** loss).
- **SACK**: receiver reports which byte ranges have been received, so sender can retransmit selectively with **multiple losses** — via header options, so compatible.
- **ECN**: routers mark packets before a drop (instead of packet loss as signal); TCP reacts via **ECE/CWR** flags. Less widespread than SACK, as it requires support from both end hosts and the network.

**ECN vs RED comparison**: RED drops randomly early with rising queue → implicit signal, only router change needed (widely deployed). ECN marks instead of drops → explicit signal, no data loss, but requires support from both end hosts + routers. RED hits faster senders relatively more often → fairer; ECN is friendlier but less deployed.

## RTP (brief, for context)

RTP (RFC 3550) typically runs on top of UDP and focuses on **playback**, not perfect delivery: **too late is often worse than lost**. Sequence number (16-bit, per packet) detects loss/ordering; **timestamp** (32-bit) says when something should be played. **Jitter** (variation in delay) is handled with a **playback buffer** — trade-off between low delay and little loss due to late arrival. RTCP provides feedback (delay, jitter, bandwidth, sync).

## QUIC (brief)

QUIC (RFC 9000, ~30% of internet traffic) is **frame-based**, runs in **userspace on top of UDP**, with built-in security (TLS 1.3 integration, sometimes **0-RTT**), **stream multiplexing** (solves **head-of-line blocking** at the connection level) and **connection migration** via **Connection IDs** (CID) — IP/port may change without breaking the connection (validated via PATH_CHALLENGE/PATH_RESPONSE; congestion state is reset upon migration). Sequence numbers are 62-bit (wraparound is no longer an issue). Congestion control based on **Cubic (RFC 8312)**.

---

## Common exam pitfalls / important to remember

- **Flow control ≠ congestion control**: flow control protects the receiver (window/buffer), congestion control protects the network (congestion window, AIMD).
- With **Go-Back-N** the receiver has window 1 and on timeout **everything from the lost segment onward** is resent — with **Selective Repeat** only the missing part, but with window size constraint ≤ half of the sequence space.
- Always calculate with **a = (RTT/2)/T_trans**: stop-and-wait is disastrous at high a (e.g. satellite), and Go-Back-N/SR then need W ≥ 1+2a for 100% efficiency.
- **Two generals problem**: no perfect solution possible — absolute mutual certainty about connection state is not enforceable.
- **End-to-end checksums** remain necessary despite lower-layer checksums, because errors can arise inside routers.
- Know the **3-way handshake** with exact SYN/ACK values, and the difference between RTO/persistence timer/TIME_WAIT.
- RTP/UDP has no ACK clock and no congestion control → can "starve" TCP flows (unfairness) and contributes to congestion collapse under high load. RTP deliberately uses no flow/congestion control because **too late is worse than lost** for real-time media, and multicast (1-to-N) makes it impractical to throttle per receiver.
- QUIC is not "TCP 2.0": it is a new protocol with its own connection identification (CID), framing and integrated security — not just an extension. Key differences: **0-RTT** setup (replay risk!), **stream multiplexing** (no HoL blocking at connection level), **connection migration** via CID (IP/port may change).
- **Retransmission belongs in the transport layer** (end-to-end argument): hop-by-hop retransmission improves performance but guarantees nothing end-to-end.
