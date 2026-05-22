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

### Wat doen leaf nodes?

Leaf nodes verbinden met een ultrapeer en doen minder discoverywerk zelf.

Zo:

- besparen zwakke toestellen resources
- wordt de discovery-overhead beter beheerd

### Vereisten voor ultrapeers

De slides noemen bijvoorbeeld:

- geen firewall
- genoeg bandbreedte
- genoeg uptime
- genoeg RAM en CPU

### belangrijk voor examen

Gnutella 0.6 verbetert Gnutella 0.4 door een **super-node-architectuur** te gebruiken waarin sterke peers meer werk dragen dan zwakke peers.

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

- Het verschil tussen een **P2P-applicatie** en een echt **P2P-netwerk**
- Waarom Napster niet volledig gedecentraliseerd was
- Het basisidee van Gnutella 0.4
- De rol van `PING`, `PONG`, `QUERY`, `QUERYHIT` en `PUSH`
- Wat het **search horizon**-probleem is
- Waarom Gnutella 0.6 ultrapeers gebruikt
- De drie kernproblemen die BitTorrent oplost
- Wat een **tracker**, **swarm**, **chunk** en **seeder** zijn
- Waarom BitTorrent **rarest first** gebruikt
- Hoe **tit-for-tat** free-riding tegengaat
- Het basisidee van **onion routing**
- De rol van **directory servers**, **Onion Proxy**, **Onion Routers** en **exit nodes**
- Waarom TOR vooral tegen **traffic analysis** probeert te beschermen
- Wat een **TOR circuit** is

## Korte eindintuitie

Als je dit hoofdstuk tot één idee terugbrengt:

P2P-systemen proberen de rand van het internet zelf mee het werk te laten doen, maar moeten dan telkens opnieuw oplossingen vinden voor drie lastige dingen: wie vindt wat, wie draagt zijn deel van het werk, en wie ziet welke informatie.

Dat is de kern van deze slides.
