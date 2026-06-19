# Deel 10 - Application 2: DHCP en DNS

In dit deel verschuift de focus binnen de application layer naar twee protocollen die niet altijd zo zichtbaar zijn voor gebruikers, maar die essentieel zijn voor bijna alles wat op een netwerk gebeurt:

- **DHCP** om hosts automatisch een netwerkconfiguratie te geven
- **DNS** om namen naar adressen te vertalen

Die twee lijken op het eerste gezicht heel verschillend, maar ze lossen eigenlijk een gelijkaardig soort probleem op: ze nemen complexe technische details weg zodat gebruikers en applicaties niet alles zelf hoeven te weten of te configureren.

## DHCP: automatische netwerkconfiguratie

**DHCP**, Dynamic Host Configuration Protocol, is gedefinieerd in RFC 2131. Het doel is eenvoudig maar cruciaal:

- automatisch IP-adressen toewijzen
- manuele configuratie vermijden
- mobiliteit van toestellen ondersteunen

DHCP gebruikt **UDP**. Dat is niet zomaar een keuze, maar een fundamentele noodzaak.

### Waarom DHCP UDP gebruikt en niet TCP

Dit is een klassieke examenvraag. Het antwoord gaat dieper dan "UDP is simpeler":

**TCP is fundamenteel onmogelijk voor DHCP.** TCP vereist een three-way handshake met een **geldig bron-IP-adres** en een **geldig bestemmings-IP-adres**. Tijdens DHCPDISCOVER heeft de client **geen van beide**:

- De client heeft nog geen eigen IP-adres (dat is precies wat hij probeert te krijgen)
- De client kent het IP-adres van de DHCP-server niet

Bovendien:

- TCP is **unicast** en **connection-oriented** -- het kan niet broadcasten
- DHCP heeft **broadcast** nodig: de client moet naar `255.255.255.255` zenden vanuit `0.0.0.0`
- UDP laat toe om vanuit `0.0.0.0` naar `255.255.255.255` te zenden **zonder voorafgaande state**

Het gaat dus niet om voorkeur, maar om **onmogelijkheid**: TCP kan structureel niet werken in een situatie waar de zender nog geen netwerkconfiguratie heeft.

### waarom bestaat dit?

Zonder DHCP zou elk toestel:

- handmatig een IP-adres moeten krijgen
- handmatig een subnetmask en gateway moeten kennen
- handmatig aangepast moeten worden als het van netwerk verandert

Dat is op schaal onwerkbaar, zeker voor laptops, smartphones en tijdelijke toestellen.

## DHCP geeft leases, geen permanente adressen

Een belangrijk idee uit de slides is dat DHCP-adressen meestal niet permanent zijn. Ze worden toegekend als **lease** voor een bepaalde periode.

Dat heeft voordelen, maar er zit een duidelijke **trade-off** in:

| Lease-duur | Voordeel | Nadeel |
|------------|----------|--------|
| **Kort** (minuten) | Efficiënt hergebruik van schaarse adressen; snel vrijkomen bij inactieve hosts | Meer server-load door frequente vernieuwingen |
| **Lang** (dagen/weken) | Minder overhead voor server en netwerk | Adressen blijven geblokkeerd door hosts die al lang niet meer actief zijn |

De keuze hangt af van de omgeving:

- **Cafe-wifi met veel korte bezoekers**: korte leases (30 min) zodat adressen snel vrijkomen
- **Kantoornetwerk met vaste werkstations**: lange leases (dagen) om onnodige vernieuwingen te vermijden
- **IoT-netwerk met veel devices en beperkte adresruimte**: korte leases

### intuition

Een DHCP-adres is eigenlijk geen "eigendom", maar eerder een tijdelijke reservering.

## Het DHCP-berichtformaat

De slides tonen het DHCP message format en benadrukken dat een client niet alles van dat bericht kan invullen. In het begin weet een client immers vaak nog bijna niets.

Belangrijke velden zijn:

- **OpCode**: request of reply
- **Hardware Type**: bijvoorbeeld Ethernet
- **Hardware Address Length**: bijvoorbeeld 6 bytes voor Ethernet
- **Hop Count**
- **Transaction ID**
- **Seconds**
- **Client IP address**
- **Your IP address**
- **Server IP address**
- **Gateway IP address**
- **Client hardware address**
- **Server host name**
- **Boot file name**
- **Options**

### Transaction ID

Dit veld is praktisch heel belangrijk. Het laat een client toe om antwoorden te koppelen aan de juiste request.

### Options

Het type DHCP-bericht zelf zit niet als apart hoofdveld in de header, maar wordt via een **option** meegegeven. De slides tonen onder andere deze types:

- `DHCPDISCOVER`
- `DHCPOFFER`
- `DHCPREQUEST`
- `DHCPDECLINE`
- `DHCPACK`
- `DHCPNAK`
- `DHCPRELEASE`
- `DHCPINFORM`

Er bestaan daarnaast nog veel meer DHCP-opties.

### belangrijk voor examen

De **message type** van DHCP zit in een **option**, niet als vast hoofdveld in de basisheader.

## De eerste fase: discovery

Wanneer een client nog geen IP-adres heeft, zit hij in een lastig startpunt:

- hij wil een server vinden
- maar hij heeft zelf nog geen bruikbaar adres

Daarom zijn in deze fase de berichten **broadcast**.

De standaardlogica is:

client  
\--> `DHCPDISCOVER`  
server  
\--> `DHCPOFFER`

Omdat de client nog geen adres heeft, moet dit allemaal heel algemeen en broadcastachtig beginnen.

## Een adres effectief verkrijgen

Na de offers kiest de client een aanbod en bevestigt hij dat met een request.

De klassieke flow is:

client  
\--> `DHCPDISCOVER`  
server  
\--> `DHCPOFFER`  
client  
\--> `DHCPREQUEST`  
server  
\--> `DHCPACK`

Vanaf dat moment heeft de client een bruikbaar IP-adres en kan verdere communicatie via normale IP verlopen.

### De broadcast-logica per fase

Elk bericht in de DHCP-flow heeft een specifieke reden om broadcast of unicast te zijn:

| Fase | Richting | Bron-IP | Doel-IP | Waarom broadcast? |
|------|----------|---------|---------|-------------------|
| DISCOVER | Client -> all | `0.0.0.0` | `255.255.255.255` | Client kent noch eigen IP noch server-IP |
| OFFER | Server -> all | Server-IP | `255.255.255.255` | Client heeft nog geen IP om unicast te ontvangen |
| REQUEST | Client -> all | `0.0.0.0` | `255.255.255.255` | Informeert ook **afgewezen servers** dat hun aanbod niet gekozen is |
| ACK | Server -> client | Server-IP | Client-IP of broadcast | Bevestigt de lease |

Let op: de REQUEST is ook broadcast, niet alleen om de gekozen server te bereiken, maar ook zodat **andere DHCP-servers** weten dat hun OFFER niet geaccepteerd is en hun gereserveerde adres weer kunnen vrijgeven.

### belangrijk voor examen

De basisflow van DHCP is:

- `DISCOVER`
- `OFFER`
- `REQUEST`
- `ACK`

Dat is een klassiek examenpunt. Ken ook de broadcast-logica en waarom elk bericht broadcast moet zijn.

## Leases vernieuwen

Een lease blijft niet eeuwig geldig. De slides zeggen dat een lease vernieuwd wordt wanneer ongeveer **50% van de lease-tijd verstreken is**.

Als de DHCP-server die vernieuwing weigert en een `DHCPNACK` terugstuurt, dan moet het adres worden vrijgegeven.

Dat toont dat DHCP dynamisch blijft: een host mag een adres gebruiken zolang de server dat blijft toestaan.

## Een adres vrijgeven

Wanneer een host zijn adres niet meer nodig heeft, kan hij expliciet een:

- `DHCPRELEASE`

sturen.

Dat helpt om adresruimte sneller opnieuw bruikbaar te maken.

## Duplicate Address Detection

Zelfs als een server een adres toekent, wil je nog vermijden dat er per ongeluk al een ander toestel met datzelfde adres bestaat.

Daarom doen hosts na toewijzing vaak **duplicate address detection**.

Als de client merkt dat het adres al in gebruik is, stuurt hij een:

- `DHCPDECLINE`

De slides zeggen expliciet dat dit via **ARP** gebeurt.

### waarom is dit nodig?

Omdat adressen in de praktijk fout geconfigureerd kunnen zijn, of omdat er lokale inconsistenties kunnen ontstaan. DHCP alleen garandeert niet magisch dat de rest van het subnet zich perfect gedraagt.

### belangrijk voor examen

**Duplicate address detection** na DHCP-toewijzing gebeurt met **ARP**, en bij conflict stuurt de client een `DHCPDECLINE`.

## DNS: namen in plaats van adressen

Na DHCP schakelen de slides over naar **DNS**, het Domain Name System.

Het basisprobleem is eenvoudig:

- machines werken met IP-adressen
- mensen werken liever met namen

In de vroege internetdagen werd dat nog opgelost met één centraal bestand, `hosts.txt`, dat mappings van namen naar IP-adressen bevatte. Hosts downloadden dat periodiek en hielden lokaal een kopie bij.

Dat werkte even, maar schaalde slecht.

### waarom schaalde dat niet?

Omdat je bij groeiende aantallen hosts en organisaties:

- één centraal bottleneckbestand krijgt
- veel coördinatie nodig hebt
- weinig flexibiliteit hebt

Dus kwam DNS als hiërarchische, gedistribueerde oplossing.

### Waarom DNS over UDP (en niet TCP)

Dit is een veelgestelde examenvraag. De redenering heeft meerdere facetten:

1. **DNS-queries zijn klein en kort**: een typische query past in een enkel UDP-datagram (< 512 bytes). TCP's three-way handshake zou per lookup drie extra berichten kosten, wat de latency verdrievoudigt voor een simpele naamresolutie.

2. **Elke iteratieve stap zou een aparte TCP-verbinding nodig hebben**: een lokale resolver doet vaak 2-4 iteratieve stappen (root -> TLD -> authoritative). Met TCP zou dat 2-4 aparte TCP-handshakes betekenen.

3. **Anycast**: UDP maakt **anycast** mogelijk. Meerdere fysieke root-servers delen hetzelfde IP-adres, en het dichtstbijzijnde exemplaar antwoordt. Met TCP zou de verbindingsstaat problemen geven als packets naar verschillende fysieke servers gaan.

4. **Statelessness**: DNS-servers zijn **stateless** -- ze hoeven geen verbindingsstaat bij te houden per client. Dit maakt ze veel schaalbaarder, vooral voor root- en TLD-servers die miljoenen queries per seconde verwerken.

**Uitzondering**: bij te grote antwoorden (> 512 bytes, of met DNSSEC) valt DNS terug op TCP. Zone transfers tussen DNS-servers gebruiken ook TCP omdat die betrouwbaarheid en grote datahoeveelheden vereisen.

## Een paar praktische DNS-notes

De slides geven twee leuke maar belangrijke opmerkingen.

### DNS-namen zijn niet case-sensitive

Dus:

- `WWW.GOOGLE.COM`
- `www.google.com`

worden als dezelfde DNS-naam behandeld.

Maar het padgedeelte van een URL kan wel degelijk case-sensitive zijn.

### DNS-namen eindigen technisch op een punt

Volledig uitgeschreven eindigt een DNS-naam op een punt, zoals:

- `www.google.com.`

In de praktijk laten we dat laatste punt bijna altijd weg.

### belangrijk voor examen

DNS-namen zelf zijn **niet case-sensitive**, maar URL-resourcepaden kunnen dat wel zijn.

## Hiërarchie in domeinnamen

De grote kracht van DNS is **hiërarchie**.

Een naam zoals:

`nix.cs.kuleuven.ac.be`

bevat zijn hiërarchische structuur al in de naam zelf.

Van rechts naar links krijg je:

- land: `be`
- type/academisch domein: `ac`
- organisatie: `kuleuven`
- suborganisatie of afdeling: `cs`
- machine of host: `nix`

De slides vergelijken dit ook met een postadres. Dat is een goede intuïtie:

- bovenaan brede regio
- lager steeds specifieker

### waarom is die hiërarchie zo belangrijk?

Omdat ze:

- lookup complexiteit beheersbaar maakt
- verantwoordelijkheid kan delegeren
- gedistribueerde opslag mogelijk maakt

## Top Level Domains

De slides onderscheiden drie grote soorten **TLDs**:

- landcodes zoals `.be`
- generieke namen zoals `.com`, `.org`, `.net`
- bepaalde speciale Amerikaanse domeinen zoals `.edu`, `.gov`, `.mil`

De administratie daarvan gebeurt sinds 1998 onder **ICANN**. Lager in de hiërarchie wordt verantwoordelijkheid gedelegeerd naar **registrars**.

Voorbeelden uit de slides:

- `.com` via Verisign
- `.be` via DNS Belgium
- `.mil` via de Amerikaanse overheid

Dat maakt duidelijk dat hiërarchie in DNS niet alleen technisch is, maar ook organisatorisch en bestuurlijk.

## Organisaties en suborganisaties

Een organisatie krijgt een domeinnaam via een registrar en is daarna vrij om intern verdere namen te structureren.

Belangrijk detail:

die interne substructuur hoeft niet telkens expliciet met hogere niveaus in de hiërarchie te worden afgestemd.

Dat maakt DNS praktisch schaalbaar. Zodra je een zone beheert, krijg je intern veel autonomie.

## Waarom hiërarchie helpt

De slides noemen drie grote voordelen van hiërarchie:

- bottlenecks vermijden
- managementcomplexiteit spreiden
- regionale of organisatorische controle toelaten

Dat is eigenlijk precies waarom DNS op wereldschaal nog werkbaar is.

## DNS-records

DNS-data wordt opgeslagen in **resource records**.

Een volledig record bevat onder andere:

- **Domain Name**
- **Time to Live**
- **Class**
- **Type**
- **Value**

### TTL

De **Time to Live** geeft aan hoe stabiel of cachebaar een record is.

Dat is heel belangrijk, want DNS draait in de praktijk sterk op caching.

## Belangrijke recordtypes

De slides noemen een reeks recordtypes. De belangrijkste die je conceptueel moet kennen zijn:

- `SOA`: start of authority voor een zone
- `A`: IPv4-adres van een host
- `NS`: naamserver voor een domein
- `MX`: mailserverinformatie
- `CNAME`: canonical name, een alias
- `PTR`: pointer, vaak voor reverse mapping
- `HINFO`
- `TXT`

### voorbeelden

- Een `A`-record koppelt een naam aan een IPv4-adres.
- Een `MX`-record zegt welke mailserver mail voor een domein wil ontvangen.
- Een `NS`-record zegt welke naamserver bevoegd is voor een stuk van de hiërarchie.

### belangrijk voor examen

Je moet minstens de betekenis kennen van:

- `A`
- `NS`
- `MX`
- `CNAME`

## Hiërarchische distributie van DNS-data

De slides tonen mooi het verschil tussen:

- een **leaf name server**
- een **branch name server**
- de **root name server**

### Leaf server

Die bevat concrete records voor een zone dicht bij de uiteindelijke hosts. Daar vind je vaak directe mappings naar IP-adressen.

### Branch server

Die bevat deels concrete informatie en deels verwijzingen naar diepere name servers.

### Root server

Die kent vooral de TLD-naamservers en staat dus helemaal bovenaan de boom.

### intuition

Hoe hoger in de hiërarchie, hoe minder specifiek de inhoud meestal wordt.

## Zones

DNS verdeelt de namespace in **niet-overlappende zones**.

Elke zonebeheerder beslist zelf waar interne zonegrenzen getrokken worden. Daar zit opnieuw een trade-off in:

- meer servers kan helpen voor load balancing en beheer
- maar te veel zones verhogen ook de managementoverhead

De zonegrenzen volgen vaak organisatorische grenzen, maar dat hoeft niet noodzakelijk.

## Hoe een DNS-lookup navigeert door de hiërarchie

De slides leggen uit dat lookup door de hiërarchie bestaat uit twee belangrijke stukken:

- **authoritative** versus **cached** informatie
- **recursive** versus **iterative** opvraging

### Authoritative versus cached

Een **authoritative** record komt van de zonebeheerder die echt bevoegd is voor dat domein.

Een **cached** record is een tijdelijk onthouden kopie.

### Recursive lookup

Bij een recursive stap verwacht de vraagsteller een volledig antwoord. De lokale name server probeert dan het werk verder uit te voeren tot hij een finaal antwoord kan geven.

### Iterative lookup

Bij een iterative stap krijg je vaak een gedeeltelijk antwoord terug, bijvoorbeeld:

"Ik weet het niet exact, maar vraag deze andere server eens."

Dat zie je typisch hoger in de hiërarchie, bijvoorbeeld bij de rootservers.

### Belangrijk proces

Een lokale name server kent via configuratie één of meer rootservers. Van daaruit "zoomt" de lookup steeds verder in op de server die de juiste zone beheert.

### Het hybride model (split design)

In de praktijk combineert DNS recursieve en iteratieve stappen in een **hybride model**. Dit is een belangrijk examenpunt dat de prof "split design" noemt.

Het hybride model werkt omdat elke component doet waar hij goed in is:

- **Client** stuurt een **recursieve** vraag aan de lokale resolver: "Geef mij het antwoord, ik wil niet zelf rondvragen."
- **Lokale resolver** doet **iteratieve** stappen naar root/TLD/authoritative servers: "Ik weet het niet, maar probeer deze server eens."
- **Root servers** geven enkel **stateless referrals**: ze verwijzen naar TLD-servers zonder zelf het hele werk te doen.

Waarom is dit hybride model efficienter dan puur recursief of puur iteratief?

| Aspect | Puur iteratief | Puur recursief | Hybride (praktijk) |
|--------|---------------|----------------|-------------------|
| Client-complexiteit | Hoog -- client moet zelf alle stappen doen | Laag -- een vraag, een antwoord | Laag |
| Load op root servers | Hoog -- elke client begint bij root | Exploderend -- root moet alles resolven | Laag -- enkel stateless referrals |
| Caching | Geen gedeelde cache | Server cached, maar niet schaalbaar | Lokale resolver cached voor hele regio |
| Schaalbaarheid | Slecht | Slecht | Goed |

Het cruciale inzicht is dat de **lokale resolver als gedeelde cache** fungeert. Als een gebruiker in dezelfde organisatie al eerder `www.google.com` heeft opgezocht, zit het antwoord in de cache van de lokale resolver. De root- en TLD-servers worden dan niet meer belast.

### belangrijk voor examen

DNS-lookup combineert typisch:

- een **recursive** component aan de kant van de lokale name server
- en **iterative** stappen hoger in de hiërarchie
- Dit hybride model (split design) combineert lage client-complexiteit met schaalbare, stateless root-servers en effectieve caching

## DNS op grote schaal

De slides benadrukken dat er miljoenen name servers zijn en dat root- en TLD-servers kritieke infrastructuur vormen.

Omdat die zo belangrijk zijn, wordt **anycast** gebruikt om:

- redundantie te bieden
- performance te verbeteren
- weerstand tegen aanvallen te verhogen

Dat toont opnieuw hoe DNS veel meer is dan "een telefoonboek voor het internet". Het is een gigantisch gedistribueerd systeem.

## DNS-messages en resource records

De laatste schema’s tonen nog eens dat DNS-berichten in feite vooral draaien om vragen en antwoorden over resource records.

Dus conceptueel moet je DNS zien als:

- een hiërarchische namespace
- een verzameling van zones
- gevuld met resource records
- die via lookup door de hiërarchie opgezocht worden

## Samenvatting van het hoofdstuk

Dit hoofdstuk behandelt twee onmisbare ondersteunende protocollen van de application layer.

### DHCP

- geeft hosts automatisch configuratie
- werkt met leases
- gebruikt een discover-offer-request-ack-flow
- detecteert dubbele adressen via ARP

### DNS

- vertaalt namen naar adressen
- gebruikt hiërarchie om schaalbaar te blijven
- deelt verantwoordelijkheid via zones en registrars
- gebruikt resource records en caching

## Wat je echt moet kennen

### belangrijk voor examen

- Waarom **DHCP** nodig is
- Waarom DHCP **UDP** gebruikt en waarom **TCP fundamenteel onmogelijk** is (geen bron-IP, geen bestemmings-IP, broadcast nodig)
- Het idee van **leases** en de **trade-off** tussen korte en lange leases
- De basisflow `DISCOVER -> OFFER -> REQUEST -> ACK` inclusief de **broadcast-logica** per fase
- Waarom REQUEST ook broadcast is (om afgewezen servers te informeren)
- De rol van `DHCPDECLINE`, `DHCPRELEASE` en `DHCPNACK`
- Waarom duplicate address detection met **ARP** gebeurt
- Waarom het oude `hosts.txt`-model niet schaalde
- Hoe de **hiërarchie** in DNS-namen zit
- Wat **TLDs**, **registrars** en **zones** zijn
- Wat een **resource record** is
- De betekenis van `A`, `NS`, `MX` en `CNAME`
- Het verschil tussen **authoritative** en **cached** data
- Het verschil tussen **recursive** en **iterative** lookup
- Het **hybride model (split design)**: waarom de combinatie van recursief (client-kant) en iteratief (hoger in hierarchie) het beste schaalt
- Waarom **DNS over UDP** werkt (kleine queries, anycast, stateless) en wanneer DNS toch TCP gebruikt (grote antwoorden, zone transfers)
- Waarom DNS-hiërarchie schaalbaarheid en delegatie mogelijk maakt

## Korte eindintuitie

Als je dit hoofdstuk tot één idee samenvat:

DHCP geeft een toestel automatisch een plaats op het netwerk, en DNS geeft mensen een bruikbare manier om die plaatsen met namen terug te vinden.

Dat is precies waarom deze twee protocollen zo fundamenteel zijn.
