# Deel 9 - Application 1: Client/Server Systems

In dit deel komen we eindelijk op de **application layer**. Dat is de laag waar netwerksoftware zichtbaar wordt voor echte gebruikers en toepassingen. De onderliggende lagen zorgen voor adressering, routing, transport en betrouwbaarheid, maar op de application layer krijg je protocollen zoals:

- **Telnet** en **SSH** voor remote login
- **SMTP** voor e-mailtransport
- **HTTP** voor het web
- **FTP** voor bestandsoverdracht

De rode draad door deze slides is het **client/server-model**. Bij al deze protocollen zie je telkens dezelfde basisvorm:

- een client initieert contact
- een server luistert op een bekende poort
- er is een afgesproken toepassingsprotocol

Maar de details en ontwerpkeuzes verschillen sterk per toepassing.

## Waar zitten we in de stack?

De application layer staat boven de transportlaag. Dat betekent dat de protocollen hier meestal **niet zelf** bezig zijn met pakketverlies, routers of MAC-adressen. Ze vertrouwen op lagere lagen, vaak op **TCP**, om een bruikbare communicatiebasis te krijgen.

Dat is belangrijk om te onthouden:

- de application layer spreekt in termen van commando’s, berichten, documenten en sessies
- de lagere lagen zorgen ervoor dat bytes effectief van de ene machine naar de andere raken

## De drie grote protocol-stacks: OSI, TCP/IP en Tanenbaum

Op het examen wordt regelmatig gevraagd om de verschillende protocol-stacks te tekenen en protocollen op de juiste laag te plaatsen. Er bestaan drie belangrijke modellen.

### Het OSI-model (7 lagen)

Het **OSI-model** (Open Systems Interconnection) is een referentiemodel met zeven lagen:

| Laag | Naam | Functie | Voorbeeld |
|------|------|---------|-----------|
| 7 | **Application** | Netwerkdiensten voor eindgebruikers | HTTP, SMTP, DNS, DHCP, FTP |
| 6 | **Presentation** | Datarepresentatie, encryptie, compressie | SSL/TLS, JPEG, ASCII |
| 5 | **Session** | Sessiebeheer, synchronisatie | RPC, NetBIOS |
| 4 | **Transport** | End-to-end betrouwbaarheid, flow control | TCP, UDP |
| 3 | **Network** | Routing, logische adressering | IP, ICMP, OSPF, BGP |
| 2 | **Data Link** | Hop-to-hop levering, framing, MAC | Ethernet, 802.11, PPP |
| 1 | **Physical** | Bits over het medium | Koper, glasvezel, radio |

### Het TCP/IP-model (4 lagen)

Het **TCP/IP-model** (ook wel het internetmodel) is pragmatischer en heeft slechts vier lagen:

| Laag | Naam | Komt overeen met OSI | Voorbeeld |
|------|------|---------------------|-----------|
| 4 | **Application** | OSI 5+6+7 | HTTP, SMTP, DNS, FTP, SSH |
| 3 | **Transport** | OSI 4 | TCP, UDP |
| 2 | **Internet** | OSI 3 | IP, ICMP |
| 1 | **Network Access** (Host-to-network) | OSI 1+2 | Ethernet, 802.11 |

### Het Tanenbaum-model (5 lagen)

Het model dat Tanenbaum in zijn boek gebruikt, is een **hybride** met vijf lagen. Het neemt de duidelijke scheiding van de onderste lagen uit OSI, maar voegt de bovenste drie OSI-lagen samen:

| Laag | Naam | Voorbeeld |
|------|------|-----------|
| 5 | **Application** | HTTP, SMTP, DNS, DHCP, RTP |
| 4 | **Transport** | TCP, UDP |
| 3 | **Network** | IP, ICMP, OSPF, BGP |
| 2 | **Data Link** | Ethernet, 802.11, PPP |
| 1 | **Physical** | Koper, glasvezel, radio |

### Protocollen op de juiste laag plaatsen

Een typische examenvraag is: "Op welke laag zit protocol X?" Lastige gevallen:

- **DHCP**: zit op de **application layer**, maar gebruikt UDP (transport) en heeft invloed op de network layer (IP-toewijzing). Het is een applicatieprotocol dat configuratiegegevens uitwisselt.
- **RTP** (Real-time Transport Protocol): zit in het **Tanenbaum-model op de application layer** (laag 5), omdat het bovenop UDP draait. In de praktijk voegt RTP transport-achtige functies toe (sequencing, timestamps), maar het is geen deel van de transportlaag zelf. Het zit in **user space**, niet in de kernel.
- **DNS**: application layer protocol, ook al is het een ondersteunend protocol dat door andere applicaties (HTTP, SMTP) wordt aangeroepen.
- **OSPF**: draait direct op IP (protocol 89), dus het zit op de **network layer**, niet op de transport layer.
- **ARP**: zit tussen de network layer en de data link layer. In het TCP/IP-model is het onderdeel van de Network Access laag.

### belangrijk voor examen

Je moet de drie stacks kunnen tekenen en uitleggen waarom ze verschillen. Het OSI-model is een **referentiemodel** (nooit echt geimplementeerd als protocol-stack), terwijl TCP/IP een **praktisch model** is dat het internet aandrijft. Tanenbaum’s model is een didactisch compromis.

De kernvraag is altijd: "op welke laag zit dit protocol, en waarom?" Het antwoord hangt af van welk model je gebruikt.

## Applicaties die UDP gebruiken (en waarom geen TCP)

Een terugkerend examenvraagtype is: "Noem drie applicaties die UDP gebruiken en leg uit waarom ze geen TCP gebruiken." Drie belangrijke voorbeelden:

### 1. DNS

DNS-queries zijn **klein** (typisch < 512 bytes), **kort** (one-shot vraag-antwoord) en vaak **naar meerdere servers** gericht. TCP’s three-way handshake zou per lookup drie extra berichten vereisen, wat de latency verdrievoudigt voor een simpele naamresolutie. Bovendien maakt UDP **anycast** mogelijk: meerdere fysieke servers delen een IP-adres, en het dichtstbijzijnde exemplaar antwoordt. Bij te grote antwoorden valt DNS alsnog terug op TCP.

### 2. DHCP

DHCP **kan fundamenteel niet over TCP** werken. TCP vereist een three-way handshake met een geldig bron-IP en bestemmings-IP. Tijdens DHCPDISCOVER heeft de client **geen van beide**: geen eigen IP-adres, en hij kent het IP-adres van de DHCP-server niet. UDP laat toe om vanuit `0.0.0.0` naar `255.255.255.255` te broadcasten zonder voorafgaande connectie-state.

### 3. Wake-on-LAN (WoL)

Wake-on-LAN stuurt een **magic packet** als UDP-broadcast om een uitgeschakelde computer op te starten. TCP is hier fundamenteel onmogelijk omdat de doelmachine **uitgeschakeld** is en dus geen TCP-handshake kan uitvoeren. Het magic packet moet als broadcast aankomen op de netwerkinterface van de slapende machine.

### Andere voorbeelden

- **RTP** (Real-time Transport Protocol): voor live audio/video is verlies acceptabel maar vertraging niet. TCP’s retransmissie en congestion control zouden de stream vertragen.
- **SNMP** (Simple Network Management Protocol): moet ook werken als het netwerk overbelast is; TCP’s congestion control zou dan net het beheerverkeer afremmen.
- **TFTP** (Trivial File Transfer Protocol): heel simpel protocol voor boot-images ophalen, zonder de complexiteit van TCP.

### De rode draad

UDP-applicaties kiezen bewust voor UDP wanneer:

- **TCP-setup onmogelijk is** (geen geldig IP, machine uit)
- **snelheid belangrijker is dan betrouwbaarheid** (real-time media)
- **berichten klein en one-shot zijn** (DNS, SNMP)
- **broadcast nodig is** (DHCP, WoL)

## Telnet

**Telnet** is een van de oudste internettoepassingen. Het werd in de vroege internetperiode gebruikt voor **remote terminal access**: een gebruiker op de ene machine logt in op een andere machine en werkt daar alsof hij lokaal achter de terminal zit.

Historisch is Telnet belangrijk omdat het een van de eerste echte internetapplicaties was. In de slides wordt zelfs vermeld dat het eerste verkeer tussen UCLA en Stanford met Telnet-achtige terminalcommunicatie gebeurde.

### Local login versus remote login

Bij een lokale login loopt de invoer ongeveer zo:

- toetsenbord of terminal levert tekens
- terminal driver verwerkt ze
- besturingssysteem interpreteert speciale tekens
- de applicatie ontvangt de uiteindelijke input

Bij **remote login** wordt datzelfde idee over het netwerk getild. De Telnet-client aan de ene kant moet dus de input en controle-informatie via het netwerk naar een Telnet-server aan de andere kant sturen. Daar wordt die dan aangeboden aan een **pseudoterminal**, zodat het systeem denkt dat er lokaal een terminal aanhangt.

### Waarom is dat niet triviaal?

Omdat twee systemen:

- verschillende terminals kunnen hebben
- verschillende karaktersets kunnen gebruiken
- verschillende regels voor controlekarakters kunnen hanteren

Telnet moest dus niet gewoon tekst sturen, maar ook een soort gemeenschappelijke terminaltaal afspreken.

## Faseren van Telnet

De slides delen Telnet op in vier fasen:

1. **connection management**
2. **optional negotiation**
3. **control**
4. **data transfer**

### 1. Connection management

Telnet gebruikt **TCP** om een betrouwbare verbinding op te zetten en te onderhouden. De standaardpoort is **23**.

### 2. Optional negotiation

Daar worden parameters afgesproken, zoals:

- lijnlengte
- terminaltype
- andere sessie-opties

### 3. Control

Daar gaat het over controle-informatie zoals:

- end of line
- interrupts
- andere commando’s

### 4. Data transfer

Daarna kunnen de eigenlijke gegevens gestuurd worden via TCP.

### belangrijk voor examen

Telnet is een **remote login-protocol bovenop TCP**, met negotiatie- en controlegegevens naast gewone tekstdata.

## Waarom gebruiken we Telnet vandaag bijna niet meer?

Het antwoord is eigenlijk simpel:

- Telnet biedt **geen serverauthenticatie**
- Telnet verstuurt **gebruikersnaam en wachtwoord in plaintext**

Dat maakt het in moderne netwerken onveilig.

Toch blijft Telnet nuttig als **debugtool**, juist omdat het zo eenvoudig is. Als een protocol op een tekstuele manier boven TCP werkt, kan je het vaak met Telnet handmatig testen.

## SSH

**SSH**, Secure Shell, is de veilige opvolger van Telnet voor remote login.

Het doel van SSH is om precies de zwakke punten van Telnet op te lossen:

- de server moet geauthenticeerd kunnen worden
- credentials en data mogen niet in het open verstuurd worden

SSH is dus niet gewoon "Telnet met een extra wachtwoord", maar een gelaagd beveiligd remote-loginprotocol.

## De architectuur van SSH

De slides tonen drie lagen in SSH.

### SSH Transport Layer

Deze onderste laag doet:

- algoritmenegotiatie
- key exchange
- serverauthenticatie

Met andere woorden: hier wordt de veilige tunnel opgezet.

### SSH Authentication Layer

Hier gebeurt de **gebruikersauthenticatie**, bijvoorbeeld:

- wachtwoord
- of public-key authenticatie

### SSH Connection Layer

Bovenop de veilige en geauthenticeerde verbinding kunnen dan verschillende diensten draaien:

- terminalsessies
- GUI-forwarding
- file transfer
- TCP forwarding

### intuition

SSH bouwt eerst een veilige pijplijn, controleert daarna wie je bent, en pas daarna laat het echte toepassingen door die pijplijn lopen.

## Public-key cryptografie in SSH

De slides verwijzen ook naar **public key cryptography**. Dat is essentieel voor SSH.

Het basisidee:

- de server heeft een keypair
- de client kan de server identificeren via diens public key
- voor gebruikersauthenticatie kan de gebruiker ook een eigen keypair gebruiken

Bij public-key authenticatie bewijst de client dat hij de private key bezit zonder die private key zelf te versturen.

Dat is veel veiliger dan een wachtwoord in plaintext.

### belangrijk voor examen

SSH lost het Telnet-probleem op door:

- **encryptie**
- **serverauthenticatie**
- en vaak **public-key user authentication**

## E-mail als systeem

Na remote login verschuiven de slides naar **e-mail**.

E-mail lijkt voor gebruikers vaak één ding, maar architecturaal bestaat het uit meerdere onderdelen. Dat is belangrijk.

De slides zeggen dat moderne e-mail twee grote functies scheidt:

- protocollen tussen **client en mailserver**
- protocollen tussen **mailservers onderling**

Dat maakt e-mail flexibel: je kan op meerdere manieren aan je mailbox, terwijl mailservers onderling toch een gemeenschappelijke standaard hebben.

## De architectuur van e-mail

De figuur laat de klassieke keten zien:

sender user agent  
\--> mail submission  
\--> message transfer agent  
\--> SMTP tussen mailservers  
\--> message transfer agent  
\--> final delivery  
\--> receiver user agent

Dus één belangrijke les is:

e-mail voor gebruikers is niet gewoon "een bericht naar een andere gebruiker sturen", maar een hele keten van opslag, transfer en aflevering.

## Message format

E-mailclients maken berichten in het **Internet Message Format**.

Daar zitten typische header fields in zoals:

- `To`
- `Cc`
- `Bcc`
- `From`
- `Sender`
- `Received`
- `Return-Path`

Dat is belangrijk omdat e-mail niet alleen een body is. Het bevat ook metadata over herkomst, bestemming en route.

## POP

**POP**, Post Office Protocol, is een eenvoudige manier voor een client om mail op te halen van een server.

De slides beschrijven POP als:

- eenvoudig
- weinig gesynchroniseerd
- moeilijk te beheren op meerdere machines

Dat is logisch. POP is historisch gebouwd voor een model waarin je mail op één toestel ophaalt en lokaal bewaart.

In een moderne wereld met:

- laptop
- smartphone
- tablet

is dat minder handig.

## IMAP

**IMAP** is gemaakt om beter met meerdere toestellen om te gaan.

Daarbij blijven mappen zoals:

- inbox
- outbox
- drafts

gesynchroniseerd tussen client en server.

Dat geeft een veel consistenter gebruik op meerdere apparaten, maar met meer overhead en meer complexiteit dan POP.

### POP versus IMAP

- **POP**: eenvoudiger, maar slecht voor meerdere toestellen
- **IMAP**: zwaarder, maar veel beter voor synchronisatie

### belangrijk voor examen

Het kernverschil is:

- **POP** haalt mail eerder lokaal op
- **IMAP** houdt mail en mappen gesynchroniseerd tussen client en server

## Webmail over HTTP

Nog een derde model is **webmail**.

Daarbij is de browser zelf de mailclient, en staat in feite alles op de server. Daardoor heb je geen klassieke synchronisatie tussen meerdere lokale clients nodig.

Voordelen:

- lichtgewicht op de client
- bruikbaar op meerdere toestellen

Nadeel:

- je hebt een werkende internetverbinding nodig

## SMTP

**SMTP**, Simple Mail Transfer Protocol, is het protocol dat gebruikt wordt voor **mail transfer** tussen mailservers.

De slides benadrukken dat SMTP een **relay protocol** is.

Dat betekent:

- het is vooral bedoeld om mail verder te geven tussen servers
- niet om rijke mailboxsynchronisatie te doen zoals IMAP

SMTP werkt in **command/response mode** en kent drie grote fasen:

1. handshaking
2. transfer van berichten
3. closure

## Belangrijke SMTP-commando’s

De slides noemen onder andere:

- `HELO`
- `MAIL FROM`
- `RCPT TO`
- `DATA`
- `QUIT`
- `RSET`
- `VRFY`
- `EXPN`
- `HELP`

De kernflow voor een mail is meestal:

client of mailserver  
\--> `HELO`  
\--> `MAIL FROM`  
\--> `RCPT TO`  
\--> `DATA`  
\--> berichtinhoud  
\--> `.`  
\--> `QUIT`

### Telnet en SMTP

Omdat SMTP tekstueel en commando-gebaseerd is, kan je het rechtstreeks met **Telnet** testen. Dat is een mooi voorbeeld van hoe een simpel tekstprotocol boven TCP nog heel transparant kan zijn.

### belangrijk voor examen

SMTP is een **command/response protocol** voor **mail transfer tussen servers**, niet voor rijke mailboxsynchronisatie op clients.

## Het World Wide Web

Daarna gaan de slides naar het web.

Een mooie quote van Tim Berners-Lee maakt daar een belangrijk onderscheid:

- het **netwerk** gaat over verbindingen tussen computers
- het **web** gaat over informatie en hypertext-links

Met andere woorden:

het web gebruikt het netwerk, maar is er niet hetzelfde als.

## Het web als client/server-systeem

De architectuur van het web is klassiek client/server:

- de **browser** is de client
- de **webserver** bewaart resources
- **HTTP** is het protocol dat tussen beiden gebruikt wordt
- **caches** kunnen tussenin prestaties verbeteren

De client vraagt een resource op via een URI, en de server stuurt de bijhorende response terug.

## URI’s

Een **URI**, Uniform Resource Identifier, is een mensleesbare manier om een resource aan te duiden.

Belangrijk is dat een URI niet per se beperkt is tot één protocol of één type resource. De slides geven voorbeelden zoals:

- `mailto:...`
- `ftp://...`
- `http://...`

Een populaire vorm van URI is de **URL**, die protocol, server en resource combineert.

Voorbeeld:

`http://www.foo.com/index.html`

### belangrijk voor examen

Een URI is een identifier voor een resource; een **URL** is de bekendste vorm daarvan.

## DNS en namen

Webgebruikers werken liever met namen dan met IP-adressen. Daarom gebruikt het web **DNS** om namen naar IP-adressen te mappen.

Dat is een essentieel stuk van de webarchitectuur, ook al wordt DNS zelf pas later in detail behandeld.

## HTML

**HTML**, HyperText Markup Language, is de markup-taal waarin webpagina’s beschreven worden.

De slides leggen uit dat HTML:

- tekst structureert
- opmaak en betekenis aangeeft met tags
- links naar andere resources mogelijk maakt

Belangrijk is dat de markup meestal niet rechtstreeks zichtbaar is voor de gebruiker, maar wel bepaalt hoe de browser de pagina presenteert.

### intuition

HTML beschrijft niet alleen "wat er staat", maar ook "welk soort stuk informatie dit is".

## HTTP

**HTTP**, HyperText Transfer Protocol, is het protocol waarmee browser en webserver communiceren.

De slides beschrijven HTTP als:

- een **request/response protocol**
- op de application layer
- bovenop **TCP**
- traditioneel op poort **80**

Een belangrijk punt is dat klassieke HTTP **stateless** is. Het protocol zelf heeft geen ingebouwd sessiegeheugen.

### Wat betekent stateless?

Elke request wordt in principe los bekeken. Als een server context wil onthouden, moet dat via extra mechanismen zoals cookies, sessies of applicatielogica gebeuren.

## HTTP-methodes

De slides noemen een reeks HTTP-commando’s of methodes:

- `GET`
- `PUT`
- `HEAD`
- `OPTIONS`
- `POST`
- `DELETE`

Daarvan zijn vooral `GET`, `HEAD` en `POST` heel bekend in de basis.

### voorbeelden

- `GET`: vraag een document op
- `HEAD`: vraag alleen de headers op
- `POST`: stuur informatie naar de server

## HTTP testen met Telnet

Net zoals bij SMTP kan je ook klassieke HTTP/1.x handmatig testen met Telnet, omdat het in tekstvorm boven TCP draait.

Dat is didactisch heel nuttig, want het laat zien dat een browser in essentie gewoon leesbare requests naar een server stuurt.

## HTTP 1.0 en inefficiëntie

De slides benadrukken een belangrijk probleem van **HTTP 1.0**.

Een webpagina bevat vaak veel embedded objects:

- afbeeldingen
- audio
- video
- scripts

Als je voor elk object een nieuwe TCP-verbinding moet opzetten, is dat inefficiënt:

- extra round trips voor elke TCP three-way handshake
- telkens nieuwe TCP-opstart
- trager door **slow start** die bij elke nieuwe verbinding opnieuw van een klein congestion window begint

Dus HTTP 1.0 werkt, maar is nogal zwaar voor moderne pagina’s.

### Het concrete probleem

Stel: een pagina heeft 1 HTML-bestand en 10 afbeeldingen. Met HTTP 1.0 heb je **11 afzonderlijke TCP-verbindingen** nodig. Elk daarvan kost:

1. TCP three-way handshake (1,5 RTT)
2. HTTP request + response
3. TCP teardown

Dat is 11 keer de setup-overhead, en 11 keer begint TCP’s congestion window opnieuw klein (slow start).

## HTTP 1.1

**HTTP 1.1** bracht twee grote verbeteringen:

### 1. Persistent connections

Daardoor hoef je niet voor elk object opnieuw een volledige nieuwe TCP-verbinding op te zetten. De verbinding blijft open na de eerste request/response, en volgende objecten worden over **dezelfde TCP-verbinding** opgehaald.

Voordelen:

- slechts een TCP-handshake voor meerdere objecten
- het congestion window hoeft niet telkens opnieuw te groeien
- minder overhead op de server (minder sockets, minder state)

### 2. Pipelining

Daarbij wacht je niet tot het ene object volledig is afgehandeld voordat je al de volgende requests uitstuurt. Je kan dus meerdere requests sneller achter elkaar door dezelfde verbinding sturen.

### Wanneer is het voordeel van HTTP 1.1 het grootst?

Het voordeel is het grootst bij pagina’s met **veel kleine embedded objecten**. Hoe meer objecten, hoe meer TCP-handshakes je uitspaart. Bij een pagina met slechts een object maakt het weinig verschil.

### Wanneer werkt pipelining minder goed?

Pipelining heeft beperkingen:

- **Head-of-line blocking**: als het eerste antwoord traag is, blokkeren alle volgende antwoorden (de server moet in volgorde antwoorden)
- Als de browser pas **na het parsen van de HTML** weet welke bijkomende objecten nodig zijn, kan hij die niet vooraf al pipelinen
- Sommige servers en proxies ondersteunen pipelining niet correct

### Cross-layer inzicht (Hughes-favoriet)

HTTP/1.0 is **connectionless/stateless** op de applicatielaag, maar draait over **connection-oriented** TCP. Dit lijkt tegenstrijdig, maar het is een belangrijk examenpunt: een applicatieprotocol kan connectionless gedrag vertonen terwijl het transportprotocol connection-oriented is. HTTP opent en sluit verbindingen, maar onthoudt zelf geen sessie-state tussen requests.

## HTTP/2

HTTP/2 ontstond uit **SPDY**.

Het behoudt de HTTP 1.1-API, maar verandert de onderliggende vorm sterk:

- binair formaat in plaats van puur tekst
- framing
- multiplexing binnen één TCP-verbinding
- prioritisatie van content

Dat maakt het protocol efficiënter voor moderne webpagina’s met veel gelijktijdige objecten.

### belangrijk voor examen

Het grote verschil is dat **HTTP/2 multiplexing en framing** toevoegt binnen één verbinding, terwijl oudere HTTP-versies veel meer leden onder inefficiëntie door aparte objectopvragingen.

## FTP

Tot slot komt **FTP**, File Transfer Protocol.

FTP is gemaakt voor:

- bestanden ophalen
- bestanden opslaan
- directory listings bekijken

Het is dus geen generiek documentprotocol zoals HTTP, maar specifiek een bestandsprotocol.

## Twee TCP-poorten bij FTP

Een opvallend kenmerk van FTP is dat het **twee TCP-verbindingen** gebruikt:

- poort **21** voor **control** (persistent voor de hele sessie)
- poort **20** voor **data transfer** (wordt per bestandsoverdracht opgezet en weer afgebroken)

Het belangrijkste conceptuele punt is dat FTP **control en data splitst** over aparte kanalen.

De controlverbinding gebruikt een Telnet-achtig commandokanaal (NVT) om de sessie te regelen.

### Waarom twee verbindingen?

De scheiding heeft een praktisch voordeel: als je een groot bestand aan het downloaden bent en een **abort-commando** wilt sturen, wordt dat commando niet geblokkeerd door een volgelopen databuffer. Het controlekanaal blijft altijd responsief, onafhankelijk van de datatransfer.

Bij HTTP lopen control en data door dezelfde verbinding, wat dit soort situaties lastiger maakt.

## FTP en het NAT-probleem

Dit is een klassiek examenvraagstuk. Er zijn twee modi in FTP:

### Active FTP (standaard)

In active mode vertelt de client aan de server: "Maak verbinding met mij op dit IP-adres en deze poort." De client stuurt een `PORT`-commando met zijn eigen IP en poortnummer.

Het probleem achter NAT:

```text
Client (prive 192.168.1.10) --> NAT --> Server (publiek 203.0.113.5)
1. Client -> Server:21 (control) -- uitgaand, NAT maakt mapping
2. Client stuurt PORT 192.168.1.10,p  (prive-adres in payload!)
3. Server -> 192.168.1.10:p (inkomend) -- NAT heeft geen mapping
   --> packet gedropt --> data mislukt
```

Het kernprobleem is tweeledig:

- NAT houdt enkel state bij voor **uitgaande** verbindingen. De server probeert een **inkomende** verbinding op te zetten, maar NAT heeft daar geen mapping voor.
- FTP embedt **IP-adressen in de applicatie-payload**. NAT herschrijft IP-headers, maar kijkt normaal niet in de payload.

### Passive FTP (PASV) als oplossing

Bij passive mode draait de logica om:

1. De client stuurt `PASV` via het controlekanaal
2. De server opent een luisterpoort en stuurt het adres terug
3. De **client** initieert de dataverbinding naar de server (uitgaand)

Nu zijn **beide verbindingen** (control en data) uitgaand vanuit de client. NAT laat dit gewoon door, want NAT maakt automatisch mappings voor uitgaande verbindingen.

### belangrijk voor examen

FTP's active mode faalt achter NAT omdat de server een inkomende verbinding probeert op te zetten. Passive FTP lost dit op door de client beide verbindingen te laten initieren. Dit is een concreet voorbeeld van het bredere NAT-probleem: protocollen die IP-adressen in hun payload embedden, werken niet goed met NAT.

## NVT en common FTP commands

De slides verwijzen ook naar **NVT**, Network Virtual Terminal. Dat past bij het idee dat het commandokanaal tekstueel en gestandaardiseerd moet kunnen werken over verschillende systemen heen.

Typische FTP-commando’s zijn:

- `USER`
- `PASS`
- `CWD`
- `PASV`
- `RETR`
- `STOR`
- `LIST`
- `QUIT`

### Voorbeeld van een sessie

Een klassieke FTP-sessie ziet er conceptueel ongeveer zo uit:

1. stuur `USER`
2. stuur `PASS`
3. kies directory met `CWD`
4. kies eventueel passive mode met `PASV`
5. haal bestand op met `RETR`
6. sluit af met `QUIT`

### belangrijk voor examen

FTP is een **bestandsprotocol met gescheiden control- en datakanalen**, wat het architecturaal anders maakt dan bijvoorbeeld HTTP.

## Samenvatting van het hoofdstuk

Dit hoofdstuk toont vier klassieke families van application-layerprotocollen:

- **remote login**: Telnet en SSH
- **e-mail**: POP, IMAP, webmail en SMTP
- **web**: HTML, URI, HTTP
- **bestandsoverdracht**: FTP

Door die voorbeelden zie je goed wat de application layer doet:

- mensgerichte of applicatiegerichte diensten aanbieden
- bovenop transportprotocollen zoals TCP
- met eigen commando’s, berichten en semantiek

## Wat je echt moet kennen

### belangrijk voor examen

- De drie protocol-stacks **(OSI, TCP/IP, Tanenbaum)** kunnen tekenen en protocollen op de juiste laag plaatsen
- Lastige gevallen: DHCP (application layer), RTP (application layer in Tanenbaum), OSPF (network layer, direct op IP)
- Drie applicaties die **UDP** gebruiken (DNS, DHCP, Wake-on-LAN) en waarom ze geen TCP gebruiken
- Waarom Telnet historisch belangrijk is
- Waarom Telnet onveilig is
- Hoe SSH dat oplost met encryptie en authenticatie
- Het verschil tussen **POP**, **IMAP** en **webmail**
- Dat **SMTP** een relayprotocol tussen mailservers is
- De basisflow van een SMTP-sessie
- Wat een **URI** is en hoe dat verschilt van een URL
- De rol van **DNS** voor het web
- Wat **HTML** doet
- Waarom HTTP **stateless** genoemd wordt
- De belangrijkste HTTP-methodes
- Waarom **HTTP 1.0** inefficiënt is (nieuwe TCP-verbinding per object, slow start telkens opnieuw)
- Wat **persistent connections**, **pipelining** en **HTTP/2 multiplexing** verbeteren
- Het cross-layer inzicht: HTTP is connectionless/stateless op applicatielaag, maar draait over connection-oriented TCP
- Wanneer het voordeel van HTTP 1.1 het grootst is (veel kleine embedded objecten)
- Waarom FTP twee verbindingen gebruikt (control apart van data)
- Waarom **active FTP faalt achter NAT** en hoe **passive FTP (PASV)** dat oplost
- De rol van typische FTP-commando’s zoals `USER`, `PASS`, `PASV`, `RETR` en `QUIT`

## Korte eindintuitie

Als je dit hoofdstuk tot één idee reduceert:

de application layer vertaalt ruwe netwerkconnectiviteit naar concrete diensten die mensen en software echt gebruiken, en elk protocol kiest daarvoor zijn eigen stijl van commando’s, veiligheid en sessiebeheer.

Dat is de echte lijn door deze slides.
