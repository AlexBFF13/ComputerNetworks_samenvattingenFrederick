# Deel 6 - Network 4: The Internet Protocol (IP)

In dit deel komen we bij het centrale protocol van de network layer: **IP**. Waar de vorige hoofdstukken vooral gingen over routing, congestie en internetworking, draait het hier om de concrete bouwstenen van internetverkeer zelf:

- hoe een IP-packet opgebouwd is
- hoe IPv4-adressen werken
- waarom IPv6 nodig werd
- welke rol ICMP en ARP spelen

Dit hoofdstuk is dus erg fundamenteel. Veel dingen die in andere delen al gebruikt werden, krijgen hier eindelijk hun concrete vorm.

## Ontwerpprincipes van IP

De slides beginnen met algemene ontwerpprincipes. Dat lijkt misschien wat abstract, maar eigenlijk verklaren ze waarom IP eruitziet zoals het eruitziet.

De grote lijn is:

- hou het simpel
- maak duidelijke keuzes
- verwacht heterogeniteit
- denk aan schaalbaarheid
- werk modulair
- vermijd te veel speciale gevallen

Dat past perfect bij IP. Het protocol probeert niet elk probleem zelf op te lossen. Het wil vooral een **algemene, schaalbare netwerklaag** zijn die op veel verschillende technologieën kan draaien.

### intuition

IP is sterk omdat het relatief beperkt probeert te blijven. Betrouwbaarheid, flow control en andere geavanceerde functies worden meestal niet in IP zelf gestopt, maar in hogere lagen.

## De structuur van het internet

Het internet wordt in de slides voorgesteld als een verzameling van **Autonomous Systems (AS)** die losjes met elkaar verbonden zijn.

Daar zie je opnieuw de hiërarchie terug:

- **Tier 1**: backbone-netwerken die zelf geen transit betalen
- **Tier 2**: regionale netwerken en ISPs die met elkaar peeren of transit kopen
- **Tier 3**: edge-netwerken die transit kopen van hoger gelegen netwerken

Dat is belangrijk om te onthouden, omdat IP nooit op één homogeen netwerk draait. Het leeft in een wereld van heel veel netwerken die onderling gekoppeld zijn.

## De IPv4-header

De IPv4-header bevat de informatie die routers nodig hebben om een packet correct door het netwerk te krijgen.

De slides lopen die header stap voor stap af. Het is belangrijk om niet gewoon de velden uit het hoofd te leren, maar te snappen **waarom elk veld bestaat**.

## Version en IHL

### Version

Dit veld zegt welke versie van IP gebruikt wordt.

- **IPv4** is de klassieke versie
- **IPv6** is de nieuwere versie
- **IPv5** was een experimenteel protocol en werd geen algemene internetstandaard

### IHL

**IHL** staat voor **Internet Header Length**.

Dat is nodig omdat de IPv4-header **variabele lengte** heeft. Door options kan die groter zijn dan het minimum.

De lengte wordt uitgedrukt in woorden van 32 bit. De minimale header is 5 woorden, de maximale 15.

### belangrijk voor examen

IPv4 heeft een **variabele headerlengte**, daarom is het IHL-veld nodig.

## DiffServ in de IP-header

De slides geven een korte introductie tot **DiffServ**. Dat veld laat een administratief domein toe om trafficklassen te markeren voor:

- prioritering
- behandeling door routers
- policing en shaping

Dus IP wordt hier gebruikt als drager voor QoS-informatie.

### Expedited Forwarding

Dit is bedoeld voor traffic die sneller of met meer prioriteit behandeld moet worden, zoals vaak bij **VoIP**.

Routers houden dan bijvoorbeeld:

- een gewone queue
- een expedited queue

en nemen eerst packets uit de snelle queue.

### Assured Forwarding

Dit is iets rijker. De slides tonen:

- 4 prioriteitsklassen
- 3 discard-klassen

Samen geeft dat 12 traffic classes.

Het punt hier is dat QoS in IP niet noodzakelijk één simpele "snel/langzaam"-bit is. Je kan ook verschillende niveaus van voorrang en weggooibaarheid meegeven.

## Total Length en Identification

### Total Length

Dit veld zegt hoe groot het volledige IPv4-packet is:

- header
- plus payload

De maximale grootte is 64 KB.

### Identification

Dit veld is belangrijk voor **fragmentatie**. Als een groot IP-packet opgesplitst wordt in meerdere fragmenten, krijgen die hetzelfde identification field zodat de ontvanger weet welke stukken bij elkaar horen.

## DF, MF en Fragment Offset

Deze velden horen allemaal bij fragmentatie.

### DF

**DF** betekent **Don’t Fragment**.

Als dit bit gezet is, mag een router het packet niet opsplitsen. Dat is belangrijk bij **path MTU discovery**.

### MF

**MF** betekent **More Fragments**.

Dat zegt dat er nog extra fragmenten van hetzelfde oorspronkelijke packet volgen.

### Fragment Offset

Dit veld zegt waar een fragment thuishoort binnen het oorspronkelijke packet. Het offset wordt in veelvouden van 8 bytes uitgedrukt.

### belangrijk voor examen

De combinatie van **Identification**, **MF** en **Fragment Offset** maakt reassemblage van IPv4-fragmenten mogelijk.

## TTL, Protocol en Header Checksum

### TTL

**TTL**, time to live, beperkt hoe lang een packet in het netwerk mag blijven rondzwerven.

Elke router verlaagt die waarde. Als TTL 0 wordt, wordt het packet weggegooid. Dat voorkomt eindeloze lussen.

### Protocol

Dit veld zegt welk protocol in de payload zit, bijvoorbeeld:

- TCP
- UDP
- ICMP

### Header checksum

Die checksum controleert enkel de **IPv4-header**.

### Waarom moet die checksum op elke hop opnieuw berekend worden?

Omdat routers onderweg velden wijzigen, vooral TTL. Zodra TTL verandert, verandert ook de header en moet de checksum dus herberekend worden.

### belangrijk voor examen

De IPv4 **header checksum** moet per hop opnieuw berekend worden omdat de header onderweg wijzigt, vooral door TTL.

## Source, Destination en Options

De source- en destinationvelden bevatten de **32-bit IPv4-adressen** van bron en bestemming.

Daarnaast heeft IPv4 ook **options**. Die maken de header flexibel, maar maken hem ook ingewikkelder en trager, omdat de header daardoor variabele lengte krijgt.

Dat is ook een van de redenen waarom IPv6 later naar een eenvoudiger vast headerformaat gegaan is.

## IP-adressen en NSAP

De slides gebruiken de term **NSAP**, Network Service Access Point, om aan te geven dat een IP-adres een netwerklaagadres is.

Belangrijk detail:

een IP-adres hoort bij een **netwerkinterface**, niet noodzakelijk bij een volledige machine.

Dus een toestel met meerdere interfaces kan ook meerdere IP-adressen hebben.

## Hoe schrijf je IPv4-adressen?

Een IPv4-adres is eigenlijk gewoon een reeks van **32 bits**. Omdat dat niet handig leest, splitsen we die in 4 bytes en schrijven we die in decimale vorm, gescheiden door punten.

Voorbeeld:

- binair: `11000000 10101000 00000101 00000010`
- dotted decimal: `192.168.5.2`

## Prefixen en subnet masks

Een IPv4-adres bestaat uit:

- een **network prefix**
- een **host part**

We schrijven dat typisch in **CIDR-notatie**, bijvoorbeeld:

`128.208.0.0/24`

Dat betekent dat de eerste 24 bits het netwerkdeel vormen.

Je kan hetzelfde ook schrijven met een subnetmask, zoals:

`255.255.255.0`

### Waarom zijn prefixen belangrijk?

Omdat routers niet voor elk individueel IP-adres een aparte route moeten hebben. Eén route per prefix volstaat meestal.

Dat vermindert routingtabellen enorm.

### belangrijk voor examen

Routers denken meestal in **prefixen**, niet in individuele hosts.

## Subnetting

Organisaties willen hun adresruimte vaak intern verder opdelen. Dat heet **subnetting**.

Het idee is:

- je krijgt een prefix van buitenaf
- intern verdeel je die verder in kleinere subprefixen

Zo kan je het netwerk logisch opsplitsen zonder dat je voor elk stukje compleet nieuwe publieke adresblokken nodig hebt.

## Klassengebaseerde adressering en het probleem daarvan

Oorspronkelijk werkte IPv4 met vaste klassen:

- **Class A**
- **Class B**
- **Class C**

Het probleem was dat dat vaak heel verspild was.

De slides leggen dat mooi uit met het "Three Bears Problem":

- Class A was veel te groot
- Class C vaak te klein
- Class B leek vaak "juist goed"

Maar in werkelijkheid bleek zelfs Class B vaak nog gigantisch overgedimensioneerd. Veel organisaties gebruikten maar een heel klein deel van hun toegewezen adressen.

## CIDR

Daarom schakelde men over op **CIDR**, **Classless Inter-Domain Routing**.

Het idee van CIDR is:

- geen vaste klassen meer
- prefixes mogen preciezere lengtes hebben
- routes met gemeenschappelijke prefix kunnen samengevoegd worden tot een groter blok, een **supernet**

Dat maakt adressering én routing veel efficiënter.

### Longest prefix match

CIDR brengt ook een belangrijk routeringsprincipe mee:

als meerdere prefixes lijken te passen, kies je de **langste overeenkomende prefix**.

Dus de meest specifieke route wint.

### belangrijk voor examen

CIDR vervangt de starre klassenstructuur door flexibele prefixlengtes, en routers gebruiken **longest prefix match** om overlappende routes te kiezen.

## Schaarste van IPv4-adressen

IPv4-adressen zijn schaars. Daarom kwamen allerlei oplossingen in zwang om met een beperkt aantal publieke adressen meer toestellen te bedienen.

Daar komen **DHCP** en **NAT** in beeld.

## DHCP

**DHCP**, Dynamic Host Configuration Protocol, is eigenlijk een **applicatielaagprotocol**.

Het zorgt voor automatische toewijzing van IP-adressen en andere netwerkconfiguratie.

Voordelen:

- geen manuele configuratie nodig
- adressen kunnen on-demand uitgedeeld worden
- mobiele toestellen kunnen eenvoudig een nieuw lokaal adres krijgen

### DHCP leases

DHCP-adressen zijn meestal geen permanente toewijzingen. Ze worden **geleased** voor een bepaalde tijd.

Daar zit een trade-off in:

- korte leases gebruiken de adresruimte efficiënter
- lange leases verlagen de belasting op de DHCP-server

## Privéadressen

Als DHCP alleen niet genoeg is, wordt vaak een extra stap gezet:

- één publiek adres aan de buitenkant
- veel private adressen aan de binnenkant

De klassieke private IPv4-ranges zijn:

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

Dat leidt rechtstreeks naar NAT.

## NAT

**NAT**, Network Address Translation, laat een heel intern netwerk met private adressen naar buiten treden via één of enkele publieke IP-adressen.

Het basisidee:

- intern gebruik je private adressen
- naar buiten toe lijkt alles van het publieke adres van de NAT-box te komen
- inkomende antwoorden worden door de NAT-box terug vertaald naar het juiste interne toestel

### Hoe weet NAT naar welk intern toestel iets terug moet?

Daar gebruikt NAT niet alleen IP-adressen voor, maar ook **poorten/TSAPs**.

Bij uitgaand verkeer:

- wordt het private bronadres vervangen door het publieke adres
- en wordt de bronpoort vervangen of herschreven naar een waarde die als index dient in de NAT-tabel

Bij inkomend verkeer:

- kijkt de NAT-box naar die poortmapping
- en reconstrueert zo het juiste interne doel

### intuition

Eén publiek IP-adres gedraagt zich als één centraal telefoonnummer, en de poorten zijn dan een soort extensienummers.

## Problemen met NAT

De slides zijn hier terecht kritisch. NAT werkt praktisch heel goed, maar botst met de oorspronkelijke IP-filosofie.

Belangrijke problemen:

1. NAT verbreekt het idee van één adres per interface.
2. NAT breekt het **end-to-end model**.
3. NAT is in de praktijk connection-oriented.
4. NAT maakt de lagen minder netjes.
5. NAT werkt vooral goed voor TCP en UDP.
6. NAT heeft moeite met protocollen die IP-adressen in de payload verstoppen.

### belangrijk voor examen

NAT is praktisch nuttig, maar theoretisch eigenlijk een **compromis** dat het end-to-end model en de nette layering van IP aantast.

## Waarom IPv6 nodig werd

IPv6 probeert meerdere problemen tegelijk aan te pakken:

- veel grotere adresruimte
- eenvoudiger headers
- minder nood aan NAT
- betere ondersteuning voor autoconfiguratie, QoS, multicast en mobility

### Belangrijkste kenmerken

- **128-bit adressen**
- **vaste header van 40 bytes**
- extra info in **extension headers**
- stateless autoconfiguration
- ondersteuning voor QoS
- vereenvoudigde fragmentatie
- ondersteuning voor mobility
- multicast en anycast als volwaardige modellen

## IPv6-adressen schrijven

IPv6-adressen zijn veel langer dan IPv4-adressen. Daarom schrijf je ze in hexadecimale groepen gescheiden door dubbele punten.

Voorbeeld:

`2001:0db8:0000:0000:3e07:54ff:fe74:abcd`

### Verkorten

IPv6 laat twee verkortingen toe:

1. leidende nullen in een groep weglaten
2. één aaneengesloten reeks groepen met alleen nullen vervangen door `::`

Maar dat tweede mag maar **één keer**, anders wordt het adres ambigu.

Voorbeeld uit de slides:

`2001:0db8:0000:0200:0000:0000:0000:0007`

wordt:

`2001:db8:0:200::7`

### belangrijk voor examen

Bij IPv6 mag `::` maar **één keer** gebruikt worden in een adres.

## IPv6-ranges en prefixen

Ook in IPv6 schrijf je adressenblokken met:

`adres/prefixlengte`

Zoals:

- `2001:db8::/32`
- `2001:db8::/48`

Dus het prefixidee blijft, alleen wordt de adresruimte veel groter.

## IPv6-adrestypes

IPv6 gebruikt drie hoofdtypes van adressen:

- **unicast**
- **multicast**
- **anycast**

Wat opvallend ontbreekt is klassieke **broadcast**. In IPv6 wordt die rol typisch door multicast overgenomen.

### Unicast

Eén adres verwijst naar één interface.

### Multicast

Eén adres verwijst naar meerdere interfaces, en alle leden van die groep krijgen de data.

### Anycast

Eén adres verwijst naar meerdere interfaces, maar de data gaat naar één van hen, typisch de dichtstbijzijnde of best bereikbare.

Dat is handig voor load balancing en redundantie.

## Belangrijke unicast-ranges in IPv6

De slides benadrukken drie soorten unicastadressen.

### Global Unicast

- range: `2000::/3`
- routable op het globale internet

Structuur:

- 48 bits network ID
- 16 bits subnet ID
- 64 bits interface ID

Dit zijn de adressen voor toestellen die echt globaal bereikbaar moeten zijn.

### Unique Local Unicast

- range: `fc00::/7` met in de praktijk vaak `fd00::/8`

Deze zijn bedoeld voor private interne netwerken, maar dan netter dan klassieke NAT-constructies.

### Link Local Unicast

- range: `fe80::/10`

Deze zijn alleen lokaal geldig op één link en worden niet gerouteerd.

Elke IPv6-interface moet er normaal één hebben.

### belangrijk voor examen

Je moet minstens het onderscheid kennen tussen:

- **global unicast**
- **unique local**
- **link local**

## Interface Identifier

In IPv6 wordt een deel van het adres gebruikt als **Interface Identifier (IID)**.

De slides tonen hoe dat afgeleid kan worden uit EUI-64 of uit een MAC-adres. Dat is belangrijk omdat het laat zien hoe adressen deels automatisch gevormd kunnen worden.

Maar dat heeft ook privacygevolgen.

## Meerdere adressen per node

Een IPv6-node luistert vaak niet op maar één adres. De slides noemen onder andere:

- link-local
- loopback
- toegewezen unicast
- toegewezen anycast
- all-nodes multicast
- solicited-node multicast
- en voor routers ook extra routergerelateerde adressen

Dat toont dat een node in IPv6 veel rijker ingebed is in verschillende adresrollen tegelijk.

## Security en adressen in IPv6

Omdat IPv6 geen NAT meer nodig heeft om schaarste op te vangen, wordt **end-to-end bereikbaarheid** weer normaler.

Maar dat betekent ook:

- je kan niet meer blind vertrouwen op NAT als beschermlaag
- firewalls op hosts worden belangrijker

Daarnaast kan een stabiele interface identifier ook **privacyproblemen** geven, omdat toestellen zo makkelijker te volgen zijn.

## Het IPv6-datagram en de header

Een IPv6-packet bestaat uit:

- een vaste header
- optionele extension headers
- payload van hogere lagen

De minimale MTU is **1280 bytes**.

### Wat is het grote verschil met IPv4?

IPv6 heeft een **vaste headerlengte van 40 bytes**. Dat maakt verwerking eenvoudiger en voorspelbaarder dan bij IPv4, waar options de header variabel maken.

Veel velden uit IPv4 zijn ook verhuisd of verdwenen:

- fragmentatie-info zit niet in de basischeader
- options zitten in extension headers
- header checksum is verwijderd

### Waarom geen header checksum meer?

Omdat checks al elders gebeuren en men routers niet op elke hop opnieuw headerchecksums wou laten berekenen. Dat maakt forwarding efficiënter.

### belangrijk voor examen

De IPv6-header is **vast 40 bytes**, en veel complexiteit uit IPv4 is verplaatst naar **extension headers** of verwijderd.

## ICMP

**ICMP** is een control-protocol dat samenwerkt met IP. Het draagt geen gewone applicatiedata, maar foutmeldingen en diagnose-informatie.

De slides noemen een aantal typische ICMP-berichten.

### Destination unreachable

Wordt gestuurd als een packet niet geleverd kan worden, bijvoorbeeld:

- er is geen route
- DF staat aan en het packet is te groot voor de MTU

### Time exceeded

Wordt gestuurd als TTL op 0 komt. Dat wijst vaak op:

- routing loops
- of een te korte TTL

### Parameter problem

Geeft aan dat er iets mis is in de IP-header of stack.

### Source quench

Historisch choke-achtig bericht voor congestie. Belangrijk vooral als concept, minder als modern praktijkmechanisme.

### Redirect

Laat een host of router weten dat er een efficiëntere route bestaat.

### Echo request / reply

Dit zijn de bekende **ping** en **pong**.

### Timestamp request / reply

Werd gebruikt voor tijdssynchronisatie.

### Router advertisement / solicitation

Wordt gebruikt om routers aan hosts kenbaar te maken.

### belangrijk voor examen

ICMP is het controle- en foutmeldingskanaal van IP, met klassiek belangrijke types zoals:

- destination unreachable
- time exceeded
- redirect
- echo request/reply

## Traceroute

**Traceroute** gebruikt ICMP slim zonder dat het netwerk daar speciale ondersteuning voor hoeft te hebben.

De truc is:

- stuur eerst een packet met `TTL = 1`
- dan `TTL = 2`
- dan `TTL = 3`

Elke router waar TTL op 0 valt, stuurt een **time exceeded** terug. Zo kan de zender stap voor stap ontdekken welke routers op het pad liggen.

### intuition

Traceroute laat een pakket telkens net één hop verder sterven, en gebruikt de foutmeldingen om het pad zichtbaar te maken.

## ARP

Tot slot komt **ARP**, Address Resolution Protocol.

IP gebruikt **logische adressen**, maar op een lokaal medium zoals Ethernet wordt op hopniveau gerouteerd met **MAC-adressen**.

Dus ergens moet je een IP-adres kunnen omzetten naar een MAC-adres. Dat is precies wat ARP doet.

## ARP in actie

Binnen hetzelfde subnet werkt het zo:

1. Host 1 kijkt eerst in zijn **ARP-cache**
2. als daar al een mapping staat, gebruikt hij meteen het MAC-adres
3. zo niet, dan stuurt hij een **broadcast**: wie heeft dit IP-adres?
4. de juiste host antwoordt met zijn MAC-adres
5. Host 1 zet die mapping in cache en kan verder communiceren

### belangrijk detail

Dit werkt voor hosts **in hetzelfde subnet**. Voor een host op een extern netwerk verwacht je normaal niet het MAC-adres van de verre host, maar van de **default gateway/router** die de volgende hop vormt.

## Optimalisaties in ARP

De slides noemen drie belangrijke optimalisaties:

- **ARP-cache**: vermijdt herhaalde broadcasts
- een ARP-request bevat zelf ook al MAC-info, zodat de ontvanger meteen iets kan cachen
- **gratuitous ARP**: een host broadcast zijn nieuwe IP/MAC-combinatie na configuratie

Dat vermindert discovery-overhead en helpt caches up-to-date houden.

### belangrijk voor examen

ARP mapt **IP naar MAC** op een lokale link, en gebruikt daarvoor een combinatie van:

- cache
- broadcast request
- unicast antwoord

## Samenvatting van het hoofdstuk

Dit hoofdstuk zet de network layer om van abstract idee naar concrete internetpraktijk.

Je ziet:

- hoe **IPv4** opgebouwd is
- hoe **adressering en prefixen** werken
- waarom **CIDR** nodig werd
- hoe **DHCP** en **NAT** de schaarste van IPv4 mee opvingen
- waarom **IPv6** ontworpen werd
- welke rol **ICMP** speelt bij fouten en diagnose
- en hoe **ARP** IP-adressen lokaal aan MAC-adressen koppelt

## Wat je echt moet kennen

### belangrijk voor examen

- De algemene ontwerpfilosofie van IP
- De belangrijkste velden van de **IPv4-header**
- Waarvoor **Identification**, **DF/MF** en **Fragment Offset** dienen
- Waarom de **IPv4 header checksum** per hop herberekend wordt
- Wat een **NSAP** is en waarom IP-adressen aan interfaces hangen
- Hoe **prefixen**, **subnetting** en **CIDR** werken
- Wat **longest prefix match** betekent
- De private IPv4-adresranges
- Hoe **DHCP** en **leases** werken
- Hoe **NAT** ongeveer werkt en waarom het problematisch is
- De grote verschillen tussen **IPv4** en **IPv6**
- Hoe je een **IPv6-adres** noteert en verkort
- Het verschil tussen **global unicast**, **unique local** en **link local**
- Waarom IPv6 geen klassieke **broadcast** gebruikt
- De rol van **ICMP** en hoe **traceroute** werkt
- Hoe **ARP** een IP-adres naar een MAC-adres vertaalt

## Korte eindintuitie

Als je dit hoofdstuk samenvat in één gedachte:

IP is de eenvoudige maar krachtige gemeenschappelijke taal van het internet, en alles errond — adressering, controleberichten, lokale adresresolutie en de overstap naar IPv6 — probeert die taal schaalbaar en bruikbaar te houden.

Dat is de rode draad door deze slides.
