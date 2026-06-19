# Netwerklaag (Laag 3) - Korte samenvatting

De **network layer** zorgt voor **end-to-end delivery**: een packet van bron naar bestemming krijgen over meerdere routers heen, ongeacht onderliggende technologie. Twee kerntaken staan hierbij centraal: **routing** (het pad bepalen) en **forwarding** (het packet effectief doorsturen op basis van de routingtabel), plus alles rond **IP-adressering** die dat mogelijk maakt.

## Connectionless vs connection-oriented

Routers werken volgens **store-and-forward**: een packet wordt volledig ontvangen, gecontroleerd en dan pas doorgestuurd.

- **Connectionless (datagram)**: elk packet draagt het volledige eindadres en wordt apart gerouteerd. Verschillende packets van dezelfde flow kunnen via verschillende routes lopen. **IP is hét voorbeeld.**
- **Connection-oriented (virtual circuit)**: bij setup wordt een circuit afgesproken; packets dragen daarna enkel een **circuit-/labelidentifier** (lokaal geldig, kan per hop veranderen = **label switching**). **MPLS** is het voorbeeld.

| | Virtual circuits | Datagrams |
|---|---|---|
| Voordeel | eenvoudigere routing, kleinere tabellen, QoS/resource-reservatie mogelijk | geen setup, sneller starten, robuuster bij routercrash |
| Nadeel | vastgelegd pad, setup-overhead | geen garanties op volgorde/pad |

## Routing-algoritmes

Een goed algoritme is **correct, simpel, robuust, stabiel, fair en efficiënt**. Het **optimality principle**: als J op het optimale pad van I naar K ligt, dan ligt het optimale pad van J naar K ook op dat pad. Alle optimale paden naar één bestemming vormen samen een **sink tree**.

**Dijkstra**: vanaf bronnode telkens de node met de laagste voorlopige kost permanent maken, dan de buren herberekenen. Basis van link-state routing.

| | Distance vector (Bellman-Ford) | Link state (bv. OSPF) |
|---|---|---|
| Wat wordt gedeeld? | eigen beste schattingen naar buren | eigen linkkosten naar buren (link-state packets), via **flooding** |
| Berekening | `kost via buur = kost naar buur + schatting buur->bestemming` | elke router reconstrueert de hele graaf en draait zelf Dijkstra |
| Grootste probleem | **count-to-infinity**: bij wegvallen van een route ontstaat een lus in de redenering, afstand stijgt traag (3,4,5,6...) tot "infinity" omdat routers enkel lokale info hebben | meer rekenkost, opslag en control traffic |
| Convergentie | slecht nieuws traag, goed nieuws snel | sneller en correcter |

**Link-state in 5 stappen**: (1) buren ontdekken via **HELLO**, (2) linkkosten bepalen (bandbreedte/delay), (3) link-state packet maken (zender, **sequence number**, **age**, buren+kosten), (4) flooding (naar alle links behalve inkomende), (5) Dijkstra draaien. Flooding wordt beperkt via **TTL/hopcounter**, **sequence numbers** (duplicaten weggooien) en **age** (oude info negeren).

**Hiërarchische routing**: regio's groeperen zodat routers enkel toegangspunten van andere regio's kennen i.p.v. alle details. Geeft kleinere tabellen en betere schaalbaarheid, ten koste van soms niet-optimale paden.

**Delivery models**: **unicast** (1-naar-1), **broadcast** (naar alle), **multicast** (naar groep). Beste broadcast qua aantal packets = **spanning tree** (geen lussen, dekt alle routers), maar vereist globale kennis.

## Congestion control, QoS en internetworking

Congestie wordt aangepakt via een combinatie: vermijden (traffic-aware routing - werkt slecht door **oscillatie**), weigeren (**admission control**), signaleren en uiteindelijk **droppen**. Meer buffers lost het niet op (te lange wachttijden = instabiliteit).

Drie throttling-signalen/mechanismen:

| Mechanisme | Werking | Snelheid | Overhead |
|---|---|---|---|
| **Choke packets** | router stuurt apart controlpacket naar zender | traag (volledige RTT) | hoog |
| **ECN** | router markeert gewoon packet, ontvanger meldt het terug aan zender | iets trager dan choke (+1 RTT) | laag |
| **RED** | router dropt vroeg en willekeurig bij stijgende queue, treft sneldere zenders relatief vaker | snel | extra hertransmissies |

Als load-signaal is **queuing delay** een vroeger/beter signaal dan **packet loss** (dat komt te laat).

**QoS = 3 bouwstenen**: traffic shaping, packet scheduling, admission control.
- **Leaky bucket**: emmer met capaciteit **B** (burst-tolerantie) en afvoersnelheid **R** (lange-termijn rate). B=0 -> volledig vlakke output.
- **Scheduling**: FIFO (simpel, **tail drop**, niet fair) -> **Fair Queuing** (elke flow eigen queue, beurtelings - maar groot-packettrucje kan dit omzeilen) -> **Weighted Fair Queuing** (gewichten per flow, bv. realtime > achtergrond).

**Internetworking**: netwerken verschillen in adressering, MTU, routing, QoS, security. Twee aanpakken: **translation** (direct omzetten) vs **indirection** (gemeenschappelijke tussenlaag - dit is **IP**). **Switches = laag 2 (frames)**, **routers = laag 3 (packets)**. **Tunneling** = packet van het ene type inkapselen in een ander netwerktype (bv. IPv6-over-IPv4), telt als 1 hop voor hogere lagen.

**Fragmentatie**: nodig door verschillende **MTU's**. Transparant (routers fragmenteren én reassembleren - extra routerwerk) vs niet-transparant (hosts moeten reassembleren). Beter: **Path MTU Discovery** - zender stuurt met **DF-bit**, te grote packets worden gedropt + ICMP-foutmelding teruggestuurd, zender verlaagt packetgrootte. PMTUD en gewone fragmentatie zijn **incompatibel** (fragmentatie verbergt net het MTU-probleem).

## OSPF, BGP, mobility en ad-hoc

- **AS (Autonomous System)**: netwerk onder beheer van één organisatie, met **ASN**.
- **Intra-domain** (binnen een AS) -> **interior gateway protocol** = **OSPF**.
- **Inter-domain** (tussen AS'en) -> **exterior gateway protocol** = **BGP**.

**OSPF**: link-state protocol (HELLO, flooding, Dijkstra), ondersteunt hiërarchie via **areas** (**Area 0 = backbone**, stub areas hangen erop aan) en **ECMP** (load balancing over paden met gelijke kost). OSPF-berichten reizen als gewone IP-packets.

**BGP**: distance-vector-achtig maar werkt op **AS-paths** i.p.v. afstanden, en is gedreven door **policy** (welke routes accepteren/aankondigen), niet puur kortste pad. Belangrijke begrippen:
- **Peering**: AS'en wisselen verkeer uit (vaak via **IXP**), gratis, en **niet transitief** (A-B en B-C peering geeft A geen gratis transit naar C).
- **Transit**: betalen om verkeer verder gedragen te krijgen.
- **iBGP**: boundary routers binnen een AS delen externe routes; intern bereikbaarheid via OSPF. Vuistregel: OSPF kiest de weg *binnen* het AS, BGP kiest *via welk AS* je naar buiten gaat.

**Mobility**: mobiele host houdt een vast **home address**; op nieuwe locatie krijgt hij een **care-of address**, gemeld aan zijn **home agent**, die verkeer dan tunnelt. Dit geeft **triangle routing** (remote host -> home agent -> mobile host) en een **securityrisico** (vals care-of address registreren = verkeer omleiden).

**Ad-hoc netwerken**: geen vaste infrastructuur, nodes = host + router tegelijk, **resource-/energy-constrained en unreliable**. **AODV** is **reactief/on-demand**: floodt een **ROUTE_REQUEST** (met sequence number + TTL, TTL stijgt stapsgewijs om floods te beperken) tot een **ROUTE_REPLY** terugkomt langs het reverse path. AODV pakt count-to-infinity aan via **destination sequence numbers**: hogere sequence = verser, oudere routes worden verworpen. Lange time-outs blijven wel een probleem.

## IP, adressering, NAT, ICMP en ARP

**IPv4-header** (variabele lengte, vandaar **IHL**): belangrijkste velden zijn **TTL** (verlaagd per hop, voorkomt eindeloze lussen), **Protocol** (TCP/UDP/ICMP), **header checksum** (per hop herberekend omdat TTL wijzigt), en **Identification + MF + Fragment Offset** (samen verantwoordelijk voor reassemblage van fragmenten). DF-bit = Don't Fragment (zie PMTUD). Max. packetgrootte = 64 KB.

**Adressering**: 32-bit IPv4-adres = network prefix + host part, genoteerd in **CIDR** (bv. `/24`) of subnetmask (bv. `255.255.255.0`). Routers werken met **prefixen**, niet losse hosts -> bij overlap wint de **longest prefix match**. Klassen (A/B/C) waren te grofkorrelig ("Three Bears Problem") en zijn vervangen door **CIDR** (flexibele prefixlengtes, supernetting).

**DHCP** (applicatielaag) deelt adressen automatisch uit via **leases** (kort = efficiënter, lang = minder serverbelasting). **Private ranges**: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`.

**NAT**: vertaalt private adressen naar één publiek adres, gebruikt **poorten** als index in de vertaaltabel. Problemen: breekt **end-to-end model**, is impliciet connection-oriented, werkt slecht met protocollen die IP-adressen in de payload meesturen (FTP, SIP, IPsec). **Traversal**: STUN (publiek adres ontdekken), TURN (relay), UPnP (automatische port mapping), connection reversal, ALG (application-level gateway).

| | IPv4 | IPv6 |
|---|---|---|
| Adreslengte | 32 bit | 128 bit |
| Header | variabel (min. 20 bytes, IHL-veld, options) | vast **40 bytes**, extra's in extension headers |
| Checksum | header checksum, per hop herberekend | **geen** header checksum |
| Fragmentatie | door routers mogelijk | routers fragmenteren **niet** -> PMTUD verplicht; min. MTU **1280 bytes** |
| Adrestypes | unicast/broadcast/multicast | **unicast/multicast/anycast** (geen broadcast meer) |
| Config | DHCP | SLAAC (stateless autoconfiguration) |

IPv6-notatie: hex groepen met `:`, leidende nullen weglaten, en **`::` mag maar één keer** per adres gebruikt worden. Belangrijke ranges: **global unicast** (`2000::/3`, routable), **unique local** (`fc00::/7`, privénetwerken), **link local** (`fe80::/10`, niet gerouteerd, verplicht per interface).

### IPv6 SLAAC en privacy

**SLAAC** (Stateless Address Autoconfiguration): host genereert zelf een IPv6-adres uit netwerkprefix (via Router Advertisement) + **EUI-64** interface identifier (afgeleid van MAC-adres: MAC splitsen, `ff:fe` invoegen, 7e bit flippen).

**Privacyprobleem**: MAC-adres is permanent → EUI-64-adres is overal hetzelfde → gebruiker traceerbaar over netwerken + fabrikant (OUI) zichtbaar.

**Tegenmaatregelen**: RFC 4941 (random tijdelijke adressen), RFC 7217 (stabiel maar niet-afleidbaar per netwerk), MAC-randomisatie.

**SLAAC vs DHCPv6**: SLAAC = stateless, geen server nodig, schaalt beter. DHCPv6 = stateful, meer controle (DNS, logging), maar server = single point of failure.

### Subnetting: methode

1. Prefix → subnetmask (`/25` = `255.255.255.128`)
2. Netwerkadres = IP AND subnetmask
3. Broadcastadres = netwerkadres OR inverse mask (alle hostbits = 1)
4. Eerste host = netwerkadres + 1, laatste host = broadcast - 1
5. Aantal hosts = 2^(32-prefix) - 2

### IoT en IP stack (6LoWPAN)

IPv6 voor IoT (802.15.4): **6LoWPAN** = adaptielaag die IPv6 aanpast aan beperkte netwerken (headercompressie 40→paar bytes, fragmentatie bij kleine MTU). IPv4 ongeschikt: te weinig adressen, geen SLAAC. Voordeel IP op IoT: end-to-end bereikbaarheid, bestaande tools. Nadeel: overhead voor constrained devices.

**ICMP** = foutmeldings-/diagnosekanaal van IP: **destination unreachable** (o.a. Type 3/Code 4 voor PMTUD), **time exceeded** (TTL=0), **redirect**, **echo request/reply** (ping). **Traceroute** gebruikt stijgende TTL (1,2,3,...) zodat elke router op het pad een "time exceeded" terugstuurt.

**ARP**: zet IP-adres om naar MAC-adres **binnen hetzelfde subnet** via cache -> broadcast request -> unicast reply. Voor hosts buiten het subnet wordt enkel het MAC-adres van de **default gateway** gecached, niet van de eindbestemming.

## Veelgemaakte examenvalkuilen / belangrijk om te onthouden

- **PMTUD en fragmentatie zijn incompatibel**: DF=1 + te groot packet -> router dropt en stuurt ICMP Type 3/Code 4 terug (geen stille fragmentatie). Een firewall die ICMP blokkeert geeft een **PMTUD black hole**.
- **Count-to-infinity** is een fundamenteel probleem van distance-vector/Bellman-Ford door enkel lokale kennis; **AODV** lost dit gedeeltelijk op met **destination sequence numbers** (hoger = verser), maar lange time-outs blijven lastig.
- **Peering is niet transitief** - en BGP draait om **AS-paths + policy**, niet om de kortste technische route (dat is het domein van OSPF/intra-domain).
- Bij congestienotificatie: **choke packets** (traag, veel overhead) vs **ECN** (snel, weinig overhead, markeert bestaand packet) vs **RED** (vroege willekeurige drops, treft snelle zenders harder) - ken de trade-offs.
- **ARP werkt enkel binnen het subnet**: een host buiten je subnet bereik je via je default gateway, en je ARP-cache bevat dan ook enkel een entry voor die gateway, niet voor de eindbestemming.
- Reken-/notatieklassiekers: CIDR-notatie en subnetmask correct kunnen omzetten (bv. `/25` = `255.255.255.128`), **longest prefix match** bij overlappende routes, en IPv6 `::`-verkorting mag **maar één keer**.
- **IPv6 SLAAC + EUI-64 = privacyprobleem**: MAC-adres in het IPv6-adres maakt tracking over netwerken mogelijk. Herkenbaar aan `ff:fe` in het midden van het interface-ID.
- **NAT-traversal**: inkomende verbindingen geblokkeerd → STUN/TURN/UPnP/connection reversal nodig. FTP, SIP en P2P breken achter NAT.
- **6LoWPAN**: adaptielaag voor IPv6 op 802.15.4 (IoT) — headercompressie + fragmentatie.
