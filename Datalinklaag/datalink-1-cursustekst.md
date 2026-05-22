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

## Slotted ALOHA

**Slotted ALOHA** is de eerste grote verbetering.

Hier wordt tijd opgedeeld in discrete **slots**. Een node mag alleen aan het begin van een slot beginnen zenden.

Dat helpt omdat collisions dan veel strakker begrensd worden. Je hoeft niet meer rekening te houden met overlappende willekeurige stukken van twee tijdsintervallen, maar enkel met botsingen binnen hetzelfde slot.

### prijs die je betaalt

Je hebt wel **synchronisatie** nodig.

Dus:

- betere efficiëntie
- maar extra coördinatie

### belangrijk voor examen

Slotted ALOHA verbetert Pure ALOHA door zendingen alleen op slotgrenzen toe te laten, maar daarvoor is synchronisatie nodig.

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

### belangrijk voor examen

Bij BMAC zorgt een **lange preamble** ervoor dat een ontvanger die maar af en toe luistert, toch een inkomend packet kan opvangen.

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

### intuition

In plaats van al je geluk op één kanaal te zetten, spreid je de communicatie over tijd en frequentie.

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
