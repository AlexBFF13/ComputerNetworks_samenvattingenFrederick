# Deel 6 - Transport 1: Principles

In dit deel beginnen de slides aan de **transport layer**. Dat is de laag die tussen applicaties en de network layer zit, en die moet zorgen dat processen op twee hosts op een bruikbare manier met elkaar kunnen communiceren.

De grote thema's in deze slides zijn:

- welke diensten de transportlaag aanbiedt
- welke bouwstenen een transportprotocol nodig heeft
- hoe sliding window-protocollen werken
- hoe congestion control in grote lijnen werkt
- wat UDP wel en niet doet

Dit hoofdstuk is fundamenteel, want het legt de basis voor alles wat later over TCP, RTP en QUIC komt.

## De rol van de transportlaag

De transportlaag zorgt voor **end-to-end delivery** van data tussen processen op verschillende hosts. Data wordt op dit niveau verpakt in **segmenten**.

De transportlaag probeert tegelijk:

- efficiënt te zijn
- betrouwbaar te zijn
- en de kosten beheersbaar te houden

Een belangrijk idee uit de slides is dat de transportlaag vooral op de **hosts** zit. Het is de grote grens tussen:

- de wereld van applicaties
- en de wereld van het netwerk

Dat betekent dat de transportlaag veel complexiteit van lagere lagen moet afschermen voor applicaties.

## Connectionless en connection-oriented transport

De slides maken meteen een belangrijk onderscheid.

### Connectionless transport

Dit focust op:

- snelheid
- eenvoud
- weinig overhead

Er is geen connection setup nodig, maar daar staat tegenover dat er weinig ingebouwde hulp is:

- beperkte error control
- geen flow control

Veel verantwoordelijkheid komt dan bij de applicatie terecht.

### Connection-oriented transport

Dit focust op:

- betrouwbaarheid
- foutafhandeling
- duidelijke scheiding van verantwoordelijkheden

Hier bestaat de communicatie typisch uit drie fasen:

1. **establishment**
2. **data transfer**
3. **release**

Dus eerst zet je de verbinding op, dan wissel je data uit, en daarna sluit je weer netjes af.

### belangrijk voor examen

Het basisverschil is:

- **connectionless**: snel en simpel, maar weinig garanties
- **connection-oriented**: meer overhead, maar veel meer ingebouwde ondersteuning

## Segmenten en encapsulatie

Op de transportlaag spreken we over **segmenten**.

Die worden dan verder ingekapseld:

- segmenten zitten in **network-layer packets**
- die packets zitten weer in **data-link frames**

Dat lijkt eenvoudig, maar het is goed om die woorden correct te gebruiken. Een term als "packet" is dus niet automatisch hetzelfde als "segment".

## Generalized transport primitives

De slides tonen dat transportdiensten op een abstract niveau beschreven kunnen worden via een set **primitives**.

Het grote idee is dat je een protocol niet alleen als pakketformaat moet zien, maar ook als een reeks handelingen:

- verbinding opzetten
- data sturen
- data ontvangen
- verbinding sluiten

Dat helpt om de transportlaag te zien als een dienstinterface, niet alleen als bytes op de lijn.

## Berkeley sockets

De praktisch belangrijkste programmeerinterface in deze context is **Berkeley sockets**.

De slides herinneren eraan dat je eigenlijk al met een connection-oriented transportprotocol gewerkt hebt via TCP sockets.

Berkeley sockets werden uitgebracht met Berkeley UNIX en vormden de basis voor gelijkaardige modellen in Linux, Windows en macOS.

### Serverkant

Aan serverzijde zie je typisch deze flow:

1. `SOCKET`
2. `BIND`
3. `LISTEN`
4. `ACCEPT`
5. `SEND` / `RECEIVE`
6. `CLOSE`

### Clientkant

Aan clientzijde is het eenvoudiger:

1. `SOCKET`
2. `CONNECT`
3. `SEND` / `RECEIVE`
4. `CLOSE`

De client heeft normaal geen `BIND` en `LISTEN` nodig op die klassieke manier.

### intuition

De server maakt eerst een ingang klaar waar aanvragen mogen binnenkomen. De client stapt gewoon naar die ingang toe.

### belangrijk voor examen

Het verschil tussen server- en clientflow in Berkeley sockets is een klassiek basispunt, vooral de rol van `LISTEN` en `ACCEPT`.

## De vier bouwstenen van transportprotocollen

De slides structureren de kern van transportprotocollen rond vier elementen:

1. **addressing**
2. **connection management**
3. **error control**
4. **flow control**

Dat is een erg goede kapstok voor dit hoofdstuk.

## 1. Addressing

Een IP-adres alleen is niet genoeg om transportverkeer correct af te leveren, want op één host kunnen meerdere processen tegelijk netwerkverkeer gebruiken.

Daarom heb je op transportniveau **Transport Service Access Points (TSAPs)** nodig.

In de praktijk zijn **poorten** een voorbeeld van TSAPs.

De slides zetten dat tegenover **Network Service Access Points (NSAPs)**, zoals IP-adressen.

Dus:

- **NSAP**: netwerklaag-eindpunt
- **TSAP**: transportlaag-eindpunt

### Belangrijk inzicht

Eén host kan één IP-adres hebben, maar daarachter kunnen veel verschillende transportverbindingen en toepassingen actief zijn.

### Analogie

De slides gebruiken de telefooncentrale-analogie:

- één extern nummer
- meerdere interne extensies

Dat is een goed beeld voor:

- één NSAP
- meerdere TSAPs

## Poorttoewijzing en bekende diensten

Sommige poorten zijn **well known** en vast gekoppeld aan bekende diensten:

- HTTP
- SMTP
- Telnet
- SSH

Op Unix-achtige systemen vind je zulke mappings bijvoorbeeld in `/etc/services`.

Dat is praktisch, maar conceptueel ook belangrijk: transportadressering is gestandaardiseerd zodat clients weten waar ze een dienst kunnen vinden.

## Initial Connection Protocol

Niet elke server moet noodzakelijk de hele tijd als volledig actief proces draaien.

De slides beschrijven daarom het **Initial Connection Protocol (ICP)**:

- een process server luistert op een well-known TSAP
- bij een inkomende verbinding start die pas de echte server op

Op Unix-achtige systemen werd dit soort idee historisch uitgevoerd via `inetd`.

### waarom is dit nuttig?

Omdat je zo resources kan besparen:

- processen niet permanent laten draaien
- servers pas opstarten wanneer er effectief vraag is

## Multiplexing en inverse multiplexing

De slides tonen ook twee verwante ideeën:

- **multiplexing**
- **inverse multiplexing**

### Multiplexing

Verschillende applicatiestromen delen één netwerklaagverbinding of één lager kanaal.

### Inverse multiplexing

Eén hogere-stroom wordt net over meerdere lagere kanalen verdeeld.

Die begrippen tonen dat de transportlaag niet alleen over betrouwbaarheid gaat, maar ook over hoe verkeer logisch verdeeld of samengebracht wordt.

## 2. Connection management

Connection management lijkt op het eerste gezicht eenvoudig: zet een verbinding op en sluit ze later weer. Maar de slides tonen waarom dat in echte netwerken lastig is.

### Het probleem

Transportprotocollen draaien bovenop een netwerk waar packets:

- verschillende routes kunnen nemen
- verschillende snelheden kunnen ondervinden
- vertraagd kunnen raken
- gedupliceerd kunnen worden

Dat betekent dat oude of out-of-order packets later nog kunnen opduiken.

## Out-of-order packets

De figuren tonen dat packets niet noodzakelijk in volgorde aankomen:

- sommige nemen een kortere route
- andere nemen een langere route
- sommige gaan over tragere links, zoals satelliet of draadloos

Dus zelfs als jij segment 1, 2 en 3 netjes verstuurt, kan de ontvanger ze in een andere volgorde zien.

### Waarom is dat erg?

Omdat hogere toepassingen daar last van kunnen hebben.

De slides geven een mooi voorbeeld met een banktransfer:

- je verstuurt een opdracht
- die lijkt verloren
- je stuurt opnieuw
- de oude vertraagde opdracht komt later toch nog aan

Dan krijg je dubbel uitgevoerde acties.

### belangrijk voor examen

Transportprotocollen mogen niet aannemen dat packets in volgorde aankomen. Vertraging en hertransmissie maken dat gevaarlijk.

## Bounded packet lifetime

Een klassieke oplossing is om de levensduur van packets te **begrenzen**.

Het idee:

- elk packet krijgt een sequence number
- een sequence number wordt niet opnieuw gebruikt binnen een voldoende lange tijd

Zo kan de ontvanger oude, vertraagde of gedupliceerde packets herkennen en verwerpen.

De slides koppelen dat aan:

- packet lifetime
- wraparound van sequence numbers
- en de nood aan een klok bij crashes

### Waarom een klok bij crashes?

Als een systeem crasht en herstart, mag het niet per ongeluk oude sequence numbers hergebruiken die nog in het netwerk rondhangen. Daarom gebruikt men een real-time clock om na een crash met hogere sequence numbers verder te gaan.

## The two generals problem

Dit is een klassiek fundamenteel probleem in gedistribueerde systemen.

De kern is:

zelfs als twee kanten acknowledgements uitwisselen, kan je nooit met absolute zekerheid weten dat alle nodige bevestigingen ook effectief aangekomen zijn.

De slides zijn hier heel duidelijk:

- **no solution is possible**

Dat betekent dat de transportlaag alleen geen perfecte, absolute zekerheid kan geven over gedeelde kennis van verbindingstoestand.

### praktische les

Hogere protocollen en applicaties moeten zo ontworpen worden dat ze abrupte disconnecties kunnen verdragen, of dat ze probabilistisch omgaan met onzekerheid.

### belangrijk voor examen

Het **two generals problem** heeft geen perfecte oplossing; absolute wederzijdse zekerheid is niet afdwingbaar met eindige berichtuitwisseling.

## 3. Error control

Error control gaat over het detecteren en herstellen van fouten in transportcommunicatie.

De slides leggen drie grote ideeën uit:

- end-to-end checksums
- acknowledgements
- hertransmissie

## End-to-end checksums

Een checksum is een korte code die uit grotere data wordt berekend.

De ontvanger herberekent de checksum en vergelijkt die met de meegestuurde waarde. Als die niet overeenkomt, weet je dat er iets misging.

### Waarom nog een checksum op transportniveau?

Dat is een heel belangrijk punt.

Ook lagere lagen, zoals de datalinklaag, doen foutcontrole. Maar dat beschermt vooral **hop per hop**. Een packet kan nog steeds beschadigd raken **binnen een defecte router**.

Dus:

- hop-by-hop checks zijn niet genoeg
- je hebt ook een **end-to-end check** nodig

### belangrijk voor examen

Transportchecksums zijn nodig zelfs als lagere lagen ook checksums hebben, omdat fouten ook **binnen routers** kunnen ontstaan.

## Acknowledgements en retransmissie

Als je betrouwbare overdracht wil, moet de zender weten of data effectief is aangekomen.

Daarom gebruik je:

- **ACKs**
- eventueel **NACKs**

De basislogica is:

- zender stuurt segment
- ontvanger ACK't als het goed aankomt
- als ACK uitblijft, wordt opnieuw gestuurd

Dat betekent ook dat de zender oude berichten tijdelijk moet bewaren tot ze bevestigd zijn.

## 4. Flow control

Flow control gaat over de vraag:

hoe zorg je dat een snelle zender een tragere ontvanger niet overspoelt?

De grote oplossing in deze slides is de familie van **sliding window protocols**.

## Sliding windows

Een sliding window-protocol combineert:

- error control
- flow control

Zowel zender als ontvanger houden een venster bij van segmenten waarvoor nog geen definitieve bevestiging is afgerond.

Door de venstergrootte te kiezen, bepaal je hoeveel verkeer tegelijk "in flight" mag zijn.

### intuition

In plaats van na elk segment te stoppen, laat je een gecontroleerd aantal segmenten tegelijk onderweg zijn.

## Window size en stop-and-wait

De eenvoudigste vorm is een venster van grootte 1:

- **stop-and-wait**

Dat betekent:

- stuur één segment
- wacht op ACK
- stuur pas daarna het volgende

Dit gebruikt weinig geheugen en is eenvoudig, maar het is inefficiënt wanneer de **RTT hoog** is ten opzichte van de verzendsnelheid.

## Stop-and-wait in detail

De slides tonen:

- de zender bewaart het laatste segment tot ACK aankomt
- data en ACKs gebruiken alternerende nummers, typisch 0 en 1
- de zender start een timer
- als er geen ACK komt voor timeout, wordt opnieuw gestuurd

### Normale situatie

Segment komt aan, ACK komt terug, klaar.

### Als data verloren gaat

De zender krijgt geen ACK, timeout verloopt, segment wordt opnieuw gestuurd.

### Als ACK verloren gaat

Ook dan ziet de zender geen ACK en stuurt hij opnieuw. De ontvanger herkent dat het om een duplicaat gaat en ACK't nog eens.

### Piggybacking

Bij bidirectionele communicatie kan je ACKs soms meenemen samen met eigen data. Dat heet **piggybacking** en verlaagt het aantal aparte segmenten.

### belangrijk voor examen

Stop-and-wait is eenvoudig, maar inefficiënt bij hoge RTT omdat er nooit meer dan één segment tegelijk onderweg is.

## Go-Back-N

De volgende stap is **Go-Back-N**.

Hier mag de zender **W segmenten** uitsturen voor hij op ACKs moet wachten. Daardoor stijgt de efficiëntie sterk.

Belangrijke eigenschappen:

- grotere range van sequence numbers
- zender bewaart alle nog niet geACKte segmenten
- ontvanger heeft een venster van grootte 1
- out-of-order segmenten worden verworpen

### Cumulative ACKs

ACKs in Go-Back-N zijn **cumulatief**. Een ACK voor een bepaald nummer impliceert dat alle lagere segmenten ook goed aangekomen zijn.

### Wat gebeurt er bij timeout?

Als één segment in het venster timed out, stuurt de zender dat segment én alle volgende nog niet bevestigde segmenten opnieuw.

Dat is waarom het protocol "go back N" heet.

### voordeel

Veel efficiënter dan stop-and-wait in normale omstandigheden.

### nadeel

Bij verlies kan je veel **onnodige hertransmissie** krijgen van segmenten die eigenlijk wel al ontvangen waren.

### belangrijk voor examen

Bij Go-Back-N heeft de ontvanger effectief venster 1, en bij timeout worden **alle niet-bevestigde segmenten vanaf het verloren segment opnieuw gestuurd**.

## Selective Repeat

**Selective Repeat** wil precies dat nadeel van Go-Back-N oplossen.

In plaats van alle latere segmenten opnieuw te sturen, hertransmitteer je alleen wat echt ontbreekt of beschadigd is.

Dat maakt het protocol:

- bandbreedte-efficiënter

maar ook:

- complexer

### Waarom complexer?

Omdat de ontvanger nu out-of-order segmenten **mag bufferen**. Daardoor heb je aan beide kanten extra buffers, bookkeeping en variabelen nodig.

### NACKs

Een **NACK** kan gebruikt worden om expliciet te melden welk segment ontbreekt of beschadigd is.

### Venstergroottebeperking

De slides geven een belangrijk correctheidsdetail:

de window size moet **kleiner dan of gelijk aan de helft van de sequence number space** zijn.

Anders zou de ontvanger duplicaten kunnen verwarren met nieuwe segmenten.

### belangrijk voor examen

Selective Repeat is efficiënter dan Go-Back-N, maar vereist **buffers aan beide kanten** en een **venstergroottebeperking van hoogstens de helft van de sequence space**.

## Congestiecontrole: wat is congestie?

Na flow control gaan de slides naar **congestion control**.

Congestie ontstaat wanneer we te veel packets te snel het netwerk in sturen. Dan raken routers overbelast en ontstaan:

- wachtrijen
- toenemende delay
- packet loss

Een belangrijk onderscheid:

- **flow control** beschermt de ontvanger
- **congestion control** beschermt het netwerk

De network layer moet congestie signaleren, maar de transportlaag moet uiteindelijk het verzendtempo aanpassen.

## Wat willen we bereiken?

De slides noemen vier doelen:

- alle beschikbare bandbreedte benutten
- congestie vermijden
- eerlijk zijn tussen flows
- snel reageren op veranderingen

Dat is lastig, zeker zonder globale kennis.

## Throughput versus goodput

Nog een belangrijk inzicht:

we willen niet zomaar maximale **throughput**, maar goede **goodput**.

Goodput betekent:

- nuttige data die effectief aankomt

Als het netwerk te dicht tegen zijn limiet zit, leiden verliezen tot hertransmissies. Dan kan de totale nuttige opbrengst dalen en zelfs tot **congestion collapse** leiden.

## Load en delay

Naarmate load stijgt, stijgt ook delay, vooral door buffering in routers.

Kleinrock stelde daarom het idee van:

- `power = load / delay`

voor als manier om te zien wanneer je nog efficiënt werkt. Meer load is niet altijd beter als de delay daar disproportioneel door toeneemt.

## Min-max fairness

Een allocatie is **min-max fair** als je de bandbreedte van één flow niet kan verhogen zonder die van een andere flow te verlagen.

Dat is een nuttige theoretische maatstaf voor eerlijkheid.

In de praktijk is perfecte fairness moeilijk, maar het belangrijke doel is:

- geen starvation
- geen instabiel gedrag

## Twee soorten traagheid

Als een verbinding traag is, kan dat volgens de slides twee oorzaken hebben:

1. de ontvanger is zwak of traag
2. het netwerk ertussen is congested

Dat onderscheid is cruciaal:

- in het eerste geval moet je flow control aanpassen
- in het tweede geval moet je send rate omlaag door congestiesignalen

TCP doet later beide.

## AIMD

De slides eindigen de congestion control-sectie met **AIMD**:

- **Additive Increase**
- **Multiplicative Decrease**

Dat betekent:

- als het goed gaat, verhoog je voorzichtig en lineair
- bij congestie verlaag je agressief en multiplicatief

### waarom zo?

Omdat:

- het gemakkelijk is om een netwerk in congestie te duwen
- maar moeilijker om daar weer uit te raken

Dus stijgen moet voorzichtig zijn, dalen moet krachtig zijn.

### belangrijk voor examen

AIMD convergeert snel en redelijk stabiel omdat het **zacht verhoogt** maar **hard verlaagt**.

## UDP

Tot slot komen de slides bij **UDP**, User Datagram Protocol.

UDP is:

- **connectionless**
- lichtgewicht
- beschreven in RFC 768

Het laat hosts segmenten sturen zonder connection setup en zonder veel overhead.

### Wat biedt UDP niet?

UDP biedt geen:

- flow control
- ordering
- congestion control

### Wat biedt UDP wel?

UDP biedt wel:

- **ports**
- **checksum**

Dus UDP is heel bewust minimaal gehouden.

## UDP-pakketstructuur

De slides tonen dat UDP in essentie vooral de twee dingen levert die applicaties vaak nog wel nodig hebben:

- TSAP-adressering via poorten
- eenvoudige end-to-end error checking via checksum

De checksum van UDP wordt berekend over:

- de UDP-header
- en een IP **pseudo-header**

Dat is weer zo’n typisch detail dat je best conceptueel begrijpt: de checksum kijkt niet alleen naar de zuivere UDP-bytes, maar ook naar context uit de netwerklaag.

## Waarom zou je UDP gebruiken?

De slides noemen een paar belangrijke voordelen:

- lage overhead tegenover TCP
- bruikbaar voor **anycast**, **broadcast** en **multicast**
- nuttig op kleine of eenvoudige toestellen

Voorbeelden:

- DNS gebruikt graag UDP
- DHCP gebruikt broadcast en dus ook UDP

### belangrijk voor examen

UDP is nuttig wanneer je:

- lage overhead wil
- geen verbindingsopbouw nodig hebt
- of communicatievormen zoals broadcast/multicast wil ondersteunen

## Samenvatting van het hoofdstuk

Dit hoofdstuk legt de fundamenten van de transportlaag:

- welke diensten die laag aanbiedt
- waarom poorten en TSAPs nodig zijn
- hoe connection management ingewikkeld wordt door een echt netwerk
- hoe foutcontrole en flow control samenkomen in sliding windows
- hoe congestion control in grote lijnen werkt
- en waarom UDP bewust minimaal blijft

Dat is de basis waarop latere hoofdstukken over TCP, RTP en QUIC verder bouwen.

## Wat je echt moet kennen

### belangrijk voor examen

- Het verschil tussen **connectionless** en **connection-oriented** transport
- Wat een **segment** is
- Het verschil tussen **NSAP** en **TSAP**
- De basisflow van **Berkeley sockets** voor client en server
- Waarom **out-of-order** en **duplicate packets** een probleem zijn
- Het idee van **bounded packet lifetime**
- Waarom het **two generals problem** geen perfecte oplossing heeft
- Waarom transportprotocollen **end-to-end checksums** nodig hebben
- De rol van **ACKs**, **NACKs** en **retransmissie**
- Hoe **stop-and-wait** werkt
- Hoe **Go-Back-N** werkt
- Hoe **Selective Repeat** werkt
- Waarom SR een **venstergroottebeperking** heeft
- Het verschil tussen **flow control** en **congestion control**
- Het idee achter **AIMD**
- Wat UDP wel en niet aanbiedt

## Korte eindintuitie

Als je dit hoofdstuk samenvat in één gedachte:

de transportlaag probeert van een rommelig, onvoorspelbaar netwerk een bruikbare communicatiedienst te maken voor applicaties, en doet dat met adressering, volgordebeheer, foutcontrole, flow control en zorgvuldig gekozen eenvoud.

Dat is precies de rode draad door deze slides.
