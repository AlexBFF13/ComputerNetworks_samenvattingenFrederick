# Transportlaag (Laag 4) — Korte samenvatting

De transportlaag zorgt voor **end-to-end delivery** tussen processen op verschillende hosts: data wordt verpakt in **segmenten** en is de grens tussen de applicatiewereld en het netwerk. Ze probeert tegelijk efficiënt, betrouwbaar en kostenbeheersbaar te zijn.

## Connectionless vs connection-oriented

- **Connectionless**: snel, simpel, weinig overhead, maar beperkte error control en geen flow control (veel verantwoordelijkheid bij de applicatie).
- **Connection-oriented**: meer overhead, maar veel ingebouwde ondersteuning, met drie fasen: **establishment → data transfer → release**.

## Addressing: TSAP vs NSAP

- **NSAP** = netwerklaag-eindpunt (IP-adres), **TSAP** = transportlaag-eindpunt (poort). Eén IP-adres kan veel TSAPs/verbindingen hebben (analogie: één telefoonnummer, meerdere extensies).
- **Berkeley sockets**: server doet `SOCKET → BIND → LISTEN → ACCEPT → SEND/RECEIVE → CLOSE`; client doet `SOCKET → CONNECT → SEND/RECEIVE → CLOSE`.
- **ICP (Initial Connection Protocol)**: een process server luistert op een well-known TSAP en start de echte server pas op bij een binnenkomende verbinding (bv. `inetd`) — bespaart resources.

## Connection management: fundamentele problemen

- Packets kunnen **out-of-order**, vertraagd of **gedupliceerd** aankomen — een oud, vertraagd packet kan later opnieuw opduiken (klassiek voorbeeld: dubbel uitgevoerde banktransfer).
- **Bounded packet lifetime**: sequence numbers worden niet te snel herbruikt + real-time clock na een crash, zodat oude sequence numbers niet per ongeluk hergebruikt worden.
- **Two generals problem**: zelfs met ACK-uitwisseling kan je nooit met absolute zekerheid weten dat beide kanten dezelfde kennis van de verbindingstoestand hebben — **er is geen perfecte oplossing**.

## Error control

- **End-to-end checksum** is nodig zelfs als lagere lagen al checksums doen, omdat fouten ook **binnen een defecte router** kunnen ontstaan (hop-by-hop is niet genoeg).
- Basislogica: zender stuurt segment → ontvanger ACK't → geen ACK binnen timeout → retransmissie. Zender moet ongeACKte data bewaren.

## Flow control: sliding window protocollen

Flow control = hoeveel data mag "in flight" zijn zodat een **trage ontvanger** niet overspoeld wordt (vs. congestion control = bescherming van het **netwerk**).

| Protocol | Venstergrootte | Out-of-order | Hertransmissie bij verlies |
|---|---|---|---|
| **Stop-and-wait** | 1 | n.v.t. | enkel dat segment |
| **Go-Back-N** | zender: W, ontvanger: 1 | verworpen | segment k **én alle volgende** in venster |
| **Selective Repeat** | zender: W, ontvanger: W | gebufferd | enkel het ontbrekende/beschadigde segment |

- **Stop-and-wait**: data/ACK alterneren tussen 0 en 1, zender start timer; **piggybacking** = ACK meesturen met eigen data. Inefficiënt bij hoge RTT (a hoog).
- **Go-Back-N**: cumulatieve ACKs (ACK voor n impliceert alles ≤ n is goed). Bij timeout: alles vanaf het verloren segment opnieuw → kan veel **onnodige hertransmissie** geven.
- **Selective Repeat**: efficiënter, maar vereist buffers aan **beide kanten** en NACKs. **Belangrijke regel**: venstergrootte ≤ **helft van de sequence number space**, anders verwart de ontvanger duplicaten met nieuwe segmenten.

### Efficiëntieformules (sliding window)

- **a = (RTT/2) / T_trans** (propagatievertraging t.o.v. transmissietijd).
- Stop-and-wait: **η = 1 / (1 + 2a)** — bij satelliet (RTT=2600ms, a≈956) krijg je η ≈ 0,05%, dus quasi onbruikbaar.
- Go-Back-N / Selective Repeat: voor η ≈ 100% heb je **W ≥ 1 + 2a** nodig (bij hetzelfde voorbeeld: W ≥ ~1913 packets). Anders η ≈ W / (1 + 2a).
- Stop-and-wait is enkel goed wanneer **a ≈ 0**, d.w.z. bandwidth-delay product < 1 packet (trage links over korte afstand, LoRa, RS-232).

## Congestion control

- Doelen: bandbreedte benutten, congestie vermijden, fair zijn, snel reageren.
- **Goodput** (nuttige, aangekomen data) ≠ **throughput** — bij overbelasting kunnen verliezen/hertransmissies de goodput laten dalen → **congestion collapse**.
- **Kleinrock's power = load / delay**: meer load is niet altijd beter als delay disproportioneel stijgt.
- **Min-max fairness**: je kan flow A niet meer bandbreedte geven zonder flow B te benadelen.
- **AIMD** (Additive Increase, Multiplicative Decrease): voorzichtig lineair verhogen, agressief multiplicatief verlagen bij congestie — makkelijk om netwerk in congestie te duwen, moeilijker om eruit te raken.

## UDP

UDP (RFC 768) is **connectionless** en lichtgewicht:

- **Biedt niet**: flow control, ordering, congestion control.
- **Biedt wel**: ports (TSAP-adressering) en checksum (berekend over UDP-header + IP-**pseudo-header**).
- Nuttig voor: lage overhead, geen connection setup, **anycast/broadcast/multicast** (DNS, DHCP).

## TCP: betrouwbare byte-stream

TCP biedt een **betrouwbare, end-to-end byte-stream** (geen message service — berichtgrenzen gaan verloren). Verbinding = **full duplex**, tussen exact twee sockets, geïdentificeerd door (src IP, src port, dst IP, dst port, protocol).

- **Sequence number** = volgnummer van de **eerste byte** in het segment; **ACK number** = volgende verwachte byte (cumulatief — ACK=2048 betekent "alles tot en met byte 2047 ontvangen").
- Belangrijke flags: **SYN/FIN** (op-/afbouw), **ACK**, **RST** (abrupt resetten), **PSH** (snel doorgeven), **URG**, **ECE/CWR** (ECN).
- **Window size**-veld = kern van flow control: hoeveel bytes de ontvanger nog kan accepteren.

### 3-way handshake

```
client → SYN, SEQ=x          (SYN=1, ACK=0)
server → SYN, ACK, SEQ=y, ACK=x+1   (SYN=1, ACK=1)
client → ACK, ACK=y+1        (SYN=0, ACK=1)
```

Drie stappen nodig zodat beide kanten weten dat de ander bereikbaar is, sequence numbers kent, en beide richtingen klaar zijn.

### Afsluiten

Twee simplex-richtingen worden apart gesloten via **FIN**. Daarna **TIME_WAIT** (≈ 2× max packet lifetime) zodat oude segmenten kunnen "uitsterven" — voorkomt dat oude packets in een nieuwe verbinding met dezelfde socket-combinatie terechtkomen. TCP is een **state machine met 11 toestanden** (CLOSED, LISTEN, SYN_SENT, ESTABLISHED, TIME_WAIT, ...).

### Sliding window & flow control

- ACK en buffer-allocatie zijn losgekoppeld: `ACK=4096, WIN=0` betekent "alles tot 4095 ontvangen, maar geen buffer vrij" → zender mag niets nieuws sturen.
- **Window probes**: als advertised window 0 is en de update-melding verloren gaat, voorkomt een window probe een **deadlock**.

### Tinygram syndrome / Nagle / Silly Window Syndrome

- **Delayed ACKs**: ontvanger wacht (tot ~200ms) om ACK te piggybacken.
- **Nagle's algorithm**: zender buffert kleine data zolang er een onbevestigd segment onderweg is — slecht voor interactieve apps (games, SSH); uit te schakelen via **TCP_NODELAY**.
- **Silly Window Syndrome / Clarke's algorithm**: ontvanger wacht met window updates tot er minstens een MSS past of buffer halfleeg is. Nagle (zender) en Clarke (ontvanger) zijn complementair.
- **Interactie**: Nagle + delayed ACK kan systematisch ~200ms latency geven bij kleine interacties (geen echte deadlock, wel structurele vertraging). Oplossing: `TCP_NODELAY` en/of `TCP_QUICKACK`.

### Timers

| Timer | Doel |
|---|---|
| **RTO** (retransmission timeout) | wanneer opnieuw versturen bij geen ACK; gebaseerd op **SRTT** + **RTTVAR** |
| **Persistence timer** | stuurt window probe bij WIN=0 |
| **Keep-alive timer** | check of peer nog leeft (optioneel) |
| **TIME_WAIT timer** | laat oude segmenten uitsterven na sluiten |

### Congestion control in TCP

- Effectieve zendlimiet = **min(receiver window, congestion window)** — receiver window beschermt tegen trage ontvanger, congestion window tegen vol netwerk.
- TCP gebruikt **packet loss** als impliciet congestiesignaal (werkt goed op bekabelde netwerken, minder op draadloze).
- **ACK clock**: nieuwe data wordt verstuurd op het tempo waarop ACKs terugkomen, en dat tempo wordt bepaald door de **bottleneck** — zelfregulerend, voorkomt bursts.
- **Slow start**: congestion window groeit **exponentieel** (verdubbelt per RTT) tot **slow-start threshold**, daarna **additive increase** (AIMD). Na congestie wordt threshold ≈ helft van de huidige congestion window.
- **Tahoe**: viel hard terug bij verlies. **Reno**: voegde **fast recovery** toe via duplicate ACKs — hertransmit snel, val terug tot rond de nieuwe threshold, ga door met additive increase (werkt vooral goed bij **één** verlies).
- **SACK**: ontvanger meldt welke byte-ranges al ontvangen zijn, zodat zender gericht kan hertransmitteren bij **meerdere verliezen** — via header options, dus compatibel.
- **ECN**: routers markeren packets vóór een drop (i.p.v. packet loss als signaal); TCP reageert via **ECE/CWR** flags. Minder wijdverspreid dan SACK, want vereist support van beide eindhosts én netwerk.

## RTP (kort, ter context)

RTP (RFC 3550) draait meestal bovenop UDP en richt zich op **playback**, niet op perfecte aflevering: **te laat is vaak erger dan verloren**. Sequence number (16-bit, per packet) detecteert verlies/volgorde; **timestamp** (32-bit) zegt wanneer iets afgespeeld moet worden. **Jitter** (variatie in delay) wordt opgevangen met een **playback buffer** — trade-off tussen lage delay en weinig verlies door late aankomst. RTCP geeft feedback (delay, jitter, bandbreedte, sync).

## QUIC (kort)

QUIC (RFC 9000, ~30% van internetverkeer) is **frame-based**, draait in **userspace bovenop UDP**, met ingebouwde security (TLS 1.3-integratie, soms **0-RTT**), **stream multiplexing** (lost **head-of-line blocking** op connectieniveau op) en **connection migration** via **Connection IDs** (CID) — IP/poort mogen veranderen zonder de verbinding te verbreken (gevalideerd via PATH_CHALLENGE/PATH_RESPONSE; congestion state wordt gereset bij migratie). Sequence numbers zijn 62-bit (wraparound is geen issue meer). Congestion control gebaseerd op **Cubic (RFC 8312)**.

---

## Veelgemaakte examenvalkuilen / belangrijk om te onthouden

- **Flow control ≠ congestion control**: flow control beschermt de ontvanger (window/buffer), congestion control beschermt het netwerk (congestion window, AIMD).
- Bij **Go-Back-N** heeft de ontvanger venster 1 en wordt bij timeout **alles vanaf het verloren segment opnieuw** gestuurd — bij **Selective Repeat** enkel het ontbrekende deel, maar met venstergroottebeperking ≤ helft van de sequence space.
- Reken altijd met **a = (RTT/2)/T_trans**: stop-and-wait is rampzalig bij hoge a (bv. satelliet), en Go-Back-N/SR hebben dan W ≥ 1+2a nodig voor 100% efficiëntie.
- **Two generals problem**: geen perfecte oplossing mogelijk — absolute wederzijdse zekerheid over verbindingstoestand is niet afdwingbaar.
- **End-to-end checksums** blijven nodig ondanks lagere-laag checksums, want fouten kunnen binnen routers ontstaan.
- Ken de **3-way handshake** met exacte SYN/ACK-waarden, en het verschil tussen RTO/persistence timer/TIME_WAIT.
- RTP/UDP heeft géén ACK-clock en geen congestion control → kan TCP-flows "uithongeren" (unfairness) en draagt bij aan congestion collapse bij hoge belasting.
- QUIC is geen "TCP 2.0": het is een nieuw protocol met eigen connection-identificatie (CID), framing en geïntegreerde security — niet zomaar een uitbreiding.
