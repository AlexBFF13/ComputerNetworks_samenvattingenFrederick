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

### Waarom heeft RTP geen flow control en geen congestion control?

Dit is een klassieke examenvraag. De redenen zijn fundamenteel:

**Geen flow control** omdat:

- bij realtime media is de zendsnelheid bepaald door de **mediabron** (camera, microfoon), niet door de ontvanger
- als de ontvanger te traag is, is het beter om packets te droppen dan de bron te vertragen
- vertragen van de bron zou de live-ervaring vernietigen (stilte, bevroren beeld)

**Geen congestion control** omdat:

- congestion control (zoals AIMD) zou de zendsnelheid verlagen bij verlies, maar bij live media is een constante bitrate vaak essentieel voor bruikbare kwaliteit
- als je de bitrate verlaagt, daalt de kwaliteit voor alle ontvangers
- bovendien gebruikt RTP vaak **multicast**, en congestion control is veel moeilijker bij multicast omdat je met meerdere ontvangers op verschillende paden te maken hebt

### RTP en multicast

RTP is expliciet ontworpen om goed samen te werken met **multicast**. Bij multicast stuurt de zender packets naar een groepsadres, en het netwerk dupliceert die packets waar nodig.

Dit is fundamenteel anders dan TCP:

- TCP is **unicast** (punt-naar-punt) en kan geen multicast
- RTP bovenop UDP kan wel multicast, wat het ideaal maakt voor:
  - live videoconferenties met meerdere deelnemers
  - live sportstreaming naar duizenden kijkers
  - internet radio

Bij multicast is congestion control extra moeilijk omdat elke ontvanger op een ander pad zit met andere congestieniveaus. Dit is nog een reden waarom RTP dit bewust niet doet.

### Het probleem van RTP zonder congestion control

Hoewel RTP bewust geen congestion control heeft, brengt dit wel risico's mee:

- **TCP starvation**: als RTP/UDP-verkeer en TCP-verkeer dezelfde links delen, zal TCP zijn snelheid halveren bij verlies (AIMD), maar RTP stuurt gewoon door. TCP wordt dan structureel benadeeld.
- **Congestion collapse**: als veel RTP-streams tegelijk actief zijn zonder enige terugkoppeling, kan het netwerk vollopen met nutteloze packets die toch te laat aankomen.

### belangrijk voor examen

De hoofdreden waarom RTP meestal bovenop UDP draait, is dat **te laat vaak erger is dan verloren** bij realtime media. RTP heeft bewust **geen flow control** (mediabron bepaalt het tempo) en **geen congestion control** (constant bitrate nodig + multicast maakt het onpraktisch).

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

De TCP-header bevat alle informatie die nodig is om data correct af te leveren en de verbinding te beheren. Dit is circa 2 keer gevraagd op het examen, typisch als "leg alle velden uit".

De TCP-header is minimaal **20 bytes** lang (zonder opties) en maximaal **60 bytes** (met opties).

### Volledige veldoverzicht

| Veld | Grootte | Functie |
|---|---|---|
| **Source Port** | 16 bits | Identificeert het verzendende proces (TSAP) |
| **Destination Port** | 16 bits | Identificeert het ontvangende proces (TSAP) |
| **Sequence Number** | 32 bits | Volgnummer van de **eerste byte** in dit segment |
| **Acknowledgment Number** | 32 bits | Volgnummer van de volgende byte die de ontvanger **verwacht** (cumulatief) |
| **Header Length / Data Offset** | 4 bits | Lengte van de TCP-header in 32-bit woorden (geeft aan waar data begint) |
| **Reserved** | 3 bits | Gereserveerd voor toekomstig gebruik, moet 0 zijn |
| **Flags** | 9 bits | Controlbits (zie hieronder) |
| **Window Size** | 16 bits | Hoeveel bytes de ontvanger nog kan accepteren (flow control) |
| **Checksum** | 16 bits | Verplichte foutdetectie over header + data + pseudo-header |
| **Urgent Pointer** | 16 bits | Offset naar het einde van urgente data (enkel geldig als URG=1) |
| **Options** | variabel | Optionele uitbreidingen (MSS, Window Scale, SACK, timestamps, ...) |

### Source en destination port

Deze velden identificeren de communicerende processen. Een TCP-verbinding wordt uniek geidentificeerd door de **5-tuple**:

- source IP
- source port
- destination IP
- destination port
- protocol (TCP)

Poorten onder 1024 zijn **well-known ports** (HTTP=80, HTTPS=443, SSH=22, etc.).

### Sequence en acknowledgment number

Het **sequence number** is het volgnummer van de **eerste byte** in het segment. Bij de 3-way handshake kiest elke kant een **Initial Sequence Number (ISN)**.

Het **acknowledgment number** zegt welke byte de ontvanger **als volgende verwacht**. Als je ACK = 2048 stuurt, betekent dat: "ik heb alles ontvangen tot en met byte 2047".

Dit is **cumulatief acknowledgen**. Een ACK bevestigt impliciet alle bytes tot aan het genoemde nummer. Je hoeft dus niet voor elke byte een apart ontvangstbewijs te sturen.

### Header length (Data Offset)

TCP-headers hebben variabele lengte, omdat er opties kunnen toegevoegd worden. Dit veld geeft de headerlengte aan in eenheden van 32 bits (4 bytes). Minimum waarde is 5 (= 20 bytes), maximum is 15 (= 60 bytes).

### Flags (controlbits)

De negen flags in de TCP-header:

| Flag | Betekenis | Wanneer gebruikt |
|---|---|---|
| **URG** | Urgent pointer is geldig | Bij urgente data die voorrang moet krijgen |
| **ACK** | Acknowledgment number is geldig | In elk segment na de eerste SYN |
| **PSH** | Push -- geef data meteen door aan applicatie | Bij interactieve data, niet onnodig bufferen |
| **RST** | Reset -- verbreek de verbinding abrupt | Bij fouten, onverwachte segmenten, of weigering |
| **SYN** | Synchronize -- start verbindingsopbouw | Bij de 3-way handshake |
| **FIN** | Finish -- initieer verbindingsafsluiting | Bij graceful connection close |
| **ECE** | ECN-Echo -- er is congestie gedetecteerd | Ontvanger echo't ECN-markering van router |
| **CWR** | Congestion Window Reduced | Zender bevestigt dat hij cwnd heeft verlaagd |
| **NS** | ECN-nonce concealment protection | Voorkomt dat ontvanger ECN-markeringen verbergt |

### Window size

Dit veld is 16 bits en zegt hoeveel bytes de ontvanger momenteel nog kan accepteren vanaf het punt dat al bevestigd is. Dit is de kern van **flow control** via **window advertisement**.

Omdat 16 bits slechts tot 65535 gaat, kan met de **Window Scale** optie (ingesteld tijdens de handshake) de waarde verschoven worden tot veel grotere windows (tot 1 GB).

### Checksum

TCP gebruikt verplicht een checksum om fouten in header en data te detecteren. De checksum wordt berekend over:

- een IP **pseudo-header** (source IP, destination IP, protocol, TCP-lengte)
- de volledige TCP-header
- de TCP-data

De pseudo-header zorgt ervoor dat een segment dat bij de verkeerde host aankomt, gedetecteerd kan worden.

### Urgent Pointer

Als URG=1, wijst dit veld naar de **offset** binnen de data waar de urgente data eindigt. Data voor dat punt moet met voorrang aan de applicatie geleverd worden. Dit mechanisme wordt in de praktijk weinig gebruikt.

### Options

TCP kan extra info meesturen via opties in **TLV-formaat** (Type, Length, Value). Belangrijke opties zijn:

- **MSS (Maximum Segment Size)**: maximale segmentgrootte die de peer kan ontvangen (enkel in SYN)
- **Window Scale**: vermenigvuldigingsfactor voor het Window Size-veld (enkel in SYN)
- **SACK Permitted**: geeft aan dat SACK ondersteund wordt (enkel in SYN)
- **SACK blocks**: specificeert welke byte-ranges al ontvangen zijn (bij out-of-order)
- **Timestamps**: voor nauwkeurige RTT-meting en PAWS (Protection Against Wrapped Sequence numbers)

### belangrijk voor examen

Je moet elk TCP-headerveld kunnen uitleggen: de functie van **sequence number** vs **acknowledgment number** (cumulatief), de betekenis van de **flags** (SYN, ACK, FIN, RST, PSH, URG, ECE, CWR), en hoe **Window Size** flow control implementeert.

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

## TCP Flow Control in detail

TCP flow control is het **meest gevraagde transport-onderwerp op het examen** (circa 10 keer gevraagd). Het verdient daarom een grondige behandeling.

### Window advertisement

De ontvanger communiceert zijn beschikbare bufferruimte via het **Window Size**-veld in de TCP-header. Dit heet de **advertised window** of **receiver window (rwnd)**.

Het mechanisme werkt als volgt:

1. De ontvanger heeft een buffer van een bepaalde grootte (bijvoorbeeld 4096 bytes)
2. Als de applicatie data uit de buffer leest, komt er ruimte vrij
3. Bij elk ACK meldt de ontvanger hoeveel bytes hij nog kan accepteren: `WIN = buffergrootte - (ontvangen maar nog niet gelezen data)`
4. De zender mag nooit meer onbevestigde data in het netwerk hebben dan wat de ontvanger adverteert

### Voorbeeld van buffer management

Stel de ontvanger heeft een buffer van 4096 bytes:

```text
Stap 1: Zender stuurt 2048 bytes, ontvanger ACK't met WIN = 2048
        (buffer half vol, applicatie heeft nog niets gelezen)

Stap 2: Zender stuurt nog 2048 bytes, ontvanger ACK't met WIN = 0
        (buffer helemaal vol!)

Stap 3: Applicatie leest 2048 bytes uit de buffer
        Ontvanger stuurt window update: WIN = 2048

Stap 4: Zender mag weer 2048 bytes sturen
```

### Het deadlock-probleem

Het deadlock-scenario is een klassieke examenvraag:

1. De ontvanger stuurt `ACK, WIN = 0` (buffer is vol)
2. De zender stopt met zenden en wacht op een window update
3. De applicatie aan de ontvangstzijde leest data, en de ontvanger stuurt een **window update** met `WIN > 0`
4. **Die window update gaat verloren in het netwerk**
5. Nu wacht de zender op een window update die nooit komt
6. En de ontvanger denkt dat hij die update al gestuurd heeft

Resultaat: **deadlock** -- beide kanten wachten op elkaar.

### Window probes als oplossing

Om dit deadlock te voorkomen gebruikt TCP een **persistence timer**:

- als de zender een `WIN = 0` ontvangt, start hij een timer
- als de timer afloopt, stuurt hij een **window probe**: een segment met 1 byte data
- de ontvanger moet hierop reageren met een ACK dat de huidige window size bevat
- als de window nog steeds 0 is, herstart de timer (met exponential backoff)
- als de window weer > 0 is, kan de zender weer normaal data sturen

Er zijn twee uitzonderingen op de regel "stuur niets bij WIN = 0":

- **urgent traffic**: data met de URG-flag mag altijd gestuurd worden
- **window probes**: om het deadlock te doorbreken

### Usable window

De effectieve hoeveelheid data die de zender mag sturen is:

```text
usable window = min(rwnd, cwnd) - data al in flight
```

Waar:

- **rwnd** = receiver window (flow control, beschermt de ontvanger)
- **cwnd** = congestion window (congestion control, beschermt het netwerk)

De zender respecteert altijd het **minimum** van beide. Dit is het fundamentele samenspel tussen flow control en congestion control.

### belangrijk voor examen

TCP flow control gebruikt **window advertisement** door de ontvanger. Bij `WIN = 0` kan een **deadlock** ontstaan als de window update verloren gaat. De **persistence timer** met **window probes** voorkomt dit. De effectieve zendlimiet is altijd `min(rwnd, cwnd)`.

## Tinygram syndrome en delayed ACKs

Als je met TCP telkens extreem kleine stukjes data verstuurt, zoals een karakter per keer, dan is de overhead gigantisch. Je betaalt dan voortdurend voor IP-header, TCP-header en acknowledgments, terwijl de echte payload miniem is.

Concreet: een segment met 1 byte nuttige data heeft 20 bytes IP-header + 20 bytes TCP-header = **41 bytes totaal voor 1 byte data**. Dat is minder dan 2,5% efficiëntie.

Dat is het idee achter **tinygram syndrome**.

### Delayed acknowledgments

Een eerste tegenmaatregel is **delayed acknowledgments**. De ontvanger wacht dan kort (typisch **maximaal 200 ms**) met ACK'en:

- komt er snel data terug in de andere richting, dan kan de ACK **gepiggybacked** worden op dat antwoord
- gebeurt dat niet, dan wordt na de delayed ACK timer (max 200 ms) alsnog een kale ACK gestuurd

Waarom is dit nuttig?

- het vermindert het aantal kleine controlepakketten op het netwerk
- piggybacking combineert ACK + data in een enkel segment
- het halveert potentieel het aantal segmenten bij bidirectioneel verkeer

## Nagle's algorithm

**Nagle's algorithm** (RFC 896) probeert kleine segmenten te vermijden aan de **zenderkant**. De regel is eenvoudig:

1. Als er **geen** onbevestigd segment onderweg is: stuur de data meteen, ook al is het klein
2. Als er **wel** een onbevestigd segment onderweg is: buffer nieuwe kleine data tot:
   - je genoeg data hebt om een volledig segment (MSS) te vullen, OF
   - het uitstaande segment geACK'd wordt

Dit betekent dat je nooit meer dan een klein segment tegelijk "in flight" hebt. Zodra de ACK terugkomt, stuur je alle gebufferde data in een keer.

### Waarom werkt Nagle goed voor gewone toepassingen?

Bij typisch interactief gebruik (zoals een shell via telnet/SSH) typt de gebruiker langzaam. Nagle verzamelt toetsaanslagen tot de vorige ACK terugkomt, en stuurt ze dan samen. Het netwerk ziet grotere, efficiëntere segmenten.

### Wanneer werkt Nagle slecht?

- realtime systemen
- online games
- andere latency-gevoelige interacties
- applicaties die meerdere kleine write()-calls doen die samen een logisch geheel vormen

Daar wil je vaak liever onmiddellijk verzenden dan wachten op batching.

### TCP_NODELAY

Met de socket-optie `TCP_NODELAY` kan je Nagle's algorithm **uitschakelen** en elk stukje data onmiddellijk versturen.

## De gevaarlijke interactie tussen Nagle en Delayed ACKs

Dit is een **veelgevraagd examenonderwerp** (circa 4-6 keer gevraagd). De combinatie van Nagle en delayed ACKs kan leiden tot een systematische latency-verhoging.

### Het scenario stap voor stap

```text
1. Client stuurt een klein request (bijv. een HTTP header-deel) naar de server
   → Nagle: "er is nu een onbevestigd segment onderweg, buffer verdere data"

2. Client heeft nog een klein stukje data te sturen (bijv. de rest van het request)
   → Nagle: "wacht op ACK van het eerste segment"

3. Server ontvangt het eerste stuk data
   → Server wacht op meer data voor het volledige request
   → Delayed ACK: "wacht 200 ms op return data om te piggybacking"

4. Resultaat: IMPASSE
   - Client wacht op ACK (Nagle) om de rest te sturen
   - Server wacht op meer data of 200 ms timeout (Delayed ACK) om te ACK'en
   → Systematische 200 ms vertraging per interactie!
```

### Is dit een echte deadlock?

**Nee**, het is geen permanente deadlock. Na maximaal 200 ms loopt de delayed ACK timer af en stuurt de ontvanger alsnog een ACK. Maar die 200 ms per interactie is **onaanvaardbaar** voor interactieve applicaties.

### Oplossingen

- **TCP_NODELAY**: schakel Nagle uit aan de zenderkant
- **TCP_QUICKACK**: schakel delayed ACKs uit aan de ontvangstkant
- **Applicatieontwerp**: gebruik een enkele write()-call in plaats van meerdere kleine writes
- **QUIC/HTTP2**: vermijden dit probleem structureel

### belangrijk voor examen

De interactie Nagle + delayed ACKs veroorzaakt een **systematische 200 ms latency** per kleine interactie. De client wacht op ACK (Nagle), de server wacht op piggybacking-data (delayed ACK). Het is geen permanente deadlock maar een ernstig prestatieprobleem.

## Silly Window Syndrome en Clarke's algorithm

**Silly Window Syndrome (SWS)** is het spiegelprobleem van tinygram, maar dan aan de **ontvangstkant**.

### Hoe ontstaat SWS?

1. De ontvangstbuffer is vol (WIN = 0)
2. De applicatie aan de ontvangstzijde leest 1 byte uit de buffer
3. De ontvanger stuurt een window update: WIN = 1
4. De zender stuurt braaf 1 byte data
5. De buffer is weer vol (WIN = 0)
6. De applicatie leest weer 1 byte...

Resultaat: een eindeloze cyclus van piepkleine segmenten met enorme overhead, net als bij tinygram maar nu veroorzaakt door de **ontvanger**.

### Clarke's algorithm

**Clarke's algorithm** pakt dit aan aan de ontvangende kant:

- wacht met window updates tot een van deze twee voorwaarden geldt:
  - er past minstens een **maximum segment size (MSS)** in de buffer, OF
  - de buffer is minstens **half leeg**

Pas dan adverteer je de nieuwe window.

### Complementariteit

De vier mechanismen vormen samen een complete oplossing:

| Probleem | Oorzaak | Kant | Oplossing |
|---|---|---|---|
| Tinygram syndrome | Zender stuurt te kleine segmenten | Zender | **Nagle's algorithm** |
| Silly Window Syndrome | Ontvanger adverteert te kleine windows | Ontvanger | **Clarke's algorithm** |
| Te veel kale ACKs | Ontvanger stuurt ACK per segment | Ontvanger | **Delayed ACKs** |
| Nagle + Delayed ACK interactie | Beide wachten op elkaar | Beide | **TCP_NODELAY** of **TCP_QUICKACK** |

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

Het ACK clock-mechanisme is een fundamenteel concept in TCP dat verklaart hoe TCP **zelfregulerende pacing** bereikt zonder expliciete kennis van de netwerktopologie. Dit wordt circa 2 keer op het examen gevraagd.

### Het probleem

Stel een host met een 1 Gbps netwerkkaart stuurt data over een pad waar ergens een bottlenecklink van 1 Mbps zit. De host kan data 1000 keer sneller produceren dan de bottleneck aankan. Als de host gewoon zo snel mogelijk data instuurt, ontstaan er enorme bursts die zich opstapelen in routerbuffers.

### Hoe de ACK clock werkt

```text
Zender (1 Gbps) ──► Router ──► Bottleneck (1 Mbps) ──► Ontvanger
```

Het proces verloopt in stappen:

1. **Eerste burst**: de zender stuurt zijn initiële window aan data snel achter elkaar het netwerk in
2. **Bottleneck spaceert packets uit**: de trage link kan packets maar aan 1 Mbps doorlaten, dus ze komen aan de andere kant mooi gespreid aan
3. **ACKs erven de spacing**: de ontvanger stuurt ACKs terug met dezelfde tussenruimte als de packets aankwamen
4. **Zender klokkeert op ACKs**: voor elk ontvangen ACK stuurt de zender precies een nieuw segment. Omdat de ACKs gespreid aankomen, worden de nieuwe segments ook gespreid verstuurd

### Waarom dit werkt

De ACK clock creëert een **feedback loop**:

- het tempo van de ACKs weerspiegelt het tempo van de bottleneck
- de zender klokkeert zijn injectie op dat tempo
- het resultaat is een **gelijkmatige, bottleneck-aangepaste stroom** in plaats van gevaarlijke bursts

Dit is wat men TCP's **self-clocking** noemt. Het netwerk gedraagt zich als een klok die het verzendtempo reguleert.

### Smoothing van verkeer

Het ACK clock-effect zorgt ervoor dat:

- **burstiness vermindert**: na de eerste burst wordt het verkeer automatisch glad
- **routerbuffers worden gespaard**: minder kans op overflow en packet loss
- **andere flows minder gehinderd worden**: gelijkmatiger verkeer deelt bandbreedte eerlijker

### Wanneer de ACK clock verstoord raakt

De ACK clock kan verstoord raken bij:

- **delayed ACKs**: als de ontvanger ACKs uitstelt, komt de feedback trager en in bursts
- **ACK-compressie**: als meerdere ACKs samen in een queue zitten en in een burst aankomen
- **hertransmissie na timeout**: na een timeout moet TCP opnieuw beginnen met slow start, waardoor de ACK clock opnieuw opgebouwd moet worden

### belangrijk voor examen

De **ACK clock** (self-clocking) zorgt ervoor dat de zender zich automatisch afstemt op de traagste schakel in het pad. Nieuwe data wordt ingestuurd op het ritme van terugkerende ACKs, waardoor bursts worden afgevlakt. Dit werkt zonder dat de zender de bottleneck-capaciteit expliciet kent.

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

## Tahoe en Reno: TCP congestion control in detail

TCP congestion control is circa 4 keer gevraagd op het examen, vaak met de vraag om het **Tahoe/Reno-diagram** te tekenen en uit te leggen.

### De fasen van TCP congestion control

TCP congestion control bestaat uit twee hoofdfasen:

1. **Slow start**: exponentieel groeien (verdubbeling per RTT)
2. **Congestion avoidance**: lineair groeien (+1 MSS per RTT, dit is het AIMD "additive increase" deel)

De overgang tussen beide fasen wordt bepaald door de **slow-start threshold (ssthresh)**.

### Hoe slow start exact werkt

1. Begin met cwnd = 1 MSS (of soms 2-4 MSS in moderne implementaties)
2. Voor **elke ontvangen ACK**: cwnd += 1 MSS
3. Omdat een window van N segmenten N ACKs oplevert, verdubbelt cwnd effectief per RTT
4. Dit gaat door tot cwnd >= ssthresh, dan schakel over naar congestion avoidance

### Hoe congestion avoidance exact werkt

1. Voor elke RTT: cwnd += 1 MSS (lineaire groei)
2. Preciezer: voor elke ACK: cwnd += MSS * (MSS / cwnd)
3. Dit is het "additive increase" deel van AIMD

### TCP Tahoe

**TCP Tahoe** (1988) was de eerste implementatie met congestion control. Het gedrag bij verlies:

**Bij elke vorm van verliesdetectie** (timeout OF 3 duplicate ACKs):

1. ssthresh = cwnd / 2
2. cwnd = 1 MSS
3. Herstart met **slow start** vanaf het begin

### TCP Reno

**TCP Reno** (1990) verbeterde Tahoe door onderscheid te maken tussen soorten verlies:

**Bij timeout** (ernstig verlies):

1. ssthresh = cwnd / 2
2. cwnd = 1 MSS
3. Herstart met **slow start** (identiek aan Tahoe)

**Bij 3 duplicate ACKs** (mild verlies -- **fast retransmit + fast recovery**):

1. ssthresh = cwnd / 2
2. cwnd = cwnd / 2 (halveer, maar ga **niet** naar 1)
3. Hertransmit het verloren segment meteen (**fast retransmit**)
4. Ga direct naar **congestion avoidance** (lineaire groei), niet naar slow start

### Waarom 3 duplicate ACKs?

Als de zender 3 duplicate ACKs ontvangt (dus 4 keer hetzelfde ACK-nummer), is dat een sterk signaal dat een specifiek segment verloren is, maar dat latere segmenten **wel** aankomen. Het netwerk werkt dus nog, alleen dat ene segment ontbreekt.

Bij een timeout daarentegen is er helemaal geen feedback meer -- dat wijst op ernstigere congestie.

### Het Tahoe/Reno-diagram

Een typisch examendiagram ziet er zo uit (cwnd in MSS per RTT):

```text
cwnd
(MSS)
  |
32|                                    *
  |                                 *     
16|              *               *        (Reno: fast recovery)
  |           *     *         *
 8|        *          *    *
  |     *               *
 4|  *                   (3 dup ACKs: Reno halveert naar 8)
  | *
 2|*                     (Tahoe zou hier naar 1 gaan)
 1|*
  +---------------------------------------------------> tijd (RTT)
     ^ slow start    ^ congestion     ^ verlies    ^ recovery
                       avoidance
```

Leeswijzer:
- **Fase 1** (RTT 0-4): Slow start, exponentieel: 1, 2, 4, 8, 16
- **Fase 2** (RTT 4-...): Congestion avoidance, lineair: 16, 17, 18, ...
- **Verlies**: bij cwnd = 32 treedt verlies op
  - **Tahoe**: ssthresh = 16, cwnd = 1, herstart slow start
  - **Reno bij 3 dup ACKs**: ssthresh = 16, cwnd = 16, meteen congestion avoidance
  - **Reno bij timeout**: identiek aan Tahoe (cwnd = 1)

### belangrijk voor examen

Het kernverschil: bij 3 duplicate ACKs valt **Tahoe terug naar cwnd = 1** (slow start), terwijl **Reno halveert en doorgaat** met congestion avoidance (fast recovery). Bij timeout gedragen beide zich identiek. Je moet het diagram kunnen tekenen.

## Fast recovery in detail

Bij fast recovery hoeft TCP niet altijd helemaal opnieuw te starten vanaf een minimale congestion window. Via **duplicate ACKs** kan de zender afleiden dat er waarschijnlijk een packet verloren ging, terwijl latere packets wel nog aankomen.

Dan kan TCP:

- het verloren packet snel hertransmitteren (**fast retransmit**)
- terugvallen tot rond de nieuwe threshold
- daarna opnieuw additive increase doen

Dat is veel efficienter dan van nul herbeginnen.

De beperking van klassieke fast recovery is wel dat het vooral goed werkt bij **een enkel packet loss** per window. Bij meerdere verliezen in hetzelfde window presteert Reno's fast recovery slecht -- daarvoor is **SACK** nodig.

## SACK: selective acknowledgments

Daarom kwam later **SACK**.

Met gewone cumulatieve ACKs weet de zender enkel tot waar alles in orde is. Met SACK kan de ontvanger extra info meesturen over **welke hogere byte-ranges al wel ontvangen zijn**.

Dat laat de zender toe om gerichter te retransmitteren bij meerdere verliezen in een venster.

### waarom is dit handig?

Zonder SACK weet de zender: "ergens is iets mis".  
Met SACK weet de zender: "deze stukken heb je al, die specifieke gaten nog niet".

### belangrijk voor examen

SACK verbetert herstel bij **multiple packet loss** zonder de basiscompatibiliteit van TCP te breken, omdat het via header options werkt.

## ECN vs RED: congestienotificatie vergeleken

Dit onderwerp is circa 3 keer gevraagd op het examen. Het gaat over de vraag: **hoe signaleer je congestie aan de zender?**

Er zijn drie grote mechanismen, elk met een ander principe:

### RED: Random Early Detection

**RED** draait op **routers** en dropt packets **proactief** voordat de queue helemaal vol is.

Werking:

1. De router meet de **gemiddelde queue-lengte**
2. Als die boven een drempelwaarde komt, begint de router packets te droppen met een **toenemende waarschijnlijkheid**
3. Hoe voller de queue, hoe hoger de dropkans
4. TCP interpreteert het verlies als congestiesignaal en verlaagt cwnd (AIMD)

Belangrijke eigenschap: zware zenders hebben meer packets in de queue en worden daardoor **proportioneel vaker getroffen**. Dat maakt RED eerlijker dan tail-drop (waar pas gedropt wordt als de queue vol is -- dan worden net de laatst aangekomen packets getroffen, ongeacht wie ze stuurde).

### ECN: Explicit Congestion Notification

**ECN** is een elegantere aanpak waarbij congestie gesignaleerd wordt **zonder** packets te droppen.

Werking:

1. Beide eindpunten onderhandelen ECN-support tijdens de TCP handshake
2. De zender markeert packets als **ECN-capable** in de IP-header (ECT-bit)
3. Een router die congestie detecteert, zet de **CE-bit** (Congestion Experienced) in de IP-header van het packet, in plaats van het te droppen
4. De ontvanger ziet de CE-markering en stuurt een TCP-segment terug met de **ECE-flag** (ECN-Echo)
5. De zender ontvangt de ECE, verlaagt cwnd (net als bij verlies), en stuurt een segment met de **CWR-flag** (Congestion Window Reduced) om te bevestigen

### Choke Packets (ter vergelijking)

Bij **choke packets** stuurt de router een apart controle-packet terug naar de zender om te melden dat er congestie is. Dit heeft als groot nadeel dat je **extra packets** stuurt in een netwerk dat al overbelast is.

### Vergelijkingstabel

| Aspect | RED | ECN | Choke Packets |
|---|---|---|---|
| **Werking** | Router dropt packets probabilistisch | Router markeert packet, ontvanger echo't | Router stuurt apart packet naar zender |
| **Packet loss** | Ja (bewust) | Nee (markering i.p.v. drop) | Nee (maar extra packets) |
| **Extra overhead** | Negatief (hertransmissies nodig) | Geen (hergebruikt bestaand packet) | Hoog (extra packets in overbelast netwerk) |
| **Snelheid signaal** | Snel (TCP detecteert verlies direct) | Medium (~1 RTT via ontvanger) | Snel (~1 RTT direct naar zender) |
| **Deployment-vereisten** | **Enkel routers** | Routers + **beide** endpoints | Enkel zender |
| **Daadwerkelijk gedeployed?** | **Ja, breed** | Beperkt | Historisch |

### Voordelen van ECN boven RED

- **Geen nutteloos verlies**: packets worden niet gedropt, dus geen hertransmissie nodig
- **Snellere reactie**: de zender weet meteen dat er congestie is, zonder te wachten op een timeout
- **Beter voor korte flows**: bij RED kan een kort bestandje met weinig packets zwaar getroffen worden door random drop
- **Beter voor realtime**: geen onnodige hertransmissie en bijbehorende delay

### Nadelen van ECN boven RED

- **Vereist ondersteuning overal**: routers, zender EN ontvanger moeten ECN ondersteunen
- **Deployment-probleem**: als een router, firewall of middlebox ECN-bits niet correct doorstuurt, werkt het niet
- **Kan misbruikt worden**: een ontvanger kan de CE-markering negeren en niet echo'en

### Hughes-punt (belangrijk voor examen)

De prof benadrukt dat **RED daadwerkelijk breed gedeployed is** omdat het enkel op routers hoeft te draaien. ECN is eleganter maar vereist end-to-end ondersteuning, en dat maakt deployment in de praktijk veel moeilijker. Dit is een voorbeeld van hoe een theoretisch betere oplossing het verliest van een pragmatische oplossing door deployment-barrières.

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

- Waarom RTP meestal op UDP draait en waarom RTP **geen flow control en geen congestion control** heeft
- Het verschil tussen **sequence number** en **timestamp** in RTP
- Waarom RTP goed werkt met **multicast** en TCP niet
- Het risico van **TCP starvation** door RTP/UDP-verkeer
- Wat **jitter** is en hoe buffering het opvangt
- **Alle TCP-header velden** kunnen uitleggen (source/dest port, seq/ack number, flags, window, checksum, urgent pointer, options)
- De betekenis van alle **TCP flags**: SYN, ACK, FIN, RST, PSH, URG, ECE, CWR
- De **3-way handshake** en de SYN/ACK-combinaties
- Waarom TCP een **byte-stream** is en geen message protocol
- Hoe de **sliding window** werkt, inclusief `ACK` en `WIN`
- Het **deadlock-scenario** bij `WIN = 0` en hoe **window probes** dat oplossen
- **usable window = min(rwnd, cwnd)**
- Het verschil tussen **flow control** en **congestion control**
- **Tinygram syndrome**, **Nagle's algorithm**, **delayed ACKs** en **Silly Window Syndrome** + Clarke's algorithm
- De **interactie Nagle + delayed ACKs** die tot 200 ms latency leidt (en oplossingen)
- Hoe de **ACK clock** (self-clocking) werkt en verkeer gladstrijkt
- Het doel van **RTO**, **persistence**, en **TIME_WAIT** timers
- Het **Tahoe/Reno-diagram** kunnen tekenen en uitleggen
- Het verschil: Tahoe gaat altijd naar cwnd=1; Reno halveert bij 3 dup ACKs (fast recovery)
- Waarom **SACK** nuttig is bij meerdere verliezen per window
- De vergelijking **ECN vs RED**: werking, deployment, voor- en nadelen

## Korte eindintuitie

Als je alles tot een zin zou reduceren:

- **RTP** zegt: _"speel media bruikbaar af, ook als niet elk packet perfect aankomt."_
- **TCP** zegt: _"lever alle bytes correct, in volgorde, en doe dat zonder het netwerk kapot te duwen."_

Dat verschil in prioriteit verklaart bijna alle ontwerpkeuzes uit deze slides.
