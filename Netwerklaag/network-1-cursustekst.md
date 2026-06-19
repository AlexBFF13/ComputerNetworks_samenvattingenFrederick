# Deel 3 - Network 1: Theory

In dit deel verschuiven we van de transportlaag naar de **network layer**. Die laag zit lager in de stack, maar is cruciaal omdat ze ervoor moet zorgen dat data over een volledig netwerk geraakt, vaak over meerdere tussenliggende routers heen.

De grote vragen in deze slides zijn:

- welke service moet de network layer aanbieden aan de transportlaag?
- doen we dat connectionless of connection-oriented?
- hoe kiezen routers goede paden?
- wat gebeurt er als het netwerk verandert?
- hoe houden we routing schaalbaar?

Dat maakt dit hoofdstuk belangrijk, want hier begint het echte "hoe geraakt een packet door een netwerk?"-denken.

## De rol van de network layer

De network layer is de laag die zich als eerste echt met **end-to-end delivery** bezighoudt. De datalinklaag kijkt vooral naar een enkele link, maar de network layer moet een packet van bron naar bestemming krijgen over mogelijk veel hops.

Dat betekent dat de network layer moet omgaan met:

- meerdere routers
- meerdere mogelijke paden
- verschillende netwerktechnologieen
- veranderende belasting in het netwerk

De transportlaag zou die complexiteit normaal niet rechtstreeks moeten zien. Dat is juist een van de doelen van de network layer: de rommel van het netwerk verbergen achter een bruikbare dienst.

## Store-and-forward packet switching

Routers werken hier volgens het **store-and-forward**-principe.

Dat betekent:

- een router ontvangt een packet
- bewaart het tijdelijk
- controleert het
- en stuurt het daarna door naar de volgende router

Dus een packet wordt niet in een keer door het hele netwerk "doorgeduwd". Het springt stap voor stap vooruit, router per router.

### intuition

Je kan het zien als een keten van tussenstations. Elk tussenstation kijkt eerst of het pakket correct is aangekomen, en beslist dan pas waar het daarna naartoe moet.

## Welke service verwacht de transportlaag?

De slides geven drie algemene doelen voor de service van de network layer:

- ze moet onafhankelijk zijn van de gebruikte routertechnologie
- de transportlaag moet niet moeten weten hoeveel routers er zijn of hoe ze verbonden zijn
- er moet een uniforme adressering zichtbaar zijn naar boven

Dat klinkt redelijk neutraal, maar daarachter zitten twee heel verschillende filosofieen:

- **connectionless**
- **connection-oriented**

Die tegenstelling komt in deze slides voortdurend terug.

## Connectionless network layer

Bij een **connectionless** aanpak worden packets individueel het netwerk in gestuurd. Elk packet wordt apart behandeld.

De hoofdideeën zijn:

- geen connection setup nodig
- elk packet draagt het eindadres
- elk packet kan onafhankelijk gerouteerd worden

Op network-layerniveau noemen we zulke eenheden meestal **datagrams**.

### Wat betekent dat concreet?

Als host H1 vier packets naar H2 stuurt, dan is er geen garantie dat al die packets exact hetzelfde pad volgen. De routers beslissen telkens opnieuw wat op dat moment de beste uitgaande link is.

Dus packet 1, 2 en 3 kunnen bijvoorbeeld via:

`A -> C -> E -> F`

gaan, terwijl packet 4 later via:

`A -> B -> D -> E -> F`

gaat omdat de netwerktoestand veranderd is.

### waarom is dit interessant?

Omdat het netwerk daardoor flexibel wordt. Als een link druk bezet is of wegvalt, kunnen nieuwe packets een ander pad nemen zonder eerst een circuit te moeten heropbouwen.

### IP als voorbeeld

Het klassieke voorbeeld van een connectionless network-layerprotocol is **IP**.

### belangrijke begrippen

- **datagram**: een individueel packet op network-layerniveau
- **connectionless**: geen setup, elk packet apart behandeld

### belangrijk voor examen

Bij connectionless routing draagt elk packet het **eindadres**, en routers mogen verschillende packets van dezelfde flow langs **verschillende routes** sturen.

## Connection-oriented network layer

Bij een **connection-oriented** aanpak werkt men met **virtual circuits**.

Daar is het idee anders:

- eerst wordt een circuit berekend bij de setup
- daarna volgen alle packets datzelfde circuit
- wanneer de verbinding stopt, verdwijnt ook dat circuit

In plaats van een volledig eindadres mee te dragen, draagt elk packet dan vooral een **circuit identifier** of label.

### Wat verandert er voor routers?

Routers moeten niet telkens opnieuw volledig naar het eindadres kijken. Ze moeten vooral weten:

- welk label komt hier binnen?
- naar welke uitgaande lijn hoort dat?
- welk nieuw label hoort daar eventueel bij?

Dat laatste heet **label switching**.

### waarom kan het label veranderen?

Omdat labels meestal alleen **lokaal betekenis** hebben. Een binnenkomend label op router A moet dus niet hetzelfde zijn als het uitgaande label dat A gebruikt richting volgende router.

Dat is precies waarom in het voorbeeld het uitgaande label op A anders is dan het inkomende label: labels zijn niet globaal uniek, maar lokaal afgesproken.

### MPLS als voorbeeld

In de slides wordt **MPLS** gegeven als voorbeeld van een connection-oriented aanpak die effectief gebruikt wordt op het internet.

## Virtual circuits versus datagrams

De slides geven argumenten voor beide modellen. Het is belangrijk dat je niet alleen de definities kent, maar ook **waarom** iemand voor het ene of het andere model zou kiezen.

### argumenten voor virtual circuits

- routing kan eenvoudiger zijn
- routing tables kunnen kleiner zijn
- labels zijn kleiner dan volledige adressen
- resources kunnen al bij setup gereserveerd worden
- QoS is makkelijker te ondersteunen

### argumenten voor datagrams

- geen setupfase nodig
- sneller om te beginnen
- robuuster bij routercrashes
- je zit minder vast aan eerder vastgelegde paden

### intuition

Virtual circuits zijn meer "we spreken eerst een weg af".  
Datagrams zijn meer "we beslissen packet per packet".

### belangrijk voor examen

Je moet niet alleen het verschil tussen connectionless en connection-oriented kennen, maar ook de **trade-offs**:

- flexibiliteit en robuustheid bij datagrams
- eenvoud en resource control bij virtual circuits

## Routing: wat probeert een routingalgoritme te doen?

Een routingalgoritme bepaalt hoe packets door het netwerk moeten worden doorgestuurd. In een virtual-circuitnetwerk gebeurt dat vooral bij setup. In een datagramnetwerk is routing een doorlopend proces.

Een goed routingalgoritme wil niet zomaar een pad vinden, maar een pad dat ook bruikbaar is op schaal.

### Gewenste eigenschappen

De slides noemen:

- **correctness**: routes naar alle bestemmingen vinden
- **simplicity**: snel genoeg blijven werken
- **robustness**: fouten in hardware of software overleven
- **stability**: convergeren naar een gemeenschappelijke toestand
- **fairness**: gebruikers eerlijk behandelen
- **efficiency**: netwerkresources goed benutten

Die doelen botsen soms met elkaar. Een heel reactief algoritme kan bijvoorbeeld snel aanpassen, maar ook instabiel worden.

## Adaptive en non-adaptive routing

Een eerste onderscheid is tussen:

- **non-adaptive** algoritmes
- **adaptive** algoritmes

### Non-adaptive

Die houden geen rekening met de actuele toestand van het netwerk. Routes kunnen vooraf statisch ingesteld worden.

### Adaptive

Die passen hun keuzes aan op basis van:

- veranderende topologie
- veranderende traffic load

Dat kan lokaal of globaal gebeuren, en periodiek of event-driven.

Voor het internet zijn adaptive algoritmes veel belangrijker, omdat het netwerk te groot en te veranderlijk is voor puur statische routing.

## Het optimality principle

Een fundamenteel idee uit routingtheorie is het **optimality principle**.

Dat zegt:

Als router J op het optimale pad ligt van I naar K, dan moet het optimale pad van J naar K verder op datzelfde pad liggen.

Waarom is dat nuttig? Omdat het betekent dat optimale routes intern consistent zijn. Je kan optimale paden dus opbouwen uit subpaden die zelf ook optimaal zijn.

Dat idee zit onder veel kortstepadalgoritmes.

## Sink tree

De verzameling van optimale routes van alle bronnen naar een bepaalde bestemming vormt een **sink tree** met die bestemming als wortel.

Dat is een mooi conceptueel beeld:

- alle beste paden naar een bestemming
- vormen samen een boomstructuur
- zonder lussen

Routingalgoritmes proberen in essentie die optimale structuur te ontdekken.

## Kortstepadideeën

Een "kortste weg" hoeft niet altijd letterlijk de fysiek kortste weg te zijn.

De slides geven twee mogelijke definities:

- **fewest hop**: zo weinig mogelijk hops
- **lowest cost**: minimale totale kost volgens een gekozen metric, zoals delay, afstand of andere kosten

Dus "kortste pad" betekent eigenlijk: kortste volgens de metric die jij belangrijk maakt.

## Dijkstra's algoritme

Dijkstra is het klassieke algoritme om vanaf een bron de kortste paden naar alle andere nodes te vinden.

De basislogica is:

1. start aan bronnode N
2. geef alle andere nodes eerst een oneindige kost
3. geef directe buren een voorlopige kost
4. kies telkens de node met de laagste voorlopige kost
5. maak die permanent
6. update dan de kosten van zijn buren als je via die node goedkoper geraakt

Dat proces herhaal je tot alle nodes een definitieve beste kost hebben.

### intuition

Dijkstra bouwt de kortstepadboom laag per laag uit, telkens beginnend met wat op dit moment het goedkoopst zeker geweten is.

### belangrijk voor examen

Bij Dijkstra is het cruciale idee dat je telkens de **laagste voorlopige kost permanent maakt** en daarna de aangrenzende paden herberekent.

## Distance vector routing

Distance vector routing is een dynamische aanpak waarbij elke router een tabel bijhoudt met voor elke bestemming:

- de geschatte afstand
- de beste volgende hop

Die tabellen worden uitgewisseld met buren. Zo leert elke router geleidelijk betere routes kennen, zonder dat iemand een volledig globaal netwerkbeeld hoeft te hebben.

Dat maakt het conceptueel elegant en gedistribueerd.

## Bellman-Ford

Het bekendste distance-vectoralgoritme is **Bellman-Ford**.

In de slides werkt het zo:

- elke router meet eerst de afstand naar zijn directe buren
- daarna wisselt hij periodiek routing tables uit met die buren
- na ontvangst van een buurttabel berekent hij voor elke bestemming of een route via die buur goedkoper is

Het kernidee is dus:

`kost naar bestemming via buur = kost naar buur + schatting van buur naar bestemming`

Dan kies je de buur die de laagste totale kost oplevert.

### waarom werkt dit?

Omdat elke router beetje bij beetje de kennis van zijn buren gebruikt om een beter beeld te krijgen van verder gelegen bestemmingen.

### belangrijk voor examen

Bellman-Ford gebruikt alleen **lokale kennis van buren** en periodieke tabeluitwisseling om stap voor stap betere paden te leren.

## Het count-to-infinity-probleem

Dit is de grote zwakte van distance vector routing.

### Goed nieuws reist snel

Als een nieuwe route beschikbaar komt, verspreidt die informatie zich vaak vrij snel. Routers nemen de nieuwe korte route over en na een paar uitwisselingen weet iedereen ervan.

### Slecht nieuws reist traag

Als een bestemming onbereikbaar wordt, kan het misgaan. Stel dat router B denkt dat C nog een pad naar A heeft, terwijl C eigenlijk op zijn beurt vertrouwde op B. Dan krijg je een lus in de redenering:

- B denkt: "via C raak ik nog aan A"
- C denkt: "via B raak ik nog aan A"

Niemand beseft dat hij eigenlijk indirect naar zichzelf zit te kijken.

Daarom stijgt de geschatte afstand dan langzaam:

- 3
- 4
- 5
- 6

enzovoort, tot "infinity".

### kernprobleem

Geen router weet in dat model of hij **zelf** al in het voorgestelde pad zit.

### Concrete illustratie

Stel drie routers A, B en C in lijn: A--B--C. Router A is verbonden met een bestemming X via kost 1. B leert: "via A naar X, kost 2". C leert: "via B naar X, kost 3".

Nu valt de link A--X weg:

1. A zet zijn kost naar X op oneindig
2. Maar B adverteert nog "naar X, kost 2" (verouderde info)
3. A denkt: "via B kost 3" en neemt dat aan
4. B krijgt update van A: "naar X, kost 3", dus B update naar 4
5. A krijgt update: kost 4 via B, update naar 5
6. Dit herhaalt tot de maximum metric bereikt wordt

### Klassieke mitigaties

| Techniek | Werking | Lost het op? |
| --- | --- | --- |
| **Split horizon** | Adverteer een route niet terug naar de buur van wie je hem geleerd hebt | Niet volledig (faalt bij 3+ nodes in lus) |
| **Poison reverse** | Adverteer de route terug met kost = oneindig | Beter, maar faalt ook bij grotere lussen |
| **Maximum metric** | Definieer een maximum (bv. RIP: 16 = onbereikbaar) | Beperkt de duur, maar lost de lus zelf niet op |
| **Hold-down timers** | Na verlies, weiger updates voor die bestemming gedurende een timer | Vertraagt convergentie |

Geen van deze technieken lost count-to-infinity volledig op. Dat is fundamenteel de reden waarom link-state routing en AODV (met sequence numbers) ontwikkeld werden.

### Hoe AODV count-to-infinity oplost

AODV gebruikt **destination sequence numbers**. Elke route draagt een sequencenummer mee dat door de bestemming zelf wordt verhoogd. Een route met een **hoger** sequencenummer is altijd verser. Bij een linkuitval stuurt de bestemming (of de node die de uitval detecteert) een RERR met een verhoogd sequencenummer. Nodes met een verouderd (lager) sequencenummer verwerpen hun oude route **onmiddellijk** en starten geen lus meer.

Dit werkt omdat sequence numbers een **globale tijdsordening** introduceren die distance vector normaal niet heeft.

### belangrijk voor examen

Het count-to-infinity-probleem ontstaat omdat distance vector routers enkel **lokale informatie** zien en niet kunnen detecteren dat een vermeend pad eigenlijk cirkelvormig is. AODV lost dit op via **destination sequence numbers** die verouderde routes onmiddellijk herkenbaar maken.

## Waarom link-state routing opkwam

Vanwege dat probleem stapte men historisch over van distance vector naar **link-state routing** voor grotere en snellere netwerken.

Het idee is anders:

- bij distance vector deelt een router zijn beste schattingen mee
- bij link state deelt een router de toestand van zijn eigen links mee

Dus routers sturen niet gewoon "mijn beste route is X", maar eerder:

"dit zijn mijn buren en dit zijn de kosten van mijn verbindingen naar hen"

Als elke router dat weet van alle andere routers, dan kan elke router zelf de volledige graaf reconstrueren en daarop Dijkstra uitvoeren.

## Het proces van link-state routing

De slides geven vijf stappen:

1. buren ontdekken
2. linkkosten bepalen
3. een link-statepakket maken
4. dat pakket verspreiden
5. op basis van alles nieuwe routes berekenen

Dat is een heel belangrijk stuk van dit hoofdstuk.

## Stap 1: buren ontdekken

Bij het opstarten stuurt een router bijvoorbeeld een **HELLO** over elke point-to-pointlijn. Buren antwoorden dan met hun adres.

Bij broadcastmedia zoals Ethernet wordt dit iets lastiger, omdat daar veel routers tegelijk aan hetzelfde medium kunnen hangen. Dat vergroot de graaf. Daarom modelleert men soms het broadcastmedium als een soort artificiele node.

## Stap 2: linkkosten bepalen

Een link kan op verschillende manieren een kost krijgen:

- fysieke afstand
- financiele kost
- signaalsterkte
- delay
- bandbreedte

In internetcontext gebruikt men vaak een kost die samenhangt met bandbreedte of delay.

Dus ook hier weer: "beste route" hangt af van **welke metric je gekozen hebt**.

## Stap 3: een link-statepacket construeren

Zo'n packet bevat typisch:

- het adres van de zender
- een sequence number
- een age
- een lijst van kosten naar alle buren

Sequence numbers en age zijn belangrijk, omdat routers anders oude of dubbele informatie niet goed zouden kunnen herkennen.

## Stap 4: distributie via flooding

De link-statepackets worden verspreid via **flooding**.

Flooding betekent:

- als een router zo'n packet ontvangt
- stuurt hij het door op alle links
- behalve de link waarlangs het binnenkwam

Dat is heel robuust, maar potentieel ook heel duur qua overhead.

## Hoe beperk je flooding?

Zonder rem zou flooding eindeloos blijven rondgaan. Daarom gebruikt men onder meer:

- een **TTL** of hop counter
- **sequence numbers**
- een **cache** van al geziene packets

### TTL

Bij elke hop gaat de teller omlaag. Zodra die 0 wordt, stopt het packet.

### Duplicate suppression

Als een router een packet met een uniek ID of sequence number al eerder zag, dan gooit hij de kopie weg.

### Age

Als een packet te oud is, wordt het ook genegeerd. Dat helpt om oude toestand na crashes op te ruimen.

### belangrijk voor examen

Flooding is robuust, maar alleen bruikbaar als je mechanismen hebt om duplicaten en eindeloze circulatie te stoppen, zoals **TTL**, **sequence numbers** en **age**.

## Stap 5: nieuwe routes berekenen

Zodra een router alle link-stateinformatie ontvangen heeft, kan hij de volledige netwerkgraaf reconstrueren.

Daarna voert hij **Dijkstra** uit op die graaf om de beste routes naar alle bestemmingen te vinden.

Belangrijk detail uit de slides: een link wordt meestal in beide richtingen apart voorgesteld, omdat kosten asymmetrisch kunnen zijn.

Dus de kost van `A -> B` hoeft niet dezelfde te zijn als die van `B -> A`.

## Link state versus distance vector

Link state lost het count-to-infinity-probleem veel beter op, maar het is niet gratis.

### voordelen van link state

- beter zicht op de volledige topologie
- geen klassiek count-to-infinity-probleem
- vaak snellere en correctere convergentie

### nadelen van link state

- meer computationele kost
- meer opslag nodig
- meer control traffic

Dat maakt schaalbaarheid een nieuwe uitdaging. En precies daarom komt hierna **hierarchical routing**.

## Hiërarchische routing

Op internetschaal is het onrealistisch dat elke router een entry heeft voor elke andere router.

Daarom gebruikt men een hiërarchie:

- routers worden gegroepeerd in regio's
- een router kent de interne structuur van zijn eigen regio goed
- voor andere regio's kent hij vooral de toegangspunten

Dat reduceert routing-tablegrootte enorm.

### Maar wat is de prijs?

Je verliest detailkennis over verre regio's. Daardoor kan het gebeuren dat je niet altijd de absoluut beste ingang kiest voor een verre bestemming.

In de slides zie je dat 1C bijvoorbeeld niet noodzakelijk de beste toegang is voor 5C.

Dus hiërarchie geeft:

- kleinere tabellen
- minder state
- betere schaalbaarheid

maar soms ook:

- langere paden
- minder optimale routekeuze

### belangrijk voor examen

Hiërarchische routing is vooral een **schaalbaarheidsoplossing**: kleinere routing tables in ruil voor soms wat langere paden.

## Delivery models

De slides eindigen met verschillende modellen voor aflevering:

- **unicast**
- **broadcast**
- **multicast**

### Unicast

Van een bron naar een bestemming. Dit is de gewone één-op-ééncommunicatie.

### Broadcast

Van een bron naar alle hosts of routers in het relevante bereik.

### Multicast

Van een bron naar een specifieke groep ontvangers.

Dat onderscheid is belangrijk, omdat de manier waarop je packets efficiënt verspreidt sterk afhangt van welk model je gebruikt.

## Broadcast: eenvoudige maar dure aanpakken

### Naive broadcast

De bron stuurt gewoon een aparte unicast naar elke ontvanger.

Dat is erg inefficiënt:

- routers zien vaak veel dubbele traffic
- de zender moet alle bestemmingen kennen

### Multi-destination broadcast routing

Een slimmere variant is een pakket dat meerdere bestemmingen bevat. Routers bekijken dan de lijst en maken kopieën waar nodig.

Dat vermindert wat redundantie, maar legt meer werk bij routers en vereist nog altijd dat de zender alle bestemmingen kent.

## Flooding broadcast

Hier stuurt de bron één message, en routers sturen die verder op alle links behalve de inkomende link.

Ook hier gebruik je TTL en sequence numbers om te voorkomen dat het onbeperkt blijft rondlopen.

Dat is robuust, maar nog altijd niet optimaal qua bandbreedtegebruik.

## Spanning tree broadcast

De meest efficiënte broadcast in aantal pakketten krijg je met een **spanning tree**.

Een spanning tree:

- bevat alle routers
- bevat geen lussen

Als routers weten welke links tot die boom behoren, kunnen ze een broadcast precies langs die boom doorsturen. Dan krijg je het minimum aantal nodige packetkopieën.

### nadeel

Dat vraagt veel kennis van de globale structuur. Daarom is dit niet zomaar combineerbaar met elk routingmodel, en zeker niet eenvoudig in een pure distance-vectorcontext.

### belangrijk voor examen

Een spanning-treebroadcast gebruikt het **minimum aantal packets**, maar veronderstelt genoeg globale kennis om die boom te kennen.

## Samenvatting van het hoofdstuk

Dit hoofdstuk vergelijkt twee fundamentele manieren om network-layerdiensten op te bouwen:

- **connectionless datagrams**
- **connection-oriented virtual circuits**

Daarna zie je hoe routing in de praktijk gebeurt:

- **Dijkstra** voor kortstepaden
- **Bellman-Ford / distance vector** met enkel buurkennis
- **link state routing** met flooding van linkinformatie en lokaal Dijkstra
- **hierarchical routing** om schaalbaarheid mogelijk te maken

Tot slot zie je dat ook de manier van afleveren belangrijk is:

- unicast
- broadcast
- multicast

## Wat je echt moet kennen

### belangrijk voor examen

- De rol van de network layer
- Wat **store-and-forward** betekent
- Het verschil tussen **connectionless** en **connection-oriented**
- Waarom IP een **datagram**-protocol is
- Wat **virtual circuits** zijn en wat **label switching** betekent
- De argumenten voor datagrams versus virtual circuits
- De gewenste eigenschappen van routingalgoritmes
- Het **optimality principle** en het idee van een **sink tree**
- Hoe **Dijkstra** in grote lijnen werkt
- Hoe **Bellman-Ford** in grote lijnen werkt
- Wat het **count-to-infinity-probleem** is, waarom het ontstaat, welke **mitigaties** bestaan (split horizon, poison reverse, maximum metric) en hoe **AODV** het oplost via destination sequence numbers
- De vijf stappen van **link-state routing**
- Waarom flooding TTL, sequence numbers en age nodig heeft
- Waarom **hierarchical routing** nodig is en welke trade-off erbij hoort
- Het verschil tussen **unicast**, **broadcast** en **multicast**
- Waarom **spanning tree broadcast** efficiënt is

## Korte eindintuitie

Als je dit hele hoofdstuk tot de essentie herleidt:

de network layer moet packets door een complex netwerk krijgen zonder dat hogere lagen die complexiteit voelen, en routing is de kunst om dat schaalbaar, correct en efficiënt te doen.

Dat is de rode draad door alles in deze slides.
