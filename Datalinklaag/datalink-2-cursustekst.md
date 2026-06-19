# Deel 14 - Data Link 2: Ethernet en 802.11

In dit deel verschuift de focus van algemene MAC-principes naar twee heel concrete en superbelangrijke technologieën:

- **Ethernet**, het klassieke wired LAN
- **IEEE 802.11**, dus wifi

Dat is een logische stap na de vorige slides over ALOHA, CSMA en MAC-protocollen. Eerst leer je de basisideeën van medium access. Daarna kijk je hoe die ideeën in de praktijk zijn uitgewerkt in echte systemen.

De kernvraag blijft dezelfde:

hoe laten we meerdere hosts hetzelfde medium gebruiken op een efficiënte, betrouwbare en betaalbare manier?

## Ethernet: het klassieke LAN

Ethernet is de bekendste technologie voor lokale netwerken met kabels. Het is historisch gegroeid uit werk van Metcalfe en Boggs in de jaren 70 en werd later gestandaardiseerd als **IEEE 802.3**.

De slides tonen ook dat Ethernet niet één enkel systeem is, maar een familie van varianten:

- classic Ethernet
- switched Ethernet
- Fast Ethernet
- Gigabit Ethernet
- 10-Gigabit Ethernet

### waarom is Ethernet zo belangrijk?

Omdat het een heel praktische oplossing gaf voor een echt probleem:

meerdere computers in hetzelfde lokale netwerk met elkaar laten communiceren zonder dat elke machine een eigen aparte fysieke verbinding met iedereen nodig heeft.

## Het oorspronkelijke idee

De eerste Ethernet-opstellingen gebruikten één gedeelde kabel. Alle hosts hingen als het ware aan dezelfde lijn.

Dat betekende:

- iedereen deelt hetzelfde medium
- iedereen kan in principe signalen op dezelfde kabel zetten
- maar je moet botsingen vermijden of detecteren

Dit is dus echt een toepassing van het MAC-probleem uit het vorige hoofdstuk.

## Ethernet-architectuur

In de oorspronkelijke Ethernet-architectuur delen meerdere hosts één gemeenschappelijke kabel. Iedere host heeft een interface die frames op het medium kan zetten en frames van het medium kan lezen.

Het gevolg daarvan is belangrijk:

een frame dat op die kabel verschijnt, is fysiek zichtbaar voor alle interfaces op dat segment.

Dat betekent niet dat elke host het frame ook logisch moet verwerken, maar wel dat alle hosts het potentieel kunnen zien.

## Ethernet frame format

Een Ethernet-frame heeft een vaste globale structuur. De details zijn belangrijk omdat elk veld een duidelijk doel heeft.

### Preamble

Vooraan staat een **preamble**. Dat is een bitpatroon dat zender en ontvanger helpt om hun klok te synchroniseren.

Met andere woorden:

voor de echte data begint, krijgt de ontvanger eerst een kort "ritme" zodat hij de bits correct kan lezen.

### Start of frame

Daarna komt een markering die aangeeft dat de echte frame-inhoud nu begint.

### Source en destination MAC address

Elke Ethernet-interface heeft een **48-bit MAC-adres**.

Dat adres bestaat typisch uit:

- een deel dat de fabrikant identificeert
- een deel dat de specifieke interface identificeert

Er bestaat ook een **broadcast-adres**, waarbij alle bits op 1 staan. Dan is het frame bedoeld voor iedereen op het lokale segment.

### Type of length

Een heel belangrijk veld is het veld dat zegt wat er in het frame zit, of hoe lang de payload is.

Hier zit het verschil tussen **DIX Ethernet** en **IEEE 802.3**.

## DIX versus IEEE 802.3

De slides leggen uit dat oudere DIX Ethernet-frames een **type field** gebruikten, terwijl IEEE 802.3 een **length field** gebruikte.

### waarom is een type field nuttig?

Omdat de netwerkkaart of het besturingssysteem moet weten aan welk hoger protocol het frame moet worden doorgegeven.

Bijvoorbeeld:

- IPv4
- ARP
- andere higher-layer protocollen

Zonder zo’n type field weet je niet goed hoe je de payload moet interpreteren.

Bij 802.3 werd dat opgelost via een extra logical link control header. In de praktijk kon men DIX en 802.3 toch onderscheiden omdat:

- waarden **tot en met 1500** als lengte worden gelezen
- waarden **boven 1500** als type worden gelezen

### belangrijk voor examen

Het verschil tussen DIX en 802.3 draait vooral rond de interpretatie van dat veld:

- `<= 1500` betekent lengte
- `> 1500` betekent type

## Waarom is er een minimum en maximum framegrootte?

Een maximumframe is nodig omdat hardware geheugen moet reserveren om frames op te slaan. In de beginperiode was RAM duur, dus men koos een praktische bovengrens.

Maar de **minimum frame size** is minstens even belangrijk.

### waarom een minimumframe?

Ethernet moest collisions kunnen detecteren terwijl een host nog bezig is met zenden.

Als een frame te klein zou zijn, dan kan het gebeuren dat de zender al volledig klaar is met uitsturen voordat een collision-signaal van ver genoeg op de kabel terug geraakt.

Dan zou de zender ten onrechte denken dat alles goed verlopen is.

Dus:

- het frame moet lang genoeg duren
- zodat een collision nog merkbaar is tijdens de transmissie

### De wiskundige relatie

De minimale framegrootte is direct gekoppeld aan de kabellengte en de snelheid:

```text
T_transmissie >= 2 x T_propagatie
```

Bij 10 Mbps classic Ethernet:

- Minimum frame = 64 bytes = 512 bits
- T_transmissie = 512 bits / 10 Mbps = 51,2 us
- Maximale kabellengte = 51,2 us x 200 m/us / 2 = **~2560 m**

Als het frame korter zou zijn dan 64 bytes, wordt het **gepad** (padding) met opvulbytes tot aan die 64 bytes (inclusief header en CRC).

### belangrijk voor examen

De minimum frame length in classic Ethernet hangt samen met **collision detection**. Het frame moet lang genoeg zijn zodat een zender een botsing nog kan opmerken voor hij klaar is met zenden. Bij hogere snelheden wordt de maximaal toelaatbare kabellengte strenger, of moeten extra mechanismen zoals carrier extension worden gebruikt.

## Ethernet gebruikt 1-persistent CSMA/CD

Classic Ethernet gebruikt **1-persistent CSMA/CD**.

Dat betekent:

- een station luistert naar het kanaal
- als het kanaal bezet is, wacht het
- zodra het kanaal vrij is, zendt het meteen
- als er toch een collision optreedt, stopt het en probeert het later opnieuw

Hier staat **CD** voor **collision detection**. Dat past goed bij een bedraad medium, want op een kabel kan je tijdens het zenden meestal detecteren dat het signaal verstoord is.

## Binary exponential back-off

Na een collision gaat Ethernet niet gewoon "een beetje" wachten. Het gebruikt **binary exponential back-off**.

Dat betekent:

- na elke collision kies je een willekeurige wachttijd
- het bereik waaruit je kiest wordt elke keer groter

Dus eerst kies je bijvoorbeeld uit een kleine range, en na meer collisions uit een grotere. Zo vermijd je dat dezelfde hosts telkens opnieuw tegelijk proberen te zenden.

### waarom werkt dat?

Bij weinig collisions blijft de wachttijd klein, dus de vertraging blijft laag.

Bij veel collisions wordt het systeem vanzelf voorzichtiger, zodat de druk op het medium afneemt.

### belangrijk voor examen

Binary exponential back-off maakt Ethernet adaptief:

- weinig collisions -> kleine wachttijden
- veel collisions -> grotere willekeurige wachttijden

## Security en privacy in classic Ethernet

Een minder mooie kant van het oorspronkelijke ontwerp is dat elke interface frames van het hele segment kan zien.

Normaal gooit een host frames weg die niet voor hem bedoeld zijn. Maar in **promiscuous mode** kan een host die frames bewust toch bijhouden en analyseren.

Dat is belangrijk voor:

- debugging
- monitoring
- maar ook privacy- en securityproblemen

### belangrijk voor examen

Op een gedeeld Ethernet-segment kan een host verkeer van andere hosts mee observeren. Dat maakt sniffing mogelijk.

## Van één kabel naar hubs

Daarna evolueerde Ethernet naar netwerken met **hubs**.

Een hub is in essentie geen slimme schakelaar. Hij herhaalt signalen gewoon naar alle poorten. Functioneel blijft het netwerk dus nog altijd één gedeeld medium.

Een voordeel is praktische eenvoud:

- makkelijker bekabelen
- een kabelbreuk treft typisch maar één hostverbinding

Maar qua capaciteit lost een hub het fundamentele probleem niet op.

## Hub vs Switch: een fundamenteel verschil

Dit is een veelgesteld examenonderwerp. Het verschil tussen hubs en switches is architecturaal en heeft directe gevolgen voor prestaties, privacy en schaalbaarheid.

### Hub: een domme repeater

Een hub is een **fysieke repeater**. Hij kopieert elk ontvangen signaal naar **alle** poorten, zonder naar de inhoud te kijken.

Dat betekent:

- alle poorten delen dezelfde **collision domain**
- slechts een host kan tegelijk zenden (half-duplex, CSMA/CD vereist)
- de totale bandbreedte wordt gedeeld over alle hosts
- **iedereen ziet al het verkeer** (geen privacy)

Een voordeel is dat een hub goedkoop en eenvoudig is. En bij een kabelbreuk treft het typisch maar een host.

### Switch: intelligent forwarden

Een switch werkt fundamenteel anders:

- hij leert **MAC-adressen** (bouwt een MAC-tabel op)
- kijkt naar het bestemmings-MAC-adres van elk frame
- stuurt het frame alleen door naar de **juiste poort**

### Vergelijking

| Aspect | Hub | Switch |
| --- | --- | --- |
| Werking | Fysieke repeater, kopieert naar alle poorten | Leert MAC-adressen, forwardt per poort |
| Collision domain | Een gedeeld domain (alle poorten) | Per poort een apart domain |
| Duplex | Half-duplex (CSMA/CD vereist) | Full-duplex mogelijk (geen collisions) |
| Bandbreedte | Gedeeld over alle poorten | Dedicated per poort |
| Privacy | Iedereen ziet al het verkeer | Enkel je eigen unicast-verkeer |
| Kosten | Goedkoop, geen logica | Duurder, forwarding-logica nodig |

### Wanneer is een hub beter dan een switch?

Dit klinkt als een strikvraag, maar er zijn situaties:

- **Network sniffing en protocol-analyse**: als je al het verkeer wil zien (bv. met Wireshark), is een hub ideaal omdat hij alles broadcast. Bij een switch is dit enkel mogelijk met **port mirroring** (SPAN), wat configuratie vereist.
- **Lab-onderwijs**: om te demonstreren hoe collisions werken of hoe promiscuous mode data kan afluisteren.

### Privacy-vergelijking over media

| Medium | Afluisterbaarheid | Bescherming |
| --- | --- | --- |
| Hub Ethernet | Triviaal: promiscuous mode op elke poort | Fysieke toegangscontrole |
| Switched Ethernet | Moeilijk, maar MAC flooding/spoofing kan switch in hub-modus forceren | Port security, 802.1X |
| 802.11 WiFi | Inherent sniffable (radiogolven voor iedereen) | Volledig afhankelijk van encryptie (WPA2/3) |

## Wat verandert er door switching?

Bij switched Ethernet krijgt elke link typisch zijn eigen verbinding tussen host en switch.

Daardoor:

- is niet iedereen nog in dezelfde collision domain
- kunnen meerdere hosts tegelijk zenden naar verschillende poorten
- is CSMA/CD vaak niet meer nodig

Zeker bij **full duplex** links is er geen klassieke collision-situatie meer, omdat zenden en ontvangen gelijktijdig op aparte paden kan gebeuren.

### waarom is dit zo’n grote verbetering?

Omdat je van een gedeelde snelweg evolueert naar veel kleinere, onafhankelijke verbindingen die centraal gekoppeld worden.

Dat verhoogt:

- capaciteit
- prestaties
- privacy

De switch heeft wel buffers nodig, want meerdere inputpoorten kunnen tegelijk data naar dezelfde outputpoort willen sturen.

## Fast Ethernet

Fast Ethernet verhoogde de snelheid naar **100 Mbps**.

De ontwerpkeuze was bewust conservatief:

men wilde zo veel mogelijk compatibel blijven met het bestaande Ethernet-model, in plaats van een totaal nieuw protocol te bouwen.

### ontwerpcompromis

Als je de bits gewoon sneller verstuurt, dan wordt de tijd die nodig is om een bepaald aantal bits uit te sturen kleiner.

Maar collision detection hangt net af van timing en kabellengte. Dus als de bits veel sneller gaan, dan moet de maximaal bruikbare kabellengte omlaag.

Bij 100 Mbps met hub:

```text
T_transmissie = 512 bits / 100 Mbps = 5,12 us
Maximale kabellengte = 5,12 us x 200 m/us / 2 ≈ 256 m
```

Dat is **10 keer strenger** dan bij 10 Mbps. Dat was de prijs voor backwards compatibility.

## Gigabit Ethernet

Bij **Gigabit Ethernet** wilde men opnieuw ongeveer een factor 10 sneller gaan, maar zonder het hele ecosysteem weg te gooien.

De slides tonen wel dat er grenzen opdoken bij half-duplex en klassieke CSMA.

Bij gigabit-snelheden wordt collision detection op langere afstanden erg lastig. Daarom werden extra hardwaretrucs gebruikt, zoals:

- **carrier extension**
- **frame bursting**

Carrier extension maakt korte frames kunstmatig langer op de lijn, zodat collision-mechanismen nog kunnen werken.

Frame bursting laat toe om meerdere kleine frames als één langere transmissie na elkaar te versturen.

### belangrijk voor examen

Bij hogere Ethernet-snelheden wordt klassieke CSMA/CD steeds minder praktisch. Daarom zie je een evolutie naar switched en full-duplex gebruik, waar collisions in de praktijk verdwijnen.

## 10-Gigabit Ethernet

Bij **10-Gigabit Ethernet** wordt die evolutie eigenlijk volledig doorgetrokken:

- enkel full-duplex
- geen hubs
- focus op point-to-point links

Daardoor zijn de klassieke collision-problemen van gedeelde media hier in feite niet meer de kern van het ontwerp.

## Overzicht: kabellengte en CSMA/CD per Ethernet-variant

Dit overzicht is zeer examrelevant en vat de evolutie van Ethernet samen:

| Configuratie | CSMA/CD | Kabellimiet | Bepaald door |
| --- | --- | --- | --- |
| Classic 10 Mbps, hub | Ja | ~2500 m | Collision-timing |
| Fast 100 Mbps, hub | Ja | ~256 m (10x strenger) | Collision-timing |
| Switched, half-duplex | Ja (per poort) | Per poort | Collision-timing per poort |
| Switched, **full-duplex** | **Nee** | ~100 m (Cat5e) | **Enkel signaalattenuatie** |
| Gigabit, half-duplex | Ja + carrier extension | Kort | Collision-timing + carrier extension |
| 10-Gigabit | **Nee** (enkel full-duplex) | Fiber: km's, koper: ~100 m | Attenuatie |

### waarom full-duplex de collision-limiet opheft

Bij full-duplex zijn zenden en ontvangen op **aparte aderparen** (bij UTP) of aparte vezels (bij fiber). Collisions zijn dan fysiek onmogelijk, CSMA/CD wordt uitgeschakeld, en de collision-driven kabellimiet verdwijnt. De enige resterende limiet is **signaalattenuatie** (~100 m voor Cat5e).

## Overgang naar 802.11

Tot nu toe ging het over Ethernet, dus vooral **wired networking**.

Bij wifi verandert de situatie sterk.

In een draadloos netwerk:

- is het medium minder voorspelbaar
- zijn collisions moeilijker te detecteren
- zijn hidden terminals een veel groter probleem
- speelt energieverbruik veel sterker mee

Daarom gebruikt 802.11 niet CSMA/CD, maar **CSMA/CA**.

## IEEE 802.11: wifi

802.11 ondersteunt twee grote werkvormen:

- **infrastructure mode**
- **ad-hoc mode**

### Infrastructure mode

Hier communiceren wireless clients via een **access point (AP)**. Dat access point vormt meestal de brug naar een bekabeld netwerk.

Dit is de normale wifi-opstelling thuis, op school of op kantoor.

### Ad-hoc mode

Hier bouwen hosts rechtstreeks een draadloos netwerk met elkaar, zonder vaste access point-infrastructuur.

Dat is nuttig wanneer je snel een netwerk on the fly wil opzetten.

## Waarom gebruikt wifi collision avoidance?

In Ethernet kan een zender vaak merken dat er tijdens het zenden iets fout loopt op de kabel.

In wifi is dat veel moeilijker:

- een node kan tijdens het zenden niet betrouwbaar tegelijk naar het kanaal luisteren
- het eigen signaal is veel sterker dan wat hij van anderen zou horen
- hidden terminals maken het beeld onvolledig

Daarom kiest 802.11 voor **Collision Avoidance** in plaats van **Collision Detection**.

Het idee is dus:

probeer botsingen vooraf minder waarschijnlijk te maken, in plaats van ze achteraf elegant te detecteren.

## CSMA/CA

802.11 gebruikt nog altijd carrier sensing en exponential back-off, maar met een belangrijk verschil:

een station start niet automatisch meteen zodra het medium vrij lijkt.

Het kiest eerst een kleine willekeurige back-off.

### waarom een start-backoff?

Omdat in wifi acknowledgements cruciaal zijn.

Als een host onmiddellijk zou beginnen zenden zodra het kanaal vrij is, dan zou hij gemakkelijk kunnen interfereren met de ACK die eigenlijk net teruggestuurd moet worden na een vorige transmissie.

De extra start-backoff helpt dus om ruimte te laten voor die ACKs.

### belangrijk voor examen

In 802.11 is de initiële random back-off belangrijk om niet te botsen met acknowledgements en om gelijktijdige zenders uit elkaar te trekken.

## ACKs in 802.11

Omdat collisions niet goed detecteerbaar zijn, werkt wifi met **acknowledgements**.

De logica is eenvoudig:

- zender stuurt frame
- ontvanger stuurt ACK terug
- blijft de ACK uit, dan vermoedt de zender verlies of collision

Dus waar Ethernet sterk op collision detection leunde, leunt wifi veel meer op **positieve bevestiging achteraf**.

## Virtual channel sensing en NAV

802.11 gebruikt niet alleen fysiek luisteren naar het medium, maar ook **virtual channel sensing**.

Dat betekent dat nodes uit de inhoud van frames afleiden hoe lang het medium nog bezet zal zijn.

Het mechanisme daarvoor is de **Network Allocation Vector (NAV)**.

Een frame kan aangeven hoe lang de volledige uitwisseling zal duren, bijvoorbeeld:

- data
- ACK

Andere stations die dat frame horen, weten dan dat ze gedurende die tijd beter zwijgen.

### waarom is dat nuttig?

Omdat een node niet altijd elk deel van de transmissie perfect fysiek hoeft te horen. Soms is de hogere-laaginformatie genoeg om te weten dat het kanaal nog gereserveerd is.

### belangrijk voor examen

NAV is de basis van virtual channel sensing in 802.11. Een station gebruikt die informatie om af te leiden hoe lang het medium nog bezet blijft.

## RTS/CTS: oplossing voor hidden terminals

Het RTS/CTS-mechanisme is de belangrijkste aanpak in 802.11 om het hidden terminal probleem aan te pakken. Het bouwt voort op NAV.

### Hoe werkt het?

```text
1. A -> B: RTS (bevat Duration = data + CTS + ACK)
2. B -> all: CTS (bevat Duration = data + ACK) -- C hoort dit!
3. C stelt zijn NAV in en zwijgt
4. A -> B: DATA (collision-vrij)
5. B -> A: ACK
```

Het cruciale punt is stap 2: zelfs als C de RTS van A niet kon horen (hidden terminal), hoort C wel de CTS van B. Daardoor weet C dat het kanaal bezet is en stelt hij zijn NAV in.

### Lost RTS/CTS het exposed terminal probleem op?

**Nee**, het maakt het zelfs **erger**. In het exposed terminal scenario hoort C de RTS van B en stelt zijn NAV onnodig in, waardoor C nog langer wacht dan met enkel carrier sense.

### Wanneer RTS/CTS gebruiken?

RTS/CTS heeft overhead: twee extra controleframes per datatransmissie. Die overhead is enkel gerechtvaardigd wanneer de kost van een collision op een groot dataframe hoger is dan de RTS/CTS-overhead.

Daarom wordt RTS/CTS typisch alleen ingeschakeld voor **grote frames**. De standaard RTS-threshold is 2347 bytes, en in de praktijk staat het mechanisme vaak uitgeschakeld.

### belangrijk voor examen

- RTS/CTS lost het **hidden terminal** probleem op (via CTS + NAV)
- RTS/CTS lost het **exposed terminal** probleem **niet** op (maakt het zelfs erger)
- Enkel zinvol bij grote frames door de overhead van twee extra controleframes

## Wifi packet loss en TCP: een cross-layer probleem

Een belangrijk cross-layer inzicht is dat TCP packet loss op wifi **verkeerd interpreteert**.

TCP (Tahoe/Reno) interpreteert **alle** packet loss als congestie en halveert het congestion window (AIMD). Maar op wifi zijn de meeste verliezen veroorzaakt door:

- Radiointerferentie (andere netwerken, magnetrons, Bluetooth)
- Hidden terminal collisions
- Signaalzwakte / multipath fading
- Uitgeputte MAC-layer retransmissies (802.11 herprobeert intern tot 7x)

Geen van deze is congestie. TCP "bestraft" zichzelf onnodig, waardoor de throughput dramatisch daalt op een niet-gecongestioneerd netwerk.

### Oplossingen

| Aanpak | Laag | Werking |
| --- | --- | --- |
| **ECN** | Netwerk | Expliciet congestiesignaal, geen verwarring met link-fouten |
| **Data-link ARQ** | Data link | 802.11 ACKs + retransmissie maskeren link-verlies lokaal |
| **TCP Westwood** | Transport | Schat beschikbare bandbreedte via ACK-timing i.p.v. verlies |

### waarom is dit belangrijk?

Dit verklaart ook waarom 802.11 **acknowledged connectionless** service biedt (in tegenstelling tot Ethernet's **unacknowledged connectionless**). De link-layer ACKs en retransmissies proberen lokaal verlies op te lossen zodat TCP het niet als congestie misverstaat.

## Power saving in 802.11

Wifi houdt ook rekening met energieverbruik. Dit is een examonderwerp dat de twee fundamentele strategieen voor power saving in 802.11 behandelt.

De slides tonen twee basisbenaderingen die fundamenteel verschillen in **wie het wake-schema bepaalt**.

### Strategie 1: Beacon + TIM / PS-Poll (AP-gestuurd)

Een access point verstuurt periodiek **beacon frames**. Elk beacon bevat een **Traffic Indication Map (TIM)**: een bitmap die aangeeft voor welke slapende clients er gebufferde data klaarstaat.

Het mechanisme werkt als volgt:

1. De client informeert het AP dat hij in **power save mode** gaat
2. Het AP buffert alle frames bestemd voor die client
3. De client waakt enkel op voor **beacon-momenten** (typisch elke 100 ms)
4. De client controleert de TIM in het beacon
5. Als de TIM aangeeft dat er data voor hem klaarstaat, stuurt de client een **PS-Poll** frame
6. Het AP levert de gebufferde data af
7. Tussen beaconmomenten slaapt de client

Dit is geschikt voor **battery-powered devices die meestal idle zijn**: een smartphone in standby, een sensor die zelden data ontvangt.

### Strategie 2: APSD (Automatic Power Save Delivery, client-gestuurd)

Bij **APSD** bepaalt de **client** wanneer hij waker wordt, niet het AP. De client waakt op vaste intervallen die hij zelf kiest, typisch afgestemd op zijn applicatie.

Het mechanisme:

1. De client waakt enkel wanneer hij **zelf data wil sturen**
2. Het AP levert gebufferde downlink-data mee in hetzelfde wake-window
3. De client kan zo lang slapen als hij wil tussen zijn eigen wake-momenten

Dit is geschikt voor **periodieke applicaties** met voorspelbare timing, zoals VoIP (elke 20 ms een spraakframe) of periodieke telemetrie.

### Kernverschil tussen de twee strategieen

| Aspect | Beacon + TIM / PS-Poll | APSD |
| --- | --- | --- |
| Wie bepaalt wake-schema | **AP** (beacon-interval) | **Client** (eigen applicatie-ritme) |
| Geschikt voor | Idle devices, onvoorspelbaar verkeer | Periodieke applicaties (VoIP, telemetrie) |
| Efficiëntie | Minder bij voorspelbaar verkeer (wacht op beacon) | Beter bij voorspelbare timing |
| Overhead | PS-Poll per opvraging | Minimaal (data piggybacked op eigen transmissie) |

### Cross-layer vergelijking

De twee 802.11 power saving strategieen weerspiegelen dezelfde fundamentele afweging als bij BMAC vs TSMP:

- **Beacon + TIM** is vergelijkbaar met **BMAC**: het netwerk (AP) geeft periodieke signalen, de client reageert asynchroon
- **APSD** is vergelijkbaar met **TSMP**: de client werkt op een voorspelbaar schema, waardoor het energieverbruik beter geoptimaliseerd kan worden

### belangrijk voor examen

802.11 heeft twee power saving strategieen. Bij Beacon+TIM bepaalt het **AP** wanneer de client kan ophalen. Bij APSD bepaalt de **client** het schema. APSD is efficienter bij voorspelbare applicaties; Beacon+TIM is flexibeler voor onvoorspelbaar verkeer.

## 802.11 frame structure

De wifi-frameheader is uitgebreider dan bij Ethernet. Dat komt omdat wifi meer complexe situaties moet ondersteunen:

- wireless versus wired kant
- fragmentatie
- retransmissions
- power management
- sequencing
- beveiliging

### Frame control

Het **frame control**-veld bevat verschillende stukjes informatie:

- protocol version
- type en subtype
- to DS / from DS
- more fragments
- retry
- power management
- more data
- protected frame
- order

### waarom zijn die bits nodig?

Omdat wifi veel meer context moet meegeven dan classic Ethernet. In een draadloze omgeving moet een frame duidelijk kunnen zeggen:

- waar het naartoe gaat
- of het deel is van een reeks
- of het een retransmission is
- of de zender gaat slapen
- of de payload beschermd is

## To DS / From DS

Deze bits geven aan of een frame richting het distributed system gaat of eruit komt.

Simpel gezegd:

ze helpen onderscheiden of een frame:

- van een wireless client naar de infrastructuur gaat
- van de infrastructuur terug naar een wireless client gaat

Dat is belangrijk omdat wifi vaak een brug vormt tussen wireless hosts en een extern netwerk.

## Fragmentatie en retries

802.11 ondersteunt ook **fragmentatie**.

### waarom?

Omdat een draadloos kanaal foutgevoeliger is. Als grote frames vaak beschadigd raken, kan het slimmer zijn om ze in kleinere stukken te verzenden. Dan moet je bij verlies niet alles opnieuw doen.

Het **retry**-bit laat zien dat een frame opnieuw verstuurd wordt. Dat helpt de ontvanger om duplicaten te herkennen.

## Sequence number

De frameheader bevat ook een volgnummer. Dat is nodig om:

- duplicaten te detecteren
- retransmissions correct te interpreteren
- de volgorde van frames te beheren waar dat relevant is

## Checksum

Net als bij Ethernet is er een checksum om fouten te detecteren.

Dat blijft fundamenteel op linkniveau: je wil beschadigde frames zo dicht mogelijk bij de bron van het probleem herkennen.

## Packets en frames

De slides maken impliciet ook het klassieke onderscheid:

- een **packet** hoort typisch bij de network layer
- een **frame** hoort bij de data-linklaag

Een IP-packet wordt dus verpakt in een Ethernet-frame of in een 802.11-frame, afhankelijk van het medium.

## Welke soorten data-linkservice bestaan er?

Op het einde plaatsen de slides Ethernet en 802.11 ook in een bredere service-indeling.

### Unacknowledged connectionless service

Voorbeeld: **Ethernet**

Dat past goed bij een betrouwbaar medium of bij een medium waar collisions goed detecteerbaar zijn.

### Acknowledged connectionless service

Voorbeeld: **802.11**

Dat past beter bij een onbetrouwbaarder kanaal, waar expliciete bevestigingen nodig zijn.

### Acknowledged connection-oriented service

Dat lijkt meer op een betrouwbare stroom met expliciete opbouw en opvolging, zoals bij sommige klassieke point-to-pointsystemen.

### belangrijk voor examen

Ethernet en 802.11 bieden niet exact dezelfde linklaagservice:

- Ethernet is typisch unacknowledged connectionless
- 802.11 is typisch acknowledged connectionless

## Samenvatting

De rode draad van dit hoofdstuk is dat **de aard van het medium het protocolontwerp sterk bepaalt**.

Bij Ethernet:

- was collision detection haalbaar
- begon men met gedeelde kabels
- evolueerde het netwerk naar switches en full duplex
- werd CSMA/CD steeds minder centraal naarmate links punt-tot-punt werden

Bij 802.11:

- is het medium moeilijker te observeren
- zijn collisions duurder en moeilijker detecteerbaar
- zijn ACKs, back-off en NAV belangrijk
- speelt power saving een veel grotere rol

## Korte intuïtie

Je kan het verschil heel simpel zo onthouden:

- **Ethernet**: op een kabel kan je veel directer zien wat er gebeurt, dus collision detection was lang bruikbaar
- **wifi**: in de lucht is alles diffuser en onzekerder, dus je werkt voorzichtiger met collision avoidance, ACKs en extra coördinatie

Dat is misschien de belangrijkste gedachte van heel dit deel.
