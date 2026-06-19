# Deel 15 - Framing and Select Physical Layer Topics

In dit deel zitten eigenlijk twee grote thema’s door elkaar:

- **framing**, dus hoe je bits groepeert tot duidelijke frames
- een selectie uit de **physical layer**, dus via welke media signalen zich verplaatsen

Dat lijkt op het eerste gezicht een wat vreemde combinatie, maar de link is logisch. Eerst moet je weten hoe je een stroom bits logisch opdeelt in stukken. Daarna moet je weten over welk fysiek medium die bits of signalen eigenlijk reizen.

## Waarom framing nodig is

Wanneer bits gewoon als een continue stroom binnenkomen, dan heeft de ontvanger een probleem:

waar begint een frame en waar eindigt het?

Zonder zo’n afbakening kan je niet betrouwbaar weten:

- welke bits bij elkaar horen
- wanneer één boodschap stopt
- waar de volgende boodschap begint

Framing is dus het mechanisme dat een ruwe bitstroom omzet in herkenbare blokken.

## Byte count

Een eerste eenvoudige oplossing is **byte count**.

Daarbij zet je vooraan in het frame een lengteveld. Dat veld zegt hoeveel bytes het frame bevat.

De logica is simpel:

- lees eerst de lengte
- lees daarna exact zoveel bytes
- dan weet je waar het frame eindigt

### waarom is dit aantrekkelijk?

Omdat het compact en efficiënt is. Je verspilt geen extra markeringen tussen de data.

### waar zit het probleem?

Als dat lengteveld zelf beschadigd raakt, dan zit je meteen scheef. De ontvanger kan dan te weinig of te veel bytes lezen en zo de framegrenzen verliezen.

Daar ontstaat een kettingprobleem:

- één fout in de lengte
- verkeerde interpretatie van het frame-einde
- ook de start van het volgende frame wordt onduidelijk

De slides zeggen terecht dat een checksum dat niet zomaar oplost. Als je al niet meer weet waar het volgende frame begint, dan weet je ook niet meer goed waarop die checksum precies slaat.

### belangrijk voor examen

Bij **byte count** is het zwakke punt dat een fout in het lengteveld de synchronisatie volledig kan verstoren.

## Flag bytes

Een tweede aanpak werkt met **flag bytes**.

Hier gebruik je een speciale bytewaarde om het begin en einde van een frame te markeren.

Dat is intuïtief:

- speciale byte voor start/einde
- alles daartussen hoort bij hetzelfde frame

### het nieuwe probleem

Wat als diezelfde speciale flag-byte toevallig in de data zelf voorkomt?

Dan zou de ontvanger kunnen denken dat het frame al gedaan is, terwijl dat niet zo is.

## Byte stuffing

Daarom gebruik je bij flag bytes meestal **byte stuffing**.

Het idee is:

- je kiest een speciale escape-byte
- telkens wanneer een flag-byte in de data voorkomt, zet je eerst die escape-byte
- de ontvanger weet dan dat de volgende speciale byte gewone data is, en geen framegrens

Maar ook hier krijg je meteen de volgende vraag:

wat als de escape-byte zelf in de data voorkomt?

Dan moet ook die ge-escaped worden. Het principe blijft hetzelfde: je voegt extra bytes toe om ambiguïteit weg te nemen.

### intuïtie

Byte stuffing is eigenlijk een manier om te zeggen:

"dit teken lijkt speciaal, maar is hier gewoon inhoud."

## Bit stuffing

De slides geven daarna de efficiëntere variant: **bit stuffing**.

Stel dat je een speciale flag gebruikt met patroon:

`01111110`

Dat patroon wil je alleen laten voorkomen als echte framegrens, niet in gewone data.

De oplossing:

- telkens wanneer de zender **vijf opeenvolgende 1-bits** ziet in de data
- voegt hij automatisch een `0` toe

De ontvanger doet het omgekeerde:

- ziet hij vijf 1-bits gevolgd door een 0
- dan verwijdert hij die extra 0 weer

Zo kan het vlagpatroon niet per ongeluk ontstaan in de payload.

### waarom is bit stuffing vaak beter?

Omdat je niet op bytegrenzen hoeft te werken en dus fijner kan omgaan met de bitstroom. Dat maakt het vaak efficienter dan byte stuffing.

### Rekenvoorbeeld: wanneer bit stuffing efficienter is

Stel dat de byte `0xFF` (binair: `11111111`) in de data voorkomt en ook de flag-byte is:

| Methode | Resultaat | Overhead |
| --- | --- | --- |
| **Bit stuffing** | `11111` **0** `111` = 9 bits | +1 bit |
| **Byte stuffing** | ESC + 0xFF = 16 bits | +8 bits (een hele extra byte) |

Bit stuffing is hier **8x efficienter**. Hoe meer flag-achtige patronen in de data voorkomen, hoe groter het voordeel van bit stuffing.

In het algemeen:

- Bit stuffing voegt in het slechtste geval 1 bit toe per 5 databits (20% overhead)
- Byte stuffing voegt in het slechtste geval 1 byte toe per databyte (100% overhead)

### belangrijk voor examen

Bij **bit stuffing** wordt na vijf opeenvolgende 1-bits een `0` ingevoegd. De ontvanger verwijdert die opnieuw. Zo blijft het flagpatroon uniek. Bit stuffing is efficienter dan byte stuffing omdat het op bitniveau werkt in plaats van op byteniveau.

## Overgang naar de physical layer

Na framing verschuift de les naar de **physical layer**.

Hier gaat het niet meer over de logische opbouw van frames, maar over de fysieke dragers van signalen:

- koper
- glasvezel
- radio
- satellietverbindingen

De belangrijkste vraag wordt dan:

welke eigenschappen heeft een medium, en wat betekent dat voor bereik, snelheid, betrouwbaarheid en kost?

## Coaxial cable

**Coaxial cable** is een klassieke koperen transmissiemedia met:

- een massieve koperen kern
- een isolerende laag
- een afschermende buitenmantel

Die afscherming is belangrijk, want ze helpt tegen interferentie van buitenaf.

### voordelen

- goede shielding
- robuust
- historisch belangrijk in oudere netwerken

### nadelen

- minder flexibel
- moeilijker om mee te werken
- minder aantrekkelijk geworden dan twisted pair of fiber in veel moderne netwerken

## Twisted pair

**Twisted pair** is vandaag veel gebruikelijker in klassieke Ethernet-bekabeling.

De basisintuïtie is mooi simpel:

een enkele draad werkt gemakkelijk als een kleine antenne. Hij kan dus:

- storing uitstralen
- storing oppikken

Daarom worden twee draden in elkaar gedraaid.

### waarom helpt twisten?

Door de twists heffen magnetische invloeden elkaar voor een groot stuk op. Dat vermindert de impact van externe storing en beperkt ook de straling naar buiten.

Met andere woorden:

de geometrie van de kabel helpt de signaalkwaliteit beschermen.

## Cat5 en varianten

Een typisch voorbeeld is **Category 5 (Cat5)**-bekabeling.

Zo’n kabel bevat meerdere getwiste paren. Die paren kunnen gebruikt worden voor zenden en ontvangen.

Bij **full duplex** heb je aparte richtingen nodig:

- een pad voor upstream
- een pad voor downstream

Hogere categorieën, zoals Cat6 of Cat7, verbeteren de prestaties door:

- extra shielding
- meer of strakkere twists

### belangrijk voor examen

Twisted pair werkt goed omdat het draaien van de draden storingen helpt onderdrukken.

## Architectuur en bekabeling

De slides tonen ook opnieuw de overgang van één gedeeld medium naar praktischere fysieke opstellingen.

Dat is een belangrijk inzicht:

de fysieke laag en de architectuur van een netwerk hangen samen.

Een netwerk met één gedeelde kabel is fysiek en logisch anders dan een netwerk waar elke host zijn eigen verbinding naar een centrale hub of switch heeft.

De vergelijking met kapotte kerstlichtjes is hier best nuttig: in sommige topologieën kan één probleem grote delen van het systeem beïnvloeden, in andere topologieën blijft de schade veel lokaler.

## Optical fiber

**Optical fiber** werkt helemaal anders dan koper.

In plaats van elektrische signalen gebruik je **licht**.

Een glasvezelsysteem bestaat in essentie uit drie delen:

- een lichtbron
- het transmissiemedium
- een detector

### total internal reflection

De kernidee is dat licht in de vezel gevangen blijft door **total internal reflection**.

Als het licht onder de juiste hoek in het medium beweegt, wordt het telkens terug naar binnen gereflecteerd in plaats van eruit te ontsnappen.

Zo kan het over lange afstanden reizen met weinig verlies.

### voordelen van fiber

- heel hoge bandbreedte
- lange afstanden mogelijk
- weinig elektromagnetische interferentie

### nadeel

De technologie is krachtig, maar vaak duurder of delicater dan eenvoudige koperbekabeling.

### belangrijk voor examen

Fiber gebruikt licht en total internal reflection. Daardoor zijn hoge snelheden en lange afstanden mogelijk met weinig verlies.

## Wireless communication

Daarna verschuift de focus naar draadloze communicatie.

Bij wireless heb je geen vaste kabel, maar dat maakt het systeem niet eenvoudiger. Vaak net omgekeerd.

Het medium is gedeeld, onzichtbaar en veel minder stabiel dan een draad.

## Unlicensed bands

Een deel van draadloze communicatie gebeurt in **unlicensed bands**. Dat zijn frequentiebanden die je mag gebruiken zonder individuele licentie.

Dat klinkt handig, maar het heeft een prijs:

veel verschillende systemen zitten daar samen in dezelfde spectrumruimte.

Dus:

- meer congestie
- meer interferentie
- minder voorspelbaarheid

## Wireless challenges

De slides sommen een paar typische problemen van wireless netjes op. Die zijn belangrijk omdat ze verklaren waarom wireless protocollen anders ontworpen worden dan wired protocollen.

### RF interference

Het spectrum is beperkt en populaire stukken spectrum zijn druk bezet. Je moet dus altijd rekening houden met onverwachte storingen.

### Blocked paths

Niet elk signaal gaat zomaar door alles heen.

Bijvoorbeeld:

- 2.4 GHz-signalen worden sterk beïnvloed door metaal
- water absorbeert signalen
- vegetatie kan signalen verstrooien
- optische signalen worden al snel door een ondoorzichtig object geblokkeerd

### Bandwidth limitations

Niet elk draadloos systeem heeft wifi-snelheden. Low-power systemen zitten vaak veel lager in bandbreedte en gebruiken soms ook kleinere packets.

### Mobility

Nodes kunnen bewegen. Dat betekent dat verbindingen en topologie niet stabiel blijven.

### Power constraints

Een radio kost energie, zowel bij zenden als bij luisteren. Zeker in mobiele en batterijgevoede systemen is dat een centrale beperking.

### belangrijk voor examen

Belangrijke wireless uitdagingen zijn:

- interferentie
- obstructies
- lagere bandbreedte
- mobiliteit
- energiebeperkingen

## Wireless topologies

Draadloze netwerken kunnen in verschillende vormen opgebouwd worden.

De slides noemen onder andere:

- **star**
- **star-of-stars**
- **mesh**

### Star

Hier praten nodes met één centraal punt.

### Star-of-stars

Hier heb je meerdere centrale punten die elk een lokale groep bedienen.

### Mesh

Hier kunnen nodes ook via elkaar verkeer doorgeven.

Dat is vooral nuttig wanneer directe communicatie niet altijd mogelijk is.

## Hidden and exposed terminals

Ook hier komt opnieuw het probleem van **hidden** en **exposed terminals** terug.

Dat blijft fundamenteel in wireless:

- twee zenders kunnen elkaar misschien niet horen
- maar toch storen op dezelfde ontvanger

Of omgekeerd:

- een node denkt dat hij moet zwijgen
- terwijl zijn transmissie eigenlijk niemand zou hinderen

### waarom is dit zo belangrijk?

Omdat het toont dat "ik hoor niets" niet hetzelfde is als "het kanaal is echt vrij".

Dat is een van de redenen waarom wireless MAC zoveel lastiger is dan wired MAC.

## Energie en radio’s

De slides maken expliciet de opmerking dat radio’s energie verbruiken bij:

- zenden
- luisteren

Vaak kost zenden meer, maar continu luisteren is ook duur. Daarom moet je mechanismen ontwerpen waarbij radio’s regelmatig uit kunnen staan zonder dat communicatie helemaal onmogelijk wordt.

Dat idee zagen we eerder al terug bij low power MAC-protocollen.

## Frequentie-afwegingen

De keuze van frequentie is nooit gratis. Er zijn altijd trade-offs.

### Lagere frequenties

Die zijn vaak gunstiger voor langere afstanden, omdat vrije-ruimte-verlies daar meestal minder hard toeslaat.

### Hogere frequenties

Die laten kleinere antennes toe, maar hebben vaker meer last van atmosferische verzwakking en andere praktische beperkingen.

### regelgeving

Daarbovenop komen nog wettelijke beperkingen op zendvermogen en toegelaten banden.

### belangrijk voor examen

Frequentiekeuze is altijd een compromis tussen bereik, antennegrootte, propagatie-eigenschappen en regelgeving.

## Satellietcommunicatie

Satellieten zijn een aparte vorm van draadloze communicatie. Ze zijn interessant omdat ze enorme gebieden kunnen overbruggen, ook waar weinig klassieke infrastructuur is.

Maar hun eigenschappen hangen sterk af van hun **baanhoogte**.

## Geostationary satellites (GEO)

Een **geostationaire** satelliet hangt zo hoog dat hij relatief stil lijkt te staan ten opzichte van de aarde.

Daardoor:

- kan je met vaste schotels werken
- heb je met een klein aantal satellieten grote dekking

Maar de afstand is enorm, en dus ook de vertraging.

De slides noemen een latency van ongeveer **270 ms**.

### voordeel

- groot dekkingsgebied
- stabiele richting

### nadeel

- hoge latency
- krachtige zenders nodig

## Medium Earth Orbit (MEO)

**MEO-satellieten** zitten lager en hebben daardoor minder latency, typisch ergens tussen **35 en 85 ms** volgens de slides.

Ze bewegen wel aan de hemel, dus je hebt meer dynamiek in tracking en infrastructuur.

Ook heb je meer satellieten nodig dan bij GEO om wereldwijde dekking te krijgen.

## Low Earth Orbit (LEO)

**LEO-satellieten** zitten nog lager. Daardoor daalt de latency verder, maar ze bewegen veel sneller door het zicht van een gebruiker.

Eén enkele satelliet blijft dus niet lang bruikbaar voor een vaste grondgebruiker.

De oplossing is:

veel satellieten tegelijk inzetten, zodat zodra de ene weg is, de volgende verschijnt.

Dat is het basisidee achter systemen zoals **Iridium** en moderner ook **Starlink**.

### belangrijk voor examen

Lager orbit betekent typisch:

- lagere latency
- maar meer satellieten nodig
- en complexere opvolging of handover

## Communicatie met de aarde

Satellieten communiceren via radiobeams op specifieke frequenties. Omdat de afstanden groot zijn, moet die communicatie gericht en goed beheerd worden.

Daarbovenop komt vaak nog een intern schakelsysteem of doorstuurarchitectuur tussen satellieten en grondstations.

## Satellite DSL, ADSL en moderne LEO-systemen

De slides tonen een paar praktische toepassingen:

- satellite DSL
- satellite ADSL
- moderne LEO microtransceivers

De grote intuïtie is dat satellietinternet aantrekkelijk is voor plaatsen waar gewone bekabelde toegang moeilijk of duur is.

Een hybride model zoals satellite ADSL probeert kosten te drukken door uplink en downlink anders te organiseren.

Bij moderne LEO-systemen zie je dan opnieuw de trend naar:

- lagere latency
- betaalbaardere terminals
- meer continue dekking

## Starlink

Starlink is een modern voorbeeld van een **LEO-constellatie**.

De interessante stap hier is dat het niet alleen om veel satellieten gaat, maar ook om slimmere onderlinge koppelingen, bijvoorbeeld met **free-space optical communication** tussen satellieten.

Dat maakt het netwerk meer dan zomaar een verzameling losse repeaters.

## RFID

Het laatste deel gaat over **RFID**, wat een beetje tussen wired en wireless systemen in zit qua ontwerpkeuzes.

RFID wil extreem goedkope tags mogelijk maken die:

- draadloos gelezen kunnen worden
- soms ook beschreven kunnen worden
- op korte afstand bruikbaar zijn

Typische toepassingen zijn:

- track and trace
- lokalisatie
- anti-diefstal

## Waarom RFID speciaal is

RFID concurreert niet met klassieke wifi of mobiele netwerken. De vergelijking is eerder met:

- barcodes
- QR-codes

Het doel is dus niet hoge bandbreedte of groot bereik, maar:

- lage kost
- weinig complexiteit
- snelle identificatie

## RFID communication principles

Een RFID-systeem werkt met:

- een krachtige reader met antenne
- een tag

De reader zendt elektromagnetische energie uit. De tag vangt die op en gebruikt ze om actief te worden.

Daarna kan de tag informatie teruggeven.

### belangrijk inzicht

De tag hoeft dus niet altijd een eigen sterke energiebron te hebben.

## Energy transfer

Bij passieve RFID wordt de ontvangen radio-energie gebruikt om de kleine elektronica in de tag van stroom te voorzien.

De wisselspanning uit het elektromagnetische veld wordt als het ware omgezet zodat de tag kort kan functioneren.

Dat werkt typisch alleen op korte afstand, in de **near field**.

## Backscatter communication

Een RFID-tag zendt meestal niet op de klassieke manier een eigen radiosignaal uit.

In plaats daarvan gebruikt hij **backscatter**:

- de reader stuurt een draaggolf
- de tag verandert hoe hij die golf reflecteert of verstoort
- de reader merkt dat verschil op

Dat is een heel slim idee, want het vermijdt dat de tag een volwaardige zender nodig heeft.

### active tags

Er bestaan ook actieve tags met een batterij. Die kunnen veel grotere afstanden halen, maar zijn duurder en minder simpel.

### belangrijk voor examen

Bij **backscatter communication** maakt de tag geen volledig eigen signaal, maar moduleert hij de reflectie van de draaggolf van de reader.

## RFID PHY classes

De slides geven ook een reeks RFID-varianten op verschillende frequenties. Het belangrijke punt daar is niet dat je elk nummer vanbuiten kent, maar dat RFID in meerdere frequentieklassen bestaat, met verschillende trade-offs in bereik, gedrag en toepassing.

## Samenvatting

Dit deel combineert twee niveaus van netwerken:

- hoe je een bitstroom logisch afbakent in **frames**
- via welke **fysieke media** die signalen zich verplaatsen

De belangrijkste ideeën zijn:

- framing is nodig om grenzen tussen berichten te herkennen
- byte count is eenvoudig maar kwetsbaar
- flag bytes vereisen stuffing
- bit stuffing maakt een uniek bitpatroon veilig bruikbaar als framegrens
- twisted pair, coax en fiber hebben elk andere fysische eigenschappen
- wireless communicatie krijgt extra problemen zoals interferentie, obstructie, mobiliteit en energieverbruik
- satellieten tonen hoe baanhoogte rechtstreeks invloed heeft op latency en dekking
- RFID toont hoe extreem low-cost draadloze identificatie werkt via energietransfer en backscatter

## Korte intuïtie

Je kan dit hoofdstuk bijna samenvatten als:

eerst moet je weten **waar een boodschap stopt**, en daarna moet je weten **door welk medium die boodschap reist**.

Dat is precies wat framing en de physical layer samen proberen op te lossen.
