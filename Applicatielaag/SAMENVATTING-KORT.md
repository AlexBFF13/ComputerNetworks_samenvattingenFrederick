# Applicatielaag (Laag 7) — Spiekbriefje

De applicatielaag is de bovenste laag: hier worden ruwe TCP/UDP-verbindingen vertaald naar concrete diensten (web, mail, bestanden, P2P). Elk protocol kiest zijn eigen stijl van commando's, sessiebeheer en (on)veiligheid, maar leunt voor transport vrijwel altijd op **TCP** (betrouwbaar) of **UDP** (lichtgewicht, geen setup nodig).

## OSI / TCP-IP / Tanenbaum stacks

| Laag | OSI | TCP/IP | Tanenbaum |
|---|---|---|---|
| 7 | Application | Application | Application |
| 6 | Presentation | ↑ | ↑ |
| 5 | Session | ↑ | ↑ |
| 4 | Transport | Transport | Transport |
| 3 | Network | Internet | Network |
| 2 | Data Link | Network Access | Data Link |
| 1 | Physical | ↑ | Physical |

- **Lastige gevallen**: DHCP = applicatielaag (UDP broadcast); RTP = applicatielaag (bovenop UDP); OSPF = netwerklaag (direct over IP, geen TCP/UDP).
- OSI = referentiemodel (7 lagen, nooit volledig geïmplementeerd), TCP/IP = praktisch model (4 lagen, succesvol gedeployed).

## UDP-applicaties: waarom geen TCP?

| App | Waarom UDP |
|---|---|
| **DNS** | Kleine queries, stateless, anycast-compatibel; TCP fallback bij grote antwoorden |
| **DHCP** | Client heeft nog geen IP → TCP handshake onmogelijk; broadcast nodig |
| **Wake-on-LAN** | Doelhost is uit → kan geen TCP-verbinding opzetten; magic packet = broadcast |

Rode draad: TCP vereist een werkende verbinding aan beide kanten — dat is precies wat bij deze apps nog niet bestaat.

## Remote login: Telnet vs SSH

- **Telnet** (poort 23, TCP): stuurt commando's en tekst in **plaintext**, geen serverauthenticatie. Fases: connection management → optional negotiation → control → data transfer. Onveilig, maar nog steeds bruikbaar als **debugtool** voor tekstuele protocollen boven TCP (SMTP, HTTP).
- **SSH**: veilige opvolger, drie lagen:
  - **Transport layer**: key exchange + serverauthenticatie (veilige tunnel)
  - **Authentication layer**: gebruikersauthenticatie (wachtwoord of public key)
  - **Connection layer**: diensten erbovenop (terminal, file transfer, TCP forwarding)
  - Bij **public-key authenticatie** bewijst de client dat hij de private key heeft, zonder die te versturen.

## E-mail: architectuur en protocollen

E-mail is een **keten**: sender user agent → mail submission → message transfer agent → SMTP tussen mailservers → message transfer agent → final delivery → receiver user agent. Berichten volgen het **Internet Message Format** (headers: `To`, `Cc`, `Bcc`, `From`, `Received`, `Return-Path`, ...).

| | POP | IMAP | Webmail |
|---|---|---|---|
| Werking | Haalt mail lokaal op, weinig sync | Houdt mappen (inbox/drafts/...) gesynchroniseerd | Browser = client, alles op server |
| Voordeel | Eenvoudig | Goed voor meerdere toestellen | Lichtgewicht client, overal bruikbaar |
| Nadeel | Moeilijk over meerdere toestellen | Meer overhead/complexiteit | Vereist internetverbinding |

**SMTP** = command/response **relay-protocol tussen mailservers** (niet voor mailbox-sync!). Drie fasen: handshaking → transfer → closure. Flow: `HELO` → `MAIL FROM` → `RCPT TO` → `DATA` → bericht → `.` → `QUIT`. Omdat het tekstueel is, kan je het testen via Telnet.

## Het web: URI, HTML, HTTP

- **URI** = identifier voor een resource (kan `mailto:`, `ftp://`, `http://`, ...). Een **URL** is de bekendste vorm, combineert protocol + server + resource.
- Webgebruikers gebruiken namen → **DNS** vertaalt naar IP's (zie verder).
- **HTML** structureert tekst en geeft betekenis/opmaak via tags + links.
- **HTTP** = request/response protocol, op TCP, traditioneel poort 80, **stateless** (geen ingebouwd sessiegeheugen — context via cookies/sessies/app-logica). Methodes: `GET` (document opvragen), `HEAD` (alleen headers), `POST` (data sturen), plus `PUT`, `OPTIONS`, `DELETE`. Ook HTTP/1.x kan je testen via Telnet (leesbare requests).

### HTTP-versies vergeleken

| Versie | Kenmerk | Probleem/voordeel |
|---|---|---|
| **HTTP/1.0** | Eén TCP-verbinding per object | Inefficiënt: extra round trips, telkens nieuwe TCP-opstart + congestion control van klein begin |
| **HTTP/1.1** | **Persistent connections** + **pipelining** | Hergebruik van verbinding verlaagt overhead; pipelining stuurt meerdere requests zonder te wachten op elk antwoord (werkt minder goed bij afhankelijkheden tussen resources) |
| **HTTP/2** | Binair formaat, **framing**, **multiplexing**, prioritisatie — binnen één TCP-verbinding (kwam uit SPDY) | Lost het "veel aparte objecten ophalen"-probleem structureel op |

## FTP: gescheiden control en data

FTP gebruikt **twee TCP-poorten**: poort 21 voor **control** (Telnet-achtig commandokanaal, NVT), poort 20 voor **data**. Dit splitst sessiebeheer (persistent) van datatransfer (per overdracht), in tegenstelling tot HTTP waar control+data samen lopen. Commando's: `USER`, `PASS`, `CWD`, `PASV`, `RETR`, `STOR`, `LIST`, `QUIT`.

## DHCP: automatische netwerkconfiguratie

**DHCP** (RFC 2131, **UDP**) geeft hosts automatisch IP-adres, subnetmask, gateway en DNS-server als **lease** (tijdelijk, geen permanent eigendom). Kernflow:

`DHCPDISCOVER` (client, broadcast vanaf `0.0.0.0`) → `DHCPOFFER` (server) → `DHCPREQUEST` (client, broadcast) → `DHCPACK` (server)

- Het **berichttype** zit in een **option**, niet als vast headerveld.
- **Transaction ID** koppelt antwoorden aan requests.
- Lease wordt vernieuwd bij ~50% verstreken tijd; bij weigering volgt `DHCPNAK`.
- `DHCPRELEASE` = adres vrijwillig teruggeven.
- Na toewijzing: **Duplicate Address Detection via ARP** — bij conflict stuurt de client `DHCPDECLINE`.
- **Waarom geen TCP?** TCP vereist een 3-way handshake met geldig bron- én bestemmings-IP; tijdens DISCOVER heeft de client geen van beide. UDP laat broadcast vanaf `0.0.0.0` naar `255.255.255.255` toe zonder voorafgaande state.

## DNS: namen naar adressen

Vroeger één centraal `hosts.txt`-bestand — schaalde niet. DNS is een **hiërarchische, gedistribueerde** oplossing.

- Namen zijn **niet case-sensitive** (URL-paden kunnen dat wel zijn), en eindigen technisch op een punt (`www.google.com.`).
- **Hiërarchie** van rechts naar links: `nix.cs.kuleuven.ac.be` → land (`be`) → academisch domein (`ac`) → organisatie (`kuleuven`) → afdeling (`cs`) → host (`nix`). Vergelijk met een postadres: breed → specifiek.
- **TLD's**: landcodes (`.be`), generiek (`.com`, `.org`), VS-specifiek (`.edu`, `.gov`, `.mil`). Beheerd door **ICANN** (sinds 1998), gedelegeerd naar **registrars** (bv. Verisign voor `.com`, DNS Belgium voor `.be`).
- **Resource records**: Domain Name, **TTL**, Class, Type, Value. Belangrijkste types: `A` (IPv4), `NS` (naamserver), `MX` (mailserver), `CNAME` (alias), `PTR` (reverse mapping).
- **Server-hiërarchie**: leaf (concrete records) → branch (mix van records en verwijzingen) → root (kent TLD-servers).
- **Zones** = niet-overlappende stukken van de namespace; meer zones = meer load balancing maar meer beheeroverhead.
- **Authoritative** (van de bevoegde zonebeheerder) vs **cached** (tijdelijke kopie).
- **Recursive** vs **iterative** lookup: een lokale resolver doet recursief werk voor de client, en stapt zelf iteratief van root → TLD → authoritative server. **Root/TLD-servers gebruiken anycast** voor redundantie en performantie.

| | Puur iteratief | Puur recursief | Hybride (praktijk) |
|---|---|---|---|
| Caching | Slecht (geen gedeelde cache) | Goed (lokale resolver cachet voor iedereen) | Goed |
| Load op root | Hoog | Laag | Laag |
| Latency client | Hoog (meerdere round trips) | Laag (1 round trip) | Laag |
| Complexiteit client | Hoog | Laag | Laag |

## Socketprogrammering / client-server vs P2P

Klassieke applicaties (Telnet, SSH, mail, web, FTP, DHCP/DNS) volgen het **client/server-model**: client initieert, server luistert op een bekende poort, met afgesproken protocol. Bij **P2P** leveren randmachines zelf resources (opslag, bandbreedte, CPU). Onderscheid:
- **P2P-applicatie** (application-centric): gebruikt edge-resources, mag toch centrale onderdelen hebben.
- **P2P-netwerk** (network-centric): volledig gedecentraliseerd, symmetrisch, zonder hiërarchie.
Een **overlay-netwerk** is een logisch netwerk boven peers — 1 overlay-hop kan op de echte netwerklaag meerdere hops zijn, dus message passing minimaliseren.

## P2P-systemen

- **Napster**: centrale **indexering**, gedistribueerde bestandsuitwisseling → single point of failure/attack/aanklacht (werd gesloten).
- **Gnutella 0.4**: unstructured decentralized overlay, **flooding** met `PING`/`PONG` (peer discovery) en `QUERY`/`QUERYHIT` (search), TTL beperkt de flood. `PUSH` helpt peers achter een firewall. Bestandstransfer zelf via **HTTP**. Probleem: **search horizon** (te lage TTL = te weinig gevonden, te hoge TTL = netwerk overspoeld).
- **Gnutella 0.6**: **super-node**-architectuur — **ultrapeers** (sterke nodes: discovery, routing, indices van leafs) en **leaf nodes** (verbinden met 1 ultrapeer, minder werk).
  - **Monitoring/spying**: als gewone peer zie je enkel directe buren; als ultrapeer ook leaf-queries. Met **onbeperkte resources** kan een adversary veel ultrapeers draaien en zo groot deel van het zoekverkeer observeren → trade-off schaalbaarheid vs anonimiteit.
- **Chord (DHT)**: gestructureerde oplossing — consistente hashing (SHA-1) plaatst nodes/bestanden op een ring van 0..2^160-1, lookup in **O(log N)** hops via finger tables, gegarandeerd resultaat (i.p.v. TTL-onzekerheid bij Gnutella). Hashfunctie moet **uniform verdeeld** (geen hotspots) en **collision-vrij** zijn; "virtual nodes" helpen balans.
- **BitTorrent**: gericht op **content distribution** (i.p.v. discovery zoals Gnutella of routing zoals Chord). Discovery via web (torrent-bestand) → **tracker** → **swarm**. Bestand in **chunks** met **SHA-1 hash** (integriteit + parallel downloaden). **Rarest first**: zeldzame chunks eerst verspreiden tegen bottlenecks. **Seeder** = peer met alle chunks; gewone peers zijn down- én uploader. **Tit-for-tat**: peers die niet bijdragen worden **choked** (tegen free-riding). Nieuwere versies gebruiken een **DHT (Kademlia)** i.p.v. trackers.

## TOR: privacy en onion routing

**Onion routing**: bericht in meerdere encryptielagen, elke router pelt precies 1 laag af en kent alleen vorige/volgende hop — niet de volledige route, oorsprong, bestemming of inhoud. Client = **Onion Proxy** (SOCKS-interface, ontdekt routers via **directory server**, bouwt circuit). **Onion Routers** sturen verkeer door of zijn **exit node**. **Alle links versleuteld, behalve exit node → gewone internet** (daar is verkeer plaintext als het bv. HTTP is). Vaste **cell-grootte** (control cells vs relay cells) bemoeilijkt traffic analysis. Circuit wordt opgebouwd via `relay extend`; reverse path via **Circuit ID**. TOR beschermt vooral tegen **traffic analysis**, niet tegen een global passive adversary (timing-correlatie blijft mogelijk) — meer hops kan zelfs de kans op een gecompromitteerde node verhogen.

**Wat kent elke node?**

| | Bron-IP | Bestemming | Data | Vorige hop | Volgende hop |
|---|---|---|---|---|---|
| **Entry** | ja | nee | nee | — | ja |
| **Middle** | nee | nee | nee | ja | ja |
| **Exit** | nee | ja | ja (plaintext!) | ja | — |

- **Minimaal 3 nodes**: bij 2 nodes kent één node zowel bron als bestemming → volledige deanonymisatie.
- **Sybil attack**: aanvaller registreert veel nepnodes als TOR-relays → vergroot kans dat aanvaller entry én exit controleert → timing-correlatie.
- **Spying met beperkte resources**: exit nodes loggen (ziet bestemmingen, niet bronnen). Met **onbeperkte resources** (state actor): globale timing-correlatie tussen entry en exit.

---

## Veelgemaakte examenvalkuilen / belangrijk om te onthouden

- **SMTP is geen mailbox-protocol**: het is een command/response **relay**-protocol tussen servers. POP/IMAP zijn voor client ↔ server (IMAP = sync, POP = lokaal ophalen).
- **DHCP moet UDP gebruiken**: TCP vereist een handshake met geldige bron-/bestemmings-IP's, die de client tijdens DISCOVER nog niet heeft. Kernflow `DISCOVER → OFFER → REQUEST → ACK`, en **Duplicate Address Detection gebeurt via ARP** (niet via DHCP zelf).
- **DNS-namen zijn niet case-sensitive**, maar URL-paden kunnen dat wel zijn. Ken de betekenis van `A`, `NS`, `MX`, `CNAME`. DNS-lookup is een **hybride**: recursief vanuit het oogpunt van de client/lokale resolver, **iteratief** hoger in de hiërarchie (root/TLD) — dit hybride model combineert lage client-complexiteit met schaalbare, stateless root-servers.
- **FTP gebruikt 2 poorten** (21 control, 20 data) — en **Active FTP faalt achter NAT** omdat de server een inkomende verbinding naar het privé-IP van de client moet opzetten (NAT heeft daar geen mapping voor). **Passive FTP (PASV)** lost dit op: de client initieert beide verbindingen (alles outbound).
- **P2P-applicatie ≠ P2P-netwerk**: Napster was een P2P-app met centrale indexering, geen volledig P2P-netwerk. Ken het verschil tussen Gnutella 0.4 (flooding, search horizon-probleem), Gnutella 0.6 (ultrapeers/leafs) en Chord (DHT, O(log N), gegarandeerd maar minder anoniem/robuust).
- **TOR**: minstens **1 intermediate node** (3 nodes totaal) is nodig zodat geen enkele node zowel bron als bestemming kent. Enkel de link exit node → internet is onversleuteld. Meer hops = meer latency én potentieel méér kans op een gecompromitteerde node (1 - 0.8^N), zonder bescherming tegen een global passive adversary.
