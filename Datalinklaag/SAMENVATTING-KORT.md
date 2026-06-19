# Cheat Sheet: Data Link Layer (Layer 2) & Physical Layer (Layer 1)

The **physical layer** ensures that bits actually travel over a medium (copper, fiber, radio). The **data link layer** builds on top of that: it delineates a bit stream into **frames**, detects/corrects errors, and controls **who may transmit when** on a shared medium (MAC).

## Framing: where does a frame begin/end?

- **Byte count**: a length field at the front says how many bytes follow. Compact, but **weak point**: if that length field gets corrupted, you lose synchronization completely — even a checksum won't save you then (you no longer know what it applies to).
- **Flag bytes**: a special byte value marks start/end. Problem: that value can accidentally occur in the data.
  - **Byte stuffing**: add an escape byte before every flag byte (and escape byte) that occurs in the data.
  - **Bit stuffing** (more efficient): with flag pattern `01111110` the sender inserts a `0` after **5 consecutive 1-bits**; the receiver removes that `0` again. This way the flag pattern can never accidentally appear in the payload.

## Physical layer: media

| Medium | Key point |
|---|---|
| **Coax** | solid core + shielding, robust against interference, but less flexible |
| **Twisted pair** (Cat5/6/7) | two wires twisted together → magnetic disturbances largely cancel out; higher categories = more shielding/twists |
| **Optical fiber** | uses **light** + **total internal reflection** → high bandwidth, long distance, little EM interference, but more expensive/delicate |
| **Wireless** | shared, invisible medium; extra challenges: RF interference, blocked paths (metal/water/vegetation), limited bandwidth, mobility, energy |

**Frequency trade-off**: low = better range (less path loss), high = smaller antennas but more atmospheric attenuation; plus legal restrictions.

**Satellites** (latency depends on orbit height):
- **GEO**: ~270 ms, small number of satellites, fixed dishes, large coverage area.
- **MEO**: ~35–85 ms, more dynamics/tracking needed.
- **LEO** (Iridium, Starlink): lowest latency, but many satellites needed + handover complexity.

**RFID**: reader transmits energy → passive tag becomes active (near field) and responds via **backscatter** (modulates reflection of the carrier wave, does not transmit its own signal). Active tags (with battery) achieve greater range but are more expensive.

## MAC: the channel allocation problem

One shared medium, who may transmit when?
- **Static allocation** (fixed wires/slots/frequencies, e.g. FM radio): simple but **wasteful** with bursty traffic.
- **Dynamic allocation**: 5 key questions — is traffic independent? one or multiple channels? are collisions observable? continuous or slots? carrier sense possible?

### Random access protocols

- **Pure ALOHA**: transmit whenever you want; on collision → **random** wait time (randomization breaks symmetry). Vulnerable period = **2T**, max throughput **18%** (S = G·e^(-2G)).
- **Slotted ALOHA**: transmission only at slot boundaries → vulnerable period = **1T**, max throughput **37%** (S = G·e^(-G)). Requires **synchronization**.
- **Can Pure ALOHA run on 802.11?** Physically yes, but pointless: wifi has carrier sense, and ALOHA's blind transmission wastes the advantage of CSMA.
- **CSMA**: listen first (carrier sense) before transmitting. Does not fully solve collisions due to propagation delay and **hidden terminals**.

| Variant | Behavior when channel is free |
|---|---|
| **Non-persistent** | transmit; if busy → wait random time, try again later |
| **1-persistent** | wait until free, then transmit immediately (probability 1) — aggressive, higher chance of simultaneous senders |
| **p-persistent** | transmit with probability p, otherwise wait and try again — finer control |

**Hidden terminal**: A and C cannot hear each other, yet collide at common receiver B — carrier sense does not solve this (listening locally ≠ channel actually free).
**Exposed terminal**: C hears B transmitting and waits unnecessarily, while C→D would not disturb anyone.

### Energy-efficient wireless MAC

- **BMAC (Low Power Listening)**: radio is usually off, samples briefly at intervals. Sender transmits **long preamble** so that a periodically sampling receiver is guaranteed to detect activity. Condition: `Preamble_length ≥ Sample_Interval`. Asynchronous, decentralized → good scalability and suitable for **unpredictable/bursty** traffic, but with heavy traffic each preamble costs extra energy.
- **TSMP (TDMA)**: strongly **synchronized**, central manager assigns time slots → almost no collisions/retransmissions, radio can be mostly off. Also uses **frequency hopping** (spreads interference risk) and redundancy (**spatial**: different neighbor, **temporal**: different time). Good for **predictable** traffic and QoS (e.g. more slots for camera than for temperature sensor). Biggest energy cost: **(re)joining** the network. Limitations: manager = bottleneck, worse for sporadic traffic (higher latency).

## Ethernet (IEEE 802.3)

- Classic Ethernet = **1-persistent CSMA/CD** (Collision Detection) — fits well with wired medium where you can detect interference during transmission.
- **Binary exponential backoff**: after a collision the range from which you pick your random wait time grows → few collisions = short wait, many collisions = system automatically becomes more cautious.
- **Minimum frame size (64 bytes = 512 bits)** is tied to collision detection: the sender must still be transmitting when the collision signal from the farthest point returns. Requirement: `T_transmission ≥ 2×T_propagation`. At 10 Mbps this gives max ~2560 m cable length.
- **Frame format**: preamble (clock synchronization), start-of-frame, source/destination **MAC address (48-bit)**, type-or-length field, broadcast address = all bits 1.
- **DIX vs 802.3**: value `≤ 1500` = length field, value `> 1500` = type field (tells which higher protocol — IPv4, ARP — to pass the frame to).
- **Hub** = simple repeater, remains one shared medium/collision domain. **Switch** = looks at destination address and forwards only to the correct port → own collision domain per port, more capacity/privacy.
- **Full-duplex switched**: transmit/receive on separate pair (UTP) → collisions physically impossible, CSMA/CD off. Only remaining limit = **signal attenuation** (~100 m Cat5e), no longer timing.
- **Fast/Gigabit/10G Ethernet**: speed up → CSMA/CD timing becomes increasingly difficult (shorter cables needed); Gigabit uses **carrier extension** (artificially lengthen short frames) and **frame bursting**. 10G = fully full-duplex, no hubs.
- **Promiscuous mode**: host keeps frames not intended for it → sniffing possible on shared segment.

## 802.11 (WiFi): CSMA/CA

WiFi cannot reliably listen while transmitting (own signal dominates) → **Collision Avoidance** instead of Detection.

| | CSMA/CD (Ethernet) | CSMA/CA (802.11) |
|---|---|---|
| Detects collision | During transmission, on the cable | Not reliably possible |
| Strategy | Detect & stop | Avoid beforehand + confirm afterward (ACK) |
| Start after free channel | Immediately (1-persistent) | First a small **random backoff** (room for ACKs) |
| Extra mechanisms | Binary exponential backoff | NAV, RTS/CTS, backoff |

- **ACK**: sender transmits frame, receiver confirms; if ACK fails to arrive → sender suspects loss/collision.
- **NAV (Network Allocation Vector)** = virtual carrier sense: the **Duration** field in the header says how long the channel is still busy (data+ACK); other nodes remain silent until NAV = 0, even if they don't hear the actual signal.
- **RTS/CTS**: solves **hidden terminal** — A sends RTS to B, B sends CTS back; C hears B's CTS (even though C didn't hear A's RTS) and sets its NAV. Does **not solve exposed terminal** (C sometimes waits unnecessarily, actually becomes slightly worse). Has overhead (2 extra frames) → only worthwhile for **large frames** (RTS threshold, default 2347 bytes = often disabled).
- **Power saving**: (1) beacon-based — AP periodically announces whether there is buffered data, client sleeps in between; (2) client-triggered — AP holds traffic until client becomes active itself.
- **Frame**: more extensive header than Ethernet — frame control (type/subtype, **to DS/from DS**, more fragments, retry, power management...), **fragmentation** (smaller pieces at high error rate), **sequence number** (duplicates/retries), checksum.
- **Service types**: Ethernet = **unacknowledged connectionless**; 802.11 = **acknowledged connectionless**.

## Stop-and-Wait analysis

**η = 1 / (1 + 2a)** with a = propagation delay / transmission time. Good at **a ≈ 0** (short Ethernet, LoRa — slow link, short distance). Disastrous at high a (GEO satellite: a ≈ 956 → η ≈ 0.05%).

## MAC protocols with AODV

| Good | Bad |
|---|---|
| **CSMA/CA** (asynchronous, decentralized) | **TSMP** (central manager = bottleneck, synchronization impractical) |
| **B-MAC** (LPL, event-driven, scalable) | **Pure ALOHA** (no carrier sense, high loss rate) |

## LoRa MAC

LoRa = long range, low power, but **Pure ALOHA-like** → poor at high density (collisions). Improvements: TDMA slots, channel hopping, ADR (adaptive data rate). Hidden terminal is a major problem due to long range.

## Channel hopping (TSMP)

Advantages: interference resilience (interference affects only 1 frequency), multipath fading mitigation, security (harder to jam), fair spectrum distribution.

## Important formulas/numbers

- Minimum Ethernet frame: **512 bits**, requirement `T_transmission ≥ 2×T_propagation`.
- BMAC: `Preamble_length ≥ Sample_interval`; `E_tx ∝ T_preamble + T_data`.
- Bit stuffing flag: `01111110`, stuff after 5 ones.
- DIX/802.3 boundary: **1500** (≤ = length, > = type).
- MAC address: **48 bit**, broadcast = all bits 1.

## Common exam pitfalls / important to remember

- **Carrier sense does not solve hidden/exposed terminal** — this is a fundamental, repeatedly tested exam point: locally "I hear nothing" ≠ "channel is free".
- **RTS/CTS solves hidden terminal, but not exposed terminal** (actually makes it slightly worse) — and is only beneficial for large frames due to the overhead.
- **Hubs vs switches**: hub = no new collision domain, limit becomes stricter at higher speeds (Fast Ethernet+hub: ~100-200m). Switch + full-duplex: collisions physically impossible, CSMA/CD off, only limit = attenuation (~100m). **When is a hub better?** Protocol analysis/sniffing (promiscuous mode sees all traffic), lab/education, and cheaper for small network without privacy requirements.
- **Packet loss on WiFi ≠ congestion**: TCP (Tahoe/Reno) interprets loss as a congestion signal and halves the congestion window, but on 802.11 loss can come from interference, hidden terminals, fading or exhausted MAC retransmissions (up to 7x) — leads to unnecessary throughput reduction. Solutions: ECN, TCP Westwood, QUIC, link-layer ARQ.
- **BMAC vs TSMP**: BMAC = unpredictable/bursty/event-driven traffic, no synchronization needed, scalable. TSMP = predictable/periodic traffic, requires synchronization + central manager, best energy efficiency but biggest energy cost is in (re)joining.
- **Byte count framing**: vulnerable because an error in the length field derails the entire synchronization — a checksum won't save this, because you no longer know what it applies to.
