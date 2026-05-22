# Deel 4 - Network 2: Congestion Control, QoS en Internetworking

In dit deel bouwen de slides verder op de network layer, maar nu ligt de focus veel meer op drie praktische problemen:

- hoe vermijden of beheersen we **congestie**?
- hoe kunnen we bepaalde **quality of service**-beloftes doen?
- hoe verbinden we **verschillende soorten netwerken** met elkaar?

Dat maakt dit hoofdstuk heel relevant, want hier zie je dat routing alleen niet genoeg is. Zelfs als je een pad kent, moet je nog altijd omgaan met overbelasting, verschillende soorten traffic en heterogene netwerken.

## De rol van de network layer bij congestie

Een belangrijk idee uit de slides is dat **hosts zelf throttlen** (= zendsnelheid verlagen), bijvoorbeeld via mechanismen in de transportlaag zoals TCP. Maar de network layer speelt wel een grote ondersteunende rol.

Die rol is drieledig:

- congestie zoveel mogelijk vermijden via routing
- congestie signaleren aan hosts zodat zij hun zendtempo verlagen
- in het slechtste geval load afwerpen, dus packets droppen

De slides zeggen ook expliciet dat het probleem niet opgelost raakt door gewoon grotere buffers te zetten. Dat klinkt intuïtief, maar werkt niet goed als wachttijden in queues te groot worden. Dan blijven packets heel lang hangen, en dat maakt het netwerk vaak nog instabieler.

### intuition

Meer buffers lijken even alsof je "meer plaats" maakt, maar in werkelijkheid kan je zo ook gewoon meer file maken.

## Grote categorieën van congestiecontrole

De cursus onderscheidt meerdere manieren om met congestie om te gaan.

### 1. Infrastructuur uitbreiden

Je kan natuurlijk meer capaciteit bouwen of redundante infrastructuur toevoegen. Dat helpt, maar dat gebeurt op een tijdschaal van maanden of langer. Dat is dus geen snelle reactie op actuele congestie.

### 2. Traffic-aware routing

Je kan routing slim maken zodat ze rekening houdt met actuele load en traffic wegstuurt van hotspots.

### 3. Admission control

Je kan nieuwe verbindingen weigeren als het netwerk of bepaalde routers al dicht bij hun capaciteit zitten.

### 4. Throttling

Je kan hosts laten vertragen zodra congestie wordt gedetecteerd.

### 5. Packet dropping

In het slechtste geval moet het netwerk gewoon packets droppen.

### belangrijk voor examen

Congestiecontrole is niet één mechanisme, maar een combinatie van:

- vermijden
- weigeren
- signaleren
- en uiteindelijk droppen

## Traffic-aware routing

In het vorige hoofdstuk kwamen kortstepadalgoritmes zoals Dijkstra aan bod. Daar gebruikten we vaste gewichten op links. Dat laat toe om aan te passen aan topologieveranderingen, maar niet aan actuele verkeersdrukte.

**Traffic-aware routing** probeert dat wel te doen. Het idee is eenvoudig:

- als een link of buur zwaar belast is
- stijgt de kost van die route
- en verschuift traffic automatisch naar minder drukke delen van het netwerk

Dus in plaats van enkel "de kortste" route te nemen, probeer je "de beste route gegeven de huidige load" te nemen.

### Het probleem van oscillatie

Op papier klinkt dat sterk, maar de slides benadrukken een groot probleem: **oscillatie**.

Wat gebeurt er dan?

- iedereen ziet dat route A te druk is
- iedereen verschuift naar route B
- plots wordt route B te druk
- dan verschuift iedereen terug

Je krijgt dus een slingerbeweging in plaats van stabiel gedrag.

Daarom zegt de cursus dat traffic-aware routing in het internet uiteindelijk niet echt gebruikt wordt in deze pure vorm. Als je te snel reageert, krijg je instabiliteit. Als je te traag reageert, verlies je veel van het voordeel.

### belangrijk voor examen

Traffic-aware routing is aantrekkelijk omdat ze load kan ontwijken, maar in de praktijk is **oscillation** het grote probleem.

## Admission control

Admission control betekent dat het netwerk soms beslist om **een nieuwe flow of verbinding niet toe te laten**.

Dat lijkt op een busy tone in telefonie: als het systeem vol zit, wordt een nieuwe verbinding niet aangenomen.

### Waarom vooral bij connection-oriented netwerken?

Omdat je daar een setupfase hebt. Tijdens die setup kan het netwerk nog zeggen:

- ja, we hebben capaciteit
- nee, we weigeren deze flow

In een pure datagramwereld is dat moeilijker, omdat verkeer niet eerst een expliciete circuitsetup doorloopt.

## Circuit admission control

Bij **circuit admission control** moet je inschatten of het netwerk genoeg ruimte heeft voor een nieuwe flow.

Dat is niet triviaal, want veel traffic is **bursty**:

- gemiddeld misschien laag
- maar met korte pieken die veel hoger liggen

Daarom moet je niet alleen kijken naar het gemiddelde verbruik, maar ook naar:

- pieken
- burstiness
- typische historische patronen van dat type traffic

De slides verwijzen hier al naar de **leaky bucket** als manier om zo’n trafficprofiel te beschrijven.

### intuition

Twee videoverbindingen van gemiddeld 1 Mbps zijn niet noodzakelijk hetzelfde als twee verbindingen die allebei af en toe tegelijk heel hard pieken.

## Router admission control

Admission control kan ook op routerniveau bekeken worden.

De slides beschrijven het idee dat een router die dicht tegen capaciteit zit, zichzelf kan verwijderen uit routinginformatie voor nieuwe circuits. Dan zullen nieuwe verbindingen automatisch andere paden zoeken.

Dus:

- bestaande verbindingen blijven
- nieuwe verbindingen worden weggeleid van overbelaste routers

Dat is een nette manier om congestie preventief te vermijden in circuit-georiënteerde contexten.

## Throttling

**Throttling** gebeurt uiteindelijk in de transportlaag, bijvoorbeeld via het congestion window van TCP. Maar de network layer moet de signalen aanleveren die zeggen: "vertraag nu".

In deze slides worden drie belangrijke mechanismen genoemd:

- **choke packets**
- **ECN**
- **RED**

Voor al die mechanismen moet je eerst op een of andere manier de huidige load inschatten.

## Hoe schat je load in?

De slides noemen drie mogelijke signalen:

- **link utilization**
- **packet loss**
- **queuing delay**

### Link utilization

Geeft snel feedback, maar kan bursty traffic moeilijk correct vatten.

### Packet loss

Houdt rekening met alle traffic, maar komt meestal **te laat**. Als je al verlies ziet, is congestie vaak al goed bezig.

### Queuing delay

Dat is vaak een beter vroeg signaal, omdat het laat zien dat buffers beginnen vollopen nog voor er effectief veel verlies optreedt.

Daarom gebruikt men vaak een **exponentially weighted moving average** op queuing delay om load te schatten.

### belangrijk voor examen

Packet loss is een bruikbaar congestiesignaal, maar meestal een **laat** signaal. **Queuing delay** geeft vaak sneller nuttige feedback.

## Choke packets

Een **choke packet** is een expliciete melding van een congested router naar de zender.

Het idee:

- een router merkt congestie
- stuurt een datagram terug naar de zender
- die zender verlaagt dan zijn verzendtempo

De slides vermelden ook dat het packet gemarkeerd wordt zodat er niet telkens opnieuw extra choke packets worden gegenereerd voor hetzelfde probleem.

### pluspunt

Heel direct en expliciet.

### nadeel

Extra control-overhead, want je stuurt aparte meldingspackets rond.

## Explicit Congestion Notification (ECN)

**ECN** is eleganter dan choke packets.

Daar markeert een congested router niet een apart controledatagram, maar markeert hij gewone packets die toch al onderweg zijn. Die bereiken de ontvanger, en die ontvanger laat de zender dan weten dat er vertraagd moet worden.

Dus het pad is:

router  
\--> markeert packet  
\--> ontvanger ziet markering  
\--> ontvanger meldt dit aan zender  
\--> zender throttlet

### vergelijking met choke packets

- **minder overhead**
- maar ook **tragere reactie**, omdat de informatie eerst via de ontvanger moet terugkeren

### belangrijk voor examen

ECN verlaagt de overhead ten opzichte van choke packets, maar de reactie komt indirecter en dus vaak iets later.

## Random Early Detection (RED)

**RED** vertrekt van een slim maar wat contra-intuïtief idee:

als packet loss een goed congestiesignaal is, waarom zouden we dan niet **vroeger al af en toe bewust packets droppen** zodra we zien dat delay en queue-opbouw stijgen?

Dat geeft twee voordelen:

- packet loss wordt een vroegere en betrouwbaardere waarschuwing
- transportlagen zoals TCP reageren sneller

Dus RED wacht niet tot de queue helemaal vol is. Het begint al eerder met willekeurig droppen zodra congestie nadert.

### Waarom helpt random dropping vooral tegen snelle senders?

Omdat snelle senders meer packets in de queue hebben. Als je willekeurig uit die queue dropt, is de kans groter dat je packets van die agressievere flow raakt.

Dat is slim, want je hoeft niet per packet perfect bij te houden wie exact de grootste boosdoener is.

### belangrijk voor examen

RED dropt **vroeg en willekeurig** om congestie sneller zichtbaar te maken voor transportprotocollen, en treft daardoor relatief vaker de snelste senders.

## Quality of Service (QoS)

Na congestiecontrole verschuift de focus naar **quality of service**.

Het ideale antwoord op QoS zou gewoon zijn: overal overprovisioning. Met andere woorden: altijd meer capaciteit voorzien dan nodig.

Maar dat is niet realistisch omdat:

- het duur is
- piekbelasting veel hoger kan zijn dan de gemiddelde belasting
- traffic snel kan veranderen

Dus moeten we QoS slimmer organiseren met beperkte middelen.

## QoS is een probleem in drie delen

De slides formuleren drie bouwstenen:

1. **traffic shaping**
2. **packet scheduling**
3. **admission control**

Dat is een mooie structuur om te onthouden.

### Traffic shaping

Beschrijf en controleer het trafficpatroon.

### Packet scheduling

Bepaal hoe routerresources verdeeld worden tussen flows.

### Admission control

Laat niet meer flows toe dan wat de beloofde QoS nog haalbaar maakt.

### belangrijk voor examen

QoS draait in deze cursus rond drie zaken:

- traffic shaping
- scheduling
- admission control

## Traffic flows

Een **traffic flow** is gewoon de stroom data van bron naar bestemming.

Dat kan je in een connection-oriented context vaak zien als alle packets van een verbinding. In een connectionless context is het eerder alle packets tussen bepaalde processen of endpoints.

Voor zo’n flow willen we garanties of minstens afspraken maken over:

- bandbreedte
- delay
- jitter
- loss

## Traffic shaping

Veel traffic is bursty: het gemiddelde ligt laag, maar korte pieken kunnen veel hoger liggen.

Om daarmee om te gaan, wordt het toegelaten trafficprofiel vaak vastgelegd in een **SLA** tussen klant en provider.

Daarin staat in essentie:

- dit soort traffic mag je sturen
- buiten dat profiel kunnen we traffic policed behandelen

De vraag wordt dan: hoe beschrijf je zo’n patroon op een eenvoudige manier?

## De leaky bucket

Het klassieke model is de **leaky bucket**.

De metafoor:

- je hebt een emmer met capaciteit **B**
- onderaan zit een klein gat
- water loopt eruit aan vaste snelheid **R**
- als de emmer leeg is, komt er niets meer uit
- als de emmer vol zit en er komt nog water bij, dan gaat dat verloren

Omgezet naar netwerken:

- **B** bepaalt hoeveel burst je tijdelijk kan opvangen
- **R** bepaalt de lange-termijn uitstroomsnelheid

### Wat doet dit met traffic?

Zolang er nog plaats is in de bucket, kan traffic even sneller binnenkomen dan de gemiddelde afvoersnelheid. Maar als de bucket vol geraakt, wordt de traffic effectief afgevlakt tot snelheid **R**.

Dus:

- korte bursts mogen
- permanente overbelasting niet

## Leaky bucket in actie

De slides tonen een voorbeeld met een host die heel snel kan zenden, maar gemiddeld minder traffic hoeft door te sturen.

Het belangrijke inzicht is dat de parameters **R** en **B** bepalen hoe hard je de traffic afvlakt:

- grote **B** laat meer burst toe
- kleine **B** maakt de shaping strenger
- **B = 0** maakt de output volledig vlak

Dus met andere woorden:

- **unsmoothed**: veel pieken blijven zichtbaar
- **partially smoothed**: pieken worden wat afgevlakt
- **totally smoothed**: bijna constante output

### Hosts versus routers

De slides maken nog een praktisch onderscheid:

- op **hosts** wordt extra traffic meestal gequeued tot die verzonden mag worden
- op **routers** wordt extra traffic eerder gediscard

Dat verschil is logisch: routers zitten al onder druk en moeten niet eindeloos verkeer van anderen blijven bufferen.

### belangrijk voor examen

Bij een leaky bucket beschrijven **R** en **B** samen hoeveel gemiddelde rate en hoeveel burstiness toegelaten zijn.

## Packet scheduling

Zelfs als traffic mooi beschreven is, blijft de vraag: wie mag wanneer over de link?

Routers hebben beperkte resources:

- bandbreedte
- buffer space
- CPU

Scheduling bepaalt hoe die resources verdeeld worden tussen flows.

## FIFO

De simpelste aanpak is **FIFO**: first in, first out.

Packets worden gewoon in volgorde doorgelaten, en als de queue vol is, worden nieuwe packets gedropt aan de staart van de queue: **tail drop**.

Dat is eenvoudig, maar erg gevoelig voor misbruik:

- agressieve senders vullen de queue
- andere flows lopen vertraging of verlies op

Dus FIFO is simpel, maar niet eerlijk.

## Fair Queuing (FQ)

Bij **fair queuing** krijgt elke flow zijn eigen queue. Als de lijn vrij is, neemt de router beurtelings een packet uit elke queue.

Dat voorkomt dat één agressieve flow alles blokkeert.

### maar er is nog een trucje

De slides stellen een goeie vraag: hoe zou je dit als transportontwikkelaar proberen te omzeilen?

Antwoord: door **grote packets** te sturen.

Als elke flow per beurt één packet mag sturen, dan wint een flow met grotere packets alsnog meer bytes per ronde.

Dus fair queuing op packetniveau is niet automatisch perfect eerlijk op byte-niveau.

## Weighted Fair Queuing (WFQ)

In de praktijk wil je flows soms ook niet allemaal identiek behandelen.

Met **weighted fair queuing** krijgt elke flow een gewicht. Dat gewicht bepaalt hoeveel bytes per ronde voor die flow mogen worden afgevoerd.

Zo kan je bijvoorbeeld:

- realtime traffic meer geven
- achtergrondtraffic minder geven

Dat is een handig mechanisme om QoS-prioriteiten concreet te maken.

### belangrijk voor examen

FQ probeert flows gelijk te behandelen, terwijl **WFQ** expliciet verschillende gewichten en dus verschillende service-aandelen toelaat.

## Admission control in QoS-context

Admission control komt hier terug, maar nu in QoS-termen.

Het netwerk beslist of een nieuwe flow nog aanvaardbaar is gegeven alle bestaande reservaties en beloften. Voor elke aanvaarde flow moet er langs het hele pad genoeg capaciteit zijn.

Dat betekent:

- alle routers op het pad moeten mee kunnen
- één bottleneckrouter kan de hele QoS-belofte breken

Als een flow op één pad niet aanvaard kan worden, kan eventueel een andere route geprobeerd worden.

## Wat weet de applicatie?

De applicatie hoeft normaal niet zelf met buffers, CPU en routerinterne details bezig te zijn.

Wel moet ze vaak duidelijk kunnen maken wat voor soort noden ze heeft:

- sommige apps hebben harde eisen
- andere hebben wat speelruimte
- sommige apps kunnen terugschalen als de QoS slechter wordt

De slides geven als voorbeeld video die bijvoorbeeld van 30 fps naar 24 fps kan dalen.

Dat is belangrijk, want QoS is niet altijd alles-of-niets. Soms is er ruimte voor **graceful degradation**.

## Internetworking en heterogeniteit

Het laatste grote thema is **internetworking**: verschillende soorten netwerken met elkaar verbinden op grote schaal.

Waarom verschillen netwerken eigenlijk?

- andere fysieke omstandigheden, zoals Ethernet versus satelliet
- hergebruik van bestaande infrastructuur, zoals telefoonlijnen, kabel-tv of power lines

De waarde van netwerken stijgt ook met schaal, dus het is enorm belangrijk dat die verschillende netwerken toch samenwerken.

## Hoe kunnen netwerken verschillen?

De precieze details op de slide zijn minder belangrijk dan het grote beeld: netwerken kunnen verschillen in bijna alles wat operationeel relevant is, zoals:

- aangeboden service
- adressering
- maximale packetgrootte
- routing
- kwaliteit van service
- beveiliging

Dus internetworking is lastig omdat je niet gewoon twee identieke systemen aan elkaar hangt.

## Twee benaderingen van internetworking

De slides noemen:

- **translation**
- **indirection**

### Translation

Een apparaat zet pakketten van het ene type rechtstreeks om naar het andere type.

### Indirection

Hogere lagen gebruiken een gemeenschappelijke tussentaal of gemeenschappelijke laag die per netwerk anders gemapt wordt.

Het klassieke succesvoorbeeld van die tweede aanpak is **IP**.

### belangrijk voor examen

IP is een voorbeeld van **indirection**: verschillende onderliggende netwerken worden samengebracht door één gemeenschappelijke network layer.

## De rol van routers

Routers zitten precies op de grenzen tussen netwerken. Daarom moeten ze extra werk doen om end-to-end routing mogelijk te maken ondanks verschillen tussen de onderliggende netwerken.

Dat is ook meteen het verschil met switches.

## Routers versus switches

### Switches

Werken op **laag 2**, dus de datalinklaag. Ze denken in termen van **frames** en houden zich niet bezig met IP-routingvragen.

### Routers

Werken op **laag 3**, dus de network layer. Ze denken in termen van **packets** en halen netwerklaaginfo uit pakketten om routingbeslissingen te nemen.

### belangrijk voor examen

Switches verwerken **frames op layer 2**, routers verwerken **packets op layer 3**.

## Tunneling

**Tunneling** gebruik je wanneer bron- en bestemmingsnetwerk eigenlijk compatibel zijn, maar er tussenin een ander type netwerk zit.

Het voorbeeld in de slides is heel klassiek:

- twee IPv6-netwerken
- verbonden over een tussenliggend IPv4-netwerk

Omdat directe vertaling niet altijd mogelijk is, kan je het oorspronkelijke packet inkapselen in een packet van het tussennetwerk. Voor de hogere lagen voelt dat alsof de twee uiteinden toch rechtstreeks met elkaar verbonden zijn.

### waarom niet gewoon vertalen?

Omdat vertaling fundamentele grenzen heeft. Een 128-bit IPv6-adres past bijvoorbeeld niet zomaar in een 32-bit IPv4-adres.

### multiprotocol routers

De routers aan zo’n tunnel moeten beide protocollen begrijpen. Dat zijn **multi-protocol routers**.

Voor de hogere lagen telt de volledige tunnel vaak als één enkele hop.

### intuition

Een tunnel is alsof je een volledig pakket in een beschermende buitenverpakking stopt om het door een vreemd tussengebied te krijgen, en het aan de andere kant weer uitpakt.

## Routinguitdagingen bij internetworking

De slides noemen vier klassieke moeilijkheden:

1. verschillende routingalgoritmes
2. verschillende metrics voor "beste pad"
3. verborgen interne paden door hiërarchie
4. extreme schaal

Dat toont dat internetworking niet enkel een adresseringsprobleem is. Ook de betekenis van routinginformatie kan per netwerk verschillen.

## Fragmentatie

Een ander groot heterogeniteitsprobleem is **MTU**. Niet elk netwerk kan even grote packets aan.

Als een packet te groot is voor een bepaalde link of netwerktechnologie, heb je twee grote strategieen:

- **transparante fragmentatie**
- **niet-transparante fragmentatie**

### Transparante fragmentatie

Routers fragmenteren en assembleren opnieuw na de smallere hop.

Voordeel:

- makkelijker voor hosts

Nadeel:

- meer werk voor routers
- en routers zijn vaak al zwaar belast

### Niet-transparante fragmentatie

Routers fragmenteren eventueel, maar assembleren niet opnieuw. De hosts moeten dat oplossen.

Voordeel:

- minder werk voor routers

Nadeel:

- meer complexiteit voor hosts

## Waarom fragmentatie liefst vermijden?

De slides zijn daar heel duidelijk in: fragmentatie is duur in beide modellen en wordt best vermeden.

Daarom moet de zender liefst de **path MTU** kennen: de grootste packetgrootte die langs het volledige pad past zonder fragmentatie.

## Path MTU discovery

Bij **path MTU discovery** stuurt de zender packets uit en leert hij onderweg wat de kleinste bruikbare MTU op het pad is.

Als een router een te groot packet ontvangt, dan:

- dropt hij dat packet
- en stuurt hij een foutmelding terug

De zender verlaagt dan zijn packetgrootte.

### Is dit compatibel met gewone fragmentatie?

Volgens de slides: nee.

Waarom niet? Omdat fragmentatie het probleem juist zou verbergen. Als routers alles gewoon zouden fragmenteren, dan zou de zender nooit echt leren dat zijn packets te groot zijn voor het pad.

Daarom moet bij path MTU discovery de **DNF/DF-flag** gezet worden, zodat routers het packet niet fragmenteren maar echt terugmelden dat het te groot was.

### belangrijk voor examen

Path MTU discovery werkt alleen goed als routers **niet fragmenteren**, zodat de zender leert welke packetgrootte echt past op het volledige pad.

## Samenvatting van het hoofdstuk

Dit hoofdstuk laat zien dat de network layer meer doet dan enkel routes berekenen.

Ze moet ook:

- congestie helpen vermijden en signaleren
- QoS praktisch ondersteunen
- heterogene netwerken met elkaar laten samenwerken

De drie grote blokken zijn:

- **congestion control**
- **QoS**
- **internetworking**

En in elk blok zie je opnieuw hetzelfde patroon: de theorie is mooi, maar de echte uitdaging zit in schaal, trade-offs en praktische beperkingen.

## Wat je echt moet kennen

### belangrijk voor examen

- De rol van de network layer bij congestie
- De verschillende benaderingen van congestiecontrole
- Waarom **traffic-aware routing** moeilijk is door oscillatie
- Wat **admission control** doet
- Het verschil tussen **choke packets**, **ECN** en **RED**
- Waarom **queuing delay** vaak een nuttiger vroeg signaal is dan packet loss
- De drie bouwstenen van **QoS**
- Wat een **traffic flow** is
- Hoe de **leaky bucket** werkt en wat **R** en **B** betekenen
- Het verschil tussen **FIFO**, **FQ** en **WFQ**
- Waarom admission control nodig is voor QoS-garanties
- Het verschil tussen **translation** en **indirection** bij internetworking
- Het verschil tussen **switches** en **routers**
- Wat **tunneling** is en waarom het nodig kan zijn
- Het verschil tussen transparante en niet-transparante fragmentatie
- Waarom **path MTU discovery** fragmentatie liever vermijdt

## Korte eindintuitie

Als je dit hoofdstuk tot één kernidee reduceert:

de network layer moet niet alleen packets verplaatsen, maar ook slim omgaan met schaarste, kwaliteitsverschillen en incompatibele netwerken.

Dat is precies wat deze slides proberen op te bouwen.
