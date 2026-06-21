# Deel 8 - Transport 3: QUIC

In deze slides verschuift de focus van klassieke transportprotocollen naar **QUIC**. Dat protocol is ontstaan omdat het internet en de toestellen errond veranderd zijn. We gebruiken vandaag veel meer mobiele toestellen, veel meer multimedia, en veel meer applicaties die tegelijk meerdere soorten data over dezelfde verbinding sturen. Klassieke TCP kan veel, maar botst daar ook op limieten.

QUIC probeert die limieten op te lossen door een modern transportprotocol te bouwen bovenop UDP, met ingebouwde security, lage opstartlatency, stream multiplexing en ondersteuning voor connection migration.

## Waarom QUIC nodig was

De motivatie voor QUIC komt uit twee evoluties tegelijk.

Ten eerste zijn **hosts** veranderd. Een smartphone heeft vandaag vaak meerdere netwerkinterfaces tegelijk, zoals WiFi en 4G of 5G. Dat betekent dat een verbinding soms van netwerkpad moet veranderen terwijl de applicatie gewoon wil blijven doorwerken.

Ten tweede zijn ook **applicaties** veranderd. Moderne websites zijn niet meer alleen HTML-tekst. Ze bevatten video, scripts, API-calls, afbeeldingen en andere parallelle datastromen. De onderliggende transportlaag moet daar dus beter mee kunnen omgaan.

TCP had wel uitbreidbaarheid via options, maar in de praktijk bleek dat lastig:

- middleboxes behandelen TCP-options niet altijd correct
- sommige hosts of tussenliggende systemen droppen of negeren opties
- nieuwe uitbreidingen uitrollen op TCP is daardoor traag en moeilijk

Daarom ontstond het idee om een nieuw transportprotocol te bouwen in **userspace**, bovenop UDP.

### waarom bovenop UDP?

Niet omdat UDP op zich alle gewenste diensten aanbiedt, maar omdat UDP een eenvoudige drager is waarboven je zelf een nieuw protocol kan ontwerpen zonder vast te zitten aan de historische beperkingen van TCP in kernels en middleboxes.

## De ontwerpdoelen van QUIC

QUIC is niet zomaar "TCP maar iets nieuwer". Het is ontworpen met een duidelijke lijst doelen:

- ingebouwde encryptie in het transportprotocol
- betrouwbare, in-order stream multiplexing
- lage latency bij het opzetten van een verbinding
- connection migration
- implementatie in userspace bovenop UDP

Die combinatie is de kern. QUIC wil tegelijk **veilig**, **snel opstartbaar**, **flexibel** en **geschikt voor moderne applicaties** zijn.

### belangrijke begrippen

- **userspace**: het protocol zit niet vast in de OS-kernel, maar kan sneller evolueren in softwarebibliotheken
- **stream multiplexing**: meerdere logische datastromen tegelijk over een verbinding
- **connection migration**: de verbinding behouden terwijl IP-adres of poort veranderen

## QUIC in het kort

QUIC is gestandaardiseerd in **RFC 9000** van mei 2021. In de slides wordt vermeld dat het ongeveer 30% van het internetverkeer vertegenwoordigt, wat meteen toont dat dit geen nicheprotocol is.

Je kan QUIC zien als een soort combinatie van:

- transportfunctionaliteit zoals bij TCP
- security zoals bij TLS

maar dan in een strakker geïntegreerd ontwerp bovenop UDP.

QUIC gebruikt **Connection IDs** in plaats van enkel IP-adres en poort om een verbinding te herkennen. Dat is erg belangrijk voor migratie: als het netwerkpad verandert, kan dezelfde verbinding toch verder blijven bestaan.

Daarnaast is QUIC een **frame-based protocol**. Dat betekent dat de inhoud niet gewoon een ongestructureerde byte-stream is zoals bij TCP, maar dat verschillende soorten informatie in afzonderlijke frame types verpakt worden.

### belangrijk voor examen

QUIC is geen kleine uitbreiding op TCP. Het is een nieuw transportprotocol bovenop UDP met eigen verbindingidentificatie, ingebouwde security en eigen framing.

## Twee soorten headers

QUIC gebruikt twee grote headerformaten:

- **Long Header**
- **Short Header**

### Long Header

De Long Header wordt gebruikt in de vroege fase van een verbinding, dus tijdens setup en voordat alles volledig operationeel en versleuteld is.

Daar zitten zaken in zoals:

- versie-informatie
- connection setup context
- info die nodig is tijdens handshake en negotiation

### Short Header

De Short Header wordt gebruikt nadat de verbinding is opgezet en encryptie actief is. Het doel daar is vooral **efficiente datatransmissie**.

De grote intuïtie is dus:

- eerst meer expliciete info om de verbinding veilig op gang te krijgen
- daarna een compactere header om overhead te verlagen

## Connection IDs

Bij TCP hangt een verbinding sterk samen met het tuple van bron-IP, bronpoort, doel-IP en doelpoort. QUIC pakt dat anders aan met **Connection IDs (CID)** van variabele lengte.

Dat heeft een groot voordeel: de identiteit van de verbinding hangt minder hard vast aan het actuele netwerkpad.

Dus als een client van WiFi naar mobiel internet overschakelt, dan kan de server de verbinding nog altijd herkennen via de CID, ook al zijn IP en poort veranderd.

### intuition

Een CID is als het ware een stabieler label voor de connectie dan "van dit IP naar die poort".

## QUIC is frame-based

In QUIC worden verschillende soorten informatie in **frames** verpakt. Dat maakt het protocol veel explicieter en modulairer.

Voorbeelden:

- data van applicatiestreams
- acknowledgments
- cryptografische handshake-data
- flow-control updates
- path validation

Dat frame-based ontwerp maakt het makkelijker om verschillende functies netjes naast elkaar te laten bestaan binnen dezelfde verbinding.

## ACKs in QUIC

QUIC gebruikt acknowledgments, maar niet op exact dezelfde manier als klassieke TCP.

De slides tonen dat een ACK in QUIC **ranges** kan bevatten. Dat betekent dat de ontvanger niet alleen zegt "ik heb alles tot hier", maar ook compacter kan meedelen welke stukken ontvangen zijn en welke ontbreken.

Een ACK kan dus bijvoorbeeld impliciet zeggen:

- packet 0 ontvangen
- packet 2 ontvangen
- packet 4 ontvangen
- packet 6 ontvangen
- packets 1, 3 en 5 ontbreken

Dat is nuttig omdat QUIC beter kan omgaan met gaten in ontvangstvolgorde zonder alles te reduceren tot een puur cumulatief ACK-model.

### waarom is dit handig?

Bij verlies of out-of-order aankomst geeft range-based acknowledgment meer precieze informatie. Daardoor kan de zender beter inschatten wat echt verloren is en wat gewoon later aankwam.

## Sequence numbers in QUIC

Een slide vergelijkt TCP en QUIC op vlak van sequence numbers.

TCP gebruikt klassieke 32-bit sequence numbers. Daardoor kunnen die uiteindelijk wrappen. QUIC gebruikt veel grotere nummers, in de slides voorgesteld als **62-bit**. Het idee daarachter is eenvoudig: wraparound wordt in de praktijk geen probleem meer.

Dat maakt het protocol eenvoudiger voor lange, zware of langdurige stromen.

### belangrijk voor examen

Het punt van de grotere QUIC-nummers is niet "meer bits is leuk", maar dat sequence numbers in de praktijk **niet meer wrappen**, wat ontwerp en interpretatie eenvoudiger maakt.

## Stream multiplexing

Een van de belangrijkste vernieuwingen van QUIC is **stream multiplexing**.

Bij TCP heb je een byte-stream. Alles zit in dezelfde geordende stroom. Als ergens iets ontbreekt, dan blokkeert dat de levering van latere data aan de applicatie, zelfs als die latere bytes eigenlijk al toegekomen zijn. Dat is het bekende **head-of-line blocking**-probleem.

QUIC pakt dat slimmer aan door meerdere onafhankelijke streams binnen dezelfde verbinding toe te laten.

### Wat betekent dat concreet?

Stel dat een webpagina tegelijk drie dingen downloadt:

- een stukje HTML
- een afbeelding
- een script

Bij QUIC kunnen die elk in een aparte stream zitten. Als in stream 1 een packet ontbreekt, dan hoeft dat de andere streams niet stil te leggen. Alleen de getroffen stream wacht op herstel.

### intuition

TCP zegt min of meer: "alles in een rij".  
QUIC zegt: "meerdere rijen naast elkaar".

Dat maakt QUIC veel geschikter voor moderne webtoepassingen waar veel parallel gebeurt.

### belangrijk voor examen

QUIC vermindert **head-of-line blocking op verbindingsniveau** door meerdere onafhankelijke streams binnen een connectie te ondersteunen.

## Types van streams

Niet elke stream in QUIC is hetzelfde. Streams kunnen:

- **client-initiated** of **server-initiated** zijn
- **unidirectional** of **bidirectional** zijn

Dus je krijgt vier basissoorten:

- client-initiated bidirectional
- server-initiated bidirectional
- client-initiated unidirectional
- server-initiated unidirectional

Dat geeft QUIC veel flexibiliteit. Sommige datastromen moeten in beide richtingen kunnen lopen, andere niet.

### voorbeeld

Een request-response patroon is vaak bidirectional.  
Een push van zuivere metadata of controle-info kan unidirectional zijn.

## Low-latency security

Een heel belangrijk ontwerpdoel van QUIC is dat security niet als een aparte, logge extra bovenop transport voelt. Bij klassieke combinaties zoals TCP + TLS moest je vaak meerdere round trips spenderen voor alles klaar was.

De slides vergelijken:

- TCP + TLS 1.2
- TCP + TLS 1.3
- TCP + TLS 1.3 met early data
- QUIC + TLS 1.3
- QUIC + TLS 1.3 met early data

Het algemene patroon is duidelijk: QUIC verlaagt de **connection establishment latency** doordat transport en cryptografische setup veel nauwer geintegreerd zijn.

### 1-RTT en 0-RTT

Bij een **nieuwe verbinding** kan QUIC typisch in **1 RTT** operationeel zijn. De transport handshake en de TLS 1.3 handshake lopen tegelijk, in plaats van na elkaar zoals bij TCP + TLS.

Vergelijking van opstartlatency:

| Combinatie | Aantal RTTs voor eerste applicatiedata |
|---|---|
| TCP + TLS 1.2 | 3 RTT (1 TCP handshake + 2 TLS handshake) |
| TCP + TLS 1.3 | 2 RTT (1 TCP handshake + 1 TLS handshake) |
| TCP + TLS 1.3 + early data | 2 RTT (maar data kan al bij de eerste TLS-flight mee) |
| QUIC + TLS 1.3 (nieuw) | **1 RTT** (transport + crypto tegelijk) |
| QUIC + TLS 1.3 (hervat) | **0 RTT** (early data met opgeslagen keys) |

### 0-RTT: hoe en wanneer

Bij **0-RTT** stuurt de client meteen applicatiedata mee in het allereerste packet, nog voor de handshake afgerond is. Dit kan alleen als:

- de client eerder al een verbinding met deze server had
- de client een **session ticket** of **pre-shared key** van die eerdere sessie heeft opgeslagen
- de server deze keys nog accepteert

### Security-risico's van 0-RTT

0-RTT is niet gratis -- er zijn belangrijke security trade-offs:

- **Replay attacks**: een aanvaller kan een 0-RTT packet opvangen en later opnieuw afspelen. De server kan niet met zekerheid onderscheiden of het een origineel of herhaald verzoek is. Daarom is 0-RTT enkel veilig voor **idempotente** operaties (operaties die je veilig meerdere keren kan uitvoeren, zoals een GET-request, maar niet een bankovermaking).

- **Geen forward secrecy**: 0-RTT data wordt versleuteld met keys van de vorige sessie. Als die keys gecompromitteerd zijn, is de 0-RTT data ook leesbaar.

### belangrijk voor examen

QUIC verlaagt opstartlatency door **transport en TLS 1.3 sterk te integreren** (1 RTT voor nieuwe verbindingen). Bij hervatte sessies is **0-RTT early data** mogelijk, maar dit brengt het risico van **replay attacks** mee en is daarom enkel veilig voor idempotente operaties.

## Congestion control in QUIC

De slides zeggen expliciet dat QUIC voor congestion control gebaseerd is op **Cubic (RFC 8312)** en dat het nog altijd werkt met een impliciet en niet-perfect congestiesignaal, vooral via:

- packet loss
- packet latency

Het detail van Cubic zelf wordt hier niet uitgewerkt, dus voor deze cursustekst is het belangrijker om het juiste niveau te onthouden: QUIC doet wel degelijk congestion control, maar dat is niet het kernverschil waar deze slides op focussen.

## Connection migration

Een van de sterkste eigenschappen van QUIC is **connection migration**.

Dat betekent dat een client van IP-adres en poort mag veranderen terwijl de verbinding actief blijft. Dat is enorm handig voor mobiele toestellen.

### Praktische situatie

Een smartphone is verbonden via WiFi en communiceert met een server. Tijdens die sessie valt WiFi weg en schakelt het toestel over naar mobiel internet. Bij klassieke protocollen is dat vaak fataal voor de verbinding, omdat het netwerkpad volledig verandert.

Bij QUIC hoeft dat niet.

### Hoe gaat dat in grote lijnen?

De server ziet verkeer plots van een nieuw pad komen. Dan wil hij eerst zeker weten dat dit nog steeds dezelfde client is. Daarom wordt het nieuwe pad **gevalideerd**.

In de figuren gebeurt dat met onder meer:

- `PING`
- `PATH_CHALLENGE`
- `PATH_RESPONSE`

Pas nadat die path validation gelukt is, wordt het nieuwe pad normaal gebruikt.

### Waarom die voorzichtigheid?

Omdat een server niet blind extra veel data naar een nieuw adres mag beginnen sturen. Eerst moet hij controleren of dat adres echt geldig is en bij dezelfde peer hoort.

Daarom zie je in de slides ook dat de verzendsnelheid in het begin beperkt wordt op het nieuwe pad.

### Congestion state reset

Een belangrijk detail is dat bij migration de **congestion control state** gereset wordt. Dat is logisch: een nieuw netwerkpad kan totaal andere eigenschappen hebben dan het vorige. Je mag dus niet zomaar aannemen dat oude schattingen nog kloppen.

### belangrijk voor examen

Bij QUIC kan een verbinding actief blijven terwijl IP en poort veranderen, dankzij:

- **Connection IDs**
- **path validation**
- heropbouw van pad-specifieke toestand zoals congestion state

## QUIC frame types

De laatste slides geven enkele belangrijke frame types. Die moet je vooral conceptueel kunnen plaatsen.

### STREAM

Bevat applicatiedata. Dit is de eigenlijke inhoud van een stream.

### ACK

Bevestigt ontvangen packets.

### CRYPTO

Draagt cryptografische handshake-data, bijvoorbeeld TLS record stream-informatie.

### PATH_CHALLENGE en PATH_RESPONSE

Worden gebruikt om een nieuw netwerkpad te valideren.

### MAX_DATA en MAX_STREAM_DATA

Deze lijken op het receive-windowidee uit TCP, maar dan op respectievelijk:

- connectieniveau
- streamniveau

Ze regelen dus hoeveel data de peer nog mag sturen.

### NEW_CONNECTION_ID

Stelt een nieuwe CID voor die gebruikt kan worden voor de verbinding.

### MAX_STREAMS

Beperkt hoeveel streams gebruikt mogen worden.

### PING

Checkt of de peer nog bereikbaar is.

### CONNECTION_CLOSE

Sluit de verbinding af en geeft een foutoorzaak mee.

## QUIC versus TCP op hoofdlijnen

Nu we alles samenleggen, zie je dat QUIC niet alleen "sneller TCP" wil zijn. Het neemt een andere architecturale positie in.

| Aspect | TCP | QUIC |
|---|---|---|
| **Encryptie** | Optioneel, apart via TLS | Verplicht, TLS 1.3 ingebouwd |
| **Connection setup** | 1-3 RTT (TCP + TLS) | 1 RTT nieuw, 0 RTT hervat |
| **Datamodel** | Byte-stream (een enkele geordende stroom) | Frame-based met meerdere streams |
| **Head-of-line blocking** | Ja -- een verloren segment blokkeert alles | Nee -- enkel de getroffen stream wacht |
| **Connection migration** | Nee -- gebonden aan IP/poort 4-tuple | Ja -- overleeft WiFi naar 4G switch |
| **Implementatie** | OS kernel | User space (snellere updates) |
| **Uitbreidbaarheid** | TCP options, maar middleboxes blokkeren vaak | Frame types, versleuteld dus onzichtbaar voor middleboxes |
| **Congestion control** | AIMD / Reno / Cubic in kernel | Cubic (RFC 8312), pluggable in user space |
| **Verbindingsidentificatie** | IP/poort 4-tuple | Connection ID (variabele lengte) |
| **Sequence numbers** | 32-bit (kan wrappen) | 62-bit (wrapt in praktijk nooit) |
| **ACKs** | Cumulatief (+ optioneel SACK) | Range-based (altijd, rijkere info) |

### Waarom QUIC niet TCP kan vervangen voor alles

QUIC is geen universele vervanger:

- QUIC heeft **geen unreliable mode** -- het is altijd betrouwbaar. Voor verlies-tolerante realtime media (waar RTP/UDP voor dient) is QUIC niet geschikt.
- QUIC draait in **user space**, wat meer CPU-overhead kan geven dan kernel-TCP
- QUIC is versleuteld, waardoor netwerk-debugging en monitoring moeilijker is
- Sommige firewalls blokkeren of throttlen UDP-verkeer, wat QUIC kan hinderen

## Wat je echt moet kennen

### belangrijk voor examen

- Waarom QUIC ontwikkeld werd (middleboxes blokkeren TCP-opties, mobiele toestellen, moderne applicaties)
- Waarom QUIC bovenop **UDP** gebouwd is (middleboxes begrijpen UDP, user space vermijdt kernel-ossificatie)
- De rol van **Long Header** (setup) en **Short Header** (data na handshake)
- Wat **Connection IDs** oplossen (verbinding niet gebonden aan IP/poort)
- Waarom QUIC **frame-based** is (modulair, verschillende frametypes voor data/ACK/crypto/flow control)
- Hoe QUIC-ACKs **range-based** rijkere informatie geven dan TCP's cumulatieve model
- Waarom **stream multiplexing** head-of-line blocking oplost
- Het verschil tussen **unidirectional** en **bidirectional** streams
- De **latency-vergelijking**: TCP+TLS 1.2 = 3 RTT, TCP+TLS 1.3 = 2 RTT, QUIC nieuw = 1 RTT, QUIC hervat = 0 RTT
- Wat **0-RTT early data** betekent en het **replay attack risico** (enkel voor idempotente operaties)
- Hoe **connection migration** werkt: Connection IDs + PATH_CHALLENGE/PATH_RESPONSE + congestion state reset
- QUIC heeft **wel** congestion control (Cubic) maar is **geen** vervanging voor RTP (geen unreliable mode)
- De vergelijkingstabel **QUIC vs TCP** op alle hoofdpunten

## Korte eindintuitie

Als je QUIC in een zin moet samenvatten:

QUIC probeert een modern internetprotocol te zijn dat de sterke kanten van transport en security samenbrengt, beter omgaat met parallelle applicatiestromen, en mobieler is dan klassieke TCP-verbindingen.

Dat is precies waarom QUIC zo relevant is voor het moderne web.
