# Deel 11 - Application 3: Peer-to-Peer

In dit deel verschuift de focus van klassieke client/server-systemen naar **peer-to-peer**-systemen. Dat zijn toepassingen waarin de rand van het netwerk, dus de gebruikersmachines zelf, niet alleen consumenten van diensten zijn, maar ook mee resources leveren.

De drie grote thema’s zijn:

- **resource discovery** in P2P-netwerken
- **efficiënte content distribution**
- **privacy en anonimiteit**

De slides werken dat uit aan de hand van vier bekende voorbeelden:

- **Napster**
- **Gnutella**
- **BitTorrent**
- **TOR**

Dat zijn geen willekeurige voorbeelden. Samen tonen ze vier verschillende manieren waarop P2P-systemen met schaal, verantwoordelijkheid, performantie en privacy omgaan.

## Wat bedoelen we met peer-to-peer?

De slides geven twee definities.

### Application-centric

Een P2P-applicatie is een toepassing die resources gebruikt die zich aan de rand van het internet bevinden. Dat zijn dus de machines van de gebruikers zelf:

- opslag
- uploadbandbreedte
- CPU
- online tijd

### Network-centric

Een strengere definitie zegt dat een echt P2P-netwerk volledig gedecentraliseerd is, zonder hiërarchie en met symmetrische communicatie tussen nodes.

### Waarom zijn die twee definities niet hetzelfde?

Omdat een **P2P-applicatie** niet noodzakelijk op een volledig **P2P-netwerk** hoeft te draaien.

Je kan best een toepassing bouwen die gebruikmaakt van resources van peers, maar toch nog centrale servers gebruikt voor indexering, controle of coördinatie.

Dat onderscheid is belangrijk voor Napster.

## Napster

**Napster** was een van de eerste echt grote P2P-toepassingen. Het maakte grootschalige muziekdeling mogelijk zonder dat Napster zelf voor enorme opslag en dataverkeer moest betalen.

De truc was eenvoudig:

- gebruikers boden zelf bestanden aan
- Napster gebruikte die gebruikersmachines als infrastructuur

Dat maakte het systeem heel krachtig, maar ook juridisch en technisch interessant.

## Hoe Napster werkte

De slides beschrijven Napster als volgt:

- elke peer uploadde een lijst van zijn bestanden naar een **centrale server**
- andere gebruikers konden die server doorzoeken
- de eigenlijke data werd daarna tussen gebruikers uitgewisseld

Dus:

de **indexering** was centraal, maar de **bestandsuitwisseling** was gedistribueerd.

### waarom was dat slim?

Omdat centrale indexering zoeken veel makkelijker maakt.

### waarom was dat ook zwak?

Omdat de centrale server:

- een single point of failure werd
- een single point of attack werd
- en een single juridische target werd

Dat laatste is exact wat ook gebeurde: Napster werd juridisch aangepakt en moest sluiten.

### belangrijk voor examen

Napster was een **P2P-applicatie met centrale indexering**. Dat gaf goede zoekfunctionaliteit, maar ook een duidelijk single point of failure en verantwoordelijkheid.

## Het algemene P2P application pattern

De slides halen uit Napster een meer algemeen patroon:

1. geef gebruikers een reden om resources te delen
2. indexeer beschikbare resources
3. gebruik die gedeelde resources om een gedistribueerde dienst te bouwen

Dat is een handige mentale kapstok voor bijna elk P2P-systeem.

### Incentives zijn cruciaal

Een P2P-systeem werkt alleen als gebruikers ook effectief bijdragen. Als iedereen alleen wil downloaden en niemand wil uploaden of delen, valt het systeem stil.

## P2P-app is niet hetzelfde als P2P-netwerk

Deze slide is inhoudelijk belangrijker dan ze op het eerste gezicht lijkt.

Een P2P-app kan edge resources gebruiken, maar toch centraal blijven op een cruciaal punt. Dat beperkt:

- schaalbaarheid
- robuustheid
- juridische ontkoppeling

Daarnaast maken de slides ook een mooie observatie:

als een centrale partij weet welke data er gedeeld wordt, kan die partij verantwoordelijk gehouden worden voor die data.

Dus centralisatie is niet alleen een technisch risico, maar ook een juridisch en organisatorisch risico.

## Overlay-netwerken

Een belangrijk concept in P2P is het **application-level overlay network**.

Dat is een logisch netwerk tussen peers bovenop het echte internet.

Maar de slides waarschuwen terecht:

- één hop in het overlay-netwerk
- kan in de echte network layer meerdere hops zijn

Dus berichten rondsturen op een overlay is niet gratis. Daarom moet je application-level message passing zo veel mogelijk beperken.

### intuition

Een P2P-overlay lijkt misschien op een kort pad tussen twee peers, maar op het echte internet kan dat onderliggend veel langer zijn.

## Gnutella 0.4

**Gnutella** werd ontwikkeld als een meer gedecentraliseerd alternatief voor Napster.

In plaats van centrale indexservers bouwt Gnutella een **unstructured decentralized overlay network** bovenop TCP/IP.

Elke host doet mee aan het netwerk door:

- peers te helpen ontdekken
- zoekboodschappen verder te sturen
- onderhoudsverkeer te dragen

Dat lost het Napster-probleem van de centrale server op, maar creëert een nieuw probleem: **overhead**.

## De bijdrage van Gnutella

Gnutella doet in wezen hetzelfde als Napster, maar zonder centrale servers.

Voordelen:

- geen single point of failure
- geen enkele grote indexserver nodig
- moeilijker om één centrale verantwoordelijke partij juridisch te treffen

Daarnaast geeft het een beetje beperkte anonimiteit, omdat een gebruiker meestal vooral zijn directe buren kent.

Maar die voordelen hebben een kost: discovery gebeurt via veel meer berichtverkeer.

## Gnutella-berichten

De slides noemen vijf belangrijke berichttypes:

- `PING`
- `PONG`
- `QUERY`
- `QUERYHIT`
- `PUSH`

### PING en PONG

Deze worden gebruikt voor **peer discovery**.

- `PING` wordt gebroadcast
- peers die beschikbaar zijn antwoorden met `PONG`

Een `PONG` bevat informatie zoals:

- IP-adres
- poort
- metadata over de peer

### QUERY en QUERYHIT

Deze worden gebruikt voor **resource search**.

- `QUERY` zoekt naar content
- `QUERYHIT` zegt dat een peer iets passends heeft

### PUSH

Deze is belangrijk voor peers achter een firewall. Als een peer niet rechtstreeks bereikbaar is, kan een andere peer via `PUSH` vragen dat de firewall-peer zelf de verbinding opzet en de file "push’t".

## Gnutella-fase 1: verbinding opbouwen

Een nieuwe peer komt het netwerk binnen via een bekende peer en begint daarna discovery te doen.

De logica:

- de nieuwe peer opent een TCP-verbinding naar een bekende node
- die node floodt een `PING`
- beschikbare peers sturen `PONG` terug
- de `PONG`s reizen terug langs het pad van de inkomende `PING`

Die broadcast wordt beperkt met een **TTL**. Elke peer verlaagt die TTL en gooit het bericht weg als die 0 wordt.

### waarom TTL?

Omdat volledige flood search anders totaal oncontroleerbaar zou worden.

## Gnutella-fase 2: zoeken

Zoeken werkt analoog:

- een peer floodt een `QUERY`
- peers sturen die verder, met dalende TTL
- een peer met passende content stuurt een `QUERYHIT` terug langs het pad van de query

Belangrijk is dat `QUERYHIT` meteen ook informatie bevat om de file later op te halen, zoals:

- adres
- poort

De zoekfase en de transferfase zijn dus logisch gescheiden.

## Gnutella-fase 3: file transfer

Na een `QUERYHIT` probeert de requester een **directe download** op te zetten met de doelpeer, via **HTTP**.

Dat is interessant:

Gnutella gebruikt zijn eigen overlay voor discovery, maar leunt voor de eigenlijke dataoverdracht op HTTP.

Als de doelpeer achter een firewall zit, kan een `PUSH` gebruikt worden zodat die peer zelf de verbinding initieert.

## Het search horizon-probleem

De grote zwakte van Gnutella 0.4 is dat **broadcast search niet schaalbaar** is.

Daarom beperkt men queries met een TTL. Maar dat geeft meteen een fundamenteel probleem:

- te lage TTL: je vindt te weinig
- te hoge TTL: je overspoelt het netwerk

Dat spanningsveld noemt de cursus de **search horizon**.

De slides geven een sterk voorbeeld: zelfs een eenvoudige zoekopdracht op Napster-schaal zou gigantische hoeveelheden dataverkeer kunnen losmaken.

### belangrijk voor examen

Het **search horizon**-probleem is de spanning tussen:

- genoeg ver zoeken om content te vinden
- maar niet zoveel flooden dat discovery onbetaalbaar wordt

## Gnutella 0.6

De oplossing in **Gnutella 0.6** is een hiërarchischer model met:

- **ultrapeers**
- **leaf nodes**

Dit is een klassiek **super-node**-idee.

### Wat doen ultrapeers?

Sterke nodes nemen meer verantwoordelijkheid op zich:

- peer discovery
- meer routing- en zoekwerk
- lijsten van resources van leaf nodes bijhouden
- fungeren als **proxy** voor hun leaf nodes

### Wat doen leaf nodes?

Leaf nodes verbinden met een ultrapeer en doen minder discoverywerk zelf.

Zo:

- besparen zwakke toestellen resources
- wordt de discovery-overhead beter beheerd

### Vereisten voor ultrapeers

De slides noemen bijvoorbeeld:

- geen firewall (moet direct bereikbaar zijn)
- genoeg bandbreedte
- genoeg uptime
- genoeg RAM en CPU

### Gnutella 0.4 versus 0.6: vergelijkingstabel

| Aspect | 0.4 (vlak) | 0.6 (hierarchisch) |
|--------|-----------|-------------------|
| Topologie | Alle peers gelijk | Ultrapeers + leaf nodes |
| Zoekverkeer | Flooding via **alle** peers | Geconcentreerd op ultrapeers |
| Schaalbaarheid | Slecht (O(N) berichten) | Beter |
| Anonimiteit | Hoger (enkel buren kennen je) | **Lager** -- een ultrapeer ziet alles van zijn leaves |

### belangrijk voor examen

Gnutella 0.6 verbetert Gnutella 0.4 door een **super-node-architectuur** te gebruiken waarin sterke peers meer werk dragen dan zwakke peers. Maar dit gaat **ten koste van anonimiteit**: een ultrapeer kent de bestandslijsten, zoektermen en IP-adressen van al zijn leaf nodes.

## Gnutella: transport-keuze

Een subtiel maar examenwaardig punt is de transportkeuze binnen Gnutella:

- **Discovery** (PING/PONG/QUERY): past bij **UDP** -- kleine berichten, connectionless, flooding-vriendelijk
- **Bestandsoverdracht**: gebruikt **TCP/HTTP** -- betrouwbaar, geordend, een volledig bestand vereist dat alle bytes correct aankomen

In de originele implementatie draait het overlay-netwerk zelf over **TCP-verbindingen** tussen peers. Maar conceptueel past de discovery-logica beter bij UDP-achtig gedrag.

## Monitoring en spionage in Gnutella

Dit is een favoriet examenvraagtype: "Hoe zou je het Gnutella-netwerk kunnen bespioneren?"

### Spionage met beperkte resources (1 node)

- **Als gewone peer in 0.4**: je ziet enkel QUERY/QUERYHIT die via jouw directe buren passeren. Beperkt zicht, want je ziet alleen verkeer dat toevallig door jou gerouteerd wordt.
- **Als leaf node in 0.6**: je ziet enkel je eigen verkeer. Minimaal zicht.
- **Als ultrapeer in 0.6**: je ziet **alle** QUERY's van jouw leaf nodes plus die van buur-ultrapeers. Je kent: zoektermen, IP-adressen, bestandslijsten en activiteitspatronen van je leaves. Dit is veel meer informatie dan een gewone peer in 0.4 ooit ziet.

### Spionage met onbeperkte resources

- **In 0.4**: deploy honderden nodes met hoge degree (veel buren) zodat je een groot deel van het flooding-verkeer ziet.
- **In 0.6**: word ultrapeer op meerdere plekken in het netwerk. Als je 10% van alle ultrapeers controleert, zie je een significant deel van al het zoekverkeer.

### Hughes-punt

Gnutella 0.6 koopt **performantie** maar verliest **anonimiteit**. Copyright-handhaving en surveillance worden triviaal als je een ultrapeer controleert: je weet precies wie wat zoekt en wie welke bestanden aanbiedt.

## BitTorrent

Waar Gnutella vooral een discoveryprobleem oplost, is **BitTorrent** vooral ontworpen voor **efficiënte content distribution**.

De slides zetten dat mooi neer:

- Chord is voor object routing
- Gnutella is voor resource discovery
- BitTorrent is voor content distribution

Dat is een belangrijk onderscheid. BitTorrent draait niet in de eerste plaats om "iets vinden", maar om "iets groot efficiënt verspreiden".

## Het volledige file-sharingprobleem

De slides formuleren drie grote deelproblemen:

1. hoe vind je de file?
2. hoe repliceer je de content efficiënt?
3. hoe stimuleer je uploadgedrag?

BitTorrent geeft op elk van die drie vragen een praktisch antwoord.

## Discovery in BitTorrent

BitTorrent gebruikt traditioneel **webinfrastructuur** om de torrentbestanden te vinden. Een torrent beschrijft hoe je een file kan downloaden.

Zo’n torrent bevat dan weer informatie over **trackers** die weten welke peers deelnemen aan de **swarm**.

Dus:

- het web helpt je de torrent vinden
- de torrent helpt je de swarm vinden

Dat is een slimme combinatie van klassieke webtechnologie en P2P-distributie.

## Chunks en hashes

Een file wordt in BitTorrent opgesplitst in veel kleine **chunks**.

Elke chunk heeft een **SHA-1 hash**.

Dat is belangrijk om twee redenen:

- je kan de integriteit van chunks controleren
- je kan fijnmazig en parallel downloaden van meerdere peers

De peer krijgt van de tracker adressen van de swarm, en daarna kan hij chunks bij verschillende peers ophalen.

## Rarest first

BitTorrent gebruikt een slimme strategie voor replicatie:

- peers delen welke chunks ze hebben
- nieuwe downloads kiezen bij voorkeur de **rarest pieces first**

Waarom?

Omdat zeldzame stukken anders het bottleneckrisico vormen. Door die eerst extra te verspreiden, verhoog je de totale beschikbaarheid van de file in de swarm.

### belangrijk voor examen

BitTorrent gebruikt **rarest first** om de beschikbaarheid van minder verspreide chunks te maximaliseren.

## Seeders en peers

Een peer die alle chunks heeft, is een **seeder**.

Gewone peers zijn tegelijk:

- downloader
- uploader

Dat is een fundamenteel verschil met klassieke client/server file downloading.

## Free-riding en tit-for-tat

Het derde grote probleem is **free-riding**: gebruikers die wel downloaden maar niet willen uploaden.

BitTorrent pakt dat aan met een **tit-for-tat**-achtig handelssysteem:

- peers meten de prestaties van anderen
- ze blijven vooral data uitwisselen met peers die zelf ook goed bijdragen

Peers die weinig teruggeven, worden **choked** en krijgen minder of geen service.

Dat systeem zorgt er in de praktijk ook voor dat peers met gelijkaardige snelheden elkaar vaak vinden.

### belangrijk voor examen

BitTorrent gebruikt een **tit-for-tat**-mechanisme om uploadgedrag te stimuleren en free-riders af te remmen.

## Trackers en nieuwere versies

De slides vermelden nog dat nieuwere versies van BitTorrent een **DHT** gebruiken, gebaseerd op **Kademlia**, om adressen van peers op te slaan.

Dat vermindert de afhankelijkheid van trackers en maakt het systeem weer wat gedecentraliseerder.

## Chord: gestructureerde P2P met DHT

Chord wordt in de slides kort vermeld bij BitTorrent, maar het is een belangrijk onderwerp op zich. Chord lost het discovery-probleem op een fundamenteel andere manier op dan Gnutella.

### Wat is een DHT?

Een **Distributed Hash Table** (DHT) is een gedistribueerde datastructuur die key-value paren opslaat over meerdere nodes. Het doel: gegeven een key, vind de juiste node die de waarde bewaart, zonder flooding of centrale server.

### Hoe Chord werkt

Chord plaatst zowel **nodes** als **bestanden** op een circulaire ring van 0 tot 2^160 - 1 (de SHA-1 keyspace).

- Elke node krijgt een positie op de ring via `hash(IP-adres)`
- Elk bestand krijgt een positie via `hash(bestandsnaam)` of `hash(bestandsinhoud)`
- Een bestand wordt opgeslagen op de **eerste node die gelijk aan of groter is** dan de hash van het bestand (de **successor**)

### Finger tables

Om niet langs elke node op de ring te moeten wandelen, houdt elke node een **finger table** bij. Die bevat verwijzingen naar nodes op exponentieel groeiende afstanden op de ring:

- finger[1] = successor op afstand 2^0
- finger[2] = successor op afstand 2^1
- finger[k] = successor op afstand 2^(k-1)

Dit geeft een lookup-complexiteit van **O(log N)** hops, in plaats van O(N) bij Gnutella's flooding.

### Vereisten voor de hashfunctie

De hashfunctie moet twee eigenschappen hebben:

1. **Uniforme verdeling**: keys moeten gelijkmatig over de ring verdeeld worden. Als de verdeling scheef is, krijgen sommige nodes veel meer bestanden dan andere (hotspots), en wordt het systeem ongebalanceerd.

2. **Collision-resistentie**: verschillende bestanden mogen niet dezelfde hash krijgen, anders worden ze op dezelfde node geplaatst en is het onduidelijk welk bestand je opvraagt.

**Virtual nodes** helpen bij niet-perfecte verdeling: een fysieke node neemt meerdere posities in op de ring, wat de load beter spreidt.

### Gnutella over Chord: de vergelijking

Een belangrijke examenvraag is: "Wat als je Gnutella's discovery vervangt door Chord?" Dit geeft:

| Aspect | Gnutella (flooding) | Gnutella op Chord |
|--------|--------------------|--------------------|
| Zoek-overhead | O(N) berichten | O(log N) gerichte hops |
| Zoekgarantie | Nee (TTL-horizon) | Ja, als het bestand bestaat |
| Privacy | Hoger (breed verspreide query) | **Lager** (deterministisch pad, traceerbaar) |
| Keyword search | Ja (substring matching) | **Nee** -- enkel exacte hash-lookups |

### Examenvalkuil: keyword search

Chord ondersteunt **geen keyword of substring search**. Als je zoekt naar "Beatles", moet je de exacte hash van een bestandsnaam kennen. In Gnutella kan een QUERY een substring bevatten die door elke peer lokaal gematcht wordt.

Om keyword search op Chord te bouwen, zou je een **extra indexlaag** nodig hebben die trefwoorden mapt naar bestandshashes. Dat voegt weer complexiteit toe.

### Privacy-vergelijking: Chord vs Gnutella

| Aspect | Gnutella 0.4 | Chord |
|--------|-------------|-------|
| Wie ziet je zoekterm? | Alle nodes langs het flood-pad | Enkel de nodes langs het O(log N) pad naar de key |
| Traceerbaarheid | Moeilijk (query verspreidt breed) | Makkelijker (deterministisch pad, hash van zoekterm is publiek) |
| Wie weet dat jij een bestand hebt? | Enkel jij (tot je een QUERYHIT stuurt) | De predecessor-node weet dat jij de key beheert |

Chord is dus **minder anoniem** dan Gnutella: het deterministisch routeringspad maakt het makkelijker om te achterhalen wie wat zoekt of opslaat.

## Privacy en anonimiteit

De laatste grote blok gaat over **TOR**.

De slides kaderen anonimiteit als een socio-technisch thema:

- het kan gebruikers beschermen tegen censuur
- maar het kan ook misbruikt worden

Dat dubbele karakter is belangrijk. TOR is technisch interessant, maar ook maatschappelijk beladen.

## Onion routing

De basistechniek achter TOR is **onion routing**.

Het idee:

- een bericht wordt in meerdere lagen encryptie verpakt
- elke tussenliggende router verwijdert precies één laag
- daardoor leert elke router alleen van wie hij het bericht kreeg en naar welke volgende hop het moet

Maar niet:

- de volledige route
- de echte oorsprong
- de finale bestemming
- of de volledige inhoud

### intuition

Zoals bij een ui pel je laag per laag informatie weg. Geen enkele tussenliggende node ziet het hele plaatje.

## TOR: algemene aanpak

Een TOR-client haalt eerst een lijst van beschikbare TOR-routers op via een **directory server**.

Daarna kiest de client een willekeurig pad door het TOR-netwerk, eindigend in een **exit node**.

Belangrijk detail uit de slides:

- alle links in TOR zijn versleuteld
- behalve de uitgaande link vanaf de exit node naar het gewone internet

Dus TOR beschermt sterk binnen het overlay-netwerk, maar zodra verkeer het gewone internet opgaat via de exit node, gelden de normale risico’s daar weer.

### Wat weet elke node? (examenfavoriet)

Dit is een van de vaakst gestelde TOR-vragen. De kennis per node is bewust beperkt:

| Node | Bron-IP | Bestemming | Data | Vorige hop | Volgende hop |
|------|---------|------------|------|------------|-------------|
| **Entry node** | **Ja** | Nee | Versleuteld (2 lagen) | Client | Intermediate |
| **Intermediate node** | Nee | Nee | Versleuteld (1 laag) | Entry | Exit |
| **Exit node** | Nee | **Ja** | **Plaintext** (als geen end-to-end encryptie) | Intermediate | Server |

Cruciale observatie: **geen enkele node kent zowel de bron als de bestemming**. Dat is precies het doel van het ontwerp.

### Waarom minimaal 3 nodes (entry + intermediate + exit)?

Met slechts **2 nodes** (entry die direct ook exit is, of entry -> exit zonder tussennode) kent een node zowel de bron-IP als de bestemming. Daarmee is de volledige **deanonymisatie** een feit.

De **intermediate node** breekt de directe link: de entry kent de bron maar niet de bestemming, de exit kent de bestemming maar niet de bron.

### Waarom NIET 10 hops?

Meer hops lijkt veiliger, maar is dat paradoxaal genoeg niet:

1. **Latency**: elke hop kost vertraging. 10 hops maakt TOR onbruikbaar traag.
2. **Paradoxaal onveiliger**: de kans dat minstens een node gecompromitteerd is, stijgt met het aantal nodes. Bij kans p=20% per node: N=3 geeft 49% kans op minstens 1 slechte node, N=10 geeft 89%.
3. **Globale timing-correlatie** (door een state actor) wordt niet door extra hops verslagen. Als een aanvaller zowel het begin als het eind van de communicatie kan observeren, helpen extra tussenliggende hops niet.

## Threat model

De slides zijn hier vrij realistisch over. TOR kan niet alles voorkomen.

Een aanvaller kan bijvoorbeeld:

- verkeer genereren
- wijzigen
- vertragen
- verwijderen
- zelf onion routers uitbaten
- meerdere routers compromitteren

TOR is ontworpen om vooral **traffic analysis** en monitoring moeilijker te maken, niet om een magische absolute onzichtbaarheid te bieden.

### Surveillance-scenario’s (beperkte vs onbeperkte resources)

Dit is een terugkerend examenvraagtype:

**Met beperkte resources:**

- Run een of meerdere **exit nodes**. Je kunt dan de **plaintext-inhoud** loggen van onversleuteld verkeer (HTTP, DNS-queries). Je ziet de bestemming, maar niet de afzender.
- Traffic-pattern analyse op je exit node: correleer timing en volume van inkomend/uitgaand verkeer.

**Met onbeperkte resources (state actor):**

- **Globale timing-correlatie**: observeer verkeer bij zowel de entry als de exit. Correleer packet-timing en volume end-to-end. Dit is het sterkste aanvalsmodel en TOR biedt hier **geen bescherming** tegen.
- Deploy een groot aantal malicious onion routers om de kans te verhogen dat een circuit volledig door jouw nodes loopt.

Kernpunt: TOR beschermt tegen een **lokale adversary** (iemand die een deel van het netwerk ziet), maar niet tegen iemand die **beide uiteinden** simultaan observeert.

### Sybil attack op TOR

Een **Sybil attack** is een aanval waarbij een aanvaller een groot aantal valse identiteiten (nodes) in het netwerk injecteert. In de context van TOR:

- Een aanvaller registreert honderden of duizenden onion routers bij de directory servers
- Hoe meer nodes de aanvaller controleert, hoe groter de kans dat een willekeurig gekozen circuit **volledig door zijn nodes** loopt
- Als de aanvaller zowel de entry als de exit node van een circuit controleert, kan hij timing-correlatie doen en de gebruiker deanonymiseren

**Verdediging**: TOR’s directory authorities monitoren verdacht gedrag (te veel nodes van hetzelfde IP-bereik, nodes die net zijn opgezet). Maar een goed gefinancierde aanvaller kan dit omzeilen door geografisch verspreide nodes te deployen.

## TOR-architectuur

Elke client draait een **Onion Proxy (OP)**.

Die:

- biedt een **SOCKS-interface** aan applicaties
- ontdekt routers
- bouwt circuits op in het overlay-netwerk

Daarnaast heb je **Onion Routers (ORs)** die verkeer verder sturen of als exit node optreden.

Belangrijk is dat een client niet hetzelfde is als een onion router.

## Onion routers en sleutels

TOR gebruikt public-key cryptografie.

Onion routers houden sleutelparen bij om:

- hun descriptors te ondertekenen
- sessies en circuits op te bouwen

Dat vormt de cryptografische basis van het vertrouwen in het overlay-netwerk.

## TOR cells

TOR transporteert verkeer in **fixed-size cells**.

Die bevatten onder andere:

- een **Circuit ID**
- een descriptor van het payloadtype

De slides maken onderscheid tussen:

- **control cells**
- **relay cells**

Control cells worden lokaal geïnterpreteerd door een node. Relay cells dragen end-to-end data en extra metadata.

### waarom vaste cell-grootte?

Dat helpt om traffic analysis moeilijker te maken en maakt de verwerking regelmatiger.

## Belangrijke TOR-commando’s

De slides noemen onder meer:

- `relay begin`
- `relay end`
- `relay connected`
- `relay extend`
- `relay extended`

Samen laten die toe om:

- een stream te openen
- een circuit uit te breiden
- en het circuit weer af te sluiten

## Een TOR-circuit opbouwen

De opbouw verloopt stap voor stap.

### Eerst

De client maakt via de Onion Proxy een verbinding met een bekende onion router.

### Dan

Met `relay extend` wordt het circuit verder opgebouwd, hop per hop.

### Daarna

Zodra het circuit klaar is, kan verkeer naar de exit node getunneld worden.

De slides tonen ook dat sessiesleutels tussen de client en elke hop afzonderlijk worden opgezet.

Dat is belangrijk:

een tussenliggende node ziet niet automatisch wie de hele boodschap uiteindelijk kan ontsleutelen.

## Reverse path

Responses komen terug via hetzelfde circuit.

De exit node gebruikt daarvoor de **Circuit ID** en de socketcontext om de juiste terugweg te vinden.

Dat lijkt een beetje op NAT-logica: ook daar wordt inkomend verkeer gekoppeld aan een eerder opgebouwde toestand.

### belangrijk voor examen

TOR verbergt de route door verkeer in lagen te versleutelen en via een **circuit** van onion routers te sturen; elke router kent alleen zijn **volgende hop**.

## P2P chat-applicatie ontwerpen

Dit was een examenvraag in 2024: "Ontwerp een P2P chat-applicatie." Het antwoord vereist dat je per fase van de applicatie de juiste overlay-architectuur kiest en beargumenteert.

### De fasen en overlay-keuze

| Fase | Beste overlay | Waarom |
|------|--------------|-------|
| **Login / presence** (wie is online?) | Napster-stijl centraal of Gnutella 0.6 superpeers | Je hebt een betrouwbare, doorzoekbare directory nodig. Flooding (Gnutella 0.4) is verspilling voor simpele presence-informatie |
| **Contact opzoeken** (peer discovery) | Gnutella 0.6 | Superpeers routen lookups efficient; leaf nodes blijven licht |
| **1-op-1 chat** | Directe verbinding (zoals Gnutella HTTP transfer) | Minimale latency, geen onnodige overlay-overhead |
| **Groot bestand delen naar groep** | BitTorrent (swarming, chunked) | Schaalbaar bij populaire content, parallelle downloads |

### Praktische overwegingen

- **NAT traversal**: veel chat-gebruikers zitten achter NAT. Je hebt een PUSH/relay-mechanisme nodig (vergelijk Gnutella’s PUSH) zodat peers achter NAT toch bereikbaar zijn.
- **Offline berichten**: als de ontvanger offline is, heb je tijdelijke opslag nodig. Dat vereist een server-component (zoals Napster’s centrale server) of superpeers die berichten bufferen.
- **Privacy**: voor een chat-app is end-to-end encryptie belangrijk. TOR-achtige onion routing zou extreem zijn, maar minimaal wil je encryptie op de directe verbinding.

### Kernles

Een goede P2P chat-app combineert elementen van meerdere systemen: centrale of super-node indexering voor presence, directe verbindingen voor chat, en BitTorrent-achtige distributie voor grote bestanden. Geen enkel P2P-systeem alleen dekt alle behoeften.

## Samenvatting van het hoofdstuk

Dit hoofdstuk laat drie grote P2P-thema’s zien.

### Discovery

- Napster: centrale indexering
- Gnutella: decentrale flood search
- Gnutella 0.6: super-node-architectuur

### Distribution

- BitTorrent: chunking, trackers, rarest first, tit-for-tat

### Privacy

- TOR: onion routing, circuits, exit nodes, traffic-analysebescherming

Samen tonen die hoe breed "peer-to-peer" eigenlijk is: het gaat niet alleen om files delen, maar ook om netwerkarchitectuur, incentives en privacy.

## Wat je echt moet kennen

### belangrijk voor examen

**Gnutella (topprioriteit, ~12 keer gevraagd):**

- Het verschil tussen een **P2P-applicatie** en een echt **P2P-netwerk**
- Waarom Napster niet volledig gedecentraliseerd was (centrale indexering = single point of failure)
- De rol van `PING`, `PONG`, `QUERY`, `QUERYHIT` en `PUSH` in Gnutella 0.4
- Wat het **search horizon**-probleem is (TTL te laag = te weinig gevonden, te hoog = flooding)
- Het verschil tussen **Gnutella 0.4** (vlak, flooding) en **0.6** (ultrapeers + leaf nodes)
- Dat 0.6 performantie koopt maar **anonimiteit verliest** (ultrapeer ziet alles van zijn leaves)
- **Monitoring/spionage**: wat je ziet als gewone peer (0.4), als leaf (0.6), als ultrapeer (0.6)
- Het **NAT/PUSH-mechanisme**: hoe peers achter een firewall bestanden kunnen delen
- De transport-keuze: UDP past bij discovery, TCP/HTTP bij bestandsoverdracht

**Chord (DHT, ~5 keer gevraagd):**

- Hoe Chord werkt: consistente hashing, circulaire ring, finger tables, O(log N) lookup
- Hashfunctie-vereisten: uniforme verdeling + collision-resistentie
- Chord vs Gnutella: beter zoekgarantie, maar **geen keyword search** en **minder anoniem**
- Virtual nodes voor load balancing

**TOR (~4 keer gevraagd):**

- Het basisidee van **onion routing** (laag-per-laag encryptie)
- **Wat elke node weet**: entry kent bron, exit kent bestemming, intermediate kent geen van beide
- Waarom **minimaal 3 nodes** nodig zijn (intermediate breekt de link)
- Waarom **niet 10 hops** (latency, paradoxaal onveiliger, timing-correlatie)
- **Spionage met beperkte vs onbeperkte resources** (exit nodes loggen vs globale timing-correlatie)
- **Sybil attack**: valse nodes injecteren om circuits te controleren

**BitTorrent:**

- De drie kernproblemen die BitTorrent oplost (discovery, replicatie, incentives)
- Wat een **tracker**, **swarm**, **chunk** en **seeder** zijn
- Waarom BitTorrent **rarest first** gebruikt
- Hoe **tit-for-tat** free-riding tegengaat

**P2P chat-app ontwerpen:**

- Per fase (presence, discovery, chat, file sharing) de juiste overlay kiezen en beargumenteren
- NAT traversal als praktische randkwestie

## Korte eindintuitie

Als je dit hoofdstuk tot één idee terugbrengt:

P2P-systemen proberen de rand van het internet zelf mee het werk te laten doen, maar moeten dan telkens opnieuw oplossingen vinden voor drie lastige dingen: wie vindt wat, wie draagt zijn deel van het werk, en wie ziet welke informatie.

Dat is de kern van deze slides.
