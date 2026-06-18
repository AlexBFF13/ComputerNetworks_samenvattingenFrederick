# Computernetwerken — Open-Boek Referentiegids

> G0Q43A · KU Leuven · Examen 8 juni 2026
> Geoptimaliseerd voor snelle opzoeking onder tijdsdruk. Focus: architecturale logica, cross-layer redenering, ontwerpkeuzes.

---

## Inhoudstafel

**1. Application Layer**
- 1.1 DNS: hybride resolutie-architectuur
- 1.2 DHCP: bootstrapping zonder IP
- 1.3 FTP: NAT-incompatibiliteit en passieve modus
- 1.4 Gnutella 0.4 vs 0.6: architectuur, spionage, transport
- 1.5 TOR: kennisverdeling, minimale hops, surveillance
- 1.6 HTTP 1.0 vs 1.1: persistent connections
- 1.7 P2P chat-app: overlay-ontwerpkeuze
- 1.8 NAT: applicatielaagproblemen en traversal

**2. Transport Layer**
- 2.1 RTP vs TCP: sequencing, download vs stream, congestie
- 2.2 Sliding Windows: formules, satelliet, LoRa
- 2.3 TCP Flow Control vs Go-Back-N vs Selective Repeat
- 2.4 ACK Clock: zelfregulerende pacing
- 2.5 Nagle + Delayed ACKs: interactieprobleem
- 2.6 TCP Tahoe vs Reno: verliesreactie
- 2.7 QUIC: ontwerp en verschil met TCP
- 2.8 Congestion Notificatie: Choke/ECN/RED

**3. Network Layer**
- 3.1 IPv4 Fragmentatie vs Path MTU Discovery
- 3.2 IPv6 SLAAC: privacy via EUI-64
- 3.3 IPv6: geen header checksum — waarom?
- 3.4 IoT en de IP Stack (6LoWPAN)
- 3.5 AODV vs OSPF: reactief vs proactief + count-to-infinity
- 3.6 OSPF vs BGP: intra-AS vs inter-AS
- 3.7 Subnetting: methode + routing tables + ARP
- 3.8 PDU-analyse bij multi-hop routing
- 3.9 NAT-problemen + traversal-technieken
- 3.10 Adressering per laag: IP vs MAC vs poort
- 3.11 Zigbee/BLE mesh routing naar IPv6 gateway

**4. Data Link & Physical Layer**
- 4.1 CSMA/CA: Hidden/Exposed Terminal + RTS/CTS + NAV
- 4.2 BMAC vs TSMP: preamble/sampling trade-off + fusie
- 4.3 Hub vs Switch: architectuur, collision domains, privacy
- 4.4 Collision Domains en Kabellengte (CSMA/CD)
- 4.5 802.11 Packet Loss als TCP Congestion Signaal
- 4.6 LoRa: beperkingen, ALOHA-analyse, MAC-verbeteringen
- 4.7 Pure ALOHA vs Slotted ALOHA
- 4.8 Framing: byte-stuffing vs bit-stuffing
- 4.9 Ethernet Frame Padding + Minimum Frame
- 4.10 Stop-and-Wait: wanneer goed/slecht
- 4.11 Null MAC: problemen onder belasting
- 4.12 802.11 Power Saving (twee strategieen)
- 4.13 Cross-layer: MAC-protocol keuze bij AODV

---

## 1. Application Layer

### 1.1 DNS: Hybride Resolutie-architectuur

**Waarom recursief + iteratief gecombineerd?**

| Aspect | Puur Iteratief | Puur Recursief | Hybride (actueel) |
|---|---|---|---|
| Client-complexiteit | Hoog — client chased referrals zelf | Laag — één vraag, één antwoord | Laag |
| Load op root servers | Hoog — elke client begint bij root | Exploderend — server moet alles resolven | Laag — enkel stateless referrals |
| Caching | Geen gedeelde cache | Server cached, maar niet schaalbaar | Lokale resolver cached voor hele regio |
| Schaalbaarheid | Slecht | Slecht | Goed |

Het hybride model werkt omdat elke component doet waar hij goed in is:
- **Client** → recursieve vraag aan lokale resolver (eenvoud)
- **Lokale resolver** → iteratieve stappen naar root/TLD (gedeelde cache absorbeert load)
- **Root servers** → stateless referrals (schaalbaar, anycast)

**Waarom DNS over UDP?** DNS-queries zijn klein, kort, one-shot. TCP's three-way handshake per lookup (en per iteratieve hop) verveelvoudigt overhead. UDP maakt anycast mogelijk: meerdere fysieke servers delen één IP voor redundantie. Bij te groot antwoord: fallback naar TCP.

---

### 1.2 DHCP: Bootstrapping zonder IP

**Flow: DISCOVER → OFFER → REQUEST → ACK**

| Fase | Richting | Bron-IP | Doel-IP | Waarom broadcast? |
|---|---|---|---|---|
| DISCOVER | Client → all | 0.0.0.0 | 255.255.255.255 | Client kent noch eigen IP noch server-IP |
| OFFER | Server → all/client | Server-IP | 255.255.255.255 | Client heeft nog geen IP om unicast te ontvangen |
| REQUEST | Client → all | 0.0.0.0 | 255.255.255.255 | Informeert ook afgewezen servers |
| ACK | Server → client | Server-IP | Client-IP of broadcast | Bevestigt lease |

Na ACK: client doet Duplicate Address Detection via gratuitous ARP. Bij conflict: DHCPDECLINE → herstart.

**Waarom TCP fundamenteel onmogelijk is:** TCP vereist een three-way handshake met geldig bron-IP én doel-IP. Tijdens DISCOVER heeft de client geen van beide. TCP is unicast en connection-oriented; DHCP heeft broadcast nodig. UDP laat toe vanuit 0.0.0.0 naar 255.255.255.255 te zenden zonder voorafgaande state.

**Lease-tijd trade-off (Hughes-focus):** korte leases → efficiënte herbruik van schaarse adressen, meer server-load. Lange leases → minder overhead, maar adressen blijven geblokkeerd door inactieve hosts.

---

### 1.3 FTP: NAT-incompatibiliteit

**Waarom twee TCP-verbindingen?** Control (poort 21) blijft persistent voor de hele sessie; data (poort 20) wordt per bestandsoverdracht opgezet en afgebroken. Dit scheidt commando's van datatransfer — een abort-commando wordt niet geblokkeerd door een buffervolle datastream.

**Active FTP faalt achter NAT:**
```
Client (privé 192.168.1.10) → NAT → Server (publiek 203.0.113.5)
1. Client → Server:21 (control) ✓ uitgaand, NAT maakt mapping
2. Client stuurt PORT 192.168.1.10,p  (privé-adres in payload!)
3. Server → 192.168.1.10:p (inkomend) ✗ NAT heeft geen mapping
   → packet gedropt → data mislukt
```
Kernprobleem: NAT houdt enkel state bij voor uitgaande verbindingen. FTP embedt IP-adressen in de payload; NAT herschrijft die niet.

**Passive FTP (PASV) oplossing:** server opent luisterpoort, client initieert dataverbinding uitgaand → NAT laat dit door. Beide verbindingen (control + data) zijn nu outbound vanuit client.

---

### 1.4 Gnutella: Architectuur en Spionage

**0.4 vs 0.6 kernverschil:**

| | 0.4 (vlak) | 0.6 (hiërarchisch) |
|---|---|---|
| Topologie | Alle peers gelijk | Ultrapeers + leaf nodes |
| Zoekverkeer | Flooding via alle peers | Geconcentreerd op ultrapeers |
| Schaalbaarheid | Slecht (O(N) berichten) | Beter |
| Anonimiteit | Hoger (enkel buren kennen je) | **Lager** — ultrapeer ziet alles van zijn leaves |

**Spionage-scenario (open-boek favoriet):**

*Beperkte resources (1 node):*
- **0.4 als gewone peer:** je ziet enkel QUERY/QUERYHIT die via jouw directe buren passeren. Beperkt zicht.
- **0.6 als leaf node:** je ziet enkel je eigen verkeer. Minimaal zicht.
- **0.6 als ultrapeer:** je ziet alle QUERY's van jouw leaf nodes + buurultrapeers. Je kent: zoektermen, IP-adressen, bestandslijsten, activiteitspatronen.

*Ongelimiteerde resources:*
- **0.4:** deploy honderden nodes met hoge degree → zie groot deel van flooding-verkeer
- **0.6:** word ultrapeer op meerdere plekken → 10% van ultrapeers controleren = significant deel van het verkeer zichtbaar

**Hughes-punt:** 0.6 koopt performantie maar verliest anonimiteit — copyright surveillance wordt triviaal.

**Transport-keuze:** Discovery (PING/PONG/QUERY) → UDP past: klein, connectionless, flooding-vriendelijk. Bestandsoverdracht → TCP/HTTP: betrouwbaar, geordend, volledig bestand vereist.

---

### 1.5 TOR: Kennisverdeling en Surveillance

**Wat weet elke node?**

| Node | Bron-IP | Bestemming | Data | Vorige hop | Volgende hop |
|---|---|---|---|---|---|
| Entry | **Ja** | Nee | Versleuteld (2 lagen) | Client | Intermediate |
| Intermediate | Nee | Nee | Versleuteld (1 laag) | Entry | Exit |
| Exit | Nee | **Ja** | Plaintext (als geen end-to-end encryptie) | Intermediate | Server |

**Minimum: 3 nodes (entry + 1 intermediate + exit).** Met 2 nodes (entry = exit of direct entry→exit): één node kent zowel bron als bestemming → volledige deanonymisatie. De intermediate node breekt deze directe link.

**Waarom NIET 10 hops:**
1. Latency: elke hop ≈ 100 ms → 10 hops = 1s extra
2. Paradoxaal onveiliger: kans op gecompromitteerde node = 1 − (1−p)^N. Bij p=20%: N=3 → 49%, N=10 → 89%
3. Globale timing-correlatie (state actor) wordt niet door extra hops verslagen

**Surveillance met beperkte vs onbeperkte resources:**
- **Beperkt:** run exit nodes, log plaintext verkeer + traffic-pattern analyse. Je ziet content maar zelden de afzender.
- **Onbeperkt (state actor):** globale timing-correlatie — observeer entry én exit, correleer packet-timing/volume end-to-end.

Kern: TOR beschermt tegen lokale adversary, niet tegen wie beide uiteinden simultaan observeert.

---

### 1.6 HTTP 1.0 vs 1.1

HTTP/1.0 opent een **nieuwe TCP-verbinding per object** (handshake + slow-start kost per keer). HTTP/1.1 gebruikt **persistent connections** (hergebruik van één TCP-verbinding) en **pipelining** (volgende request sturen zonder te wachten op response).

Voordeel is het grootst bij pagina's met veel embedded objecten (moderne webpagina's).

**Cross-layer inzicht (Hughes-favoriet):** HTTP/1.0 is "connectionless/stateless" op applicatielaag maar draait over connection-oriented TCP. Een applicatieprotocol kan connectionless zijn terwijl het transport connection-oriented is.

---

### 1.7 P2P Chat-app: Overlay-ontwerpkeuze

Per fase de juiste overlay kiezen en beargumenteren:

| Fase | Beste overlay | Waarom |
|---|---|---|
| Login / presence (wie is online) | Napster-stijl centraal of Gnutella 0.6 superpeers | Betrouwbare, queryable directory nodig; flooding is verspilling voor presence |
| Peer discovery / contact opzoeken | Gnutella 0.6 | Superpeers routen lookups efficiënt; leaves blijven licht |
| 1-op-1 chat | Directe verbinding (zoals Gnutella HTTP transfer) | Minimale latency, geen overhead |
| Groot bestand delen naar groep | BitTorrent (swarming, chunked) | Schaalbaar bij populaire content |

Praktische randkwestie: NAT traversal via PUSH/relay mechanisme.

---

### 1.8 NAT: Problemen op Applicatieniveau

NAT herschrijft IP/poort aan de boundary maar **niet** IP-adressen in applicatie-payloads, en blokkeert inkomende verbindingen. Protocollen die hun eigen adressen in de payload dragen, breken: FTP active mode, SIP/VoIP.

**Twee traversal-technieken:**
1. **Client-geïnitieerde verbindingen** — FTP passive mode, Gnutella PUSH
2. **Relays/helpers** — STUN/TURN (ontdek publiek mapping of relay via server), ALG (NAT herschrijft bekende payloads), UPnP port mapping

---

## 2. Transport Layer

### 2.1 RTP vs TCP

| Eigenschap | TCP | RTP |
|---|---|---|
| Sequencenummers | 32-bit, telt **bytes** | 16-bit, telt **packets** |
| Doel nummering | Volgorde, retransmissie, SACK | Verliesdetectie, volgordeherstel voor afspeelbuffer |
| Timing | Impliciet via RTT-schatting | Expliciet 32-bit timestamp (media-eenheden) |
| Retransmissie | Altijd | Nooit — te laat frame is waardeloos |
| Congestion control | Ja (AIMD) | Nee — draait op UDP |
| Multicast | Nee | Ja |

**Video downloaden (niet streamen):** TCP is de correcte keuze. Een download vereist alle bytes — een ontbrekend GOP maakt het bestand ondecodeerbaar. RTP's verlies-tolerantie is een feature voor live media, niet voor stored content.

**High-volume RTP congestieprobleem:**
- **TCP-starvation:** TCP halveert bij verlies (AIMD); RTP/UDP zendt constant → TCP krijgt structureel minder bandbreedte
- **Congestiecollaps:** RTP heeft geen feedback-loop → voortdurend nutteloze packets bij volle queues
- **Geen ACK-clock:** TCP's self-clocking smootht injectie op basis van bottleneck; RTP mist dit → burst-gedrag

---

### 2.2 Sliding Windows: Formules en Toepassing

**Kernformules:**
```
a = propagatievertraging(één richting) / transmissietijd = (RTT/2) / T_trans
T_trans = (packet_grootte × 8) / linksnelheid

Stop-and-Wait:    η = 1 / (1 + 2a)
Go-Back-N:        η = min(W / (1 + 2a), 1)     vereist: W ≥ 1 + 2a voor η = 100%
Selective Repeat: η = min(W / (1 + 2a), 1)     vereist: W ≥ 1 + 2a voor η = 100%
```

**Wanneer Stop-and-Wait efficiënt is:** als a ≈ 0, d.w.z. het bandwidth-delay product < één packetgrootte.

| Medium | a-waarde | S&W efficiëntie | Motivatie |
|---|---|---|---|
| GEO-satelliet (RTT 2600 ms, 10 Mbps, 1700 B) | ~956 | 0,052% | Catastrofaal — venster van 1913 packets nodig |
| LoRa SF12 (RTT 134 μs, 250 bps, 50 B) | ~0,00004 | 99,99% | T_trans >> T_prop → S&W optimaal |
| Serieel 9600 bps over 1 m kabel | ~0 | ~100% | T_trans = 1,42 s vs T_prop = 5 ns |

**Window constraint (Selective Repeat):** venstergrootte ≤ ½ van de sequencenummerrange. Bij grotere vensters kunnen retransmissies na ACK-verlies niet onderscheiden worden van nieuwe packets met hetzelfde nummer.

---

### 2.3 TCP Flow Control vs Go-Back-N vs Selective Repeat

| Eigenschap | TCP | Go-Back-N | Selective Repeat |
|---|---|---|---|
| Venstereenheid | Bytes (dynamisch) | Packets (vast N) | Packets (vast N) |
| Vensteradvertentie | Ontvanger adverteert vrije buffer (WIN-veld) | Protocol-parameter | Protocol-parameter |
| Ontvanger buffert out-of-order | Ja (+ SACK) | Nee — verwerpt | Ja |
| Retransmissie bij verlies | Enkel gaten (met SACK) | Verloren packet **+ alle volgende** | Enkel verloren packet |
| ACK-type | Cumulatief + optioneel SACK | Puur cumulatief | Individueel per packet |

**Window probe:** als ontvanger WIN=0 adverteert (buffer vol), kan het window-update ACK verloren gaan → deadlock. De zender stuurt periodiek een tiny probe om de ontvanger te dwingen zijn window opnieuw te adverteren.

**usable window = min(rwnd, cwnd)** — flow control beschermt de eindhost, congestion control beschermt het netwerk.

---

### 2.4 ACK Clock

```
Zender (1 Gbps) ──► Bottleneck (1 Mbps) ──► Ontvanger
```

1. Zender burst packets → bottleneck spaceert ze uit in de tijd
2. Ontvanger stuurt ACKs met dezelfde spacing als bottleneck
3. Zender stuurt nieuwe packets op ritme van inkomende ACKs → niet sneller dan bottleneck

Resultaat: zelfregulerende pace-matching zonder expliciete terugkoppeling over netwerktopologie. Dit is TCP's "self-clocking".

---

### 2.5 Nagle + Delayed ACKs: Interactieprobleem

**Delayed ACKs:** ontvanger wacht ≤200 ms op return data voor piggybacking alvorens ACK te sturen.
**Nagle:** zolang er onbevestigd segment onderweg is, buffer kleine data tot volledig segment OF tot ACK terugkomt.

**Het probleem:**
```
Client stuurt klein request → Nagle: "wacht op ACK"
Server ontvangt request → Delayed ACK: "wacht 200ms op return data"
→ Systematische 200ms latency per kleine interactie
```

Geen permanente deadlock (timer loopt af), maar onaanvaardbaar voor interactieve apps (SSH, games).

**Oplossingen:** TCP_NODELAY (disable Nagle), TCP_QUICKACK (disable Delayed ACKs). QUIC/HTTP2 vermijden dit structureel.

**Tinygram syndrome:** 1 byte data → 41 bytes header overhead (IP 20 + TCP 20 + 1 byte). Nagle lost dit op door kleine data te bufferen.

**Silly Window Syndrome (receiver-zijde):** app leest 1 byte per keer → ontvanger adverteert steeds tiny windows → zender stuurt tiny segments. Clarke's algoritme: ontvanger wacht met window-update tot hij een volledige MSS of halve buffer kan accepteren.

---

### 2.6 TCP Tahoe vs Reno

| Fase | Tahoe | Reno |
|---|---|---|
| Slow start | cwnd verdubbelt per RTT tot ssthresh | Identiek |
| Congestion avoidance | cwnd +1 MSS per RTT (lineair) | Identiek |
| Verlies (timeout) | cwnd → 1, ssthresh = ½ vorig cwnd, herstart slow start | Identiek |
| Verlies (3 dup ACKs) | Zelfde als timeout: cwnd → 1 | **Fast Recovery:** cwnd → ½, blijf in congestion avoidance |

Reno herstelt veel sneller bij packet loss door niet volledig te resetten.

**SACK (Selective ACK):** ontvanger specificeert welke byte-ranges al ontvangen zijn → zender retransmit enkel gaten. Backwards-compatible via TCP Options.

**ECN vs RED deployment:** ECN vereist support op routers + beide endpoints. RED draait enkel op routers → daadwerkelijk gedeployed. Hughes-punt: RED is wat echt gebruikt wordt.

---

### 2.7 QUIC

QUIC is een modern transport protocol bovenop **UDP in user space**.

| Eigenschap | TCP | QUIC |
|---|---|---|
| Encryptie | Optioneel (TLS apart) | **Verplicht** (TLS ingebouwd) |
| Connection setup | 1-3 RTT (TCP + TLS) | **0-1 RTT** |
| Head-of-line blocking | Ja — één verloren packet blokkeert alles | **Nee** — onafhankelijke streams |
| Connection migration | Nee — gebonden aan IP/poort | **Ja** — overleeft WiFi→4G switch |
| Implementatie | OS kernel | User space → snellere updates |

Waarom UDP eronder: middleboxes/NAT begrijpen UDP; user-space vermijdt OS-kernel ossificatie (TCP-wijzigingen vereisen OS-updates overal).

**QUIC heeft wél congestion control** (typisch CUBIC), conceptueel vergelijkbaar met Reno/Tahoe maar per-connectie en ontkoppeld van kernel TCP.

QUIC is **geen** RTP-vervanging: geen unreliable mode voor verlies-tolerante realtime media.

---

### 2.8 Congestion Notificatie

| Mechanisme | Werking | Overhead | Snelheid | Deployment |
|---|---|---|---|---|
| **Choke Packets** | Router stuurt apart packet naar zender | Hoog — extra packets in overbelast netwerk | Snel (1 RTT) | Zender only |
| **ECN** | Router markeert bit in bestaand packet; ontvanger echo't via ECE | Geen — hergebruikt bestaande packets | Medium (~1 RTT via ontvanger) | Routers + beide endpoints |
| **RED** | Router dropt packets probabilistisch vóór queue vol is | Negatief — nutteloze hertransmissies | Snel — TCP interpreteert verlies direct | **Routers only** → daadwerkelijk gedeployed |

RED: drop-probability stijgt met gemiddelde queue-lengte. Zware zenders hebben meer packets in queue → worden proportioneel vaker getroffen → eerlijk.

---

## 3 — Network Layer

### 3.1 IPv4 Fragmentatie vs Path MTU Discovery

Het DF-bit bepaalt het fundamentele gedrag bij MTU-mismatch:

| DF-bit | Routergedrag bij te groot packet | Feedback aan zender |
|---|---|---|
| DF = 0 | Router splitst packet stilzwijgend | Geen — zender leert path MTU nooit |
| DF = 1 (PMTUD) | Router dropt packet, stuurt ICMP Type 3 Code 4 | Ja — met bottleneck-MTU |

Ze zijn **incompatibel**: fragmentatie verbergt de MTU-mismatch, PMTUD onthult haar. Je kunt niet tegelijk fragmenteren en de MTU ontdekken.

**PMTUD Black Hole:** firewalls die ICMP blokkeren → zender ontvangt nooit MTU-info → verbinding "hangt" (SYN-ACK lukt, data te groot, geen foutmelding). Oplossing: RFC 4821 (Packetization Layer PMTUD — probeert geleidelijk kleinere segmenten via TCP zonder ICMP-afhankelijkheid).

**Fragmentatie vs segmentatie (examenvalkuil):**

- Fragmentatie (netwerklaag): router splitst te groot packet → reassembly bij **bestemming**, niet bij volgende router
- Segmentatie (transportlaag): TCP verdeelt byte-stream in segments <= MSS **voor** verzending

**IPv6:** fragmentatie door routers is **verboden**; PMTUD is verplicht. Dit vereenvoudigt routers (geen reassembly-state nodig) en maakt forwarding sneller.

---

### 3.2 IPv6 SLAAC en EUI-64 Privacy

SLAAC leidt interface-ID af uit MAC-adres via EUI-64: MAC `00:1A:2B:3C:4D:5E` → insert `FFFE`, flip U/L-bit → `021A:2BFF:FE3C:4D5E`.

**Privacy-problemen:**

1. **Permanente tracking:** MAC is hardware-permanent → zelfde interface-ID op elk netwerk → traceerbaar over locaties
2. **Adresvoorspelbaarheid:** kennis van MAC → IPv6-adres voorspelbaar op elk netwerk
3. **OUI-lekkage:** eerste 24 bits MAC = fabrikant → toesteltype onthuld

**Tegenmaatregelen:** RFC 7217 (opake, stabiele adressen per netwerk — niet afleidbaar), RFC 4941 (tijdelijke privacy-adressen — roteren periodiek).

**SLAAC vs DHCPv6:** SLAAC lekt meer (host leidt adres af uit hardware, geen centrale controle). DHCPv6 kan randomiseren, houdt logs, past in managed netwerk. SLAAC past bij IoT/always-on devices die zonder server moeten autoconfigureren.

**Herkenning:** een adres met `ff:fe` in het midden is de EUI-64 fingerprint → waarschijnlijk SLAAC.

---

### 3.3 IPv6: Geen Header Checksum — Waarom?

**Redenering (niet triviaal):** error detection gebeurt al op twee andere lagen:

- **Link-layer:** Ethernet FCS, 802.15.4 CRC — detecteert bitfouten per hop
- **Transport-layer:** TCP/UDP checksum — end-to-end integriteitscontrole

Een IPv6 header checksum zou **redundant** zijn. Maar het cruciale argument is **performantie**: een router decrementeert de hop limit bij elk packet. Met een header checksum zou herberekening nodig zijn bij **elke hop** → vertraagt forwarding op high-speed routers die miljoenen packets/seconde verwerken. IPv4 heeft dit probleem wel: de header checksum moet per hop opnieuw berekend worden vanwege het TTL-veld.

---

### 3.4 IoT en de IP Stack (6LoWPAN)

IEEE 802.15.4 max frame: **127 bytes**. Dat maakt standaard IP-headers problematisch:

| Barriere | IPv4 | IPv6 |
|---|---|---|
| Header | 20-60 bytes | 40 bytes vast (1/3 van frame!) |
| Adresruimte | 32-bit, uitgeput, vereist NAT | 128-bit, voldoende |
| Autoconfiguratie | DHCP (server nodig) | SLAAC (serverloos) |
| Fragmentatie | Door routers (duur) | Verboden door routers; min MTU 1280 bytes |
| End-to-end | NAT breekt directe M2M | Geen NAT nodig |

**Winnaar: IPv6**, maar niet ongewijzigd. **6LoWPAN** is de adaptielaag:

- Comprimeert 40-byte IPv6-header naar **2-3 bytes** (link-local prefix en EUI-64 interface-ID zijn afleidbaar → niet meesturen)
- Handelt fragmentatie over meerdere 802.15.4-frames af (eigen fragmentatieheader, niet IPv6)
- SLAAC maakt serverloze mesh-netwerken mogelijk

**Waarom niet gewoon IPv4 met kleinere header?** NAT breekt machine-to-machine communicatie, DHCP vereist infrastructuur, en het adresruimteprobleem maakt IPv4 onhoudbaar voor miljarden IoT-devices.

---

### 3.5 AODV vs OSPF + Count-to-Infinity

| | OSPF (proactief, link-state) | AODV (reactief, distance-vector) |
|---|---|---|
| Routes | Continu onderhouden, volledige topologie | On-demand, enkel bij actieve communicatie |
| Overhead bij geen verkeer | Hoog — constante flooding van link-state info | **Geen** — geen control traffic |
| Geheugen per router | O(N) — volledig topologiebeeld | Minimaal — enkel actieve routes |
| Geschikt voor | Stabiele netwerken (campus, enterprise) | Dynamische ad-hoc (mobiel, sensor mesh) |

**Count-to-Infinity (Distance Vector):**
"Good news travels fast, bad news travels slowly." Bij link-uitval adverteert buurnode de oude route → andere node adopteert die + 1 → lus → afstanden stijgen langzaam richting oneindig.

Mitigaties: split horizon, poison reverse, maximum metric (RIP: 16), hold-down timers — maar geen van deze lost het volledig op.

**AODV's oplossing:** destination sequence numbers. Route met **hoger** sequencenummer = verser. Bij uitval: destination verhoogt sequencenummer, stuurt RERR. Nodes met verouderde route (lager nummer) verwerpen die **onmiddellijk** → geen count-to-infinity lus mogelijk.

---

### 3.6 OSPF vs BGP: Intra-AS vs Inter-AS

| | OSPF | BGP |
|---|---|---|
| Scope | Binnen een AS | Tussen ASes (ISPs) |
| Algoritme | Link-state (Dijkstra) | Path-vector met beleid |
| Kennis per router | Volledige area-topologie | Border routers: reachability + policy |
| Schaal | Niet voor country-scale | Ontworpen voor global Internet |
| Routeringskriterium | Naief shortest-path (kost) | Policy: kosten, bandbreedte, welke ASes vermijden |
| Transport | Direct op IP (protocol 89) | **TCP** (poort 179) |

**Waarom OSPF connectionless:** flood link-state advertisements naar alle buren, broadcast-stijl. TCP-sessie per buurnode zou zwaar en zinloos zijn voor flooding. (Technisch: OSPF draait direct op IP protocol 89, niet letterlijk op UDP — maar het is connectionless in geest.)

**Waarom BGP over TCP:** inter-AS sessies zijn **langlevend** (dagen/weken), moeten betrouwbaar zijn (geen verloren routing-updates tussen ISPs). TCP biedt betrouwbaarheid + ordered delivery + flow control. Een verloren BGP-update kan een heel AS onbereikbaar maken.

**Peering vs Transit (examenfavoriet):** peering is het gratis uitwisselen van verkeer tussen twee ASes, typisch via een IXP. Peering is **niet transitief**: als A peert met B en B peert met C, kan A niet gratis via B naar C. Daarvoor is transit (betaald) nodig.

---

### 3.7 Subnetting: Methode + Routing Tables + ARP

**Methode (werkt voor elk masker):**
```
1. CIDR /n  -->  eerste n bits = netwerk, rest = host
2. Netwerkadres       = IP AND masker
3. Broadcast          = netwerk met alle hostbits = 1
4. Eerste host        = netwerk + 1
5. Laatste host       = broadcast - 1
6. #bruikbare hosts   = 2^(32-n) - 2
7. Zelfde subnet?     --> (IP_1 AND masker) == (IP_2 AND masker)
8. Prive (RFC 1918):  10/8, 172.16/12, 192.168/16
```

**Voorbeeld netwerktopologie (192.168.22.0/24):**

| Subnet | CIDR | Bruikbaar bereik |
|---|---|---|
| LAN A | 192.168.22.0/25 | .1 - .126 (126 hosts) |
| Inter-router | 192.168.22.128/30 | .129 - .130 (2 hosts) |
| LAN B | 192.168.22.192/26 | .193 - .254 (62 hosts) |

**Routing Table Router 1:**

| Bestemming | Masker | Gateway | Interface |
|---|---|---|---|
| 192.168.22.0 | /25 | direct connected | eth0 |
| 192.168.22.128 | /30 | direct connected | eth1 |
| 192.168.22.192 | /26 | 192.168.22.130 | eth1 |

**ARP Cache na HTTP-request PC-A (LAN A) naar Webserver (LAN B):**

| Host | Leert via ARP |
|---|---|
| PC-A | Gateway R1 MAC (enkel!) |
| R1 | PC-A MAC + R2 MAC |
| R2 | R1 MAC + Webserver MAC |
| Webserver | R2 MAC (enkel!) |

ARP werkt **enkel binnen hetzelfde subnet** — PC-A kent nooit de MAC van de webserver, enkel die van zijn gateway.

---

### 3.8 PDU-analyse bij Multi-hop Routing

**Kernprincipe — wat verandert per hop en wat niet:**

```
PC-A ---[R1]---[R2]--- Webserver
```

| Veld | Scope | Verandert per hop? |
|---|---|---|
| Src/Dst IP (NSAP) | End-to-end | **Nee** (behalve bij NAT) |
| Src/Dst MAC | Per link | **Ja** — elk hop heeft nieuwe src/dst MAC via ARP |
| Src/Dst Poort (TSAP) | End-to-end | **Nee** (behalve bij NAT) |
| TTL/Hop Limit | Per hop | **Ja** — decrementeert per router |

**Na NAT:** private IP/poort wordt herschreven naar publiek IP/poort aan de NAT-boundary. De applicatie-payload wordt **niet** herschreven.

**Veelgemaakte fouten:**

- Default gateway = IP-adres van router-interface op **jouw** subnet, NIET de switch (switches zijn L2, hebben geen IP-forwarding)
- Broadcasts kruisen **geen** routers — een router is per definitie een broadcast-domain boundary
- Een switch leert MAC-adressen maar routeert niet — frames naar onbekende MAC worden geflooded

---

### 3.9 NAT-problemen + Traversal-technieken

NAT herschrijft IP/poort aan de boundary maar **niet** adressen in applicatie-payloads, en blokkeert inkomende verbindingen (geen mapping in de NAT-tabel).

**Protocollen die breken:**

- FTP active mode: client stuurt `PORT 192.168.1.10,p` (prive-IP in payload) → server probeert inbound verbinding → NAT blokkeert
- SIP/VoIP: IP-adres in SDP-payload → peer belt naar prive-adres → onbereikbaar

**Twee categorieen traversal-technieken:**

| Categorie | Technieken | Principe |
|---|---|---|
| Client-geinitieerd | FTP PASV, Gnutella PUSH | Draai de verbindingsrichting om zodat alles outbound is |
| Relay/helper | STUN, TURN, ALG, UPnP | Externe server ontdekt/relayed mapping, of NAT herschrijft payload |

**Kan een plain router met een publiek IP NAT vervangen?** Nee — het delen van een publiek IP over meerdere hosts vereist adrestranslatie. Zonder NAT-functionaliteit kan een router slechts een host bedienen per publiek IP.

---

### 3.10 Adressering per Laag — Waarom Alle Drie Nodig?

| Laag | Adrestype | Scope | Functie |
|---|---|---|---|
| Netwerk (L3) | IP-adres (NSAP) | Logisch, end-to-end | **Routing** tussen netwerken |
| Data Link (L2) | MAC-adres | Fysiek, next-hop | **Delivery** op de lokale draad/link |
| Transport (L4) | Poortnummer (TSAP) | End-to-end | **Demultiplexing** naar juiste applicatie |

**Waarom niet alleen IP?** IP-adressen zijn logisch en veranderen niet per hop, maar de link-laag weet niets van IP. Ethernet/WiFi-hardware levert frames af op basis van MAC. ARP vertaalt IP naar MAC **per subnet**. Zonder MAC zou elke NIC elk frame moeten verwerken op IP-niveau → inefficient.

**Waarom niet alleen MAC?** MAC-adressen zijn vlak (geen hierarchie) → niet routeerbaar. IP-adressen zijn hierarchisch (prefix = netwerk) → aggregeerbaar in routing tables. Zonder IP zou elke router een entry nodig hebben voor elk device op het Internet.

**Poortnummers:** zonder poorten kan een host niet onderscheiden of een inkomend packet voor de webserver (80), SSH (22) of DNS (53) is. Poorten zijn het demultiplexing-mechanisme van de transportlaag.

---

### 3.11 Zigbee/BLE Mesh Routing naar IPv6 Gateway

**Scenario:** constrained devices in een mesh moeten sensordata naar een IPv6 gateway (sink) sturen.

**Best fit routing: AODV (reactief)** — low-overhead, on-demand, geen control traffic bij stilte. Maar standaard AODV is ontworpen voor ad-hoc laptops, niet voor energy-constrained sensors.

**Noodzakelijke modificaties:**

- **Energy-aware metric:** kies routes via nodes met meer batterij, niet puur shortest-path
- **Minder HELLOs:** verminder beaconing-frequentie → langere slaapperiodes
- **Langere route lifetimes:** vermijd onnodige route-discovery bij stabiele topologie

**Gateway als sink — RPL (DODAG):**
Voor verkeer dat overwegend naar een gateway stroomt is RPL (Routing Protocol for Low-Power and Lossy Networks) geschikter dan AODV. RPL bouwt een Destination-Oriented Directed Acyclic Graph (DODAG) met de gateway als root. Voordelen: structureel gericht naar de sink, mogelijkheid tot data-aggregatie onderweg, en expliciete ondersteuning voor 6LoWPAN.

**Cross-layer keuze MAC:** AODV/RPL past bij contention-based MACs (CSMA/CA, BMAC) — niet bij TDMA/TSMP die een vaste schedule veronderstellen (zie 4.8).

---

## 4 — Data Link & Physical Layer

### 4.1 CSMA/CA: Hidden/Exposed Terminal + RTS/CTS + NAV

**Hidden Terminal:**
```
A ←→ B(AP) ←→ C     (A en C horen elkaar niet)
A: "kanaal vrij" → zendt | C: "kanaal vrij" → zendt → collision bij B
```
**Waarom carrier sense faalt:** A en C luisteren lokaal, maar hun bereik overlapt niet. Het kanaal lijkt vrij voor beiden, terwijl B beide signalen tegelijk ontvangt en geen van beide kan decoderen.

**Exposed Terminal:**
```
A ←→ B ←→ C ←→ D    (B zendt naar A; C wil naar D)
C hoort B → wacht onnodig, terwijl C→D geen collision zou veroorzaken
```
**Waarom dit verspilling is:** C's transmissie naar D zou B's ontvangst niet storen (A en D liggen in tegengestelde richtingen), maar carrier sense verbiedt zenden omdat het kanaal "bezet" klinkt.

**CSMA lost geen van beide op** — het luistert enkel lokaal. Wifi kan bovendien niet betrouwbaar collision detecteren tijdens zenden (eigen signaal overheerst) → daarom Collision **Avoidance** i.p.v. Detection.

**NAV (Network Allocation Vector):** virtuele carrier sense. Node leest Duration-veld in 802.11-header → stelt timer in → zendt niet tot timer = 0. Werkt zelfs als je het datasignaal niet hoort — puur op basis van gehoorde controlframes.

**RTS/CTS lost hidden terminal op:**
```
1. A → B: RTS (Duration = data + CTS + ACK)
2. B → all: CTS (Duration = data + ACK) ← C hoort dit!
3. C stelt NAV → zwijgt
4. A → B: DATA (collision-vrij)
5. B → A: ACK
```
C hoorde A's RTS misschien niet (hidden), maar hoort B's CTS wel → weet dat kanaal bezet is.

**RTS/CTS lost exposed terminal NIET op** — verergert het zelfs: C hoort B's RTS en stelt NAV onnodig in, waardoor C nog langer wacht dan met enkel carrier sense.

**Wanneer RTS/CTS gebruiken:** enkel voor grote frames. Overhead van 2 extra frames per transmissie is enkel gerechtvaardigd als de kost van een collision op een groot dataframe hoger is dan de RTS/CTS-overhead. Standaard RTS-threshold = 2347 bytes (vaak uitgeschakeld).

---

### 4.2 BMAC vs TSMP: Preamble/Sampling Trade-off + Fusie

| Kenmerk | BMAC (LPL) | TSMP (TDMA + channel hopping) |
|---|---|---|
| Verkeer | Onvoorspelbaar, event-driven | Voorspelbaar, periodiek |
| Topologie | Dynamisch, nodes komen/gaan | Stabiel, bekende deelnemers |
| Synchronisatie | Geen vereist | Vereist (clockmaster) |
| Latency | Laag (meteen zenden) | Hoger (wacht op toegewezen slot) |
| Energie (weinig verkeer) | Goed (radio meestal uit) | Uitstekend (slot = exact geplande wake) |
| Energie (veel verkeer) | Slecht (lange preamble per packet) | Goed (geen preamble-overhead) |
| Schaalbaarheid | Goed (gedecentraliseerd) | Beperkt (centrale slottoewijzing) |
| Grootste energiekost | Preamble-zending | (Re)joining van het netwerk |

**Preamble/Sampling trade-off (kernformule):**
```
Preamble_length ≥ Sample_Interval
```
**Waarom:** de ontvanger samplet periodiek kort het kanaal. De preamble moet minstens zo lang duren als het sample-interval zodat de ontvanger gegarandeerd activiteit detecteert tijdens een sample-moment.

| Configuratie | Sample-interval | Preamble | Resultaat |
|---|---|---|---|
| **Veel verkeer** | Kort | Kort | Lage per-packet overhead, meer sample-energie |
| **Weinig verkeer** | Lang | Lang | Maximale slaaptijd, dure maar zeldzame zendingen |

**BMAC+TSMP fusie-ontwerp:** gebruik TDMA-slots als backbone voor steady/periodiek verkeer (camera-feeds, temperatuursensoren). Reserveer een LPL-venster voor event-driven/onverwachte berichten en nieuwe nodes. Nieuwe nodes joinen via BMAC (geen schedule nodig), en schakelen over naar TDMA na synchronisatie. Gesynchroniseerde slots voorkomen dat beide regimes botsen.

**Examenvalkuil:** TSMP biedt niet alleen TDMA maar ook **channel hopping** (spreidt interferentierisico over frequenties) en **redundantie** (spatial: andere buur, temporal: ander tijdstip). BMAC biedt QoS-differentiatie niet — TSMP kan meer slots toewijzen aan een camera dan aan een temperatuursensor.

---

### 4.3 Hub vs Switch: Architectuur, Collision Domains, Privacy

| Aspect | Hub | Switch |
|---|---|---|
| Werking | Fysieke repeater, kopieert signaal naar alle poorten | Leert MAC-adressen (MAC table), forwardt per poort |
| Collision domain | Een gedeeld domain (alle poorten) | Per poort een apart domain |
| Duplex | Half-duplex (CSMA/CD vereist) | Full-duplex mogelijk (geen collisions) |
| Bandbreedte | Gedeeld over alle poorten | Dedicated per poort |
| Privacy | Iedereen ziet al het verkeer | Enkel je eigen unicast-verkeer |
| Kosten/complexiteit | Goedkoop, geen logica | Duurder, forwarding-logica |

**Wanneer hub beter dan switch?** Network sniffing, protocol-analyse, lab-onderwijs — je wilt dat alle hosts al het verkeer zien via promiscuous mode. Bij een switch is dit enkel mogelijk met **port mirroring** (SPAN), wat configuratie vereist.

**Privacy-vergelijking over media:**

| Medium | Afluisterbaarheid | Bescherming |
|---|---|---|
| Hub Ethernet | Triviaal: promiscuous mode op elke poort | Fysieke toegangscontrole |
| Switched Ethernet | Moeilijk, maar MAC flooding/spoofing kan switch in hub-modus forceren | Port security, 802.1X |
| 802.11 WiFi | Inherent sniffable (radiogolven voor iedereen) | Volledig afhankelijk van encryptie (WPA2/3) |

---

### 4.4 Collision Domains en Kabellengte (CSMA/CD)

**Kernvereiste:** CSMA/CD werkt alleen als de zender een collision detecteert **terwijl hij nog zendt**:
```
T_transmissie ≥ 2 × T_propagatie
```
De minimale framegrootte (64 bytes = 512 bits) is zo gekozen dat aan deze eis wordt voldaan.

**Rekenvoorbeeld 10 Mbps:** T_tx = 512 bits / 10 Mbps = 51,2 μs → max kabel = 51,2 μs × 200 m/μs / 2 ≈ **2560 m**.
**100 Mbps met hub:** T_tx = 512 bits / 100 Mbps = 5,12 μs → max kabel ≈ **256 m** (10x strenger!).

| Configuratie | CSMA/CD | Kabelbeperking |
|---|---|---|
| Classic 10 Mbps, hub | Ja | ~2500 m (collision-timing) |
| Fast 100 Mbps, hub | Ja | ~256 m (**strenger**) |
| Switched, half-duplex | Ja (per poort) | Per poort |
| Switched, **full-duplex** | **Nee** | **Enkel attenuatie** (~100 m Cat5e) |

**Waarom full-duplex de limiet opheft:** zenden en ontvangen op aparte aderparen → collisions fysiek onmogelijk → CSMA/CD uitgeschakeld → de collision-driven kabellimiet verdwijnt. Enige resterende limiet is signaalattenuatie (~100 m voor Cat5e).

**Gigabit Ethernet:** bij 1 Gbps zou T_tx = 5,12 ns (onwerkbaar kort). Oplossing: **carrier extension** verlengt korte frames kunstmatig tot 512 bytes (4096 bits) + **frame bursting** voor efficientie. Bij 10G: enkel full-duplex, geen hubs, CSMA/CD overbodig.

---

### 4.5 802.11 Packet Loss als TCP Congestion Signaal

**Kernprobleem:** TCP interpreteert **alle** packet loss als congestie → halveert cwnd (AIMD). Op WiFi zijn de meeste verliezen door:

- Radiointerferentie (andere netwerken, magnetrons, Bluetooth)
- Hidden terminal collisions
- Signaalzwakte/multipath fading
- Uitgeputte MAC-layer retransmissies (802.11 herprobeert intern tot 7x)

Geen van deze is congestie. TCP "bestraft" zichzelf onnodig → throughput daalt dramatisch op een niet-gecongestioneerd netwerk.

**Oplossingen:**

| Aanpak | Laag | Werking |
|---|---|---|
| **ECN** | Netwerk | Expliciet congestiesignaal, geen verwarring met link-fouten |
| **Data-link ARQ** | Data link | 802.11 ACKs + retransmissie maskeren link-verlies lokaal |
| **TCP Westwood** | Transport | Schat beschikbare bandbreedte via ACK-timing i.p.v. verlies |

**Cross-layer inzicht:** data-link retransmissie (hop-by-hop, lage latency) en transport retransmissie (end-to-end, hogere latency) zijn complementair. Lokale recovery op een lossy wireless hop voorkomt dat TCP het verlies als congestie misverstaat. Dit is waarom 802.11 **acknowledged connectionless** service biedt (in tegenstelling tot Ethernet's unacknowledged connectionless).

---

### 4.6 LoRa: Beperkingen, ALOHA-analyse, MAC-verbeteringen

**Karakteristieken:** bereik ~20 km, datarate 250 bps (SF12) – 50 kbps (SF7), T_prop ≈ 67 μs (verwaarloosbaar t.o.v. airtime).

**1. Duty Cycle (regulatoir):** ISM 868 MHz, max 1% duty cycle per kanaal.
Bij SF12, 50 bytes → T_airtime ≈ 1,6 s → wachttijd = 1,6 / 0,01 = **160 s**. Effectieve throughput ≈ **2,5 bps**.

**2. Collision-analyse (Pure ALOHA):**
```
P(collision) = 1 - e^(-2 * lambda * T_airtime)
```
100 nodes, elk 1 pkt/min → lambda = 100/60 ≈ 1,67 pkt/s, T_airtime = 1,6 s → P ≈ **99,5%** collision. LoRa schaalt fundamenteel niet met Pure ALOHA bij hoge node-densiteit.

**3. Sliding Window keuze:** a = T_prop / T_frame = 67 μs / 1600 ms ≈ 0 → Stop-and-Wait efficiency ≈ 99,99%. S&W is optimaal voor LoRa (geen baat bij sliding window).

**4. Hidden terminal:** nodes 35+ km apart horen elkaar niet maar delen dezelfde gateway → carrier sense (CSMA) onmogelijk op deze schaal → star-topologie met centrale gateway + ALOHA-achtige toegang is onvermijdelijk.

**MAC-verbeteringen om ALOHA-problemen te mitigeren:**

| Techniek | Wat het oplost |
|---|---|
| TDMA/slotting (Class B) | Vermijdt blinde collisions door tijdslots |
| CSMA/Listen-Before-Talk | Carrier sense waar mogelijk (korte range) |
| Channel hopping | Bestrijdt narrowband interferentie |
| Adaptive Data Rate | Verkort airtime waar signaal sterk genoeg → kleiner collision-window |
| Duty-cycle coordinatie | Spreidt zendmomenten over gateways |
| ACKs + retransmissie | Betrouwbaarheid voor kritieke data |

| Scenario | Aanbevolen protocol | Motivatie |
|---|---|---|
| Enkele node, licht verkeer | Stop-and-Wait | Eenvoud, efficiency ≈ 100% |
| Dense (>50 nodes) | TDMA (Class B) | Vermijdt collisions |
| Mobiele nodes | ALOHA (Class A, best effort) | Geen synchronisatie mogelijk |
| Kritieke data | Confirmed (Class A + ACK) | Hertransmissie bij verlies |

---

### 4.7 Pure ALOHA vs Slotted ALOHA

| Kenmerk | Pure ALOHA | Slotted ALOHA |
|---|---|---|
| Wanneer zenden | Op elk willekeurig moment | Enkel bij slotgrenzen |
| Vulnerable period | 2 x frametijd | 1 x frametijd |
| Max channel efficiency | 1/(2e) ≈ **18%** | 1/e ≈ **37%** |
| Synchronisatie nodig | Nee | Ja (globale klok) |

**Waarom slotting de throughput verdubbelt:** in Pure ALOHA kan een frame botsen met elk frame dat begint in de 2T-periode rondom het eigen frame. Door alle frames te dwingen op slotgrenzen te beginnen, kan een frame enkel botsen met frames in **hetzelfde** slot → vulnerable period halveert → dubbele maximale throughput.

**Trade-off:** slotting vereist tijdsynchronisatie. In netwerken waar synchronisatie moeilijk of duur is (grote afstanden, geen centrale klok), kan Pure ALOHA de enige optie zijn.

**Kan Pure ALOHA op 802.11?** Fysiek mogelijk (zelfde radio), maar een slechte keuze: 802.11 gebruikt bewust CSMA/CA met collision avoidance, backoff en RTS/CTS. ALOHA's blinde transmissies verspillen een gedeeld radiokanaal, negeren hidden terminals, en bieden geen ACK-mechanisme.

---

### 4.8 Framing: Byte-Stuffing vs Bit-Stuffing

**Probleem:** framegrenzen moeten gemarkeerd worden met een flag-patroon. Als dat patroon toevallig in de data voorkomt, denkt de ontvanger dat het frame eindigt → escaping nodig.

**Byte-stuffing:** flag-byte markeert grenzen. Bij elke flag of ESC in de data: voeg ESC-byte in voor het betreffende byte. Simpel te implementeren, maar duur bij veel voorkomens van de flag-waarde.

**Bit-stuffing:** flag = `01111110`. Regel: na 5 opeenvolgende 1-bits in data, voeg automatisch een 0 in. Ontvanger verwijdert elke 0 na 5 enen. Het flag-patroon (zes opeenvolgende enen) kan zo nooit per ongeluk in de payload ontstaan.

**Rekenvoorbeeld — byte 0xFF (11111111):**

| Methode | Resultaat | Overhead |
|---|---|---|
| Bit-stuffing | 11111**0**111 = 9 bits | +1 bit |
| Byte-stuffing | ESC + 0xFF = 16 bits | +8 bits (hele ESC-byte) |

Bit-stuffing is hier **8x efficienter**. Hoe meer flag-achtige patronen in de data, hoe groter het voordeel van bit-stuffing.

**Alternatief — byte count:** lengteveld vooraan specificeert framelengte. Compact, maar fataal zwak punt: als het lengteveld corrupt raakt, verliest de ontvanger synchronisatie volledig en een checksum redt dit niet (je weet niet meer waarop hij slaat).

---

### 4.9 Ethernet Frame Padding + Minimum Frame

**Waarom 64 bytes minimum:** CSMA/CD vereist dat de zender nog aan het zenden is wanneer het collision-signaal terugkeert van het verste punt op het netwerk. Bij 10 Mbps, max ~2500 m kabel:
```
T_propagatie (round trip) ≈ 51.2 us
T_transmissie (64B = 512 bits bij 10 Mbps) = 51.2 us  -->  precies gelijk
```
Als de payload korter is dan nodig, wordt het frame **gepad** met opvulbytes tot 64 bytes (inclusief header + CRC). Zonder padding zou een kort frame al verzonden zijn voor de collision terugkomt → collision ondetecteerbaar.

**Gigabit Ethernet:** bij 1 Gbps zou 64 bytes slechts 0,512 μs duren → veel te kort. Oplossing: **carrier extension** vergroot het minimum effectief tot 512 bytes. **Frame bursting** laat meerdere korte frames achter elkaar zenden binnen een enkele carriersessie, waardoor de carrier-extension-overhead gedeeld wordt.

---

### 4.10 Stop-and-Wait: Wanneer Goed/Slecht

**Throughput:** efficiency ≈ 1 / (1 + 2a), waarbij a = T_propagatie / T_frame.

Stop-and-Wait is efficient wanneer **bandwidth-delay product < framegrootte** (a ≈ 0).

| Scenario | RTT | Frame | Throughput | Oordeel |
| --- | --- | --- | --- | --- |
| Kort Ethernet (100m) | ~10 μs | 1500 B | ~1,2 Gbps | **Goed** (a ≈ 0) |
| LoRa (SF12, 20 km) | ~134 μs | 1600 ms airtime | ~2,5 bps | **Goed** (a ≈ 0) |
| Satelliet (GEO) | ~540 ms | 1000 B | ~15 kbps | **Slecht** (a >> 1) |
| Trans-Atlantisch | ~80 ms | 1500 B | ~150 kbps | **Slecht** |

**Waarom slecht bij hoog RTT:** de zender wacht idle op ACK terwijl de pijplijn leeg is. De link is het grootste deel van de tijd onbenut. Oplossing: **sliding window** (Go-Back-N of Selective Repeat) vult de pijplijn.

**Methodiek voor examenvragen:** (1) schat RTT van het medium, (2) bereken a = T_prop / T_frame, (3) als a << 1: S&W volstaat, als a >> 1: sliding window nodig.

---

### 4.11 Null MAC: Problemen onder Belasting

**Definitie:** geen carrier sense, geen backoff, geen retransmissie — luister 100% van de tijd, zend onmiddellijk.

| Situatie | Probleem |
|---|---|
| Hoog verkeer | Veel collisions, geen exponential backoff om load te spreiden, geen recovery → **goodput collapst** naar nul |
| Dicht bevolkt (stadscentrum) | Veel nodes, veel interferentie, geen mechanisme om te wachten → near-zero bruikbare throughput |
| Licht verkeer, weinig nodes | Enige scenario waar Null MAC werkbaar is — collisions zijn zeldzaam |

**Waarom dit een examenonderwerp is:** het illustreert dat MAC-protocollen niet optioneel zijn. Elk mechanisme (carrier sense, backoff, retransmissie) lost een specifiek probleem op. Null MAC = worst-case baseline om andere protocollen tegen af te zetten.

---

### 4.12 802.11 Power Saving (twee strategieen)

| Strategie | Mechanisme | Use case |
|---|---|---|
| **Beacon + TIM / PS-Poll** | AP stuurt periodiek beacon met Traffic Indication Map. Slapende client waakt enkel voor beacons, ziet of hij geflagged is, stuurt PS-Poll om gebufferde data op te halen | Battery-powered device dat meestal idle is (sensor, telefoon in standby) |
| **APSD (Automatic Power Save Delivery)** | Client waakt enkel wanneer hij zelf wil zenden (vaste intervallen). AP levert gebufferde downlink-data mee in hetzelfde wake-window | Periodieke reporting (VoIP, telemetrie) — voorspelbare timing |

**Kernverschil:** bij Beacon+TIM bepaalt het **AP** wanneer de client kan ophalen (beacon-interval). Bij APSD bepaalt de **client** het schema — efficienter bij voorspelbare applicaties.

**Cross-layer link:** vergelijk met BMAC (client-side sampling) vs TSMP (netwerk-gestuurd schema). Dezelfde afweging: wie controleert het wake-schema, en hoe voorspelbaar is het verkeer?

---

### 4.13 Cross-layer: MAC-protocol Keuze bij AODV

AODV is reactief/on-demand: routes worden pas ontdekt wanneer nodig (RREQ flooding). Topologie is dynamisch — nodes komen en gaan. Dit vereist een MAC dat **op elk moment kan zenden zonder vooraf geconfigureerde schedule**.

| MAC | Past bij AODV? | Waarom |
|---|---|---|
| **CSMA/CA** | **Goed** | Zendt wanneer nodig, geen schedule, tolereert nodes die komen/gaan |
| **BMAC** | **Goed** | Idem + energiebesparing via LPL; asynchroon, geen centrale coordinatie |
| **TSMP/TDMA** | **Slecht** | Vereist pre-established schedule + tijdsynchronisatie → clashed met AODV's dynamische route discovery |
| **Pure ALOHA** | **Slecht** | Geen carrier sense, geen backoff → hoge collision rate bij RREQ flooding |

**Kernredenering:** scheduled MACs (TSMP/TDMA) veronderstellen een stabiele topologie met bekende deelnemers. AODV veronderstelt het tegenovergestelde. Een topologiewijziging bij TDMA forceert kostbare herscheduling, terwijl bij CSMA/CA of BMAC een nieuwe node gewoon kan beginnen zenden.

**Waarom Pure ALOHA ook slecht is:** AODV's route discovery gebruikt flooding (RREQ naar alle buren). Zonder carrier sense of backoff leidt dit tot massale collisions — precies wanneer het netwerk communicatie het hardst nodig heeft.
