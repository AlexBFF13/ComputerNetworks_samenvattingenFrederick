# Computer Networks — Examenoverzicht per laag

> **Doel:** Snel inzicht in welke onderwerpen het vaakst terugkomen op het examen, zodat je gericht kunt studeren.
> **Bronnen:** Wiki-examen, opgeloste examenvragen, Ward & Kevin bundel, Modelexamen 2024 (Maxim), examen 10 juni 2025, examen 24 juni 2025.
> **Jaren geanalyseerd:** 2013–2025 (14 examenmomenten)

---

## 1. Application Layer

| # | Onderwerp | Aantal keer | Jaren |
|---|-----------|:-----------:|-------|
| 1 | **Gnutella 0.4 / 0.6** — search (PING/PONG/QUERY/QUERYHIT), routering, TTL, verschillen 0.4 vs 0.6, monitoring/spying, NAT/PUSH | ~12 | '13, '14, '15, '17, '18, '20, '22, '24, '25-1, '25-2 |
| 2 | **DNS** — recursief vs iteratief, waarom UDP i.p.v. TCP, diagram, efficiëntie van split design | ~8 | '14, '15, '18, '21, '22, '25-1, '25-2 |
| 3 | **DHCP** — werking (diagram), waarom UDP niet TCP, lease/renew/decline | ~8 | '14, '15, '17, '18, '21, voorbeeldexamen |
| 4 | **OSI / TCP-IP / Tanenbaum stacks** — tekenen, protocol per laag plaatsen, RTP/DHCP layer-discussie | ~7 | '18, '20, '21, '22, '25-1, '25-2 |
| 5 | **Chord** — DHT, Gnutella over Chord, hash-vereisten, vergelijking privacy | ~5 | '13, '15, '17 |
| 6 | **TOR** — entry/exit/middle nodes, info per node, spying (limited vs unlimited resources), Sybil attack | ~4 | '13, '14, '15, '24 |
| 7 | **HTTP 1.0 vs 1.1** — persistent connections, pipelining, wanneer voordeel het grootst | ~3 | '14, '15, '17 |
| 8 | **FTP** — waarom aparte control/data verbinding, passive mode, NAT-probleem | ~2 | '17, '18 |
| 9 | **UDP-applicaties** — 3 apps die UDP gebruiken + waarom geen TCP (Wake on LAN, DNS, DHCP...) | ~3 | '15, '20 |
| 10 | **P2P chat-applicatie ontwerpen** (Napster/Gnutella/BitTorrent) | 1 | '24 |

**Topprioriteit voor studeren:** Gnutella (0.4/0.6 verschillen + message types), DNS (split design + waarom UDP), DHCP (werking + waarom geen TCP), en OSI-layering.

---

## 2. Transport Layer

| # | Onderwerp | Aantal keer | Jaren |
|---|-----------|:-----------:|-------|
| 1 | **TCP Flow Control** — sliding window, window advertisement, window probe, deadlock, buffer management | ~10 | '14, '15, '17, '18, '21, '22, '24, '25-2, voorbeeldexamen |
| 2 | **Sliding Window Protocols** — stop-and-wait, go-back-N, selective repeat, beperkingen sequence numbers, vergelijking | ~6 | '14, '15, '17, '20, voorbeeldexamen |
| 3 | **Silly Window Syndrome + Tinygram** — definitie, Nagle's algorithm, delayed ACKs als oplossing, interactie Nagle + delayed ACKs | ~6 | '18, '20, '21, '25-1, voorbeeldexamen |
| 4 | **Delayed ACKs + Nagle's algorithm** — werking, waarom nuttig, interactie (deadlock!) | ~4 | '13, '14, '15, '21 |
| 5 | **RTP** — features, verschil met TCP, header velden, waarom geen flow/congestion control, multicast | ~4 | '15, '17, '22, voorbeeldexamen |
| 6 | **TCP Congestion Control** — Tahoe/Reno diagram, slow start, AIMD, threshold | ~4 | '17, '22, '24, '25-1 |
| 7 | **ECN vs RED** — werking ECN, voordelen/nadelen t.o.v. RED | ~3 | '18, '24, '25-1 |
| 8 | **Retransmission** — waarom in transport layer, voordelen in data link / application layer | ~3 | '13, '15, '20 |
| 9 | **QUIC** — ontwerp, verschil met TCP (streams, 0-RTT, connection migration, congestion control) | ~2 | '24, '25-1 |
| 10 | **TCP header** — alle velden uitleggen | ~2 | '21, '22 |
| 11 | **ACK clock** — werking, smoothing traffic | ~2 | '14, '17 |
| 12 | **Leaky bucket** — werking, R en B, grafiek tekenen | ~1 | '15 |

**Topprioriteit voor studeren:** TCP Flow Control (absoluut #1!), Silly Window + Tinygram + Nagle, sliding window protocolvergelijking, en TCP congestion control (Tahoe/Reno diagram).

---

## 3. Network Layer

| # | Onderwerp | Aantal keer | Jaren |
|---|-----------|:-----------:|-------|
| 1 | **IPv6 autoconfiguratie / privacy / security** — MAC-based SLAAC, waarom onveilig, vergelijking DHCPv6, experiment ontwerpen | ~9 | '13, '14, '15, '17, '24, '25-1, '25-2 |
| 2 | **IP addressing / Subnetting** — broadcast adres berekenen, CIDR, hostbereik, adrestype herkennen | ~6 | '15, '20, '22, '25-1, '25-2 |
| 3 | **Congestion Notifications vergelijking** — RED vs ECN vs Choke Packets (snelheid, efficiëntie, invloed) | ~6 | '14, '15, '21, '22, voorbeeldexamen |
| 4 | **AODV** — werking (ROUTE_REQUEST/REPLY), reactief vs proactief, wanneer beter dan OSPF | ~6 | '17, '20, '22, '24, '25-2, voorbeeldexamen |
| 5 | **MTU / IP Fragmentation** — onverenigbaarheid, ICMP rol, DF-flag, transparant vs niet-transparant | ~5 | '17, '18, '24, voorbeeldexamen |
| 6 | **Distance Vector Routing / Count-to-infinity** — werking, probleem, hoe AODV het oplost | ~5 | '13, '15, '22, voorbeeldexamen |
| 7 | **BGP vs OSPF** — intradomain vs interdomain, link state vs path vector, TCP vs UDP | ~4 | '13, '15, '22, voorbeeldexamen |
| 8 | **NAT** — problemen op application layer, technieken om te omzeilen | ~2 | '15, '25-1 |
| 9 | **IoT en IP stack** — IPv4 vs IPv6 voor IoT, voor/nadelen IP op 802.15.4 | ~2 | '18, voorbeeldexamen |
| 10 | **Round Robin fair queuing** — werking, misbruik | ~1 | '14 |

**Topprioriteit voor studeren:** IPv6 autoconfiguratie/privacy (bijna elk jaar!), IP subnetting oefeningen, congestion notification vergelijking (RED/ECN/Choke), en AODV werking.

---

## 4. Data Link & Physical Layer

| # | Onderwerp | Aantal keer | Jaren |
|---|-----------|:-----------:|-------|
| 1 | **B-MAC vs TSMP** — wanneer welke kiezen, LPL (Low Power Listening), preamble configuratie bij hoge/lage data rate | ~11 | '13, '14, '15, '17, '18, '20, '21, '22, '24, '25-1 |
| 2 | **Hidden / Exposed Terminal Problem** — uitleg + diagram, MAC-oplossingen (MACA, CSMA/CA, RTS/CTS), CSMA helpt niet bij exposed | ~6 | '14, '15, '17, '24, voorbeeldexamen |
| 3 | **Pure ALOHA / Slotted ALOHA** — werking, verschil, kan Pure ALOHA op 802.11? (ja maar zinloos) | ~6 | '13, '14, '15, '17, '24, voorbeeldexamen |
| 4 | **Stop-and-Wait analyse** — welk medium goed/slecht, throughput berekening | ~4 | '14, '18, '21, '25-1 |
| 5 | **Switches vs Hubs** — werking, voordelen switch, wanneer hub beter | ~4 | '14, '15, '24 |
| 6 | **Ethernet** — kabellengte limiet (classic vs switched vs full duplex), padding, CSMA/CD | ~4 | '15, '20, '22 |
| 7 | **802.11 power saving** — beacon frames, APSD, twee strategieën | ~2 | '20, '22 |
| 8 | **Byte/bit stuffing** — werking, wanneer bit stuffing efficiënter | ~2 | '13, '14 |
| 9 | **MAC-protocollen bij AODV** — welke 2 goed (CSMA, B-MAC), welke 2 slecht (TSMP, Pure Aloha) | ~1 | '24 |
| 10 | **LoRa MAC** — wanneer goed/slecht, verbetering voorstellen | ~1 | '25-2 |
| 11 | **Channel hopping** — voordelen | ~1 | voorbeeldexamen |

**Topprioriteit voor studeren:** B-MAC vs TSMP (elk jaar!), Hidden/Exposed terminal, Pure ALOHA (+ 802.11 vraag), en Stop-and-Wait analyse.

---

## Samenvatting: Top 5 onderwerpen per laag

| Laag | Must-know onderwerpen |
|------|-----------------------|
| **Application** | Gnutella 0.4/0.6, DNS (split design + UDP), DHCP (werking + UDP), OSI layering |
| **Transport** | TCP Flow Control, Silly Window/Tinygram/Nagle, Sliding Window vergelijking, TCP Congestion Control (Tahoe diagram) |
| **Network** | IPv6 autoconfiguratie/privacy, IP subnetting, RED/ECN/Choke vergelijking, AODV, Count-to-infinity |
| **Data Link** | B-MAC vs TSMP (+ high/low rate config), Hidden/Exposed terminal, ALOHA (+ 802.11), Stop-and-Wait analyse |

---

## Trends per jaar

- **Recente examens (2024–2025):** Meer focus op QUIC, LoRa, AODV+MAC combinatie, en praktische IP-oefeningen. TOR spying kwam terug in 2024.
- **Vaste klassiekers (elk jaar):** B-MAC vs TSMP, TCP flow control, Gnutella, DNS, IPv6 privacy.
- **Open vs Gesloten boek wisselt per jaar:** De prof wisselt welke lagen open/gesloten zijn. Bereid alles voor alsof het gesloten boek is.
