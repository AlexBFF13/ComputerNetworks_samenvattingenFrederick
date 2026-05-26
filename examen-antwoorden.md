# Computernetwerken — Academische Examenantwoorden

> Gebaseerd op de cursusteksten uit deze repository.

---

## Deel 1 — Application Layer (Laag 7)

### 1. Gnutella bovenop een Chord Overlay-netwerk

**Wat vervangt Chord?**

De cursustekst beschrijft Gnutella als een *"unstructured decentralized overlay network"* met flooding-gebaseerde zoekopdrachten (QUERY-berichten met TTL). Het grote probleem is het **search horizon**-probleem: *"te lage TTL: je vindt te weinig; te hoge TTL: je overspoelt het netwerk."*

Chord is een **gestructureerde DHT** (Distributed Hash Table) die bestanden mapt op nodes via consistente hashing. In een hybride ontwerp:

**Wat Chord vervangt:**
- De willekeurige graaf-topologie van Gnutella → Chord geeft een deterministische ring-structuur met O(log N) finger-table-entries per node.
- Het flooding-mechanisme voor resource discovery → in plaats van QUERY-berichten te flooden, hash je de bestandsnaam en zoek je de verantwoordelijke node in O(log N) hops.

| Aspect | Gnutella alleen | Gnutella op Chord overlay |
|---|---|---|
| Netwerk-overhead | Hoog — flooding met TTL, O(N) berichten | Laag — O(log N) gerichte hops |
| Zoekgarantie | Niet gegarandeerd (TTL-horizon) | Gegarandeerd als bestand bestaat |
| Privacy / anonimiteit | Redelijk — zoekopdracht verspreidt zich breed | Lager — zoekpad is deterministisch en traceerbaar |
| Robuustheid bij uitval | Hoog — redundante paden | Medium — keystorage-nodes moeten beschikbaar zijn |
| Schaalbaarheid | Slecht bij grote netwerken | Goed — O(log N) per lookup |

**Vereisten aan de hashfunctie van Chord**

Chord gebruikt consistente hashing (SHA-1 of gelijkwaardig). De hashfunctie moet aan twee eisen voldoen:

1. **Uniforme (random) verdeling over de keyspace**: elke node krijgt een gelijkmatig deel van de 2^160 keyruimte verantwoordelijk. Nodes worden via hashing geplaatst op een circulaire ring van 0 tot 2^160 − 1.

2. **Uniciteit (collision-vrij)**: twee verschillende bestanden of nodes mogen niet dezelfde hashwaarde krijgen, anders zou één node verantwoordelijk zijn voor twee conflicterende resources.

**Effect als de hash niet random of uniform verdeeld is:**

- **Hotspot-probleem**: als bepaalde sleutels geconcentreerd zijn rond bepaalde waarden (b.v. alle bestandsnamen beginnen met een letter van A-D), dan dragen de nodes die verantwoordelijk zijn voor dat deel van de ring een disproportioneel grote load. Dit zijn zogenaamde *hot nodes* die overbelast raken terwijl de rest van het netwerk onderbenut is.
- **Ongelijke verantwoordelijkheidsgebieden**: als nodes via een slechte hashfunctie geclusterd worden op de ring (in plaats van uniform verdeeld), krijgen sommige nodes een enorm groot "virtueel segment" van de keyspace en andere bijna niets.
- **Praktische oplossing**: Chord gebruikt "virtual nodes" (elke fysieke node heeft meerdere virtuele posities op de ring) en een cryptografisch sterke hashfunctie (SHA-1) om statistisch uniform te verspreiden.

---

### 2. Spionage in Gnutella 0.4 versus Gnutella 0.6

**Gnutella 0.4 architectuur**

Vlak netwerk: alle peers zijn gelijk. QUERY-berichten worden via flooding met TTL doorgegeven. Elke node ziet berichten van zijn directe buren.

**Gnutella 0.6 architectuur**

Hiërarchisch: ultrapeers (supernodes) verbinden leaf nodes. Ultrapeers dragen het zoekverkeer; leaf nodes connecteren alleen met ultrapeers.

---

**Scenario A: Ongelimiteerde resources**

*Gnutella 0.4:*
- Stel honderden nodes op die verbinden met veel peers tegelijk.
- Elk inkomende QUERY passeert via flooding, wat betekent dat jouw nodes een groot deel van het netwerk zien.
- Door als centrale knooppunten te fungeren (hoge degree) zie je het meeste verkeer.
- Je ziet: **alle QUERY-keywords** die over jou passeren, IP-adressen van aanvragers, TTL (geeft positie in het netwerk).
- Via QUERYHIT zie je: bestandsnamen, bestandsgroottes, IP van de node die het bestand heeft.

*Gnutella 0.6:*
- Word ultrapeer op meerdere plekken in het netwerk.
- Ultrapeers zien **alle QUERY's van hun leaf nodes** + QUERY's van buurultrapeers.
- Als je 10% van de ultrapeers controleert, zie je al een significant deel van het verkeer (leaf nodes verbinden met slechts 1-3 ultrapeers).
- Je ziet: welke leaf node welke keywords zoekt, en via QUERYHIT wie welk bestand aanbiedt.

**Scenario B: Beperkte resources (1 node)**

*Gnutella 0.4 — als gewone peer:*
- Je verbindt met enkele peers (typisch 5-10 directe verbindingen).
- Je ziet enkel QUERY's die via jouw directe verbindingen passeren.
- Zichtbare data:
  - **QUERY**: zoektermen, unieke MessageID, TTL, IP van de zender (jouw directe buur)
  - **QUERYHIT**: bestandsnaam, filesize, SHA-hash, IP+poort van de aanbieder
  - **PING/PONG**: IP-adressen en poorten van actieve peers

*Gnutella 0.6 — als leaf node bij een ultrapeer:*
- Je verbindt enkel met de ultrapeer. Je ziet alleen berichten die jouw ultrapeer jou stuurt.
- Je krijgt geen routing-QUERY's van andere leaf nodes te zien.
- Beperkte zichtbaarheid: je kan enkel jouw eigen zoekopdrachten en antwoorden observeren.

*Gnutella 0.6 — als ultrapeer (beperkte resources):*
- Je beheert meerdere leaf nodes en peert met andere ultrapeers.
- Je ziet alle QUERY's van jouw leaf nodes (hun zoekgedrag).
- **Afleidbare informatie**:
  - Tijdstip van zoekopdrachten → activiteitspatronen van gebruikers
  - Zoektermen → interesses, mogelijk illegale content
  - IP-adres van leaf nodes → geolokalisatie
  - Bestandsnamen in QUERYHIT → wat gebruikers op hun computer hebben

**Vergelijkingstabel:**

| Aspect | 0.4 (gewone peer) | 0.4 (centrale peer) | 0.6 (leaf node) | 0.6 (ultrapeer) |
|---|---|---|---|---|
| Zichtbaar verkeer | Beperkt (nabijheid) | Groot deel netwerk | Eigen verkeer | Alle leaf nodes + peers |
| Anonimiteit gebruikers | Relatief hoog | Laag | Hoog | Laag |
| Resources vereist | Laag | Hoog | Laag | Medium |
| Detecteerbaarheid | Laag | Hoog (hoge degree) | Laag | Medium |

---

### 3. TOR-netwerken

**Welke informatie bezit elke node?**

| Node | Bron-IP | Bestemming-IP | Data | Vorige hop | Volgende hop | Sessiesleutels |
|---|---|---|---|---|---|---|
| Entry Node | **Ja** (echte client) | Nee | Versleuteld (2 lagen) | Client (direct) | Intermediate node IP | Gedeelde sleutel met client |
| Intermediate Node | Nee | Nee | Versleuteld (1 laag) | Entry node IP | Exit node IP | Gedeelde sleutel met client |
| Exit Node | Nee | **Ja** (echte server) | Plaintext (als HTTP) | Intermediate node IP | Server (internet) | Gedeelde sleutel met client |

De cursustekst: *"alle links in TOR zijn versleuteld, behalve de uitgaande link vanaf de exit node naar het gewone internet."*

**Hoeveel intermediate nodes zijn wiskundig minstens nodig voor anonimiteit?**

Het minimum is **1 intermediate node** (totaal 3 nodes in het circuit: Entry + Intermediate + Exit).

Redenering:
- Met **0 intermediate nodes** (Entry + Exit): de entry node kent de echte bron-IP, de exit node kent de echte bestemming. Eén gecompromitteerde adversary die beide controleert, of één rechterlijk bevel aan beide providers, levert volledige deanonymisatie op. Bovendien kent de entry node zowel de bron als de volgende hop (= exit = bestemming). Een adversary met toegang tot entry en kennis van uitgangsverkeer kan correleren.
- Met **1 intermediate node** (3 nodes): geen enkele node kent zowel bron als bestemming. Een adversary moet zowel de entry ALS de exit node controleren voor correlatie-aanvallen. De intermediate node fungeert als een "firewall" die de directe link breekt.

**Nadelen van 10 hops:**

1. **Latency**: elke hop voegt een volledige RTT toe. Bij 50-200 ms per hop: 10 hops × gemiddeld 100 ms = 1 seconde extra latentie per richtingswisseling.

2. **Verhoogde detectiekans**: als een adversary 20% van de TOR-nodes controleert, is de kans dat minstens één gecompromitteerde node op een pad zit: `1 - (0.8)^N`. Voor N=3: ~49%. Voor N=10: ~89%. Meer hops kan paradoxaal *minder* veilig zijn.

3. **Geen extra bescherming tegen globale observatie**: een *global passive adversary* (NSA-niveau) kan nog altijd timing-correlatie doen, ongeacht het aantal hops. Het fundamentele aanvalsmodel wordt niet door extra hops verslagen.

4. **Vaste cell-grootte in TOR**: TOR gebruikt fixed-size cells om traffic analysis te bemoeilijken, maar dat helpt niet bij globale timing-correlatie.

---

### 4. DNS Resolutie-architectuur

**Vergelijking: Puur Iteratief vs. Puur Recursief**

De cursustekst: *"DNS-lookup combineert typisch een recursive component aan de kant van de lokale name server en iterative stappen hoger in de hiërarchie."*

| Kenmerk / Metriek | Puur Iteratieve DNS | Puur Recursieve DNS |
|---|---|---|
| **Caching Effectiviteit** | Laag — elke client doet zijn eigen lookup; gedeelde cache-infrastructuur ontbreekt | Hoog — de lokale resolver cached antwoorden voor alle clients tegelijk (TTL-gebaseerd) |
| **Load op Root Name Servers** | Hoog — elke client begint zelf bij de root voor elke query | Laag — roots sturen enkel stateless referrals terug; lokale resolvers bufferen root-antwoorden |
| **Garantie op Partiële Data** | Laag — als een tussenliggende server uitvalt, heeft de client geen fallback | Medium — de recursieve server kan zijn eigen cache raadplegen en gedeeltelijk antwoorden |
| **End-to-End Latency (client)** | Hoog — de client maakt meerdere round trips (root → TLD → authoritative) | Laag — de client communiceert enkel met de lokale resolver en wacht op één antwoord |
| **Complexiteit voor client** | Hoog — de client implementeert de volledige iteratieve logica | Laag — de client stelt één vraag en krijgt één antwoord |
| **Schaalbaarheid** | Slecht — root servers worden bottleneck bij miljoenen directe clients | Goed — lokale resolvers absorberen het gros van de queries via caching |

**Waarom het hybride model superieur is:**

Het hybride model (client → lokale resolver recursief, lokale resolver → root/TLD iteratief) combineert de voordelen van beide:
- De **client** heeft de eenvoud van een recursieve aanpak (één vraag, één antwoord).
- De **root servers** zijn stateless en schaalbaar (enkel referrals, geen state bijhouden).
- De **lokale resolver** heeft een grote gedeelde cache die de load op de rest van de hiërarchie drastisch verlaagt.

---

### 5. FTP en NAT-incompatibiliteit

**Waarom twee afzonderlijke TCP-poorten?**

De cursustekst: *"FTP is een bestandsprotocol met gescheiden control- en datakanalen, wat het architecturaal anders maakt dan HTTP."*

De scheiding van control (poort 21) en data (poort 20) biedt twee voordelen:
1. **Persistente sessies**: het controlekanaal blijft open voor de hele FTP-sessie (meerdere bestandsoverdrachten), terwijl datakanalen per overdracht worden opgezet en afgebroken.
2. **Orthogonaliteit**: commandoverwerking en datatransfer kunnen onafhankelijk verlopen, wat eenvoudigere implementatie van de toestandsmachine mogelijk maakt.

**Waarom Active FTP onvermijdelijk faalt achter NAT**

```
Client (192.168.1.10, achter NAT) → Server (203.0.113.5)
Stap 1: Client opent TCP-verbinding naar server:21 (control) ✓ NAT laat dit door
Stap 2: Client stuurt: PORT 192.168.1.10,200,105 (= privé-IP:51309)
Stap 3: Server probeert TCP SYN naar 192.168.1.10:51309 (van buiten naar binnen)
→ NAT heeft GEEN mapping voor inkomende verbinding op poort 51309
→ NAT dropt het packet
→ Dataverbinding mislukt
```

Het fundamentele probleem: NAT houdt enkel state bij voor *uitgaande* verbindingen. Een inkomende verbinding die de *server* initieert naar het privé-IP van de client, is voor de NAT-box onbekend en wordt gedropt. Firewalls versterken dit: inkomende verbindingen op willekeurige hoge poorten zijn typisch geblokkeerd.

**Hoe Passive FTP (PASV) dit oplost**

```
Client → Server: PASV
Server → Client: 227 Entering Passive Mode (203,0,113,5,200,105)
                 (= verbind met 203.0.113.5:51309)
Client → Server: TCP SYN naar 203.0.113.5:51309 (CLIENT initieert data!)
NAT: uitgaande verbinding → NAT maakt mapping aan → ✓ werkt
```

Met PASV initieert de **client** beide verbindingen (control én data). Alle TCP-verbindingen zijn outbound vanuit het NAT-netwerk. NAT ondersteunt dit naadloos. De server geeft zijn eigen publieke IP en een willekeurige poort terug via het controlekanaal.

---

### 6. DHCP en UDP

**De vier fasen van de DHCP-uitwisseling**

De cursustekst: *"De basisflow van DHCP is: DISCOVER → OFFER → REQUEST → ACK."*

**Fase 1 — DHCPDISCOVER (Client → Broadcast)**
- De client heeft nog geen IP-adres.
- Bron-IP: `0.0.0.0` (onbekend), doel-IP: `255.255.255.255` (broadcast).
- De client kiest een willekeurige **Transaction ID** om antwoorden te koppelen.
- Vraag: "Is er een DHCP-server die mij een adres kan geven?"

**Fase 2 — DHCPOFFER (Server → Broadcast/Unicast)**
- De DHCP-server antwoordt met een voorstel: een IP-adres, subnetmask, gateway, DNS-servers, en de **lease-tijd**.
- Bevat dezelfde Transaction ID als de DISCOVER.
- Omdat de client nog geen adres heeft, kan dit als broadcast teruggestuurd worden.
- Meerdere servers kunnen aanbieden; de client kiest één.

**Fase 3 — DHCPREQUEST (Client → Broadcast)**
- De client bevestigt zijn keuze voor een specifieke server (via Server Identifier option).
- Nogmaals broadcast: andere servers die ook een OFFER stuurden, weten zo dat hun aanbod afgewezen is.
- Bron-IP: nog steeds `0.0.0.0` (het adres is nog niet officieel geaccepteerd).

**Fase 4 — DHCPACK (Server → Client)**
- De server bevestigt de toewijzing. Nu is het IP-adres officieel geleased.
- Na ontvangst doet de client **Duplicate Address Detection** via ARP (gratuitous ARP) om te controleren of het adres al in gebruik is.
- Bij conflict: client stuurt DHCPDECLINE en begint opnieuw.

**Waarom is DHCP over TCP fundamenteel onmogelijk?**

TCP vereist een **three-way handshake** om een verbinding op te zetten: `SYN → SYN-ACK → ACK`. Daarvoor heeft een host nodig:
1. Een geldig **bron-IP-adres** voor zijn SYN-pakket.
2. Een geldig **doel-IP-adres** van de server (de client weet niet welke DHCP-server aanwezig is).

Tijdens DHCPDISCOVER heeft de client **noch een bron-IP, noch een server-IP**. Er is geen manier om een TCP-handshake te initiëren. TCP is per definitie unicast en connection-oriented; DHCP heeft een broadcast nodig in de discovery-fase. UDP laat toe dat de client zendt vanuit `0.0.0.0` naar `255.255.255.255` zonder enige voorafgaande state, wat precies past bij de bootstrapping-situatie van een DHCP-client.

---

## Deel 2 — Transport Layer (Laag 4)

### 1. RTP versus TCP

**Sequencenummers**

| Eigenschap | TCP | RTP |
|---|---|---|
| Grootte | 32-bit | 16-bit (+ 32-bit timestamp apart) |
| Wat wordt genummerd | Bytes (bytepositie in de stream) | Packets (oplopend per packet met ±1) |
| Doel | Volgordebeheer, retransmissie, SACK, flow control | Verliesdetectie, volgordeherstel voor afspeelbuffer |
| Timing | Impliciet via RTT-schatting | Expliciet via 32-bit RTP-timestamp (mediatingeenheid) |

**Ontwerplogica van het verschil:**

TCP is een *byte-stream*: elke byte telt. 32-bit geeft ruimte voor lange sessies (bij 1 Gbps duurt wraparound: 2^32 bytes / 125 MB/s ≈ 34 seconden — vandaar ook PAWS-bescherming). RTP is ontworpen voor *realtime media*. De cursustekst: *"bij video of audio is retransmissie vaak niet handig. Als een videoframe te laat aankomt, dan is het al waardeloos."* Packets tellen volstaat voor verliesdetectie. De 32-bit timestamp geeft het exacte afspeelmoment aan in media-eenheden (b.v. 90 kHz klok voor video), wat een bytepositie niet uitdrukt. 16-bit voor sequencenummers is voldoende voor korte vensters in real-time applicaties.

**RTP versus TCP voor video-download (niet streamen)**

TCP is de betere keuze:
- RTP biedt geen garanties voor volledige bestandsoverdracht; packets die verloren gaan worden niet hertransmitteerd.
- Een videobestandsdownload vereist **alle bytes**, want een ontbrekend GOP (Group of Pictures) maakt het bestand onbruikbaar of ondecodeerbaar.
- TCP's betrouwbare, geordende byte-stream is de architectureel correcte keuze voor file transfers.
- RTP's tolerantie voor verlies is een feature voor live media, niet voor stored-content delivery.

**Congestieprobleem van high-volume RTP**

RTP draait bovenop UDP. UDP heeft geen congestion control. Bij hoge belasting:
- **Unfairness (TCP-starvation)**: TCP-flows detecteren congestie (via verlies of ECN) en halveren hun congestion window (AIMD). RTP/UDP zendt aan constante rate. Op een gedeelde link krijgen TCP-flows structureel minder bandbreedte dan RTP-flows.
- **Congestiecollaps**: als al het RTP-verkeer gedropt wordt door volle queues en er geen snelheidsaanpassing plaatsvindt, worden voortdurend nutteloze packets verstuurd. Dit is precies de collapse die TCP-congestiecontrole wil vermijden.
- **Geen ACK-clock**: TCP's self-clocking mechanisme smootht de injectiesnelheid automatisch op basis van de bottleneck. RTP mist dit en kan burst-gedrag veroorzaken die routerqueues instantaan vult.

---

### 2. Sliding Window Protocols in extreme netwerkomgevingen

**Wiskundige evaluatie over een satellietverbinding (RTT = 2600 ms, packetgrootte = 1700 bytes)**

Basisformule voor efficiëntie:
- a = propagatievertraging (één richting) / transmissietijd = (RTT/2) / T_trans

**Stop-and-Wait maximale throughput**

Stel een linksnelheid van 10 Mbps:
- Transmissietijd: T_trans = (1700 × 8) / (10 × 10^6) = 13.600 / 10.000.000 = **1,36 ms**
- Propagatievertraging: RTT/2 = 1300 ms
- a = 1300 / 1,36 ≈ **956**
- Efficiëntie Stop-and-Wait: η = 1 / (1 + 2a) = 1 / (1 + 1912) ≈ **0,052%**
- Maximale throughput: 0,00052 × 10 Mbps ≈ **5,2 kbps**

Dit is catastrofaal: de zender zendt 1 packet van 1,36 ms, en wacht vervolgens 2600 ms op de ACK.

**Vergelijkingstabel voor de drie protocollen:**

| Protocol | Vereist venster voor η = 100% | η als W te klein | η formule |
|---|---|---|---|
| Stop-and-Wait | W = 1; werkt enkel als a << 1 | 1 / (1 + 2a) ≈ 0,052% | 1 / (1 + 2a) |
| Go-Back-N | W ≥ 1 + 2a = 1913 | W / (1 + 2a) | min(W/(1+2a), 1) |
| Selective Repeat | W ≥ 1 + 2a = 1913 | W / (1 + 2a) | min(W/(1+2a), 1) |

Voor 100% efficientie over satelliet met RTT 2600 ms en pakket 1700 bytes: venster van **minstens 1913 packets** nodig.

**Op welk type medium presteert Stop-and-Wait wel efficiënt?**

Stop-and-Wait is efficiënt wanneer **a ≈ 0**, d.w.z. de propagatievertraging is verwaarloosbaar klein vergeleken met de transmissietijd:

- **Trage links over korte afstanden**: een seriële verbinding van 9600 bps over 1 meter kabel. T_trans voor 1700 bytes = (13600 bits) / 9600 bps = 1,42 seconden. Propagatievertraging: 1 m / (2×10^8 m/s) = 5 ns. a ≈ 0 → η ≈ 100%.
- **LoRa/LPWAN-netwerken**: bij SF12 (250 bps) en een packet van 50 bytes: T_trans = 1,6 s, RTT ≈ 134 μs. a ≈ 0,00008 → η ≈ 99,98%.
- **Seriële RS-232 verbindingen** met lage baudrate.

Het algemene principe: Stop-and-Wait is efficiënt wanneer het **bandwidth-delay product** (bandbreedte × RTT) kleiner is dan één packetgrootte.

---

### 3. TCP Flow Control, Congestie en ACK Clock

**Vergelijking TCP Flow Control met Go-Back-N**

| Eigenschap | TCP Flow Control | Go-Back-N |
|---|---|---|
| Venstereenheid | Bytes (dynamisch, door ontvanger geadverteerd) | Packets (maximaal N, vast per protocol) |
| Vensteradvertentie | Ontvanger stuurt `WIN`-veld (vrije bufferruimte) | Venstergrootte is protocol-parameter |
| Retransmissie bij verlies | Timeout of 3 duplicate ACKs → retransmit ontbrekende bytes (met SACK: selectief) | Timeout → retransmit verloren packet **en alle volgende** nog niet bevestigde packets |
| Ontvanger buffert out-of-order? | Ja, gebufferd en met SACK bevestigd | Nee — verworpen, venster van grootte 1 aan ontvangerzijde |
| ACK-type | Cumulatief (+ optioneel SACK) | Puur cumulatief |

**Algoritmisch verschil bij hertransmissie:**

Go-Back-N: bij verlies van packet k worden packets k, k+1, k+2, ... tot k+N-1 **allemaal opnieuw gestuurd**, ook al zijn k+1, k+2, ... al correct aangekomen bij de ontvanger.

TCP zonder SACK: vergelijkbaar — retransmit vanaf het laatste cumulatief bevestigde punt.

TCP met SACK: de ontvanger geeft aan welke ranges al ontvangen zijn. De zender retransmit enkel de specifieke gaten. Veel efficiënter bij meerdere verliezen per venster.

**ACK Clock: mechanisme voor heterogene netwerken**

De cursustekst: *"nieuwe data wordt alleen het netwerk ingestuurd aan het tempo waarop ACKs terugkeren. ACKs keren terug aan het tempo waarop de bottlenecklink data kan verwerken."*

Werking bij een snelle → trage link overgang:

```
Zender (1 Gbps) ──────► Trage bottleneck (1 Mbps) ──────► Ontvanger
                            ↑ packets worden hier
                              uniform uitgesmeerd
```

1. Zender injecteert een burst packets op de snelle link.
2. De trage bottleneck spaceert packets uit in de tijd (pacing door beperkte bandbreedte).
3. Packets arriveren bij de ontvanger uniform gespreid.
4. De ontvanger stuurt ACKs terug — ook uniform gespreid.
5. ACKs reizen snel terug naar de zender, maar arriveren met dezelfde spacing als de bottleneck.
6. De zender stuurt nieuwe packets op het ritme van inkomende ACKs: niet sneller dan de bottleneck.

**Resultaat**: zelfregulerende pace-matching zonder expliciete terugkoppeling over netwerktopologie. Routerbuffers worden niet systematisch overbelast. Dit is de basis van TCP's "self-clocking" gedrag.

---

### 4. Interactie Delayed ACKs en Nagle's Algorithm

**Delayed Acknowledgements:**
De ontvanger wacht tot 200 ms (of tot return data voor piggybacking) alvorens een ACK te sturen. Doel: minder kleine controlepakketten.

**Nagle's Algorithm:**
Zolang er een onbevestigd segment onderweg is, worden nieuwe kleine data gebufferd tot genoeg data voor een volledig segment verzameld is, of tot de ACK terugkomt. Doel: tinygram syndrome vermijden.

**De deadlock/degradatie:**

```
Stap 1: Client stuurt klein request (< 1 MSS)
        → Nagle: "Er is nu een uitstaand onbevestigd segment. Buffer nieuwe data."

Stap 2: Server ontvangt request
        → Delayed ACK: "Wacht max 200ms op return data om de ACK te piggybacken."

Stap 3: Server heeft klein response, maar wacht ook:
        → Server's Nagle: "Wacht op ACK van client (als server ook piggybacking doet)"

Resultaat:
- Client wacht op ACK van server (Nagle blokkering)
- Server wacht 200ms timer (Delayed ACK)
→ 200ms vertraging per kleine interactie, systematisch
```

Dit is geen permanente deadlock (de 200ms-timer loopt af), maar veroorzaakt **systematische 200ms-latency** bij elke kleine interactie. Voor interactieve toepassingen (SSH, games, HTTP/1.x met kleine requests) is dit onaanvaardbaar.

**Oplossingen:**
- `TCP_NODELAY`: disablet Nagle aan de zenderkant (Nagle's algorithm uitschakelen).
- `TCP_QUICKACK`: disablet Delayed ACKs aan de ontvangerkant.
- Modern: HTTP/2 en QUIC vermijden dit structureel via betere stream multiplexing.

---

## Deel 3 — Network Layer (Laag 3)

### 1. IPv4 Fragmentation versus Path MTU Discovery

**Fundamentele incompatibiliteit via de DF-bit**

De DF-bit (Don't Fragment) in de IPv4-header bepaalt het gedrag van routers bij te grote packets:

| DF-bit | Routergedrag bij te groot packet | Feedback aan zender |
|---|---|---|
| DF = 0 (fragmentation aan) | Router splitst packet stilzwijgend op | **Geen** — zender leert nooit de werkelijke path MTU |
| DF = 1 (PMTUD aan) | Router dropt packet en stuurt ICMP terug | **Ja** — ICMP Destination Unreachable (Type 3, Code 4) |

Ze zijn incompatibel omdat ze tegengestelde aannames maken: fragmentation *verbergt* de MTU-mismatch voor de zender, PMTUD *onthult* haar.

**Expliciete rol van ICMP Destination Unreachable**

ICMP Destination Unreachable (Type 3, Code 4 = "Fragmentation Needed and DF bit is set") is het **signaalkanaal** van PMTUD:
- De router dropt het packet dat DF = 1 heeft maar te groot is.
- De router stuurt ICMP Type 3/Code 4 terug naar de bronhost, inclusief de MTU van de bottleneck-link.
- De bronhost verlaagt zijn packetgrootte tot onder die MTU en herverzendt.
- Dit convergeert totdat de zender de minimale MTU langs het hele pad kent.

**PMTUD Black Hole:** als firewalls ICMP blokkeren (een veel voorkomende misconfiguratie), ontvangt de zender nooit de MTU-informatie. De verbinding "hangt": grote packets worden gedropt maar de zender weet niet waarom. Oplossing: RFC 4821 (Packetization Layer PMTUD) detecteert dit via TCP-gedrag zonder ICMP.

**Architecturale aanpassing voor compatibiliteit**

Een "Fragment + Notify" schema: de router fragmenteert het packet (backwards-compatibel) EN stuurt een ICMP-informatiegericht bericht terug.

Dit is **niet efficiënt** om de volgende redenen:
- De fragmentatie is al gebeurd: voor korte sessies convergeert de oplossing nooit.
- Extra ICMP-storm bij drukke links.
- Routers moeten state bijhouden over welk bronpad al een ICMP ontving (anders: herhaalberichten).
- IPv6 lost dit principiëler op: fragmentatie door routers is verboden; PMTUD is verplicht.

---

### 2. Congestion Notificatie: Vergelijkingstabel

| Notificatie mechanisme | Werking | Snelheid van feedback | Extra Netwerk Overhead | Efficiëntie bij snelle zenders |
|---|---|---|---|---|
| **Choke Packets** | Router genereert een apart controlpacket richting de zender wanneer congestie gedetecteerd wordt | Traag — volledige RTT + verwerking; zender reageert pas na terugkomst choke packet | Hoog — extra packets in het al overbelaste netwerk | Laag — één choke packet per flow; zender ontvangt geen continuus signaal |
| **Explicit Congestion Notification (ECN)** | Router zet een markeerbit in een bestaand doorgaand packet; ontvanger echo't dit via TCP ECE-bit terug; zender reduceert snelheid | Iets trager dan choke — markering gaat eerst naar ontvanger, dan via ACK terug; ±1 RTT meer | Laag — enkel een bit zetten; geen extra packets | Goed — signaal per flow, werkt proportioneel |
| **Random Early Detection (RED)** | Router dropt packets willekeurig al vóór de queue vol is, zodra gemiddelde queuelength boven drempel stijgt | Snel — drops zijn onmiddellijk; TCP interpreteert verlies direct | Negatief effect — nutteloze hertransmissies van gedropt verkeer | Hoog — snelle senders hebben meer packets in de queue en worden proportioneel vaker getroffen |

---

### 3. IoT en de IP Stack

**Technologische barrières van IPv4 en IPv6 voor IEEE 802.15.4**

De cursustekst: *"resource constrained: weinig geheugen, opslag en rekenkracht; energy constrained: nodes moeten lang op batterij werken; unreliable: uitval van nodes is normaal."*

IEEE 802.15.4 maximale framegrootte: 127 bytes.

| Barrière | IPv4 | IPv6 |
|---|---|---|
| Header overhead | 20 bytes minimaal (variabel tot 60 bytes) | 40 bytes vast — al een derde van het totale frame |
| Adresruimte | 32-bit — uitgeput, vereist NAT; NAT breekt directe M2M communicatie | 128-bit — voldoende voor elk sensorobject op aarde |
| Autoconfiguratie | DHCP vereist server | SLAAC — nodes configureren zich zelfstandig |
| Fragmentatie | Kan door routers (duur); fragmentatie van 127-byte frames verergert overhead | Routers fragmenteren niet (PMTUD verplicht); minimale MTU 1280 bytes (probleem voor 802.15.4!) |
| NAT-compatibiliteit | Slechte NAT-interactie voor IoT push-verkeer | Geen NAT nodig → directe end-to-end communicatie |

**Welk protocol heeft de overhand?**

IPv6 via **6LoWPAN** (IPv6 over Low-Power Wireless Personal Area Networks, RFC 4944) wint op de volgende gronden:
- 6LoWPAN comprimeert de 40-byte IPv6-header naar typisch 2-3 bytes voor lokaal verkeer.
- Fragmentatie van grote IPv6-packets over meerdere 802.15.4-frames wordt door 6LoWPAN afgehandeld.
- SLAAC maakt server-loze netwerken mogelijk.
- De IETF heeft IoT-specifieke protocollen (CoAP, ROLL) ontworpen bovenop IPv6/6LoWPAN.

---

### 4. IPv6 SLAAC en Privacy-inbreuk via EUI-64

Bij standaard SLAAC leidt een node zijn 64-bit Interface Identifier af uit zijn MAC-adres (EUI-64 formaat):
- De 48-bit MAC `00:1A:2B:3C:4D:5E` wordt `02:1A:2B:FF:FE:3C:4D:5E` (FFFE ingevoegd, U/L-bit omgedraaid).
- Het resulterende IPv6-adres: `prefix::021A:2BFF:FE3C:4D5E`.

**Privacy-inbreuk:**
1. **Permanente tracking**: het MAC-adres is hardware-permanent. De interface identifier blijft constant, ook als de node van netwerk wisselt (b.v. thuis → kantoor → openbaar wifi). Een tracker die het IPv6-adres ooit zag, kan het toestel herkennen op elk netwerk.
2. **Adresvoorspelbaarheid**: een aanvaller die het MAC-adres kent (via fysieke toegang, ARP-sniffing, of eerder Bluetooth-scanning) kan het IPv6-adres van het toestel op elk netwerk voorspellen en gerichte aanvallen of fingerprinting uitvoeren.
3. **OUI-informatie**: de eerste 24 bits van een MAC identificeren de fabrikant (Organizationally Unique Identifier). Dit verraadt het toesteltype, wat aanvallers helpt bij het selecteren van exploits.

**Tegenmaatregelen:** RFC 7217 (stabiele maar opake adressen) en RFC 4941 (tijdelijke privacyadressen die regelmatig veranderen) pakken dit aan, maar vereisen extra implementatiewerk op resource-arme IoT-devices.

---

### 5. AODV versus OSPF in ad-hoc netwerken + Count-to-Infinity

**Waarom OSPF slecht presteert in ad-hoc topologieën**

OSPF is een *proactief* link-state protocol: alle routers floden voortdurend link-state-informatie om een consistente topologieview te onderhouden. In ad-hoc netwerken:
- Topologie verandert constant (nodes bewegen, batterijen raken leeg).
- Elke topologiewijziging triggert een nieuwe flooding-cyclus.
- **Overhead**: voor N nodes met snelle veranderingen wordt het netwerk gedomineerd door control traffic in plaats van applicatiedata.
- **Geheugen**: een volledig topologiebeel vereist O(N) state per router — onaanvaardbaar voor resource-arme ad-hoc nodes.

**Waarom AODV superieur presteert**

De cursustekst: *"AODV is reactief: routes worden pas gezocht wanneer iemand effectief data wil sturen."*

- AODV onderhoud **geen proactieve routing tables**. Geen control traffic als er geen communicatie is.
- Routes worden on-demand ontdekt via ROUTE_REQUEST flooding (met TTL-beperking).
- Nodes met bekende routes sturen ROUTE_REPLY terug; onderweg wordt de route opgeslagen.
- **Adaptiviteit**: als een route uitvalt, wordt een nieuwe ROUTE_REQUEST gestart. Oude routes worden via route maintenance verwijderd.

**Hoe AODV het Count-to-Infinity-probleem ondervat**

Bij klassiek Distance Vector (Bellman-Ford) kennen routers geen volledig padoverzicht. Een lus in de redenering (B denkt via C, C denkt via B) leidt tot langzaam oplopende afstanden.

AODV's aanpak: elke ROUTE_REPLY en ROUTE_REQUEST draagt een **destination sequence number** (geslagen door de bestemmingsnode zelf). Een route met een *hogere* sequencenummer is per definitie verser.

- Als een route uitvalt, verhoogt de destination zijn sequencenummer en stuurt een route-error.
- Nodes die een verouderde route (lager sequencenummer) zien, verwerpen die onmiddellijk.
- Een count-to-infinity lus produceert routes met oude sequencenummers → worden snel verworpen.

**Beperking**: de cursustekst erkent dat *"lange time-outs nog altijd lastig blijven"* bij gelijktijdige storingen. AODV lost het fundamentele probleem beter op dan Bellman-Ford, maar is niet perfect bij snelle topologiewijzigingen.

---

### 6. Adresruimte en Routing Tables

**Netwerktopologie (192.168.22.0/24)**

Subnetindeling voor enterprise-netwerk:

| Subnet | CIDR | Beschrijving | Bruikbare adressen |
|---|---|---|---|
| LAN A | 192.168.22.0/25 | PC-A, Switch 1, Fileserver | .1 – .126 |
| Inter-router link | 192.168.22.128/30 | R1 ↔ R2 point-to-point | .129 – .130 |
| LAN B | 192.168.22.192/26 | Webserver, Switch 2 | .193 – .254 |

**IP-adrestoewijzing:**

| Host/Interface | IP-adres | Subnetmask | Default Gateway |
|---|---|---|---|
| PC-A | 192.168.22.10 | 255.255.255.128 (/25) | 192.168.22.1 |
| Fileserver | 192.168.22.20 | 255.255.255.128 (/25) | 192.168.22.1 |
| Router 1 — eth0 (LAN A) | 192.168.22.1 | 255.255.255.128 | — |
| Router 1 — eth1 (WAN link) | 192.168.22.129 | 255.255.255.252 | — |
| Router 2 — eth0 (WAN link) | 192.168.22.130 | 255.255.255.252 | — |
| Router 2 — eth1 (LAN B) | 192.168.22.193 | 255.255.255.192 (/26) | — |
| Webserver | 192.168.22.200 | 255.255.255.192 | 192.168.22.193 |

**Routing Table van PC-A:**

| Bestemming | Masker | Gateway | Interface |
|---|---|---|---|
| 192.168.22.0 | /25 | — (direct) | eth0 |
| 0.0.0.0 | /0 | 192.168.22.1 | eth0 |

**Routing Table van Router 1:**

| Bestemming | Masker | Gateway | Interface |
|---|---|---|---|
| 192.168.22.0 | /25 | — (direct) | eth0 |
| 192.168.22.128 | /30 | — (direct) | eth1 |
| 192.168.22.192 | /26 | 192.168.22.130 | eth1 |

**ARP Cache na succesvolle HTTP-request (PC-A → Webserver):**

PC-A stuurt het packet naar zijn default gateway (Router 1), omdat de webserver in een ander subnet zit.

| Host | ARP Cache inhoud |
|---|---|
| PC-A | 192.168.22.1 → MAC(R1-eth0) |
| Router 1 | 192.168.22.10 → MAC(PC-A); 192.168.22.130 → MAC(R2-eth0) |
| Router 2 | 192.168.22.129 → MAC(R1-eth1); 192.168.22.200 → MAC(Webserver) |
| Webserver | 192.168.22.193 → MAC(R2-eth1) |

**Belangrijk**: PC-A heeft enkel een ARP-entry voor Router 1 (zijn gateway), niet voor de webserver. ARP werkt enkel binnen hetzelfde subnet.

---

## Deel 4 — Data Link Layer & Physical Layer (Lagen 1–2)

### 1. BMAC versus TSMP: Wanneer welk protocol?

**Vergelijking per omstandigheid**

| Omstandigheid | BMAC (LPL) | TSMP (TDMA) |
|---|---|---|
| Verkeerspatroon | Onvoorspelbaar, event-driven, bursty | Voorspelbaar, periodiek, geregeld |
| Topologie-stabiliteit | Dynamisch (nodes komen en gaan) | Stabiel met bekende deelnemers |
| Latency-eis | Hoog (onmiddellijke transmissie mogelijk) | Laag (wacht op het juiste tijdslot) |
| Energie bij weinig verkeer | Goed — nodes slapen lang | Uitstekend — precies op geplande momenten actief |
| Energie bij veel verkeer | Slecht — lange preamble per packet | Goed — geen preamble nodig |
| Synchronisatiebehoefte | Geen | Vereist (tijdsmaster aanwezig) |
| Schaalbaarheid | Goed | Beperkt door slottoewijzing |

**BMAC overwint TSMP**: bij event-driven toepassingen (brandalarm, bewegingssensor, sporadische berichten) waar de transmissietijdstip onbekend is en lage latency vereist is.

**TSMP overwint BMAC**: bij regelmatige sensor-polling (temperatuur elke 5 minuten), wanneer nodes gesynchroniseerd zijn en collisions vermeden moeten worden.

**Wiskundige afstelling van Preamble Length en Sample Interval in BMAC**

De fundamentele voorwaarde:

```
Preamble_length ≥ Sample_Interval
```

De zender moet een preamble uitzenden die minstens zo lang duurt als het slaapinterval van de ontvanger, zodat de ontvanger gegarandeerd activiteit detecteert tijdens zijn volgende sample.

Bij **extreem lage datarate** (b.v. 1 kbps):
- Sample interval: T_sample = 500 ms (compromis energie/latency)
- Preamble length: ≥ 500 ms
- Preamble in bits: 500 ms × 1000 bps = **500 bits = 62,5 bytes**
- Elke transmissie kost dus veel meer energie dan de nuttige payload, maar nodes slapen lang.

Bij **extreem hoge datarate** (b.v. 1 Mbps):
- Sample interval: T_sample = 10 ms (radio sneller actief, minder energiebesparing nodig)
- Preamble length: ≥ 10 ms
- Preamble in bits: 10 ms × 1.000.000 bps = **10.000 bits** maar dit is een kleine fractie van de totale datatransmissie.
- De preamble-overhead is relatief laag bij hoge datarates.

**Optimale keuze**: T_sample is een trade-off tussen:
- Korter → lagere latency, maar meer energie voor samples.
- Langer → hogere latency, maar meer energiebesparing.

Energiekosten van zender: E_tx ∝ T_preamble + T_data = T_sample + T_data.

**BMAC + TSMP fusieschema**

```
Tijdframe structuur:
|──── TDMA-slots (periodieke data) ────|── LPL-venster ──|
     ↑ bekende nodes, gesynchroniseerd      ↑ BMAC-stijl samples
       minimum energie                        voor event-driven data
```

- Nieuwe nodes joinen via BMAC (luisteren op een TSMP-beacon zonder synchronisatie).
- Na synchronisatie: schakel over naar TDMA voor geplande slots.
- LPL-venster naast TDMA: nodes doen korte samples buiten hun geplande slots voor sporadisch verkeer.
- Frequentiehopping van TSMP blijft actief in beide zones.

---

### 2. Collision Domains en Kabellengte

**Achtergrond: waarom bestaat de kabellengtelimiet?**

Classic Ethernet gebruikt **CSMA/CD** (Collision Detection). De minimum framegrootte (64 bytes = 512 bits) is zo gekozen dat de zender nog *bezig is met zenden* wanneer het collision-signaal van het verste punt op de kabel terugkeert:

```
Vereiste: T_transmissie ≥ 2 × T_propagatie (round-trip)
Bij 10 Mbps: 512 bits / 10 Mbps = 51,2 μs ≥ 2 × T_prop
→ Max propagatievertraging: 25,6 μs
→ Bij signaalsnelheid ≈ 200.000 km/s: max kabellengte ≈ 2560 m
```

**Fast Ethernet (100 Mbps) met Hubs: limiet verdwijnt NIET**

Hubs zijn elektrische signaalrepeaters. Ze creëren geen nieuwe collision domains — het hele segment blijft één collision domain. Bij 100 Mbps:

```
512 bits / 100 Mbps = 5,12 μs ≥ 2 × T_prop
→ Max propagatievertraging: 2,56 μs
→ Max kabellengte: ~256 m (inclusief hubvertraging: nog korter)
```

De limiet wordt **strenger**, niet versoepeld. Fast Ethernet met hubs heeft een maximale segmentlengte van ±100m (mede door hub-doorlooptijd). De limiet verdwijnt niet.

**Switches met Full-Duplex bekabeling: limiet verdwijnt WEL**

Switches isoleren collision domains: elke poort is een eigen, onafhankelijk collision domain. Met full-duplex:
- Zenden en ontvangen verlopen op **afzonderlijke paren** (b.v. bij UTP: paar 1-2 voor TX, paar 3-6 voor RX).
- Er is geen gedeeld medium meer → **collisions zijn fysiek onmogelijk**.
- CSMA/CD wordt uitgeschakeld in de NIC-firmware.
- De kabellengtegrens die voortvloeide uit collision detection verdwijnt volledig.

De enige resterende kabellengtelimiet bij full-duplex switched Ethernet is signaalattentuatie: typisch **100 meter** voor Cat5e UTP (bepaald door signaalsterkte, niet door timing).

| Configuratie | Collision Domain | CSMA/CD nodig | Kabellengtegrens (collision) |
|---|---|---|---|
| Classic Ethernet (10 Mbps, hub) | Gedeeld (alle hosts) | Ja | ~2500 m |
| Fast Ethernet (100 Mbps, hub) | Gedeeld | Ja | ~100–200 m |
| Switched Ethernet, half-duplex | Per poort | Ja (per poort) | Per poort < 200 m |
| Switched Ethernet, **full-duplex** | Per poort | **Nee** | **Verdwijnt** (enkel attenuatie) |

---

### 3. 802.11 Wi-Fi: Packet Loss als TCP Congestion Signaal

**Het fundamentele probleem**

TCP Tahoe en Reno gebruiken **packet loss als impliciet congestiesignaal**: verlies van een segment = congestie → halveer congestion window (AIMD). Dit werkt op bekabelde netwerken omdat verlies bijna uitsluitend door congestie wordt veroorzaakt.

Op een 802.11 WLAN-verbinding zijn er **meerdere oorzaken van packet loss**, zonder dat congestie betrokken is:
- **Radiointerferentie**: andere WiFi-netwerken op hetzelfde kanaal, magnetrons, Bluetooth.
- **Hidden terminal-collisions**: twee nodes die elkaar niet horen, botsen op de AP.
- **Signaalzwakte (fading)**: de client beweegt van de AP weg; packets worden gecorrumpeerd.
- **MAC-laag retransmissies die uitputten**: 802.11 retransmitteert intern tot 7 keer; als die allemaal falen, ziet de transportlaag één verlies.

**Gevolg voor TCP:**
```
Scenario: Lichte WiFi-interferentie, netwerk 10% bezet
→ 802.11 MAC retransmitteerde intern 7 keer maar faalde
→ TCP ziet: packet loss!
→ TCP concludeert: "Congestie! Halveer congestion window."
→ Throughput daalt van 10 Mbps naar 5 Mbps
→ Oorzaak: niet congestie maar radiointerferentie
```

TCP "bestraft" zichzelf onnodig, wat leidt tot dramatisch lagere throughput op WiFi dan de beschikbare bandbreedte rechtvaardigt.

**Oplossingen:**
- **ECN**: expliciet congestiesignaal vermijdt de verwarring met link-layer fouten.
- **TCP Westwood**: schat bandbreedte via ACK-patronen en distinguisheert type verlies.
- **QUIC**: applicatielaag kan verliestype beter interpreteren.
- **WiFi-link layer ARQ**: 802.11 maskeert fouten al intern; ideaal gedrag als MAC de meeste fouten intern herstelt voordat TCP ze ziet.

---

### 4. CSMA/CA: Hidden/Exposed Terminal, NAV en RTS/CTS

**Hidden Terminal Problem**

```
Node A ←→ Node B (AP) ←→ Node C
A en C kunnen elkaar NIET horen

A: "Kanaal vrij" → begint te zenden
C: "Kanaal vrij" → begint ook te zenden (hoort A niet)
→ Collision bij B
→ Beide transmissies verloren
```

Carrier Sense lost dit niet op: A en C luisteren elk lokaal, maar hun bereiken overlappen niet. Ze zien allebei een "vrij" kanaal.

**Exposed Terminal Problem**

```
Node A ←→ Node B ←→ Node C ←→ Node D
B zendt naar A. C wil zenden naar D.

C: "Kanaal bezet" (hoort B) → wacht onnodig
→ C→D zou A niet storen (D is buiten bereik van A)
→ Onnodige inefficiëntie
```

**Kan traditionele CSMA dit oplossen?** Nee. CSMA luistert enkel lokaal. De fundamentele oorzaak (asymmetrisch bereik) is niet detecteerbaar via carrier sense alleen.

**Network Allocation Vector (NAV)**

NAV is een *virtuele* carrier sense timer. Een node die een frame ontvangt, leest de **Duration-veld** in de 802.11-header. Dit veld geeft aan hoe lang het kanaal bezet zal zijn (inclusief data + ACK). De node stelt zijn NAV in op deze duur en zendt niet totdat de NAV op 0 staat.

```
A ────── DATA (Duration: 1000 μs) ──────► B
         ↑ C hoort dit frame
         C stelt NAV = 1000 μs
         C wacht 1000 μs (zelfs als C B's ACK niet hoort)
```

NAV stelt nodes in staat om het kanaal als "bezet" te beschouwen zonder het signaal te horen — virtuele carrier sense.

**RTS/CTS mechanisme**

RTS (Request To Send) en CTS (Clear To Send) lossen het hidden terminal-probleem op:

```
Stap 1: A stuurt RTS naar B (klein frame, Duration = data + CTS + ACK tijd)
Stap 2: B stuurt CTS naar A (Duration = data + ACK tijd)
         ↑ C hoort CTS van B → stelt NAV → wacht
         (C hoorde A's RTS misschien niet, maar hoort B's CTS wel)
Stap 3: A stuurt DATA naar B (C zwijgt door NAV)
Stap 4: B stuurt ACK naar A
```

**Waarom werkt dit voor het hidden terminal-probleem?**
- A en C kunnen elkaar niet horen (hidden).
- Maar zowel A als C kunnen B horen (ze zijn allebei in bereik van de AP).
- C hoort B's CTS en weet: "B is bezig, ik moet wachten" → geen collision bij B.

**Exposed terminal en RTS/CTS:** RTS/CTS lost het exposed terminal-probleem *niet* op. C hoort B's RTS en stelt zijn NAV in — C wacht onnodig, ook al zou C→D geen interferentie veroorzaken. RTS/CTS maakt het exposed terminal-probleem zelfs iets erger.

**Wanneer RTS/CTS te gebruiken?**

RTS/CTS heeft overhead (2 extra frames per datatransmissie). Het is daarom voordelig enkel voor grote frames:
- 802.11 heeft een configureerbare RTS-threshold (typisch 2347 bytes = uitgeschakeld).
- Voor kleine frames: overhead van RTS/CTS is te groot t.o.v. voordeel.
- Voor grote frames (video, FTP): collision van een groot frame is duurder dan de RTS/CTS-overhead.

---

### 5. LoRa Netwerk: Fundamentele Beperkingen

**LoRa-karakteristieken:**
- Bereik: tot 20 km (landelijk), 2–5 km (stedelijk)
- Datarate: 250 bps (SF12, max bereik) tot 50 kbps (SF7, min bereik)
- Propagatievertraging: 20 km / (3×10^8 m/s) ≈ **67 μs** — verwaarloosbaar

**1. Datatransmissiesnelheid (Duty Cycles)**

LoRaWAN opereert in het licentievrije ISM-band (868 MHz in Europa). Regelgeving beperkt het luchtkanaalgebruik:
- **Duty cycle: maximaal 1%** per frequentiekanaal.
- Bij 1% duty cycle en een transmissietijd van 1,6 seconden (SF12, 50 bytes):
  - Verplichte wachttijd: 1,6 s / 0,01 = **160 seconden** tussen opeenvolgende transmissies.
  - Maximale throughput: 50 bytes / 160 s ≈ **2,5 bytes/seconde = 20 bps effectief**.

Dit is een absolute regulatoire limiet, niet een technische. LoRa is ontworpen voor sporadische sensordata, niet voor continue datastromen.

**2. Botsingskans**

Transmissietijd bij SF12 voor 50 bytes: ±1,6 seconden. Bij SF7 voor 50 bytes: ±50 ms.

Het "collision window" is het tijdsinterval waarbinnen twee transmissies elkaar kunnen storen. Bij ALOHA:

```
P(collision voor packet X) = 1 - e^(-2λ × T_airtime)
waarbij λ = aankomstrate packets per seconde
```

Bij T_airtime = 1,6 s en λ = 1 packet/minuut per node, 100 nodes:
- Totale λ = 100/60 ≈ 1,67 packets/s
- P(collision) = 1 - e^(-2 × 1,67 × 1,6) ≈ 1 - e^(-5,3) ≈ **99.5%** collisions!

Dit toont dat LoRa bij hoge node-densiteit fundamenteel schaalprobleem heeft met ALOHA. Oplossing: meerdere kanalen (LoRaWAN biedt 8 kanalen), frequentiespringen, en TDMA-scheduling.

**3. Payload Size Limiet**

LoRa's maximum payload:
- SF12 (max bereik): maximaal **51 bytes** payload (door duty cycle + coding overhead)
- SF7 (max throughput): maximaal **222 bytes** payload
- Praktisch: typisch 10–50 bytes per bericht

Dit past bij IoT-toepassingen (temperatuur: 2 bytes, GPS: 9 bytes) maar sluit bulkdata-overdracht volledig uit.

**4. Hidden Terminal op grote schaal**

Bij een bereik van 20 km: twee nodes die 35+ km van elkaar liggen, kunnen dezelfde gateway horen maar elkaar niet. CSMA werkt niet: carrier sense is lokaal en onbetrouwbaar over zulke afstanden. De standaard LoRaWAN-oplossing is een **sternetwerk** (star topology) met centrale gateway → ALOHA-achtige toegang → gateway coördineert timing via downlink (Class B beacons).

**5. Best passende Sliding Window Protocol**

Wiskundige analyse (zie ook sectie Transport Layer 2):
- Propagatievertraging: 67 μs (één richting)
- Transmissietijd SF12, 50 bytes: 1600 ms
- a = 67 μs / 1600 ms ≈ 0,000042 → **a ≈ 0**
- Stop-and-Wait efficiëntie: 1/(1+2×0,000042) ≈ **99,99%**

Stop-and-Wait is praktisch optimaal voor LoRa vanwege de enorme transmissietijd versus de verwaarloosbare propagatievertraging.

**Aanbeveling per scenario:**

| Scenario | Protocol | Motivatie |
|---|---|---|
| Enkele node, lichte belasting | Stop-and-Wait (ALOHA) | Eenvoud, efficiëntie door lage a |
| Dense netwerk (>50 nodes/gateway) | TDMA via LoRaWAN Class B | Vermijdt collisions, duty cycle-aware scheduling |
| Mobile nodes | ALOHA (best effort) | Geen synchronisatie mogelijk bij bewegende nodes |
| Kritieke applicaties | Confirmed messages (Class A ACK) | ACK per packet, hertransmissie bij verlies |
