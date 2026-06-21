# Solutions Open Questions Exam Computer Networks (17 June 2026)

This document contains the answers to the open questions from the exam of June 17, 2026, based on the course material.

---

## 1. Closed Book

### 1.1 Data Link Layer

#### 1.1.1 Question 1 (10 marks)
**(a) Would listening or transmitting likely consume more energy on**
* **i. a star topology?**
* **ii. a mesh topology?**

**Be sure to explain your answer.**

##### i. Star topology
* **Answer:** **Transmitting** consumes the most energy here for the end devices (leaf nodes).
* **Explanation:** In a star topology, all leaf nodes communicate directly with a central gateway and bear no responsibility for forwarding other nodes' packets. Consequently, leaf nodes can spend most of their time in a low-power deep sleep mode. They only wake up to transmit data and possibly listen briefly for an ACK. Since the radios almost never need to actively listen to the medium (idle listening is virtually zero), the active energy consumption is dominated by transmitting. *(The gateway does listen continuously, but it is typically connected to the mains power grid).*

##### ii. Mesh topology
* **Answer:** **Listening (listening / idle listening)** consumes the most energy here.
* **Explanation:** In a mesh topology, nodes must cooperate to route packets for their neighbors over multiple hops. To make this possible, the nodes' radios must be active regularly or continuously (idle listening) to intercept incoming transmissions from neighbors. Even when using low-power MAC protocols such as B-MAC (with Low Power Listening), nodes cumulatively spend many times more energy/time periodically sampling the medium and listening for activity than actually transmitting their own data. Listening is the absolute primary cause of energy consumption here.

---

**(b) How do MAC protocols ensure low energy consumption? Give concrete MAC protocol examples and explain how they achieve this.**

* **Answer:** MAC protocols minimize energy consumption by addressing the four major causes of energy waste: **idle listening** (listening to an empty channel), **collisions** (collisions leading to expensive retransmissions), **overhearing** (unnecessarily listening to data intended for other nodes), and **control overhead** (transmitting control data).
* **Concrete examples from the course:**
  1. **B-MAC (Berkeley MAC) - Asynchronous approach:**
     * Utilizes **Low Power Listening (LPL)**. Nodes sleep most of the time and only wake up periodically for a very short duration (sample interval) to check for activity on the channel via Clear Channel Assessment (CCA).
     - If the channel is quiet, they immediately go back to sleep.
     - To send data, the sender transmits a **long preamble** that lasts at least as long as the receiver's sample interval (`Preamble_length >= Sample_Interval`). The receiver wakes up, detects the preamble, remains awake, and subsequently receives the data.
     - This avoids the need for complex time synchronization while keeping idle listening to a minimum.
  2. **TSMP (Time Synchronized Mesh Protocol) / TDMA - Synchronized approach:**
     * Divides time into tightly coordinated timeslots (**Time Division Multiple Access - TDMA**).
     - A central manager assigns specific slots to nodes for transmission and reception. All nodes are precisely synchronized.
     - Nodes turn on their radios only during their exactly scheduled slots and go straight to sleep mode outside of them.
     - This completely eliminates idle listening and prevents collisions (as slots are exclusively assigned), at the cost of overhead to maintain synchronization and join the network.

---

#### 1.1.2 Question 2 (5 marks)
**Given is a satellite connection with bandwidth 3 Mbps, frame size 1500 B, and RTT 250 ms. Calculate the throughput for stop-and-wait.**
**Which protocol would you use to improve the throughput? Explain why.**

##### 1. Calculation of the throughput for Stop-and-Wait:
* **Given:**
  * Bandwidth ($R$) = $3 \text{ Mbps} = 3 \times 10^6 \text{ bits/s}$
  * Frame size ($L$) = $1500 \text{ Bytes} = 1500 \times 8 \text{ bits} = 12,000 \text{ bits}$
  * Round Trip Time ($RTT$) = $250 \text{ ms} = 0.25 \text{ s}$
* **Transmission time of one frame ($T_{\text{trans}}$):**
  $$T_{\text{trans}} = \frac{L}{R} = \frac{12,000 \text{ bits}}{3,000,000 \text{ bits/s}} = 0.004 \text{ s} = 4 \text{ ms}$$
* **Total cycle duration per frame for Stop-and-Wait ($T_{\text{total}}$):**
  $$T_{\text{total}} = T_{\text{trans}} + RTT = 4 \text{ ms} + 250 \text{ ms} = 254 \text{ ms} = 0.254 \text{ s}$$
* **Throughput:**
  $$\text{Throughput} = \frac{L}{T_{\text{total}}} = \frac{12,000 \text{ bits}}{0.254 \text{ s}} \approx 47,244 \text{ bits/s} \approx 47.24 \text{ kbps}$$
* *(This corresponds to a link utilization of only $U = \frac{4 \text{ ms}}{254 \text{ ms}} \approx 1.57\%$.)*

##### 2. Protocol to improve the throughput:
* **Choice:** A **sliding window protocol**, specifically **Selective Repeat (SR)**.
* **Explanation:**
  * Stop-and-Wait performs extremely poorly on this link because the sender must wait idly $98.43\%$ of the time for the ACK over the long satellite connection.
  * A sliding window protocol solves this by allowing the sender to send multiple frames in succession without waiting for an ACK. To achieve $100\%$ efficiency, the window size must be $W \ge \frac{T_{\text{trans}} + RTT}{T_{\text{trans}}} = 63.5 \implies W \ge 64$ frames.
  * **Selective Repeat (SR)** is preferred here over **Go-Back-N (GBN)**. Given the large bandwidth-delay product ("long fat pipe"), a single lost frame in GBN would lead to the retransmission of the entire outstanding window (at least 64 packets), wasting valuable bandwidth. Selective Repeat retransmits only the lost frame and buffers out-of-order received frames at the receiver, which is optimal for satellite links.

---

### 1.2 Transport Layer

#### 1.2.1 Question 1 (10 or 12 marks)
**What specific features of RTP make it well suited for real-time live streaming applications, such as broadcasting a World Cup match to millions of viewers?**

* **Answer:**
  1. **UDP-based transport:** RTP runs on top of UDP. It avoids the latency of TCP handshakes and, crucially, automatic retransmissions. In live streaming, a packet that arrives too late is useless (the video frame has already passed). TCP's congestion control (AIMD) would halve the sending rate at the slightest packet loss, leading to stuttering video. UDP allows the application itself to decide how to handle loss.
  2. **Sequence Numbers (16-bit):** These allow the receiver to detect packet loss and correctly reorder out-of-order received packets in the playout buffer before they are passed to the video decoder.
  3. **Timestamps (32-bit):** These determine the exact playback time of the media units, independent of network delays (jitter compensation). This enables smooth playout and synchronization between different streams (such as audio and video / lip-sync) via RTCP.
  4. **Payload Type field (7-bit):** Indicates the compression format/codec being used (e.g., H.264, AAC). This allows the stream to dynamically adapt to changing network conditions during transmission by switching bitrates or codecs.
  5. **Multicast support:** RTP is designed to support IP multicast. With millions of concurrent viewers, unicast (as with TCP) would immediately overload servers and backbone links. Multicast ensures that the stream is sent only once and is duplicated by network routers where necessary.
  6. **SSRC (Synchronization Source) and CSRC (Contributing Source) identifiers:** SSRC uniquely identifies each media stream (e.g., specific camera angles). CSRC shows which audio streams have been mixed (e.g., stadium sound mixed with the commentator), letting the receiver know which sources are actively contributing.

---

#### 1.2.2 Question 2 (5 marks)
**Draw TCP Tahoe congestion control graph. Annotate to explain the different phases.**

With **TCP Tahoe**, the congestion window ($cwnd$) behavior is as follows:

```text
cwnd (MSS)
  ^
32|                                         *
  |                                      *
16|             *                     *        <-- ssthresh halved to 8
  |          *     *               *
 8|       *           *         *
  |    *                 *   *
 4|  *                     *
 2| *
 1|*
  +---------------------------------------------> Time (RTTs)
   |<- Slow Start ->|<- Cong Avoidance ->|
    (exponential)       (linear)
                    ^
                    Loss event (Timeout or 3 duplicate ACKs):
                    - ssthresh = cwnd / 2 (halved to 8)
                    - cwnd = 1 MSS
                    - Return to Slow Start
```

##### Explanation of the phases:
1. **Slow Start ($cwnd < ssthresh$):** The congestion window starts at 1 MSS and doubles every RTT (exponential growth). This continues until $cwnd$ reaches the threshold ($ssthresh$).
2. **Congestion Avoidance ($cwnd \ge ssthresh$):** From this point on, $cwnd$ increases linearly by 1 MSS per RTT to carefully probe the network for capacity.
3. **Loss Event (Timeout or 3 duplicate ACKs):** Unlike TCP Reno, TCP Tahoe reacts to **both** events in exactly the same way:
   * The `ssthresh` is halved to $\frac{1}{2}$ of the current $cwnd$.
   * The $cwnd$ is rigorously reset to **1 MSS**.
   * The sender restarts in the **Slow Start** phase from 1 MSS.

---

#### 1.2.3 Question 3 (5 marks)
**What is ECN? Give one advantage and one disadvantage compared to Choke Packets and RED.**

* **ECN Definition:** **Explicit Congestion Notification** is a mechanism at the network and transport layer levels (IP and TCP). When a router detects congestion in its queue, instead of dropping the packet (as with RED), it marks the ECN bits in the IP header (CE - Congestion Experienced). The receiver notices this and echoes the warning back to the sender via the ECE (ECN-Echo) flag in the TCP ACK. The sender responds by halving its transmission rate (as if packet loss had occurred) and acknowledges this via the CWR (Congestion Window Reduced) flag.

##### Comparison with Choke Packets:
* **Advantage over Choke Packets:** ECN generates **no extra network overhead**. Choke Packets require the router itself to generate and send back a new packet, which adds load to an already congested network. ECN simply marks existing packets.
* **Disadvantage compared to Choke Packets:** ECN reacts **slower**. A Choke Packet goes directly from the router to the sender (approx. 0.5 RTT). ECN must first travel to the receiver and only returns to the sender via the ACK (approx. 1 RTT).

##### Comparison with RED (Random Early Detection):
* **Advantage over RED:** ECN **prevents packet loss**. RED signals congestion by preventively dropping packets, which leads to retransmissions and latency (harmful for short data streams). ECN delivers the data and warns without loss.
* **Disadvantage compared to RED:** ECN requires **end-to-end support** in the TCP/IP stack of both the sender and the receiver. RED runs exclusively on the router; clients and servers do not need to be modified. This makes RED much simpler to deploy in practice (consequently, RED is widely deployed, whereas ECN is barely used).

---
---

## 2. Open book

### 2.1 Application Layer

#### 2.1.1 Question 1
**How can we censor DNS traffic? Be specific about which layers and which devices would need to be involved.**

Censorship of DNS traffic (port 53) can occur at three different layers:

##### 1. IP Blocking / Routing Redirection (Network Layer - Layer 3)
* **Devices:** Border routers of the ISP or national firewalls (BGP routers).
* **Method:**
  * **IP Blocking:** The IP addresses of known alternative or public DNS servers (such as Google's `8.8.8.8` or Cloudflare's `1.1.1.1`) are blocked on border routers. IP packets to these destinations are simply dropped.
  * **Routing Redirection:** All IP packets with destination port 53 (DNS) are intercepted by routers and forcibly redirected to a censored DNS server run by the local authority/ISP, regardless of the target IP entered.

##### 2. Port Blocking / Protocol Filtering (Transport Layer - Layer 4)
* **Devices:** Stateful firewalls, Deep Packet Inspection (DPI) gateways.
* **Method:** Blocking all outgoing UDP and TCP traffic on port 53 crossing the ISP border. This prevents users from reaching external DNS servers and forces them to use the local, censored ISP DNS resolvers.

##### 3. DNS Payload Inspection & Injection (Application Layer - Layer 7)
* **Devices:** Deep Packet Inspection (DPI) appliances, DNS Proxies.
* **Method:** Since traditional DNS traffic is unencrypted in plaintext, a DPI device scans the UDP payload for domain names on a blacklist.
  * **DNS Spoofing (Injection):** As soon as a forbidden query (e.g., to a censored news site) is detected, the DPI device immediately injects a forged DNS response back with an incorrect IP (e.g., `127.0.0.1` or the IP of a blocking page). Because this fake response (injected close to the user) arrives faster than the real response from the legitimate DNS server, the client accepts it and ignores the subsequent real response.
  * **DNS Sinkholing:** The local DNS server refuses to resolve the query and returns a manipulated response (such as NXDOMAIN).

---

#### 2.1.2 Question 2
**How would a copyright enforcement agency go about monitoring BitTorrent activity to detect illegal file sharing?**

A copyright agency uses an **active monitoring strategy** to collect the IP addresses of infringers:

1. **Obtain the Infohash:** The agency searches public torrent indexing websites for the copyrighted file and extracts the unique **infohash** (cryptographic hash) from the `.torrent` file or the magnet link.
2. **Locate the Swarm:** To find active downloaders/uploaders, the agency requests peer lists through BitTorrent's three channels:
   * **Trackers:** They send an HTTP/UDP `announce` with the infohash to the torrent's trackers. The tracker responds with the IP addresses and ports of peers in the "swarm".
   * **DHT (Distributed Hash Table):** They query the Kademlia DHT using the infohash as a search key to find peers downloading trackerless.
   * **PEX (Peer Exchange):** They query already connected peers via PEX messages for the IP addresses of other peers they are connected to.
3. **Actively Connect and Handshake:**
   * The agency uses its monitoring software to directly establish TCP connections (or UDP uTP connections) with the gathered IP addresses on their BitTorrent ports.
   * They perform the BitTorrent handshake using the infohash.
   * They exchange `HAVE` or `BITFIELD` messages. This proves that the target IP possesses and is offering specific pieces of the illegal file.
4. **Collect Evidence:**
   * The agency often actually downloads a small piece of the file from the peer to legally prove beyond doubt that the peer is actively uploading data.
   * They log the IP address, port, timestamp, infohash, client type, and downloaded bytes. This log file is used to request the identity of the infringer from the ISP, often via legal proceedings.

---

### 2.2 Network Layer

#### 2.2.1 Question 1
**Give 4 network layer concepts that are also used in Transport or Data Link layer. Why this redundancy? Is the overhead justified?**

##### Four redundant concepts:
1. **Addressing:**
   * *Network:* IP addresses (NSAP) to route packets across network boundaries.
   * *Transport:* Port numbers (TSAP) to distinguish specific applications/processes on a host.
   * *Data Link:* MAC addresses to identify physical network interface cards on the same local medium.
2. **Error Detection:**
   * *Network:* IPv4 header checksum (omitted in IPv6).
   * *Transport:* TCP/UDP checksum over the entire payload and header (end-to-end).
   * *Data Link:* CRC / Frame Check Sequence (FCS) in frames to immediately drop bit errors on the physical medium.
3. **Flow / Congestion Control:**
   * *Network:* Congestion notifications such as RED and ECN (in routers).
   * *Transport:* TCP sliding window (flow control) and $cwnd$ (congestion control).
   * *Data Link:* RTS/CTS handshakes, backoff timers, or stop-and-wait flow control.
4. **Fragmentation / Segmentation:**
   * *Network:* IP fragmentation by routers when the link MTU is exceeded.
   * *Transport:* TCP segmentation into MSS-sized units by the sending host.
   * *Data Link:* Framing (bit/byte stuffing and preambles to divide a data stream into frames).

##### Why this redundancy?
* **The End-to-End Argument (Saltzer):** Error control or reliability at lower layers (such as the data link layer) is never sufficient to guarantee end-to-end correctness. A packet can become corrupt in a router's memory, or a router might crash. Only the endpoints (transport layer) can guarantee that the data arrived correctly.
* **Local Efficiency:** Relying *only* on the transport layer for error correction is terrible for performance on lossy links (like Wi-Fi). If a frame drops on the first hop, it is much faster to recover it locally at the data link layer (Wi-Fi ARQ) than to wait for an RTT-long TCP timeout.

##### Is the overhead justified?
* **Yes, the overhead is completely justified.**
  * The header overhead (e.g., 20B IP + 20B TCP + 18B Ethernet) is minuscule compared to typical payloads (1500B).
  * It guarantees modular independence: the data link hardware does not need to understand IP routing, and applications do not need to concern themselves with the physical carrier of the bits.
  * It provides an optimal compromise between **fundamental reliability** (end-to-end in the transport layer) and **fast performance** (hop-by-hop in the data link layer).

---

#### 2.2.2 Question 2
**What adaptations would be needed to run OSPF – which is normally used in LAN networks — on unreliable wireless nodes that do not move (e.g. IoT devices in a building)?**

OSPF is designed for stable, wired LAN environments with ample resources. To run OSPF successfully on unreliable, stationary wireless IoT nodes in a building, the following adaptations are necessary:

1. **Hysteresis and Link-Quality Metrics (against lossy links):**
   * *Problem:* Wireless links often drop briefly due to interference or moving objects. Standard OSPF reacts to this immediately by flooding Link-State Advertisements (LSAs). This causes a constant LSA storm that drains the limited bandwidth and battery of IoT nodes.
   * *Adaptation:* Implement **damping/hysteresis** on neighbor state transitions. Declare a link "down" only after a longer period of inactivity. Additionally, use link-quality metrics like **ETX (Expected Transmission Count)** instead of static, bandwidth-based link costs.
2. **Reduced HELLO overhead (against battery depletion):**
   * *Problem:* OSPF routers continuously send periodic HELLO packets to detect neighbors. This forces IoT radios to stay on constantly (idle listening), which quickly drains batteries.
   * *Adaptation:* Significantly increase the HELLO interval ("slow hello"), or link neighbor detection to existing link-layer beacons from the MAC protocol to avoid redundant overhead.
3. **Pruned / Directed LSA Flooding (against broadcast storms):**
   * *Problem:* OSPF uses flooding (reliable broadcast) to distribute link-state info. On a shared wireless medium, blind flooding leads to massive collisions.
   * *Adaptation:* Use a pruned flooding mechanism, such as **Multipoint Relays (MPRs)** (similar to OLSR), where only a selected subset of nodes is responsible for forwarding LSAs.
4. **Hierarchical Areas and LSDB restriction (against memory limits):**
   * *Problem:* OSPF requires every router to maintain a complete Link-State Database (LSDB) of the topology and run the computationally intensive Dijkstra algorithm on it. IoT nodes lack sufficient RAM and CPU power for this.
   * *Adaptation:* Divide the building into very small, strict **OSPF Areas (stub areas)**. This ensures that IoT nodes only need to store the local topology and use a simple default route to the gateway for external traffic, minimizing memory and CPU usage.
