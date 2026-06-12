# Spiekbriefje: Datalinklaag (Laag 2) & Physical Layer (Laag 1)

De **physical layer** zorgt dat bits effectief over een medium (koper, fiber, radio) reizen. De **datalinklaag** bouwt daarop verder: ze bakent een bitstroom af in **frames**, detecteert/corrigeert fouten, en regelt **wie wanneer mag zenden** op een gedeeld medium (MAC).

## Framing: waar begint/eindigt een frame?

- **Byte count**: lengteveld vooraan zegt hoeveel bytes volgen. Compact, maar **zwak punt**: als dat lengteveld corrupt raakt, verlies je de synchronisatie volledig — ook checksum redt je dan niet meer (je weet niet meer waarop hij slaat).
- **Flag bytes**: speciale bytewaarde markeert start/einde. Probleem: die waarde kan toevallig in de data voorkomen.
  - **Byte stuffing**: voeg een escape-byte toe vóór elke flag-byte (en escape-byte) die in de data voorkomt.
  - **Bit stuffing** (efficiënter): bij flagpatroon `01111110` voegt de zender na **5 opeenvolgende 1-bits** automatisch een `0` toe; ontvanger verwijdert die `0` weer. Zo kan het vlagpatroon nooit per ongeluk in de payload ontstaan.

## Physical layer: media

| Medium | Kernpunt |
|---|---|
| **Coax** | massieve kern + afscherming, robuust tegen interferentie, maar minder flexibel |
| **Twisted pair** (Cat5/6/7) | twee draden in elkaar gedraaid → magnetische storingen heffen elkaar grotendeels op; hogere categorieën = meer shielding/twists |
| **Optical fiber** | gebruikt **licht** + **total internal reflection** → hoge bandbreedte, lange afstand, weinig EM-interferentie, maar duurder/delicater |
| **Wireless** | gedeeld, onzichtbaar medium; extra uitdagingen: RF-interferentie, blocked paths (metaal/water/vegetatie), beperkte bandbreedte, mobiliteit, energie |

**Frequentie-afweging**: laag = beter bereik (minder path loss), hoog = kleinere antennes maar meer atmosferische verzwakking; plus wettelijke beperkingen.

**Satellieten** (latency hangt af van baanhoogte):
- **GEO**: ~270 ms, klein aantal satellieten, vaste schotels, groot dekkingsgebied.
- **MEO**: ~35–85 ms, meer dynamiek/tracking nodig.
- **LEO** (Iridium, Starlink): laagste latency, maar veel satellieten nodig + handover-complexiteit.

**RFID**: reader zendt energie → passieve tag wordt actief (near field) en antwoordt via **backscatter** (moduleert reflectie van de draaggolf, zendt geen eigen signaal). Actieve tags (met batterij) halen groter bereik maar zijn duurder.

## MAC: het channel allocation problem

Eén gedeeld medium, wie mag wanneer zenden?
- **Statische allocatie** (vaste draden/slots/frequenties, bv. FM-radio): eenvoudig maar **verspilling** bij bursty verkeer.
- **Dynamische allocatie**: 5 kernvragen — is verkeer onafhankelijk? één of meerdere kanalen? zijn collisions observeerbaar? continu of slots? carrier sense mogelijk?

### Random access protocollen

- **Pure ALOHA**: zend wanneer je wilt; bij collision → **random** wachttijd (randomisatie breekt symmetrie). Simpel maar inefficiënt.
- **Slotted ALOHA**: zenden mag enkel op slotgrenzen → minder/strakker begrensde collisions, maar vereist **synchronisatie**.
- **CSMA**: luister eerst (carrier sense) voor je zendt. Lost collisions niet volledig op door propagatiedelay en **hidden terminals**.

| Variant | Gedrag als kanaal vrij is |
|---|---|
| **Non-persistent** | zend; als bezet → wacht random tijd, probeer later opnieuw |
| **1-persistent** | wacht tot vrij, zend dan meteen (kans 1) — agressief, meer kans op gelijktijdige zenders |
| **p-persistent** | zend met kans p, anders wacht en probeer opnieuw — fijnere regeling |

**Hidden terminal**: A en C horen elkaar niet, botsen toch bij gemeenschappelijke ontvanger B — carrier sense lost dit niet op (lokaal luisteren ≠ kanaal écht vrij).
**Exposed terminal**: C hoort B zenden en wacht onnodig, terwijl C→D niemand zou storen.

### Energiezuinige draadloze MAC

- **BMAC (Low Power Listening)**: radio staat meestal uit, sample af en toe kort. Zender stuurt **lange preamble** zodat een periodiek samplende ontvanger gegarandeerd activiteit detecteert. Voorwaarde: `Preamble_length ≥ Sample_Interval`. Asynchroon, gedecentraliseerd → goed schaalbaar en geschikt voor **onvoorspelbaar/bursty** verkeer, maar bij veel verkeer kost elke preamble extra energie.
- **TSMP (TDMA)**: sterk **gesynchroniseerd**, centrale manager wijst tijdslots toe → bijna geen collisions/retransmissies, radio kan grotendeels uit. Gebruikt ook **frequency hopping** (spreidt interferentierisico) en redundantie (**spatial**: andere buur, **temporal**: ander tijdstip). Goed voor **voorspelbaar** verkeer en QoS (bv. meer slots voor camera dan voor temperatuursensor). Grootste energiekost: het **(re)joinen** van het netwerk. Beperkingen: manager = bottleneck, slechter bij sporadisch verkeer (hogere latency).

## Ethernet (IEEE 802.3)

- Classic Ethernet = **1-persistent CSMA/CD** (Collision Detection) — past goed bij bedraad medium waar je verstoring kan detecteren tijdens zenden.
- **Binary exponential backoff**: na collision groeit het bereik waaruit je je random wachttijd kiest → weinig collisions = korte wachttijd, veel collisions = systeem wordt vanzelf voorzichtiger.
- **Minimum framegrootte (64 bytes = 512 bits)** is gekoppeld aan collision detection: de zender moet nog zenden wanneer het collision-signaal van het verste punt terugkomt. Vereiste: `T_transmissie ≥ 2×T_propagatie`. Bij 10 Mbps geeft dit max ~2560 m kabellengte.
- **Frame format**: preamble (klok-synchronisatie), start-of-frame, source/destination **MAC-adres (48-bit)**, type-of-length veld, broadcast-adres = alle bits 1.
- **DIX vs 802.3**: waarde `≤ 1500` = lengteveld, waarde `> 1500` = typeveld (zegt aan welk hoger protocol — IPv4, ARP — het frame door te geven).
- **Hub** = simpele repeater, blijft één gedeeld medium/collision domain. **Switch** = kijkt naar bestemmingsadres en stuurt enkel naar juiste poort → eigen collision domain per poort, meer capaciteit/privacy.
- **Full-duplex switched**: zenden/ontvangen op apart paar (UTP) → collisions fysiek onmogelijk, CSMA/CD uit. Enige resterende limiet = **signaalattenuatie** (~100 m Cat5e), niet meer timing.
- **Fast/Gigabit/10G Ethernet**: snelheid omhoog → CSMA/CD-timing wordt steeds lastiger (kortere kabels nodig); Gigabit gebruikt **carrier extension** (korte frames kunstmatig verlengen) en **frame bursting**. 10G = volledig full-duplex, geen hubs.
- **Promiscuous mode**: host houdt frames bij die niet voor hem bedoeld zijn → sniffing mogelijk op gedeeld segment.

## 802.11 (wifi): CSMA/CA

Wifi kan tijdens zenden niet betrouwbaar luisteren (eigen signaal overheerst) → **Collision Avoidance** i.p.v. Detection.

| | CSMA/CD (Ethernet) | CSMA/CA (802.11) |
|---|---|---|
| Detecteert collision | Tijdens zenden, op de kabel | Niet betrouwbaar mogelijk |
| Strategie | Detecteer & stop | Vermijd vooraf + bevestig achteraf (ACK) |
| Start na vrij kanaal | Meteen (1-persistent) | Eerst kleine **random backoff** (ruimte voor ACK's) |
| Extra mechanismen | Binary exponential backoff | NAV, RTS/CTS, backoff |

- **ACK**: zender stuurt frame, ontvanger bevestigt; blijft ACK uit → zender vermoedt verlies/collision.
- **NAV (Network Allocation Vector)** = virtuele carrier sense: het **Duration**-veld in de header zegt hoe lang het kanaal nog bezet is (data+ACK); andere nodes zwijgen tot NAV = 0, zelfs als ze het eigenlijke signaal niet horen.
- **RTS/CTS**: lost **hidden terminal** op — A stuurt RTS naar B, B stuurt CTS terug; C hoort B's CTS (ook al hoorde C A's RTS niet) en stelt zijn NAV in. Lost **exposed terminal niet** op (C wacht soms onnodig, wordt zelfs iets erger). Heeft overhead (2 extra frames) → enkel zinvol bij **grote frames** (RTS-threshold, default 2347 bytes = vaak uit).
- **Power saving**: (1) beacon-based — AP meldt periodiek of er buffered data is, client slaapt tussendoor; (2) client-triggered — AP bewaart verkeer tot client zelf actief wordt.
- **Frame**: uitgebreidere header dan Ethernet — frame control (type/subtype, **to DS/from DS**, more fragments, retry, power management...), **fragmentatie** (kleinere stukken bij hoge foutkans), **sequence number** (duplicaten/retries), checksum.
- **Service-types**: Ethernet = **unacknowledged connectionless**; 802.11 = **acknowledged connectionless**.

## Belangrijke formules/getallen

- Minimum Ethernet frame: **512 bits**, vereiste `T_transmissie ≥ 2×T_propagatie`.
- BMAC: `Preamble_length ≥ Sample_interval`; `E_tx ∝ T_preamble + T_data`.
- Bit stuffing flag: `01111110`, stuf na 5 enen.
- DIX/802.3 grens: **1500** (≤ = lengte, > = type).
- MAC-adres: **48 bit**, broadcast = alle bits 1.

## Veelgemaakte examenvalkuilen / belangrijk om te onthouden

- **Carrier sense lost hidden/exposed terminal niet op** — dit is een fundamenteel, herhaald examenpunt: lokaal "ik hoor niets" ≠ "kanaal is vrij".
- **RTS/CTS lost hidden terminal op, maar exposed terminal niet** (maakt het zelfs iets erger) — en is enkel voordelig bij grote frames door de overhead.
- **Hubs vs switches**: hub = geen nieuw collision domain, limiet wordt bij hogere snelheid strenger (Fast Ethernet+hub: ~100-200m). Switch + full-duplex: collisions fysiek onmogelijk, CSMA/CD uit, enige limiet = attenuatie (~100m).
- **Packet loss op wifi ≠ congestie**: TCP (Tahoe/Reno) interpreteert verlies als congestiesignaal en halveert het congestion window, maar op 802.11 kan verlies komen van interferentie, hidden terminals, fading of uitgeputte MAC-retransmissies (tot 7x) — leidt tot onnodige throughput-daling. Oplossingen: ECN, TCP Westwood, QUIC, link-layer ARQ.
- **BMAC vs TSMP**: BMAC = onvoorspelbaar/bursty/event-driven verkeer, geen synchronisatie nodig, schaalbaar. TSMP = voorspelbaar/periodiek verkeer, vereist synchronisatie + centrale manager, beste energie-efficiëntie maar grootste energiekost zit in (re)joinen.
- **Byte count framing**: kwetsbaar omdat een fout in het lengteveld de hele synchronisatie doet ontsporen — een checksum redt dit niet, want je weet niet meer waarop hij slaat.
