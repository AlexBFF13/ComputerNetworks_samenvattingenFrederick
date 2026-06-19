# Computer Networks — Examenoverzicht per laag

> **Doel:** Snel inzicht in welke onderwerpen het vaakst terugkomen op het examen, zodat je gericht kunt studeren.
> **Bronnen:** Wiki-examen, opgeloste examenvragen, Ward & Kevin bundel, Modelexamen 2024 (Maxim), examen 10 juni 2025, examen 24 juni 2025.
> **Jaren geanalyseerd:** 2013–2025 (14 examenmomenten)

---

## 1. Application Layer

**Gesloten boek** (App was gesloten in: '14, '15, '17-jun27, '18, '20, '21, '22, '25-1, '25-2)

| # | Onderwerp | Keer | Jaren (gesloten) |
|---|-----------|:----:|------------------|
| 1 | **Gnutella 0.4 / 0.6** — message types, routering, TTL, 0.4 vs 0.6 | ~8 | '14, '15, '18, '20, '22, '25-1, '25-2 |
| 2 | **DNS** — recursief vs iteratief, waarom UDP, split design | ~8 | '14, '15, '18, '21, '22, '25-1, '25-2 |
| 3 | **DHCP** — werking (diagram), waarom UDP niet TCP, lease/renew/decline | ~6 | '14, '15, '17, '18, '21, voorbeeldexamen |
| 4 | **OSI / TCP-IP / Tanenbaum stacks** — tekenen, protocol per laag plaatsen | ~7 | '18, '20, '21, '22, '25-1, '25-2 |
| 6 | **TOR** — kennisverdeling, minimale hops | ~3 | '13, '14, '15 |
| 7 | **HTTP 1.0 vs 1.1** — persistent connections, pipelining | ~3 | '14, '15, '17 |
| 8 | **FTP** — control/data verbinding, passive mode | ~1 | '18 |
| 9 | **UDP-applicaties** — waarom geen TCP (Wake on LAN, DNS, DHCP...) | ~1 | '20 |

**Open boek** (App was open in: '15-jun23, '17-jun19, '24)

| # | Onderwerp | Keer | Jaren (open) |
|---|-----------|:----:|--------------|
| 1 | **Gnutella** — spionage scenario, Chord overlay, NAT/PUSH | ~4 | '13, '15, '17, '24 |
| 5 | **Chord** — DHT, Gnutella over Chord, hash-vereisten, privacy | ~3 | '13, '15, '17 |
| 6 | **TOR** — spying (limited vs unlimited resources), Sybil attack | ~1 | '24 |
| 8 | **FTP** — NAT-probleem, passive mode | ~1 | '17 |
| 9 | **UDP-applicaties** — 3 apps + waarom geen TCP | ~2 | '15 |
| 10 | **P2P chat-applicatie ontwerpen** (Napster/Gnutella/BitTorrent) | 1 | '24 |

**Topprioriteit voor studeren:** Gnutella (0.4/0.6 verschillen + message types), DNS (split design + waarom UDP), DHCP (werking + waarom geen TCP), en OSI-layering.

---

## 2. Transport Layer

**Gesloten boek** (TL was gesloten in: '15-jun23, '18, '20, '21, '22, '24, '25-1, '25-2)

| # | Onderwerp | Keer | Jaren (gesloten) |
|---|-----------|:----:|------------------|
| 1 | **TCP Flow Control** — sliding window, window advertisement, window probe, deadlock | ~7 | '15, '18, '21, '22, '24, '25-2, voorbeeldexamen |
| 3 | **Silly Window Syndrome + Tinygram** — Nagle's algorithm, delayed ACKs, interactie | ~5 | '18, '20, '21, '25-1, voorbeeldexamen |
| 5 | **RTP** — features, verschil met TCP, header velden, multicast | ~3 | '15, '22, voorbeeldexamen |
| 6 | **TCP Congestion Control** — Tahoe/Reno diagram, slow start, AIMD, threshold | ~3 | '22, '24, '25-1 |
| 7 | **ECN vs RED** — werking ECN, voordelen/nadelen t.o.v. RED | ~3 | '18, '24, '25-1 |
| 9 | **QUIC** — ontwerp, verschil met TCP (streams, 0-RTT, connection migration) | ~2 | '24, '25-1 |
| 10 | **TCP header** — alle velden uitleggen | ~2 | '21, '22 |
| 2 | **Sliding Window Protocols** — vergelijking, beperkingen sequence numbers | ~3 | '15, '20, voorbeeldexamen |
| 8 | **Retransmission** — waarom in transport layer | ~1 | '20 |
| 4 | **Delayed ACKs + Nagle's algorithm** — interactie (deadlock!) | ~1 | '21 |

**Open boek** (TL was open in: '14, '15-jun5, '17)

| # | Onderwerp | Keer | Jaren (open) |
|---|-----------|:----:|--------------|
| 1 | **TCP Flow Control** — vergelijking met Go-Back-N, buffer management | ~3 | '14, '17 |
| 4 | **Delayed ACKs + Nagle's algorithm** — werking, waarom nuttig, interactie | ~2 | '14, '15 |
| 11 | **ACK clock** — werking, smoothing traffic | ~2 | '14, '17 |
| 2 | **Sliding Window Protocols** — stop-and-wait, go-back-N, selective repeat | ~2 | '14, '17 |
| 5 | **RTP** — sequencing vs TCP, download vs stream, congestieprobleem | ~1 | '17 |
| 6 | **TCP Congestion Control** — omgang met heterogene netwerken | ~1 | '17 |
| 8 | **Retransmission** — waarom transport layer, voordelen in data link / application layer | ~1 | '15 |
| 12 | **Leaky bucket** — werking, R en B, grafiek tekenen | ~1 | '15 |

**Topprioriteit voor studeren:** TCP Flow Control (absoluut #1!), Silly Window + Tinygram + Nagle, sliding window protocolvergelijking, en TCP congestion control (Tahoe/Reno diagram).

---

## 3. Network Layer

**Gesloten boek** (NL was gesloten in: '14-jun6, '17-jun19, '20, '21, '22)

| # | Onderwerp | Keer | Jaren (gesloten) |
|---|-----------|:----:|------------------|
| 4 | **AODV** — werking (ROUTE_REQUEST/REPLY), reactief vs proactief | ~4 | '17, '20, '22, voorbeeldexamen |
| 3 | **Congestion Notifications** — RED vs ECN vs Choke Packets | ~3 | '21, '22, voorbeeldexamen |
| 2 | **IP addressing / Subnetting** — broadcast adres, CIDR, hostbereik | ~2 | '20, '22 |
| 6 | **Distance Vector / Count-to-infinity** — werking, probleem | ~3 | '13, '22, voorbeeldexamen |
| 7 | **BGP vs OSPF** — intradomain vs interdomain, link state vs path vector | ~3 | '13, '22, voorbeeldexamen |
| 10 | **Round Robin fair queuing** — werking, misbruik | ~1 | '14 |

**Open boek** (NL was open in: '14-jun16, '15, '17-jun27, '18, '24, '25-1, '25-2)

| # | Onderwerp | Keer | Jaren (open) |
|---|-----------|:----:|--------------|
| 1 | **IPv6 autoconfiguratie / privacy / security** — SLAAC, waarom onveilig, vergelijking DHCPv6 | ~7 | '13, '14, '15, '17, '24, '25-1, '25-2 |
| 2 | **IP addressing / Subnetting** — broadcast adres berekenen, CIDR, hostbereik, adrestype | ~4 | '15, '25-1, '25-2 |
| 5 | **MTU / IP Fragmentation** — onverenigbaarheid, ICMP rol, DF-flag | ~4 | '17, '18, '24, voorbeeldexamen |
| 3 | **Congestion Notifications** — RED vs ECN vs Choke Packets (snelheid, efficiëntie) | ~2 | '14, '15 |
| 8 | **NAT** — problemen op application layer, technieken om te omzeilen | ~2 | '15, '25-1 |
| 9 | **IoT en IP stack** — IPv4 vs IPv6 voor IoT, voor/nadelen IP op 802.15.4 | ~2 | '18, voorbeeldexamen |
| 4 | **AODV** — wanneer beter dan OSPF, Zigbee/BLE mesh routing | ~2 | '24, '25-2 |
| 6 | **Distance Vector / Count-to-infinity** — hoe AODV het oplost | ~1 | '15 |
| 7 | **BGP vs OSPF** — waarom TCP vs connectionless | ~1 | '15 |

**Topprioriteit voor studeren:** IPv6 autoconfiguratie/privacy (bijna elk jaar!), IP subnetting oefeningen, congestion notification vergelijking (RED/ECN/Choke), en AODV werking.

---

## 4. Data Link & Physical Layer

**Gesloten boek** (DL was gesloten in: '14-jun16, '15, '17, '20, '21, '22, '24)

| # | Onderwerp | Keer | Jaren (gesloten) |
|---|-----------|:----:|------------------|
| 1 | **B-MAC vs TSMP** — wanneer welke kiezen, LPL, preamble configuratie | ~8 | '13, '15, '17, '20, '21, '22, '24 |
| 2 | **Hidden / Exposed Terminal Problem** — MAC-oplossingen (CSMA/CA, RTS/CTS) | ~5 | '14, '15, '17, '24, voorbeeldexamen |
| 3 | **Pure ALOHA / Slotted ALOHA** — werking, verschil, kan Pure ALOHA op 802.11? | ~6 | '13, '14, '15, '17, '24, voorbeeldexamen |
| 5 | **Switches vs Hubs** — werking, voordelen switch, wanneer hub beter | ~3 | '14, '15, '24 |
| 6 | **Ethernet** — kabellengte limiet (classic vs switched vs full duplex), padding, CSMA/CD | ~3 | '15, '20, '22 |
| 7 | **802.11 power saving** — beacon frames, APSD, twee strategieën | ~2 | '20, '22 |
| 9 | **MAC-protocollen bij AODV** — welke 2 goed (CSMA, B-MAC), welke 2 slecht (TSMP, Pure Aloha) | ~1 | '24 |
| 11 | **Channel hopping** — voordelen | ~1 | voorbeeldexamen |
| 4 | **Stop-and-Wait analyse** — welk medium goed/slecht, throughput berekening | ~1 | '21 |

**Open boek** (DL was open in: '14-jun6, '18, '25-1, '25-2)

| # | Onderwerp | Keer | Jaren (open) |
|---|-----------|:----:|--------------|
| 1 | **B-MAC vs TSMP** — wanneer welke kiezen, LPL, preamble configuratie hoge/lage rate | ~3 | '14, '18, '25-1 |
| 4 | **Stop-and-Wait analyse** — welk medium goed/slecht, throughput berekening | ~3 | '14, '18, '25-1 |
| 10 | **LoRa MAC** — wanneer goed/slecht, verbetering voorstellen | ~1 | '25-2 |
| 8 | **Byte/bit stuffing** — werking, wanneer bit stuffing efficiënter | ~1 | '14 |
| 5 | **Switches vs Hubs** — wanneer hub beter dan switch | ~1 | '14 |

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
