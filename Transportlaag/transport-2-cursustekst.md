# Deel 7 - Transport 2: RTP en TCP

In dit deel bekijken we twee grote thema's uit de transportlaag: **RTP** voor realtime multimedia en **TCP** voor betrouwbare byte-stream communicatie. Die twee lijken op het eerste gezicht ver uit elkaar te liggen, maar ze vertrekken eigenlijk van dezelfde vraag: _welke service heeft de applicatie nodig van het netwerk?_

Bij video, audio en live communicatie is **timing** vaak belangrijker dan perfectie. Bij webverkeer, bestanden en veel applicatieprotocollen is **betrouwbaarheid** belangrijker dan snelheid op het niveau van individuele packets. Daarom krijg je in de praktijk twee heel verschillende oplossingen.

## RTP: transport voor realtime media

RTP staat voor **Real-time Transport Protocol** en is beschreven in RFC 3550. Het is ontworpen voor toepassingen zoals audio, video en andere multimediastromen. Zulke applicaties hebben meestal gelijkaardige noden: ze moeten data op tijd afleveren, verschillende mediastromen op elkaar afstemmen, en kunnen vaak beter omgaan met een klein beetje verlies dan met extra vertraging.

Het belangrijke idee is dus: RTP probeert niet om elk packet koste wat het kost correct te krijgen, maar helpt de ontvanger om **media correct af te spelen**.

### waarom bestaat dit?

Bij realtime media heb je vaak meerdere stromen tegelijk:

- audio
- video
- metadata of timing-info

Die stromen moeten met elkaar **gesynchroniseerd** blijven. Als het beeld een halve seconde achterloopt op het geluid, dan voelt de applicatie meteen slecht aan. RTP is gemaakt om dat soort problemen op een generieke manier aan te pakken.

### Waar zit RTP in de stack?

Dat is geen volledig eenduidig verhaal. Je kan argumenteren dat RTP bij de **applicatielaag** zit, omdat het typisch in user space draait en bovenop UDP wordt gebruikt. Maar je kan ook argumenteren dat het op de **transportlaag** lijkt, omdat het een algemene dienst aanbiedt die niet gebonden is aan een specifieke applicatie.

De praktische realiteit is meestal:

client app  
\--> RTP library  
\--> UDP  
\--> IP  
\--> netwerk

Dus conceptueel zit RTP een beetje tussen applicatielaag en transportlaag in.

### Hoe werkt RTP?

Aan de zendkant neemt een RTP-library verschillende mediastromen binnen, zoals audio of video. Die worden verpakt in RTP-packets en daarna meestal via **UDP** verzonden. Aan de ontvangende kant haalt de RTP-library die data terug uit de packets, ordent alles correct en geeft het door aan de afspeelcomponent.

RTP doet dus vooral drie dingen:

- structuur geven aan realtime data
- timing-info meesturen
- de ontvanger helpen met volgorde en synchronisatie

## Waarom RTP op UDP bouwt

Dit is een kernidee.

Bij video of audio is **retransmissie** vaak niet handig. Als een videoframe te laat aankomt, dan is het meestal al waardeloos: het afspeelmoment is voorbij. Dan heb je meer aan een klein stukje verlies dan aan extra wachttijd.

Bij video moet je ook niet altijd elk frame exact hebben. Onze zintuigen zijn vrij tolerant. Een ontbrekend frame kan soms genegeerd of geinterpoleerd worden. Met andere woorden: de ontvanger kan een redelijke gok maken op basis van de frames ervoor en erna.

UDP past daar goed bij omdat het:

- geen ingebouwde retransmissie afdwingt
- geen connection setup nodig heeft
- weinig overhead heeft
- multicast makkelijker ondersteunt

### belangrijke begrippen

- **retransmissie**: een packet opnieuw versturen als het verloren ging
- **interpolatie**: ontbrekende info schatten op basis van omliggende data
- **multicast**: een stream efficient naar meerdere ontvangers sturen

### belangrijk voor examen

De hoofdreden waarom RTP meestal bovenop UDP draait, is dat **te laat vaak erger is dan verloren** bij realtime media.

## Wat zit er in een RTP-packet?

RTP voegt een header toe waarmee de ontvanger beter begrijpt hoe de media moeten geinterpreteerd worden.

Belangrijke velden zijn:

- **Version**: welke RTP-versie gebruikt wordt
- **P**: of er padding aanwezig is
- **X**: of er een extension header is
- **CC**: hoeveel contributing sources vermeld worden
- **M**: een marker-bit met profielspecifieke betekenis, bijvoorbeeld het begin van een frame
- **Type**: welk encoding-algoritme gebruikt werd
- **Sequence number**: een oplopend nummer om verlies en volgorde te detecteren
- **Timestamp**: wanneer de eerste byte van deze media-eenheid werd gegenereerd
- **SSRC**: synchronisation source, dus de identificatie van de stream
- **CSRC**: contributing sources, typisch nuttig als data van meerdere bronnen gemengd werd

### intuition

De **sequence number** zegt: "welk packet komt na welk?"  
De **timestamp** zegt: "wanneer hoort deze media afgespeeld te worden?"

Dat verschil is belangrijk. Volgorde alleen is niet genoeg. Voor media moet je ook weten **op welk moment** data hoort.

## RTCP: control naast RTP

Naast RTP heb je **RTCP**, het Real-time Control Protocol. Dat hoort erbij en zorgt voor controle- en feedbackinformatie.

RTCP helpt onder meer bij:

- feedback over delay en jitter
- bandbreedte- en congestie-info
- synchronisatie tussen verschillende stromen
- leesbare namen voor bronnen

RTP draagt dus de mediadata, RTCP helpt bij het **monitoren en afstemmen** van die sessie.

## Jitter en playback

Packets doen er niet altijd even lang over om over het netwerk te reizen. Zelfs als de gemiddelde delay ongeveer hetzelfde blijft, kan de **variatie** in die delay groot zijn. Dat noemen we **jitter**.

Jitter is gevaarlijk voor multimedia omdat het de afspeelkwaliteit verstoort. Packets kunnen te laat komen, waardoor audio kan haperen of video schokkerig wordt.

### Hoe vangen we jitter op?

Met een **buffer** aan de ontvanger. Die wacht een klein beetje voordat de data afgespeeld wordt, zodat kleine timingverschillen opgevangen kunnen worden.

Maar daar zit een trade-off in:

- een grotere buffer vermindert verlies door late packets
- een grotere buffer verhoogt ook de totale vertraging

Dus je moet een goed **playback point** kiezen: het moment waarop je begint afspelen.

### Live media versus stored streaming

Hier is de vrijheid niet hetzelfde:

- bij **live media** wil je de vertraging zo klein mogelijk houden
- bij **stored streaming** kan je meer bufferen, omdat een paar seconden extra wachttijd vaak aanvaardbaar zijn

Dat betekent dat je voor live toepassingen minder speelruimte hebt: je kan niet zomaar blijven wachten op late packets.

### belangrijk voor examen

Het playback point is altijd een afweging tussen:

- **lage delay**
- **weinig packet loss door te late aankomst**

## TCP: betrouwbare byte-stream service

Na RTP schakelen we over naar TCP. TCP staat voor **Transmission Control Protocol** en is het klassieke transportprotocol voor betrouwbare communicatie op het internet.

De kernbelofte van TCP is:

- een **betrouwbare**
- **end-to-end**
- **byte-stream**

bovenop een netwerk dat zelf onbetrouwbaar kan zijn.

Dat is meteen het grote verschil met RTP/UDP. TCP wil verbergen dat het onderliggende netwerk packets kan verliezen, dupliceren, vertragen of in een andere volgorde afleveren.

## De TCP transport entity

De software die TCP implementeert heet de **transport entity**. Die zit vaak in de kernel van het besturingssysteem, maar dat hoeft niet per se.

Die transport entity is verantwoordelijk voor:

- byte-stream opdelen in segmenten
- betrouwbare transmissie
- segmenten opnieuw samenstellen
- efficient gebruik van bandbreedte
- congestie vermijden

### intuition

Voor een applicatie voelt TCP alsof je gewoon bytes in een stroom schrijft en aan de andere kant in dezelfde volgorde terugkrijgt. Alles wat met packets, verlies en hertransmissie te maken heeft, probeert TCP voor jou af te handelen.

## Poorten en verbindingen

TCP gebruikt **ports** als adressen op transportniveau. Dat zijn de TSAP-adressen. Poorten onder 1024 zijn gereserveerd voor bekende diensten.

Een TCP-verbinding is:

- **full duplex**: beide richtingen tegelijk mogelijk
- tussen exact **twee sockets**
- een verbinding tussen twee endpoints, geen multicast of broadcast

Belangrijk is ook dat TCP een **byte-stream** aanbiedt, geen message service. Als een applicatie vier blokken wegschrijft, wil dat niet zeggen dat die aan de andere kant in exact dezelfde blokken terug opduiken. TCP mag zelf beslissen hoe het de data segmenteert.

### belangrijke begrippen

- **socket**: combinatie van IP-adres en poort
- **full duplex**: beide richtingen tegelijk actief
- **byte-stream**: continue stroom bytes zonder ingebouwde berichtgrenzen

### belangrijk voor examen

TCP bewaart **volgorde en betrouwbaarheid**, maar **niet de oorspronkelijke berichtgrenzen** van de applicatie.

## Sequence numbers en segmentatie

In TCP krijgt **elke byte** een volgnummer. Dat is belangrijk: sequence numbers slaan dus niet enkel op packets, maar op posities in de byte-stream.

Een TCP-segment heeft een beperkte grootte. Dat hangt deels af van de TCP- en IP-headers, maar ook van beperkingen lager in de stack, zoals de MTU van Ethernet. Omdat hersegmenteren inefficient is, probeert TCP via **MTU discovery** te leren hoe groot segmenten best mogen zijn.

## De TCP-header

De TCP-header bevat alle informatie die nodig is om data correct af te leveren en de verbinding te beheren.

### Source en destination port

Deze velden zeggen van welke transportpoort het segment komt en naar welke transportpoort het moet.

Een verbinding wordt in de praktijk geidentificeerd door:

- source IP
- source port
- destination IP
- destination port
- protocol

### Sequence en acknowledgment number

Het **sequence number** is het volgnummer van de **eerste byte** in het segment.

Het **acknowledgment number** zegt welke byte de ontvanger **als volgende verwacht**. Als je ACK = 2048 stuurt, betekent dat: "ik heb alles ontvangen tot en met byte 2047".

Dit is cumulatief acknowledgen. Je hoeft dus niet voor elke byte een apart ontvangstbewijs te sturen.

### Header length

TCP-headers hebben variabele lengte, omdat er opties kunnen toegevoegd worden. Daarom moet in de header staan waar de eigenlijke data begint.

### Flags

Belangrijke flags zijn:

- **ACK**: acknowledgment field is geldig
- **SYN**: wordt gebruikt bij verbinding opzetten
- **FIN**: wordt gebruikt bij verbinding sluiten
- **RST**: abrupt resetten van een verbinding
- **PSH**: data snel doorgeven, niet onnodig bufferen
- **URG**: urgent pointer is geldig
- **ECE** en **CWR**: gebruikt voor Explicit Congestion Notification

### Window size

Dit veld zegt hoeveel bytes de ontvanger momenteel nog kan accepteren vanaf het punt dat al bevestigd is. Dat is dus de kern van **flow control**.

### Checksum

TCP gebruikt verplicht een checksum om fouten in header en data te detecteren.

### Options

TCP kan extra info meesturen via opties in **TLV-formaat**: type, length, value. Dat maakt uitbreiding mogelijk zonder het basisformaat volledig te veranderen.

## Verbinding opzetten: de 3-way handshake

TCP gebruikt een **3-way handshake** om een verbinding correct op te zetten.

De basisflow is:

client  
\--> `SYN, SEQ = x`  
server  
\--> `SYN, ACK, SEQ = y, ACK = x + 1`  
client  
\--> `ACK, ACK = y + 1`

### waarom drie stappen?

Omdat beide kanten moeten weten:

- dat de andere kant bereikbaar is
- dat sequence numbers bekend zijn
- dat beide richtingen klaar zijn voor data

Met twee stappen zou die kennis niet symmetrisch genoeg zijn.

### SYN en ACK bits

Bij een normale succesvolle handshake krijg je:

- stap 1: `SYN=1, ACK=0`
- stap 2: `SYN=1, ACK=1`
- stap 3: `SYN=0, ACK=1`

### belangrijk voor examen

De betekenis van de drie handshake-berichten en de waarden van **SYN** en **ACK** zijn klassieke examenvragen.

## Verbinding afsluiten

Een TCP-verbinding afsluiten is subtieler dan opzetten. Je moet het eigenlijk zien als **twee simplex-richtingen** die apart gesloten worden.

Als een kant een `FIN` stuurt en die `FIN` bevestigd wordt, dan mag die richting geen nieuwe data meer sturen. In de andere richting kan communicatie nog even doorgaan.

Daarna komt de **TIME_WAIT**-fase. TCP wacht daar nog een tijd, typisch tweemaal de maximale packet lifetime, zodat oude of vertraagde segmenten uit het netwerk verdwenen zijn voordat de verbinding echt weg mag.

### waarom bestaat TIME_WAIT?

Om te vermijden dat oude packets van een vorige verbinding per ongeluk in een nieuwe verbinding met dezelfde socket-combinatie terechtkomen.

## TCP als state machine

TCP wordt vaak als een **state machine** bekeken met 11 toestanden. Dat klinkt theoretisch, maar het is praktisch heel nuttig. In elke toestand zijn alleen bepaalde gebeurtenissen legaal.

Voorbeelden van toestanden zijn:

- `CLOSED`
- `LISTEN`
- `SYN_SENT`
- `SYN_RCVD`
- `ESTABLISHED`
- `FIN_WAIT_1`
- `FIN_WAIT_2`
- `CLOSE_WAIT`
- `LAST_ACK`
- `TIME_WAIT`

Die toestanden maken duidelijk dat TCP niet gewoon "verbonden of niet verbonden" is. Er zitten overgangsfasen tussen die belangrijk zijn voor betrouwbaarheid.

## De sliding window van TCP

TCP gebruikt een **sliding window** om meerdere bytes of segmenten tegelijk "in flight" te hebben zonder voor elk segment apart te moeten wachten.

Een belangrijk idee uit de slides is dat TCP **acknowledgment** en **buffer allocation** van elkaar loskoppelt.

De ontvanger doet twee dingen:

- bevestigen welke bytes al ontvangen zijn
- melden hoeveel extra ruimte nog beschikbaar is

Dat zie je in de `ACK` en de `WIN`-waarde.

### Voorbeeld

Als een receiver `ACK = 4096, WIN = 0` stuurt, betekent dat:

- alles tot byte 4095 is ontvangen
- maar de ontvanger heeft op dit moment geen extra buffer vrij

De zender mag dan dus tijdelijk niets nieuws sturen, zelfs al is de overdracht correct verlopen.

### belangrijk voor examen

TCP gebruikt de sliding window niet alleen voor betrouwbaarheid, maar ook voor **flow control**.

## Receiver control en window probes

Als de advertised window 0 wordt, mag de zender normaal geen gewone data meer sturen. Toch zijn er twee uitzonderingen:

- **urgent traffic**
- **window probes**

Window probes zijn nodig om een deadlock te vermijden. Stel dat de receiver later opnieuw buffer vrij heeft en een window update verstuurt, maar die update raakt verloren. Dan zou de zender anders voor altijd blijven wachten. Een window probe laat de zender checken of er intussen weer plaats is.

## Tinygram syndrome en delayed ACKs

Als je met TCP telkens extreem kleine stukjes data verstuurt, zoals een karakter per keer, dan is de overhead gigantisch. Je betaalt dan voortdurend voor IP-header, TCP-header en acknowledgments, terwijl de echte payload miniem is.

Dat is het idee achter **tinygram syndrome**.

Een eerste tegenmaatregel is **delayed acknowledgments**. De ontvanger wacht dan kort met ACK'en:

- komt er snel data terug in de andere richting, dan kan de ACK gepiggybacked worden
- gebeurt dat niet, dan wordt na timeout alsnog een ACK gestuurd

Zo verminder je het aantal kleine controlepakketten.

## Nagle's algorithm

**Nagle's algorithm** probeert kleine segmenten te vermijden aan de zenderkant. Het idee is simpel: zolang er nog een onbevestigd segment onderweg is, buffer je nieuwe kleine data tot je genoeg hebt voor een groter segment, of tot de vorige data geacknowledge-d is.

Dat werkt goed voor veel gewone toepassingen, maar slecht voor interactieve of realtime toepassingen.

### voorbeelden waar dit slecht werkt

- realtime systemen
- online games
- andere latency-gevoelige interacties

Daar wil je vaak liever onmiddellijk verzenden dan wachten op batching.

### link met `TCP_NODELAY`

Met `TCP_NODELAY` kan je dat bufferend gedrag uitschakelen en onmiddellijk sturen.

## Silly Window Syndrome en Clarke's algorithm

**Silly Window Syndrome** ontstaat als de zender grote blokken wil sturen, maar de ontvanger telkens maar heel kleine stukjes buffer vrijmaakt en adverteert. Dan krijg je opnieuw veel kleine, inefficiente transmissies.

**Clarke's algorithm** pakt dat aan aan de ontvangende kant:

- wacht met window updates
- tot er minstens een maximum segment past
- of tot de buffer ongeveer half leeg is

Dat is complementair aan Nagle:

- Nagle voorkomt dat de zender kleine segmenten uitstuurt
- Clarke voorkomt dat de ontvanger om kleine segmenten vraagt

## Timers in TCP

TCP gebruikt meerdere timers. Die zijn essentieel voor betrouwbaarheid en voor het vermijden van vastgelopen situaties.

### Retransmission timeout (RTO)

De **RTO** bepaalt hoe lang TCP wacht op een ACK voordat het een segment opnieuw verstuurt.

Als die timeout:

- **te kort** is, krijg je onnodige retransmissies
- **te lang** is, reageert TCP traag op echt verlies

Dus TCP moet die waarde dynamisch aanpassen.

### SRTT en RTTVAR

TCP schat de round-trip time met een **Smoothed RTT (SRTT)**. Dat is een geeffend gemiddelde van gemeten RTT's.

Maar alleen een gemiddelde is niet genoeg. Als de delay heel wisselend is, moet TCP daar rekening mee houden. Daarom gebruikt het ook **RTTVAR**, de variatie in RTT.

De RTO wordt bepaald door beide:

- geschatte gemiddelde RTT
- geschatte variatie op RTT

Het idee is: hoe onvoorspelbaarder het netwerk, hoe voorzichtiger je moet zijn met retransmissie.

### Persistence timer

Deze timer hoort bij het probleem van een window van 0. Als de timer afloopt, verstuurt TCP een **window probe**.

### Keep-alive timer

Die controleert of de andere host nog leeft als er lang niets ontvangen werd. Deze feature is optioneel, omdat ze soms ook gezonde verbindingen kan beeindigen.

### TIME_WAIT timer

Die hoort bij de afsluitfase van de verbinding en laat oude segmenten uitsterven voor de connectie definitief verdwijnt.

### belangrijk voor examen

Je moet het verschil kennen tussen:

- **RTO**: voor verliesdetectie en retransmissie
- **persistence timer**: voor window probes bij `WIN = 0`
- **TIME_WAIT timer**: voor veilig afsluiten

## Congestiecontrole in TCP

Flow control en congestion control zijn niet hetzelfde.

- **flow control** beschermt de ontvanger
- **congestion control** beschermt het netwerk

Dat onderscheid is cruciaal.

TCP gebruikt voor congestiecontrole een **congestion window**. Dat bepaalt hoeveel bytes de zender tegelijk in het netwerk mag hebben.

De effectieve zendlimiet is altijd:

- minimum van de **receiver window**
- en de **congestion window**

### waarom twee windows?

Omdat er twee verschillende bottlenecks kunnen zijn:

- de ontvanger kan te traag zijn
- het netwerk kan te vol zitten

TCP moet met beide rekening houden.

## Waarom gebruikt TCP packet loss als signaal?

Historisch koos TCP ervoor om **packet loss** te gebruiken als impliciet congestiesignaal. Dat was slim omdat men de packetformaten niet grondig wou veranderen. Zo kon de oplossing snel uitgerold worden.

Op bekabelde netwerken werkt dat redelijk goed:

- packets gaan zelden verloren zonder congestie
- packets worden bijna zeker gedropt als routers vol raken

Op draadloze netwerken is dat minder ideaal, omdat verlies daar ook andere oorzaken kan hebben.

## AIMD: additive increase, multiplicative decrease

De basislogica van klassieke TCP-congestiecontrole is **AIMD**:

- bij geen congestie: voorzichtig verhogen
- bij congestie: sterk verlagen

Dat geeft een systeem dat vrij goed convergeert naar een eerlijke verdeling van bandbreedte.

### intuition

Het netwerk in congestie brengen gaat snel. Herstellen duurt langer. Daarom moet de stijging voorzichtig zijn en de daling agressief.

## Het ACK clock-principe

Een belangrijk praktisch probleem is dat een host data heel snel in een snelle link kan dumpen, terwijl verderop een tragere link zit. Dan krijg je bursts die zich opstapelen in routers en andere flows hinderen.

Het **ACK clock**-idee lost dat op. Nieuwe data wordt alleen het netwerk ingestuurd aan het tempo waarop ACKs terugkeren. En die ACKs keren terug aan het tempo waarop de bottlenecklink data kan verwerken.

Dus:

snelle zender  
\--> stuurt eerste burst  
\--> bottleneck vertraagt doorvoer  
\--> ACKs komen traag terug  
\--> zender past zijn injectietempo automatisch aan

Dat maakt de stroom veel gelijkmatiger.

### belangrijk voor examen

De **ACK clock** zorgt ervoor dat de zender zich afstemt op de traagste schakel in het pad, en dus geen grote bursts blijft injecteren.

## Slow start

Pure AIMD is te traag om van nul naar een bruikbare snelheid te gaan op een lege verbinding. Daarom gebruikt TCP eerst **slow start**.

Die naam is eigenlijk misleidend, want het groeit eerst juist **exponentieel**.

Het idee:

- start met een kleine congestion window
- voor elke ACK groeit de window snel
- daardoor verdubbelt de hoeveelheid data ongeveer per RTT

Dat gaat door tot er tekenen van congestie opduiken, of tot een drempel bereikt wordt.

## Slow-start threshold

Slow start mag niet onbeperkt blijven doorgaan, want dan vul je routerqueues te snel en krijg je congestie.

Daarom is er een **slow-start threshold**:

- initieel hoog gezet
- vaak logisch gekozen in relatie tot de flow-control window
- na congestie opnieuw ingesteld op ongeveer de helft van de huidige congestion window

Zolang je onder die threshold zit, gebruik je exponentiele groei. Eens erboven schakel je over op additive increase.

## Tahoe en Reno

**TCP Tahoe** was de vroege variant waarin slow start centraal zat. Bij packet loss viel Tahoe vrij hard terug en begon het opnieuw op te bouwen.

**TCP Reno** verbeterde dat met **fast recovery**.

## Fast recovery

Bij fast recovery hoeft TCP niet altijd helemaal opnieuw te starten vanaf een minimale congestion window. Via **duplicate ACKs** kan de zender afleiden dat er waarschijnlijk een packet verloren ging, terwijl latere packets wel nog aankomen.

Dan kan TCP:

- het verloren packet snel hertransmitteren
- terugvallen tot rond de nieuwe threshold
- daarna opnieuw additive increase doen

Dat is veel efficienter dan van nul herbeginnen.

De beperking van klassieke fast recovery is wel dat het vooral goed werkt bij **een packet loss**.

## SACK: selective acknowledgments

Daarom kwam later **SACK**.

Met gewone cumulatieve ACKs weet de zender enkel tot waar alles in orde is. Met SACK kan de ontvanger extra info meesturen over **welke hogere byte-ranges al wel ontvangen zijn**.

Dat laat de zender toe om gerichter te retransmitteren bij meerdere verliezen in een venster.

### waarom is dit handig?

Zonder SACK weet de zender: "ergens is iets mis".  
Met SACK weet de zender: "deze stukken heb je al, die specifieke gaten nog niet".

### belangrijk voor examen

SACK verbetert herstel bij **multiple packet loss** zonder de basiscompatibiliteit van TCP te breken, omdat het via header options werkt.

## ECN: explicit congestion notification

Nog een verdere stap is **ECN**. Daarbij wacht je niet tot een packet verloren gaat, maar laat je routers expliciet aangeven dat congestie eraan komt.

In dat model:

- IP markeert packets als ECN-capable
- routers zetten een congestiemarkering wanneer buffers vollopen
- TCP gebruikt dan de `ECE` en `CWR` flags om daarop te reageren

Dat is eleganter dan packet loss als signaal, maar het vraagt ondersteuning van beide eindhosts en van het netwerk ertussen. Daardoor is het minder breed doorgebroken dan bijvoorbeeld SACK.

## Samenvatting van het grote verschil tussen RTP en TCP

RTP en TCP lossen totaal verschillende problemen op.

### RTP

- bedoeld voor realtime media
- draait meestal bovenop UDP
- accepteert dat sommige packets verloren mogen gaan
- gebruikt sequence numbers en timestamps voor juiste afspeelvolgorde en timing
- werkt samen met RTCP voor feedback en synchronisatie

### TCP

- bedoeld voor betrouwbare byte-stream communicatie
- gebruikt sequence numbers, ACKs en sliding windows
- doet flow control en congestion control
- gebruikt timers, retransmissie en state management
- probeert het netwerk efficient en eerlijk te gebruiken

## Wat je echt moet kennen

### belangrijk voor examen

- Waarom RTP meestal op UDP draait
- Het verschil tussen **sequence number** en **timestamp** in RTP
- Wat **jitter** is en hoe buffering het opvangt
- De betekenis van de belangrijkste TCP-header fields
- De **3-way handshake** en de SYN/ACK-combinaties
- Waarom TCP een **byte-stream** is en geen message protocol
- Hoe de **sliding window** werkt, inclusief `ACK` en `WIN`
- Het verschil tussen **flow control** en **congestion control**
- Het doel van **RTO**, **persistence**, en **TIME_WAIT**
- Het idee achter **AIMD**, **slow start**, **threshold**, en **fast recovery**
- Waarom **SACK** nuttig is
- Het basisidee van **ECN**

## Korte eindintuitie

Als je alles tot een zin zou reduceren:

- **RTP** zegt: _"speel media bruikbaar af, ook als niet elk packet perfect aankomt."_
- **TCP** zegt: _"lever alle bytes correct, in volgorde, en doe dat zonder het netwerk kapot te duwen."_

Dat verschil in prioriteit verklaart bijna alle ontwerpkeuzes uit deze slides.
