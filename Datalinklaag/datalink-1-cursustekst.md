# Deel 13 - Data Link 1: Principles and Classic Protocols

In dit deel verschuift de focus naar de **data-linklaag**, en meer specifiek naar **Medium Access Control (MAC)**. Hier gaat het niet meer over routing door een groot netwerk of end-to-end transport, maar over een veel directer probleem:

hoe laat je meerdere nodes één gedeeld medium gebruiken zonder dat alles constant botst?

De grote thema's in deze slides zijn:

- basisprincipes van MAC
- klassieke random access-protocollen zoals Aloha en CSMA
- energiezuinige MAC-protocollen zoals BMAC
- tijdgesynchroniseerde aanpakken zoals TSMP

Dit hoofdstuk is vooral belangrijk omdat het laat zien dat "toegang tot het medium" op een gedeeld kanaal op zich al een moeilijk ontwerpprobleem is, zeker in draadloze en energiegevoelige netwerken.

## Het channel allocation problem

MAC gaat over het delen van **één broadcastmedium**.

Dat medium kan verschillende vormen aannemen:

- een stuk radiospectrum
- een kabel
- een glasvezel

Zodra meerdere nodes hetzelfde medium delen, krijg je automatisch een probleem van **kanaaltoewijzing**: wie mag wanneer zenden?

De slides zeggen dat er twee basisopties zijn:

- **statische allocatie**
- **dynamische allocatie**

## Statische kanaaltoewijzing

Bij **statische allocatie** verdeel je het medium vooraf.

Dat kan op verschillende manieren:

- verschillende draden
- verschillende tijdslots
- verschillende frequenties

Een klassiek voorbeeld is FM-radio: elk station krijgt zijn eigen vaste kanaal.

### waarom is dat vaak verspilling?

Omdat je het medium op voorhand opdeelt, zelfs als veel delen op een bepaald moment niet gebruikt worden.

Dus als je N stukken hebt en maar een paar daarvan actief zijn, dan blijft veel capaciteit ongebruikt.

### belangrijk voor examen

Statische allocatie is eenvoudig, maar typisch inefficiënt wanneer het gebruik van het medium wisselend en bursty is.

## Dynamische toewijzing en de vijf aannames

Wanneer je dynamische toewijzing wil doen, moet je eerst duidelijk maken wat voor soort systeem je hebt.

De slides geven vijf belangrijke vragen:

1. is het verkeer onafhankelijk?
2. is er één gedeeld kanaal of meerdere?
3. zijn collisions observeerbaar?
4. werk je continu of in slots?
5. kan je carrier sense doen of niet?

Dat zijn geen kleine details. Ze bepalen mee welk MAC-protocol logisch is.

## Draadloze uitdagingen

Bij draadloze MAC-protocollen komen er nog extra moeilijkheden bovenop.

### Interferentie en occlusie

Het bruikbare spectrum is beperkt en druk bezet. Interferentie kan bovendien plots en lokaal opduiken.

### Energie

De radio is vaak de grootste energieverbruiker van een node. En belangrijk:

een radio verbruikt energie bij:

- zenden
- maar ook bij luisteren

### Node loss

In veel sensor- of veldtoepassingen kunnen nodes op elk moment verdwijnen:

- door schade
- door lege batterijen
- door ruwe omgevingen

Dat betekent dat een goed MAC-protocol ook robuust moet omgaan met onbetrouwbare aanwezigheid van nodes.

## ALOHA: het klassieke startpunt

De eerste case study is **ALOHA**, historisch gegroeid uit het probleem om de eilanden van Hawaii via radioverbindingen toegang te geven tot een centrale computer.

Die context is belangrijk, want ze toont waarom random access-protocollen ontstonden:

- er was geen klassieke telefooninfrastructuur
- maar men wilde wel meerdere afgelegen sites laten communiceren

## Pure ALOHA

**Pure ALOHA** is extreem eenvoudig:

- een node stuurt wanneer hij wil
- als twee zendingen tegelijk gebeuren, krijg je een collision
- verloren frames worden later opnieuw verzonden

### Waarom een random wachttijd na collision?

Omdat anders alle betrokken nodes meteen tegelijk opnieuw zouden zenden en dus opnieuw zouden botsen.

Dat is een mooi basisidee:

randomisatie breekt symmetrie.

### nadeel

Pure ALOHA is eenvoudig, maar inefficiënt. Omdat nodes eender wanneer mogen zenden, kunnen collisions relatief gemakkelijk optreden.

### Throughput en vulnerable period

De **vulnerable period** bij Pure ALOHA is **2T** (twee keer de frametijd). Dat is de periode waarin een ander frame kan beginnen dat overlapt met jouw frame: zowel frames die net voor jou beginnen als frames die net na jou beginnen kunnen een collision veroorzaken.

De maximale channel utilization van Pure ALOHA is:

```text
S_max = 1/(2e) ≈ 18,4%
```

Dat betekent dat zelfs onder ideale omstandigheden meer dan 80% van de capaciteit verloren gaat aan collisions en idle time.

## Slotted ALOHA

**Slotted ALOHA** is de eerste grote verbetering.

Hier wordt tijd opgedeeld in discrete **slots**. Een node mag alleen aan het begin van een slot beginnen zenden.

Dat helpt omdat collisions dan veel strakker begrensd worden. De **vulnerable period** halveert naar **1T**: een frame kan enkel botsen met andere frames die in **hetzelfde** slot beginnen, niet met frames die deels overlappen vanuit het vorige of volgende interval.

### prijs die je betaalt

Je hebt wel **synchronisatie** nodig.

De maximale throughput verdubbelt ten opzichte van Pure ALOHA:

```text
S_max = 1/e ≈ 36,8%
```

Dus:

- betere efficiëntie (dubbel zo hoog)
- maar extra coördinatie (synchronisatie)

### Kan Pure ALOHA op 802.11?

Dit is een typische examenvraag. Het antwoord is: **fysiek is het mogelijk** (dezelfde radio-hardware), maar het is **een zeer slechte keuze**. 802.11 gebruikt bewust CSMA/CA met collision avoidance, backoff en RTS/CTS. Pure ALOHA's blinde transmissies zouden:

- het gedeelde radiokanaal enorm verspillen
- het hidden terminal probleem volledig negeren
- geen ACK-mechanisme bieden
- bij RREQ flooding (zoals bij AODV) tot massale collisions leiden

Kortom: het kan, maar het is zinloos en veel slechter dan CSMA/CA.

### belangrijk voor examen

- Pure ALOHA: vulnerable period = 2T, max throughput ≈ **18%**
- Slotted ALOHA: vulnerable period = 1T, max throughput ≈ **37%** (verdubbeling)
- Slotted ALOHA vereist synchronisatie
- Pure ALOHA op 802.11 is fysiek mogelijk maar zinloos

## CSMA

De volgende stap is **Carrier Sense Multiple Access (CSMA)**.

Het basisidee is veel intuïtiever:

voordat je zendt, **luister** je eerst of het medium al in gebruik is.

Dat vermindert collisions, want je gaat niet blind beginnen zenden.

Maar de slides zeggen terecht dat je collisions daarmee niet volledig elimineert. Door propagatiedelay en verborgen nodes kan je nog altijd botsingen krijgen.

## Hidden en exposed terminals

De figuur over **hidden** en **exposed terminals** hoort bij de kernintuïtie van draadloze MAC.

### Hidden terminal

Twee nodes horen elkaar niet, maar storen wel op dezelfde ontvanger. Ze denken dus allebei dat het kanaal vrij is en botsen toch.

### Exposed terminal

Een node hoort activiteit en denkt daarom dat hij niet mag zenden, terwijl zijn eigen transmissie eigenlijk geen probleem zou veroorzaken voor de relevante ontvanger.

Dat toont dat "luisteren naar het medium" in draadloze context niet hetzelfde is als perfect weten of zenden veilig is.

### Waarom CSMA deze problemen niet oplost

CSMA luistert enkel **lokaal**. Bij het hidden terminal probleem horen A en C elkaar niet, dus carrier sense geeft hen allebei een "vrij" signaal terwijl ze bij B botsen. CSMA helpt hier dus niet.

Bij het exposed terminal probleem is CSMA zelfs contraproductief: C hoort B zenden en concludeert dat het kanaal bezet is, terwijl C's transmissie naar D niemand zou storen. CSMA veroorzaakt hier onnodige zelfcensuur.

### MACA: Multiple Access with Collision Avoidance

**MACA** is een protocol dat specifiek ontworpen is om het hidden terminal probleem aan te pakken. Het werkt met twee korte controleframes:

1. De zender stuurt een **RTS (Request To Send)** naar de ontvanger
2. De ontvanger antwoordt met een **CTS (Clear To Send)**

Alle nodes die het CTS horen weten dat het kanaal bezet is en stellen hun transmissie uit. Dit lost het hidden terminal probleem op:

```text
A stuurt RTS naar B → B stuurt CTS → C hoort het CTS
→ C weet dat het kanaal bezet is, ook al hoorde C de RTS van A niet
```

MACA lost het **exposed terminal probleem niet op** en kan het zelfs verergeren. C hoort B's RTS en wacht, terwijl C's transmissie naar D eigenlijk geen probleem zou zijn.

802.11 wifi bouwt voort op dit MACA-principe met zijn RTS/CTS-mechanisme gecombineerd met NAV (Network Allocation Vector).

### belangrijk voor examen

- CSMA lost het hidden terminal probleem **niet** op (lokaal luisteren is onvoldoende)
- MACA/RTS/CTS lost het hidden terminal probleem **wel** op
- Geen van beide lost het exposed terminal probleem op
- RTS/CTS heeft overhead (twee extra frames) en is alleen zinvol bij grote dataframes

## Non-persistent CSMA

Bij **non-persistent CSMA**:

- luister je eerst
- als het kanaal vrij is, zend je
- als het bezet is, wacht je een willekeurige tijd en probeer je later opnieuw

Dat helpt om agressieve synchronisatie en herhaalde botsingen te vermijden.

## 1-persistent CSMA

Bij **1-persistent CSMA**:

- luister je eerst
- als het kanaal bezet is, blijf je wachten
- zodra het vrij wordt, zend je meteen met kans 1

Dit is agressiever dan non-persistent CSMA. Het medium wordt snel gebruikt zodra het vrijkomt, maar daardoor neemt ook de kans toe dat meerdere wachtende nodes tegelijk toeslaan.

## p-persistent CSMA

Bij **p-persistent CSMA**:

- luister je eerst
- als het kanaal vrij is, zend je met kans `p`
- met kans `1-p` wacht je nog even en probeer je opnieuw

Dat geeft een fijnere probabilistische regeling van gedrag op een vrij kanaal.

### belangrijk voor examen

Het verschil tussen de CSMA-varianten zit vooral in **hoe agressief** een node reageert zodra het kanaal vrij lijkt.

## BMAC en low power listening

De tweede case study is **Berkeley MAC (BMAC)**. Dit protocol is gemaakt voor draadloze sensornetwerken waar energie heel belangrijk is.

De ontwerpdoelen zijn onder andere:

- laag energieverbruik
- eenvoudige implementatie
- configureerbaarheid
- schaalbaarheid
- decentralisatie

BMAC bouwt voort op een eenvoudige carrier-sense-aanpak, maar focust sterk op het beperken van de kost van **luisteren**.

## Waarom luisteren zo belangrijk is

De slides stellen expliciet de vraag:

brengen sensornodes meer tijd door met luisteren of met zenden?

Het antwoord is: **luisteren**.

En precies daarom is low power listening zo’n sterk idee. In veel sensortoepassingen zijn uitzendingen zeldzaam. Dan is het slimmer om een beetje meer energie te steken in de zeldzame zendingen, als je daardoor continu luisteren kan verminderen.

## Low Power Listening (LPL)

Bij **Low Power Listening** zetten nodes hun radio niet constant aan.

In plaats daarvan:

- zetten ze de radio maar af en toe kort aan om te samplen
- als ze activiteit merken, blijven ze wakker om het bericht echt te ontvangen

Maar daar ontstaat een probleem:

als de ontvanger maar af en toe luistert, hoe weet de zender dan dat hij op het juiste moment zendt?

### De oplossing: een lange preamble

BMAC gebruikt een **lange preamble** voor elk bericht.

Die preamble duurt lang genoeg zodat een ontvanger die periodiek sampelt, de activiteit gegarandeerd detecteert en wakker blijft tegen dat de echte data aankomt.

### De kernrelatie: preamble en sample-interval

De preamble moet minstens zo lang zijn als het sample-interval van de ontvanger:

```text
Preamble_length >= Sample_Interval
```

Als de preamble korter is dan het sample-interval, kan het gebeuren dat de ontvanger net tussen twee samples in zit en de preamble volledig mist.

### Configuratie bij hoge vs lage data rate

Dit is een zeer frequent gevraagd examenonderwerp. De optimale configuratie van het sample-interval (en dus de preamble-lengte) hangt af van het verwachte verkeerspatroon:

| Configuratie | Sample-interval | Preamble | Geschikt voor |
| --- | --- | --- | --- |
| **Hoge data rate** (veel verkeer) | **Kort** | **Kort** | Veel packets, lage per-packet overhead, maar meer energie voor sampling |
| **Lage data rate** (weinig verkeer) | **Lang** | **Lang** | Maximale slaaptijd, dure maar zeldzame zendingen |

**Bij hoge data rate (veel verkeer):**

- Kies een **kort sample-interval** zodat de ontvanger snel reageert
- De preamble is dan ook kort, wat de per-packet overhead laag houdt
- Nadeel: de radio samplet vaker, dus het luisteren kost meer energie
- Maar bij veel verkeer is die extra sample-energie verwaarloosbaar tegenover de besparing op preamble-tijd

**Bij lage data rate (weinig verkeer):**

- Kies een **lang sample-interval** zodat de radio zo lang mogelijk uit kan staan
- De preamble moet dan ook lang zijn, wat elke individuele zending duur maakt
- Maar omdat zendingen zeldzaam zijn, is de totale energiekost toch laag
- Het is slimmer om af en toe veel te betalen voor een zending dan om constant te luisteren

### belangrijk voor examen

Bij BMAC zorgt een **lange preamble** ervoor dat een ontvanger die maar af en toe luistert, toch een inkomend packet kan opvangen. De configuratie van preamble/sample-interval hangt af van het verkeerspatroon: kort interval bij veel verkeer, lang interval bij weinig verkeer.

## Eigenschappen van BMAC

De slides noemen nog een paar praktische eigenschappen:

- clear channel assessment is mogelijk
- link-layer ACKs zijn beschikbaar
- veel parameters zijn configureerbaar
- acknowledgements en CCA zijn optioneel

Dat maakt BMAC flexibel, maar ook een protocol waarbij ontwikkelaars bewuste keuzes moeten maken.

## De kracht en limieten van decentralisatie

BMAC is volledig **gedecentraliseerd**. Dat maakt het goed schaalbaar en geschikt voor onvoorspelbaar verkeer.

Maar de limieten zijn duidelijk:

- zonder synchronisatie moet je altijd energie verspillen aan extra luisteren of extra zendtijd

Dus:

- decentralisatie geeft eenvoud en flexibiliteit
- maar kost energie-efficiëntie

## TSMP

De derde case study is **Time Synchronized Mesh Protocol (TSMP)**.

TSMP is tegelijk een MAC- en network-layerprotocol, ontwikkeld voor draadloze sensornetwerken.

De ontwerpdoelen zijn:

- betrouwbaarheid
- laag energieverbruik
- security
- self-organization
- self-healing
- minimale managementlast

De kern van TSMP is bijna het tegenovergestelde van BMAC:

- BMAC is asynchroon en gedecentraliseerd
- TSMP is sterk **gesynchroniseerd** en meer **gecoördineerd**

## Waarom centralisatie hier toch interessant kan zijn

TSMP gebruikt een **centrale manager** voor coördinatie.

Dat lijkt op het eerste gezicht zwak, en de slides noemen ook meteen de risico’s:

- hoge load op de manager
- minder robuustheid

Maar in veel sensornetwerken is er toch al een **gateway** nodig die alles uiteindelijk naar buiten brengt. Dan is een zekere centrale coördinatie minder absurd dan in algemene internetnetwerken.

## TSMP gebruikt TDMA

TSMP werkt met **Time Division Medium Access (TDMA)**.

Dat betekent:

- tijd wordt in slots verdeeld
- nodes weten precies wanneer ze mogen zenden of luisteren

Dit minimaliseert collisions:

- minder collisions
- minder retransmissies
- dus minder energieverbruik

Maar TDMA kan alleen werken als nodes **precies gesynchroniseerd** zijn.

## Hoe synchroniseren nodes?

Volgens de slides levert de master node de timing.

Bij het joinen:

- luistert een nieuwe node naar bestaande transmissies
- gebruikt die om zijn eigen klok af te stellen
- en ontdekt door een hele framecyclus wanneer hij moet zenden en luisteren

Daardoor kan de radio de rest van de tijd uitstaan.

### belangrijk voor examen

TSMP wint veel energie-efficiëntie door **nauwkeurige tijdssynchronisatie**, zodat de radio bijna alleen actief is op exact geplande momenten.

## QoS en voorspelbaarheid in TSMP

Doordat tijdslots expliciet toegewezen worden, kan TSMP vrij precies resources reserveren.

Dat laat toe om nodes verschillende hoeveelheden bandbreedte te geven, bijvoorbeeld:

- weinig slots voor een temperatuursensor
- meer slots voor een camera

Dus synchronisatie helpt niet alleen tegen collisions, maar ook voor **voorspelbare service**.

## Frequentiehopping

Een tweede sterk idee in TSMP is **frequency hopping**.

Interferentie zit vaak niet gelijk verdeeld over alle kanalen. Als je één slecht kanaal kiest, kan dat desastreus zijn. Maar als alle nodes gesynchroniseerd door een bekende kanaalsequentie hoppen, dan spreid je het risico.

De slides tonen dat een interferentiebron op een paar kanalen dan niet meer permanent alle communicatie dooddrukt, maar eerder een beheersbare packet loss veroorzaakt.

### voordelen van channel hopping

Channel hopping biedt meerdere concrete voordelen:

- **Interferentiebestendigheid**: als een externe bron (bv. een magnetron of ander wifi-netwerk) een bepaald kanaal verstoort, worden slechts enkele timeslots getroffen in plaats van alle communicatie
- **Multipath fading mitigatie**: signaalsterkte varieert per frequentie; door te hoppen vermijd je dat je permanent op een "slechte" frequentie zit
- **Beveiliging**: zonder kennis van de hopsequentie is het moeilijker om communicatie af te luisteren of te jammen
- **Eerlijke verdeling**: alle nodes delen de goede en slechte kanalen eerlijk over tijd

### belangrijk voor examen

Channel hopping is een examenvraag op zich. De kerngedachte is: in plaats van al je geluk op een kanaal te zetten, spreid je de communicatie over tijd en frequentie. Zo wordt een interferentiebron op een paar kanalen beheersbaar in plaats van fataal.

## Betrouwbaarheid in TSMP

TSMP gebruikt:

- link-layer acknowledgements
- retransmissies per hop
- meerdere mogelijke ouders voor een node

Dat geeft twee soorten redundantie:

- **spatial redundancy**: probeer via een andere buur
- **temporal redundancy**: probeer op een ander tijdstip

Dat maakt het netwerk robuuster tegen interferentie en uitval.

## TSMP-frameformaat

De slides tonen dat TSMP bovenop 802.15.4 werkt, waar de totale packetgrootte klein is.

Een groot deel van de beschikbare bytes wordt gebruikt voor:

- PHY/MAC/NET headers
- integrity codes

Daardoor blijft er relatief weinig payload over.

Dat is belangrijk, want in zulke netwerken zijn de controlkosten niet verwaarloosbaar. Het protocolontwerp moet dus heel zuinig omgaan met framing.

## Grenzen van TSMP

De slides zijn hier vrij eerlijk over.

### Grenzen van centralisatie

- de manager kan een bottleneck worden
- de adressing- en packetstructuur limiteert het aantal nodes sterk

### Grenzen van tijdssynchronisatie

- heel goed voor voorspelbaar verkeer
- maar minder goed voor erg sporadisch verkeer door hogere latency

### Grootste energiekost

De slides geven ook een mooie vraag:

waar zit de primaire energiekost van TSMP?

Het antwoord is:

- in het **(re)joinen van het netwerk**

Dat is logisch, want daar moet de node opnieuw leren wanneer en hoe hij gesynchroniseerd moet meedoen.

### belangrijk voor examen

TSMP is sterk voor voorspelbaar, laagvermogenverkeer, maar de prijs zit in:

- centralisatie
- synchronisatie-overhead
- hogere kost bij join en rejoin

## Stop-and-Wait analyse op de datalinklaag

Hoewel stop-and-wait typisch bij de transportlaag wordt besproken, is de analyse ervan ook zeer relevant voor de datalinklaag. De kernvraag is: **op welk medium werkt stop-and-wait goed, en op welk medium slecht?**

### De formule

De efficiëntie van stop-and-wait hangt af van de verhouding tussen propagatievertraging en transmissietijd:

```text
a = T_propagatie / T_transmissie
efficientie = 1 / (1 + 2a)
```

Stop-and-wait is efficiënt wanneer het **bandwidth-delay product kleiner is dan de framegrootte**, ofwel wanneer `a` dicht bij 0 ligt. De zender hoeft dan maar heel even te wachten op een ACK.

### Wanneer goed?

| Medium | Waarom goed |
| --- | --- |
| **Kort Ethernet** (100 m) | T_propagatie is verwaarloosbaar klein t.o.v. T_transmissie |
| **LoRa SF12** (20 km, 250 bps, 50 bytes) | T_transmissie = ~1600 ms, T_propagatie = ~67 us, dus a is bijna 0 |
| **Serieel 9600 bps over korte kabel** | T_transmissie >> T_propagatie |

### Wanneer slecht?

| Medium | Waarom slecht |
| --- | --- |
| **GEO-satelliet** (RTT ~540 ms, 10 Mbps) | De zender is het frame in microseconden kwijt, maar wacht honderden milliseconden op ACK: a >> 1 |
| **Trans-Atlantische kabel** (RTT ~80 ms) | Zelfde probleem: hoog bandwidth-delay product |

Bij een slecht scenario staat de zender het grootste deel van de tijd idle te wachten op een ACK terwijl de link onbenut is. De oplossing is dan een **sliding window** protocol (Go-Back-N of Selective Repeat) dat meerdere frames tegelijk "in de pijplijn" houdt.

### Methodiek voor examenvragen

1. Schat de RTT van het medium
2. Bereken `a = T_propagatie / T_transmissie`
3. Als `a << 1`: stop-and-wait volstaat
4. Als `a >> 1`: sliding window nodig

## MAC-protocollen bij AODV

Dit is een cross-layer examenvraag: welke MAC-protocollen passen goed bij het reactieve routingprotocol AODV, en welke niet?

AODV is **reactief en on-demand**: routes worden pas ontdekt wanneer nodig, via RREQ flooding. De topologie is dynamisch, nodes komen en gaan. Dit vereist een MAC-protocol dat **op elk moment kan zenden zonder vooraf geconfigureerde schedule**.

| MAC | Past bij AODV? | Waarom |
| --- | --- | --- |
| **CSMA/CA** | **Goed** | Zendt wanneer nodig, geen schedule, tolereert dynamische topologie |
| **BMAC** | **Goed** | Asynchroon, gedecentraliseerd, energiebesparing via LPL, past bij dynamische omgeving |
| **TSMP/TDMA** | **Slecht** | Vereist vooraf opgestelde schedule en tijdsynchronisatie, clasht met AODV's dynamische route discovery |
| **Pure ALOHA** | **Slecht** | Geen carrier sense, geen backoff, leidt tot massale collisions bij RREQ flooding |

### Kernredenering

Scheduled MACs zoals TSMP/TDMA veronderstellen een **stabiele topologie** met bekende deelnemers. AODV veronderstelt het **tegenovergestelde**. Een topologiewijziging bij TDMA forceert kostbare herscheduling van het hele slotschema, terwijl bij CSMA/CA of BMAC een nieuwe node gewoon kan beginnen zenden.

Pure ALOHA is slecht omdat AODV's route discovery flooding gebruikt (RREQ naar alle buren). Zonder carrier sense of backoff leidt dit tot massale collisions, precies wanneer het netwerk communicatie het hardst nodig heeft.

## LoRa MAC: wanneer goed en slecht

LoRa is een long-range, low-power draadloze technologie met specifieke MAC-uitdagingen.

### Kenmerken

- Bereik: tot ~20 km
- Datarate: 250 bps (SF12) tot 50 kbps (SF7)
- ISM 868 MHz band met **1% duty cycle** beperking
- Star-topologie met centrale gateway

### Waarom LoRa goed is

- **Sparse sensing**: enkele sensoren die zelden data sturen (bv. een keer per uur een temperatuurmeting) werken uitstekend
- **Lange afstanden**: waar wifi of Bluetooth niet reiken
- Stop-and-wait is efficiënt (a is bijna 0 door de extreem lange transmissietijd)

### Waarom LoRa slecht is bij hoge densiteit

LoRa gebruikt een **Pure ALOHA-achtig** MAC (Class A). Bij veel nodes wordt de collision-kans catastrofaal:

```text
P(collision) = 1 - e^(-2 * lambda * T_airtime)
```

Bij 100 nodes die elk 1 packet per minuut sturen met SF12 (airtime ~1,6 s) is de collision-kans ~99,5%. Het systeem schaalt dus fundamenteel niet met Pure ALOHA bij hoge node-densiteit.

### MAC-verbeteringen voor LoRa

| Verbetering | Wat het oplost |
| --- | --- |
| **TDMA/slotting (Class B)** | Vermijdt blinde collisions door tijdslots |
| **Channel hopping** | Bestrijdt narrowband interferentie |
| **Adaptive Data Rate** | Verkort airtime waar signaal sterk genoeg is, verkleint collision-window |
| **Duty-cycle coordinatie** | Spreidt zendmomenten over gateways |

### Hidden terminal bij LoRa

Carrier sense (CSMA) is bij LoRa vaak **onmogelijk**: nodes die 35+ km uit elkaar liggen, horen elkaar niet maar delen dezelfde gateway. Een star-topologie met centrale gateway en ALOHA-achtige toegang is dan onvermijdelijk.

## Samenvatting van het hoofdstuk

Dit hoofdstuk laat zien dat MAC-ontwerp altijd trade-offs bevat.

### Statisch versus dynamisch

- statische allocatie is eenvoudig maar verspilt capaciteit
- dynamische allocatie gebruikt het medium beter, maar vraagt slimme protocollen

### Random access

- ALOHA is eenvoudig maar inefficiënt
- slotted ALOHA verbetert dit met synchronisatie
- CSMA probeert collisions te verminderen door eerst te luisteren

### Energiezuinige draadloze MAC

- BMAC gebruikt low power listening en lange preambles
- TSMP gebruikt strakke tijdssynchronisatie en TDMA

Die twee laatste zijn eigenlijk twee heel verschillende filosofieën:

- **asynchroon en eenvoudig**
- versus **gesynchroniseerd en strak gepland**

## Wat je echt moet kennen

### belangrijk voor examen

- Wat het **channel allocation problem** is
- Waarom statische allocatie vaak verspilling geeft
- De vijf aannames bij dynamische MAC-protocollen
- Waarom draadloze MAC extra moeilijk is door interferentie, energie en node loss
- Hoe **Pure ALOHA** werkt
- Waarom **Slotted ALOHA** beter werkt
- Wat **carrier sense** toevoegt in CSMA
- Het verschil tussen **non-persistent**, **1-persistent** en **p-persistent** CSMA
- Wat **hidden** en **exposed terminals** zijn
- Het idee van **low power listening**
- Waarom BMAC een **lange preamble** nodig heeft
- Waarom BMAC goed is voor onvoorspelbaar verkeer
- Waarom TSMP **tijdssynchronisatie** en **TDMA** gebruikt
- Hoe **frequency hopping** helpt tegen interferentie
- Welke redundantie TSMP gebruikt
- Waar de grote beperkingen van TSMP liggen

## Korte eindintuitie

Als je dit hoofdstuk tot één gedachte herleidt:

MAC-protocollen proberen allemaal dezelfde gedeelde vraag op te lossen — wie mag wanneer het medium gebruiken — maar het beste antwoord hangt volledig af van wat je belangrijker vindt: eenvoud, efficiëntie, energieverbruik, voorspelbaarheid of robuustheid.

Dat is precies de rode draad door deze slides.
