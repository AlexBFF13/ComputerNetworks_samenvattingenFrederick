# Deel 5 - Network 3: Routing Protocols, Mobility en Ad-Hoc Networks

In dit deel verschuift de focus van algemene network-layerprincipes naar concrete protocollen en dynamische netwerken. De centrale vraag is hier niet meer alleen "hoe werkt routing in theorie?", maar eerder:

- hoe doet het internet dit in de praktijk?
- hoe routeer je tussen verschillende autonome netwerken?
- wat als hosts mobiel worden?
- wat als er zelfs geen vaste infrastructuur meer is?

Daarom vallen deze slides uiteen in twee grote blokken:

- internet routing protocols: **OSPF** en **BGP**
- dynamische netwerken: **mobile networks** en **ad-hoc networks**

## Basiswoordenschat

Voor je aan OSPF en BGP begint, moet je een paar termen goed uit elkaar houden.

### Autonomous System (AS)

Een **Autonomous System** is een stuk netwerk dat door één organisatie beheerd wordt. Zo’n AS wordt geïdentificeerd door een **ASN**.

### Intra-domain routing

Dat is routing **binnen** één AS. Daar gebruik je een **interior gateway protocol**.

### Inter-domain routing

Dat is routing **tussen** verschillende AS’en. Daar gebruik je een **exterior gateway protocol**.

### Internet Exchange Point (IXP)

Een **IXP** is een plaats waar verschillende AS’en met elkaar verbinden.

### belangrijk voor examen

Je moet het verschil kennen tussen:

- **intra-domain routing**: binnen een AS
- **inter-domain routing**: tussen AS’en

Dat onderscheid verklaart bijna volledig waarom OSPF en BGP zo verschillend zijn.

## OSPF: routing binnen een AS

**OSPF** staat voor **Open Shortest Path First** en is al sinds 1988 een IETF-standaard. Het is een typisch voorbeeld van een **interior gateway protocol** voor intra-domain routing.

De naam zegt eigenlijk al veel:

- **open**: publiek beschreven, geen gesloten vendorprotocol
- **shortest path**: werkt op basis van kortstepadberekening
- **first**: kiest de beste route volgens de gekozen metric

De slides geven zeven ontwerpregels mee. In gewone taal zeggen die vooral dat OSPF:

- open en gestandaardiseerd moet zijn
- met meerdere metrics moet kunnen werken
- snel moet reageren op veranderingen
- QoS-achtige differentiatie moet ondersteunen
- load balancing moet toelaten
- hiërarchie moet ondersteunen
- security moet voorzien

Dat maakt OSPF veel praktischer dan een puur academisch link-statealgoritme.

## OSPF bouwt op link-state routing

De kern van OSPF is nog altijd **link-state routing**. Dat proces zagen we al in het vorige deel, maar hier komt het terug in een echte internetcontext.

De logica blijft:

1. buren ontdekken
2. linkkosten bepalen
3. link-state-informatie in pakketten zetten
4. die pakketten verspreiden
5. zelf het kortstepadprobleem oplossen met Dijkstra

### Stap 1: buren ontdekken

Routers sturen bij het opstarten **HELLO**-berichten uit. Buren antwoorden met hun adres. Zo leert een router welke directe neighbors hij heeft.

### Stap 2: linkkosten bepalen

De kost van een link kan gebaseerd zijn op verschillende dingen:

- afstand
- geldelijke kost
- signaalsterkte
- bandbreedte
- delay

In internetcontext is een metric gelinkt aan bandbreedte of delay gebruikelijk.

### Stap 3: link-state-informatie bouwen

Een OSPF-router maakt een pakket met onder andere:

- zijn eigen adres
- een sequence number
- een age
- de lijst van buren met hun kosten

### Stap 4: verspreiden

Die informatie wordt met **flooding** verspreid, zodat alle routers in het relevante gebied uiteindelijk dezelfde topologie-informatie kennen.

### Stap 5: routes berekenen

Als de link-stateinformatie verzameld is, reconstrueert elke router de graaf en draait hij **Dijkstra** om zijn beste routes te bepalen.

### intuition

Bij OSPF probeert elke router eerst een gemeenschappelijk beeld van de topologie op te bouwen. Pas daarna rekent hij zelf de beste paden uit.

## OSPF-abstractie van het netwerk

De slides tonen ook dat OSPF het netwerk abstraheert als een graaf van routers en links met kosten.

Dat is belangrijk, want op dat abstractieniveau maakt het niet meer uit of een fysiek stuk netwerk achterliggend complex was. Zolang het als nodes en kosten kan voorgesteld worden, kan Dijkstra erop werken.

## OSPF en hiërarchie: areas

Een groot netwerk is te groot om als één vlakke link-stategraaf te behandelen. Daarom gebruikt OSPF **areas**.

De belangrijkste ideeën:

- er is een **backbone area**, Area 0
- andere areas hangen daar aan vast
- **stub areas** hebben typisch één border router
- routers binnen een area delen hun link-statebeeld
- border routers kennen informatie voor meerdere areas

### waarom is dit nuttig?

Omdat je zo de schaal van de routingtabellen en link-state-informatie beheersbaar houdt. Binnen een area hoef je veel detail te kennen, maar niet noodzakelijk alle details van alle andere areas.

### belangrijk voor examen

Bij OSPF is **Area 0 de backbone** die de verschillende areas met elkaar verbindt.

## Load balancing in OSPF

OSPF ondersteunt **Equal Cost MultiPath (ECMP)**.

Dat betekent:

- als er meerdere paden zijn met exact dezelfde kost
- dan kan de router die allemaal onthouden
- en traffic over die paden spreiden

Dat is een vrij eenvoudige vorm van load balancing, maar wel heel nuttig in de praktijk.

## OSPF-packets

De slides tonen ook enkele typische OSPF-berichten:

- **Hello**
- **Link state update**
- **Link state ack**
- **Database description**
- **Link state request**

Het grote praktische punt hier is dat OSPF-berichten gewoon als **standaard IP-pakketten** verstuurd worden.

### belangrijk voor examen

OSPF is een **link-state interior gateway protocol** dat hiërarchie en ECMP ondersteunt, en zijn berichten als gewone IP-pakketten verstuurt.

## Waarom BGP nodig is

Na OSPF gaan de slides naar **BGP**. Daar verandert de situatie fundamenteel.

Binnen een AS wil je meestal gewoon technisch goede paden vinden. Tussen AS’en speelt er veel meer mee dan alleen "wat is het kortste pad?"

Organisaties willen ook beleid afdwingen, bijvoorbeeld:

- geen commercieel verkeer dragen op een educatief netwerk
- bepaalde landen of providers vermijden
- een goedkopere of betrouwbaardere transit kiezen

Dus inter-domain routing is niet puur een technisch optimalisatieprobleem. Het is ook een **policy-probleem**.

## BGP: routing tussen AS’en

**BGP**, het **Border Gateway Protocol**, is het standaardprotocol voor inter-domain routing.

De slides zeggen expliciet dat BGP focust op **flexible routing policies**.

Dat is echt de kern. BGP is niet ontworpen om gewoon de numeriek kortste route te kiezen, maar om AS’en controle te geven over:

- welke routes ze aanvaarden
- welke routes ze aankondigen
- via wie ze verkeer willen sturen

## Peering en transit

Om BGP te begrijpen, moet je ook het verschil kennen tussen **peering** en **transit**.

### Transit

Een AS betaalt een andere partij om verkeer verder te dragen naar de rest van het internet.

### Peering

Twee AS’en wisselen verkeer uit voor elkaars klanten of netwerken, vaak zonder betaling voor algemene transit.

De slides zeggen ook expliciet:

- peering gebeurt vaak via **IXPs**
- peering is **niet transitief**

Dat laatste is belangrijk. Als AS1 met AS2 peert en AS2 met AS3 peert, wil dat niet zeggen dat AS1 automatisch gratis transit krijgt naar AS3.

### Stub networks

Een netwerk met maar één externe verbinding is een **stub network**. Zo’n netwerk heeft vaak geen volledige BGP-complexiteit nodig.

### belangrijk voor examen

**Peering is niet transitief**. Dat is een klassiek conceptueel punt.

## Hoe BGP werkt

De slides noemen BGP een vorm van distance vector, maar met een belangrijk verschil:

het werkt op **paden van AS’en**, niet gewoon op losse routerafstanden.

Een BGP-route bevat dus typisch:

- de bestemming of prefix
- de first hop
- de **AS-path**

Dat AS-path is cruciaal. Het zegt door welke autonome systemen de route loopt.

### waarom is dat handig?

Omdat een AS daarmee beleid kan toepassen:

- dit pad wil ik niet gebruiken
- die provider wil ik vermijden
- verkeer door dat AS is voor ons niet toegestaan

Dus BGP is veel rijker in beleid dan klassieke kortstepad-routing.

## Interne details van AS’en blijven verborgen

Een AS hoeft aan de buitenwereld niet uit te leggen hoe zijn interne routing precies werkt.

Dat betekent:

- intern kan een AS OSPF of iets anders gebruiken
- extern ziet de rest alleen de grote inter-domainadvertenties

Die interne details zijn dus **obfuscated** voor andere AS’en.

Dat is essentieel voor schaal en autonomie: elk AS houdt intern zijn eigen vrijheid.

## Internal BGP (iBGP)

Binnen een AS moeten boundary routers wel weten welke externe routes er bestaan. Maar die BGP-routes worden niet gewoon via OSPF verdeeld alsof het interne link-stateinfo was.

Daarom heb je **iBGP**:

- boundary routers leren externe routes van andere boundary routers
- de interne bereikbaarheid van die boundary routers wordt dan lokaal met OSPF of iets gelijkaardigs opgelost

Dus:

- **OSPF** zegt hoe je intern ergens geraakt
- **BGP/iBGP** zegt via welk extern AS-pad je de buitenwereld best benadert

### intuition

OSPF kiest je weg **binnen het gebouw**.  
BGP kiest via welke **organisatie of provider** je het gebouw buiten gaat.

### belangrijk voor examen

BGP kiest routes op basis van **AS-paths en policy**, niet puur op basis van een technische kortstepadmetric.

## Waarom OSPF en BGP zo verschillend zijn

Nu zie je waarom het internet twee grote routinglagen nodig heeft:

- **OSPF** voor technisch efficiënte routing binnen een AS
- **BGP** voor beleidsgestuurde routing tussen AS’en

Dat onderscheid is fundamenteel. Als je een protocol voor alles zou willen gebruiken, verlies je ofwel schaal, ofwel beleid, ofwel beide.

## Directe vergelijking OSPF vs BGP

| Eigenschap | OSPF | BGP |
| --- | --- | --- |
| **Scope** | Intra-domain (binnen een AS) | Inter-domain (tussen AS’en) |
| **Type** | Interior Gateway Protocol | Exterior Gateway Protocol |
| **Algoritme** | Link-state (Dijkstra) | Path-vector (variant van distance vector) |
| **Routeringskriterium** | Technisch shortest path (kost/metric) | **Policy**: kosten, bandbreedte, welke AS’en vermijden |
| **Kennis per router** | Volledige topologie van de area | Enkel reachability + AS-path (geen interne details van andere AS’en) |
| **Transport** | Direct op IP (protocol 89), **connectionless** | **TCP** (poort 179), **connection-oriented** |
| **Schaalbaarheid** | Beperkt tot een AS (hierarchie via areas) | Ontworpen voor het volledige internet |
| **Convergentiesnelheid** | Snel (directe flooding) | Trager (beleidsfilters, padverificatie) |
| **Informatie gedeeld** | Link-state advertisements (linkkost naar buren) | AS-paths + prefixes (bereikbaarheidsinfo) |

### Waarom OSPF connectionless transport gebruikt

OSPF verspreidt link-state advertisements via flooding naar alle buren. TCP-sessies opzetten per buurnode zou zwaar en zinloos zijn voor dit flooding-model. OSPF draait direct op IP (protocol 89) en is in geest connectionless.

### Waarom BGP TCP gebruikt

Inter-AS sessies zijn **langlevend** (dagen tot weken). Ze moeten betrouwbaar zijn: een verloren BGP-update kan een heel AS onbereikbaar maken. TCP biedt betrouwbaarheid, ordered delivery en flow control. Bovendien zorgt TCP’s retransmissie ervoor dat routinginformatie niet verloren gaat over potentieel onbetrouwbare inter-AS links.

### belangrijk voor examen

OSPF en BGP verschillen fundamenteel in **scope** (intra vs inter-domain), **algoritme** (link-state vs path-vector), **criterium** (technisch vs policy) en **transport** (connectionless vs TCP). Je moet deze vier dimensies kunnen uitleggen.

## Mobile networks

De tweede helft van de slides gaat over dynamische netwerken.

Het eerste probleem is **mobility**: hoe blijft een host bereikbaar terwijl hij van locatie verandert?

De slides maken het onderscheid tussen:

- echt mobiele toestellen, zoals telefoons
- nomadische toestellen, zoals laptops die af en toe van netwerk wisselen

De grote ontwerpkeuze is dat je mobiele hosts liefst een **vast home address** geeft. Dat is veel schaalbaarder dan constant in het hele internet routes naar hun actuele locatie te herberekenen.

## Waarom niet gewoon application-level mobility?

In de praktijk lossen we veel mobiliteit op met dingen als DHCP en DNS:

- op nieuwe locatie krijg je een nieuw lokaal adres
- via DNS blijf je externe diensten vinden

Maar dat helpt niet als **andere hosts jou** rechtstreeks en blijvend moeten kunnen bereiken terwijl jij rondbeweegt.

Daarom is er ook **network-layer mobility** nodig.

## Network-layer mobility

Het kernidee in de slides is:

- een mobiele host heeft een **home agent**
- die home agent bewaart een stabiel thuisadres
- op een nieuwe plek krijgt de host een **care-of address**
- de host meldt dat nieuwe adres aan de home agent

Wanneer de home agent daarna verkeer voor de mobiele host ontvangt, wordt dat:

- ingekapseld
- getunneld
- doorgestuurd naar het care-of address

### Client-flow

\--> verhuist naar nieuw netwerk  
\--> krijgt care-of address  
\--> meldt dit aan home agent  
\--> home agent tunnelt verkeer door

Dat is de basis van network-layer mobility.

## Triangle routing

De slides vermelden daarna **triangle routing**.

Dat betekent dat verkeer niet altijd rechtstreeks van correspondent naar mobiele host loopt. Eerst loopt het vaak via de home agent, en pas daarna verder naar de actuele locatie.

Dat maakt het pad langer en minder elegant, maar wel praktisch uitvoerbaar zonder het hele internet te laten herdenken over jouw actuele positie.

### waarom heet dat triangle routing?

Omdat je in plaats van een rechte lijn vaak een driehoek krijgt:

- remote host
- home agent
- mobile host

## Securityprobleem bij mobility

De slides wijzen terecht op een groot probleem: security.

Als iemand zich kan voordoen als jouw mobiele host en een vals care-of address kan registreren, dan kan jouw verkeer naar de verkeerde plek gestuurd worden.

Dus mobility is niet alleen een routingprobleem, maar ook een authenticatieprobleem.

### belangrijk voor examen

Bij network-layer mobility zorgen **home agent**, **care-of address** en **tunneling** voor bereikbaarheid, maar dit brengt ook duidelijke **securityrisico’s** mee.

## Ad-hoc networks

Nog dynamischer zijn **ad-hoc networks**.

Daar is er geen vaste infrastructuur. Nodes zijn tegelijk:

- host
- router

Dat komt voor in bijvoorbeeld:

- vehicular ad-hoc networks
- multi-hop IoT-netwerken

Omdat alle nodes kunnen bewegen, is statisch redeneren over de topologie bijna nutteloos. De topologie kan elk moment veranderen.

## Typische zorgen in IoT en ad-hoc netwerken

De slides noemen drie belangrijke zorgen:

- **resource constrained**: weinig geheugen, opslag en rekenkracht
- **energy constrained**: nodes moeten lang op batterij werken
- **unreliable**: uitval van nodes is normaal

Dat maakt klassieke zware routingprotocollen vaak ongeschikt.

## AODV

Als concreet voorbeeld behandelen de slides **AODV**, **Ad-Hoc On Demand Vector Routing**.

Het belangrijke verschil met klassieke proactieve routing is dat AODV **reactief** is:

- routes worden pas gezocht wanneer iemand effectief data wil sturen

Dat is logisch in een zeer dynamisch netwerk. Je wil niet de hele tijd veel control traffic sturen voor routes die misschien toch direct weer verouderen.

### belangrijk voor examen

AODV is **on-demand** en dus reactief: het zoekt routes pas wanneer dat nodig is.

## Route discovery in AODV

Wanneer een node een bestemming niet kent, floodt hij een **ROUTE_REQUEST**.

Daarbij is het netwerkmedium typisch broadcastachtig: iedereen binnen bereik kan het horen.

### Wat zit er in zo’n ROUTE_REQUEST?

De slides vermelden vooral:

- een **sequence number**
- een **TTL**

Die dienen om flooding beheersbaar te houden.

Wanneer de bestemming of een node met een bruikbare route gevonden wordt, stuurt die een **ROUTE_REPLY** terug langs het omgekeerde pad van de request.

Onderweg:

- wordt de hop count bijgewerkt
- slaan intermediate nodes de route op in hun tabellen

Zo bouw je niet alleen een pad naar de bestemming op, maar leren tussenliggende nodes ook meteen iets bij.

## Reverse paths en discovered routes

De figuur met stippellijnen en volle lijnen helpt hier goed:

- **dashed lines**: mogelijke reverse paths terug voor de reply
- **solid lines**: de uiteindelijke ontdekte route

Dat is een belangrijk inzicht, want AODV leert routes niet door vooraf een volledige kaart te hebben, maar door tijdens flooding tijdelijke terugpaden te laten ontstaan.

## Route discovery optimaliseren

Flooding is duur. Daarom wordt route discovery geoptimaliseerd.

De slides zeggen dat ROUTE_REQUESTs niet meteen met grote TTL verstuurd worden. Men begint klein:

- eerst bijvoorbeeld met `TTL = 1`
- als dat niet lukt, verhoogt men de TTL stapsgewijs

Zo vermijd je onnodige brede floods als de bestemming eigenlijk dichtbij zit.

### nadeel

NO_ROUTE time-outs kunnen lang zijn, waardoor failures soms traag verwerkt worden. In erg onstabiele netwerken geeft dat vertraging.

## Route maintenance

Als een node merkt dat een actieve buur uitvalt, dan:

- verwijdert hij routes die van die link afhingen
- waarschuwt hij relevante buren
- en gebeurt dat recursief verder

Dat helpt om oude, foutieve routes uit het systeem te halen.

De slides merken ook op dat AODV hiermee een deel van het count-to-infinity-probleem kan omzeilen, maar dat lange time-outs nog altijd lastig blijven.

## Wanneer AODV beter is dan OSPF (en omgekeerd)

Dit is een klassieke examenvraag. De keuze tussen AODV en OSPF hangt af van de netwerkomstandigheden.

### AODV is beter wanneer:

- **Topologie vaak verandert** (mobiele nodes, ad-hoc netwerken): OSPF zou continu link-state updates moeten flooden bij elke topologiewijziging, wat enorm veel overhead geeft
- **Resources beperkt zijn** (IoT, sensor nodes): AODV vereist minimaal geheugen (enkel actieve routes), terwijl OSPF de volledige topologiedatabase moet opslaan
- **Er weinig actief verkeer is**: AODV genereert geen control traffic als niemand data stuurt; OSPF stuurt continu HELLO's en link-state updates
- **Energie beperkt is**: minder control traffic = minder zenden = langere batterijduur

### OSPF is beter wanneer:

- **Topologie stabiel is** (campus, enterprise): de eenmalige kost van topologiedistributie wordt geamortiseerd over lange periodes van stabiliteit
- **Lage latency cruciaal is**: OSPF heeft routes altijd klaar; AODV moet eerst een route discovery doen (vertraging van seconden)
- **Het netwerk groot maar gestructureerd is**: OSPF's hierarchie via areas schaalt goed in gestructureerde omgevingen
- **Betrouwbaarheid cruciaal is**: OSPF convergeert snel en voorspelbaar; AODV kan lijden aan lange NO_ROUTE time-outs

### Kernredenering voor het examen

De keuze draait om de **verhouding tussen topologieveranderingen en dataverkeer**:

- Veel topologiewijzigingen, weinig verkeer: AODV (reactief = minder verspilling)
- Stabiele topologie, veel verkeer: OSPF (proactief = altijd klaar)

## AODV in perspectief

De cursus is vrij eerlijk: AODV is een belangrijk voorbeeld om de ideeen te begrijpen, maar dit onderzoeksdomein is nog lang niet "af". Bij ad-hoc routing zijn de algoritmes veel minder stabiel uitgekristalliseerd dan bij klassieke internet routing.

Dat maakt ook logisch waarom dit als hot research topic voorgesteld wordt.

## Samenvatting van het hoofdstuk

Dit hoofdstuk toont vier heel verschillende routingcontexten:

- **OSPF**: link-state routing binnen een AS
- **BGP**: policy-gedreven routing tussen AS’en
- **mobile networking**: hosts veranderen van locatie maar willen bereikbaar blijven
- **ad-hoc networking**: zelfs de infrastructuur is dynamisch of afwezig

De grote rode draad is dat routing niet overal hetzelfde doel heeft:

- soms wil je technisch de beste route
- soms wil je beleid volgen
- soms wil je vooral robuust omgaan met mobiliteit
- soms wil je met minimale overhead überhaupt nog routes vinden

## Wat je echt moet kennen

### belangrijk voor examen

- Het verschil tussen **AS**, **intra-domain** en **inter-domain routing**
- Waarom **OSPF** een intra-domain link-stateprotocol is
- De rol van **HELLO**, flooding, sequence numbers en Dijkstra in OSPF
- Wat **areas** doen in OSPF en waarom **Area 0** belangrijk is
- Wat **ECMP** betekent
- Waarom **BGP** op **policy** focust in plaats van puur op shortest path
- Het verschil tussen **peering** en **transit**
- Waarom peering **niet transitief** is
- Wat een **AS-path** is in BGP
- De rol van **iBGP** binnen een AS
- Waarom mobiele hosts een **home agent** en **care-of address** gebruiken
- Wat **triangle routing** is
- Waarom network-layer mobility een **securityprobleem** heeft
- Waarom ad-hoc netwerken anders zijn dan klassieke netwerken
- Waarom **AODV** reactief is
- Hoe **ROUTE_REQUEST** en **ROUTE_REPLY** werken
- Waarom flooding in AODV geoptimaliseerd moet worden met TTL
- Wanneer **AODV beter is dan OSPF** (dynamische topologie, beperkte resources, weinig verkeer) en omgekeerd
- Hoe AODV het **count-to-infinity-probleem** oplost via **destination sequence numbers**
- De vier dimensies waarin **OSPF en BGP** verschillen: scope, algoritme, criterium en transport
- Waarom OSPF **connectionless** transport gebruikt en BGP **TCP**

## Korte eindintuitie

Als je dit hoofdstuk tot één gedachte reduceert:

routing is geen enkel trucje, maar een familie van oplossingen die elk aangepast zijn aan hun context, van stabiele backbone-netwerken tot mobiele en volledig infrastructuurloze netwerken.

Dat is precies de lijn die door deze slides loopt.
