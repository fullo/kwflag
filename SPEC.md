# KWFlag — spec v1

Clone 2D, kawaii, di BZFlag. Pagina HTML singola pubblicata come artifact Claude.

## Gameplay v1
- Arena fissa 1800×1200 con blocchi "caramella" (rettangoli) e bordo.
- Carro armato: avanti/indietro + rotazione (WASD / frecce), sparo (Spazio). Touch: stick virtuale + pulsante.
- Proiettili: stelline, velocità 420 px/s, vita 1,6 s, max 3 attivi, cadenza 0,35 s, si fermano sui muri.
- Colpo = eliminazione. Respawn dopo 3 s in un punto libero, lontano dai nemici; 1,5 s di invulnerabilità.
- Punteggio tutti-contro-tutti: uccisioni e morti. Kill feed.
- Solo / connessione assente: allenamento con bot locali (spariscono quando entra un altro giocatore).
- v1.1: livelli bot Facile (2 bot) / Normale / Difficile (3 bot): reazione, dispersione della mira,
  cadenza, gittata, velocità dei colpi. I bot mirano al bersaglio visibile più vicino, quindi combattono anche tra loro.

## Rete (capability `room`, solo presence)
- Niente server: ogni client pubblica il proprio stato assoluto in `presence` (~15 Hz).
- Si usa solo `presence` perché è l'unico canale che possono usare tutti gli invitati
  (gli eventi `emit` sono riservati a chi ha "Può modificare").
- Presence `v:1`: `n` nome, `h` colore, `x,y,a` posizione e angolo, `vx,vy` velocità,
  `al` vivo, `iv` invulnerabile, `b` proiettili `[id,x,y,vx,vy]`, `kb` mappa `{p<peer>: n}` = chi mi ha colpito quante volte.
- Autorità della vittima: ogni client verifica i proiettili altrui contro il proprio carro e, se colpito,
  incrementa `kb[tiratore]`. Uccisioni di X = somma di `kb[X]` su tutti i peer.
- Limiti noti: niente persistenza (chi ricarica azzera il punteggio), barare è facile, possibili messaggi persi.

## v1.2 — Arene generate
- Generatore con seed (PRNG mulberry32): simmetria a specchio o rotazione di 180°, pezzi (barre, blocchi, pilastri, L)
  con corridoi ≥ 80 px, verifica di connettività (flood fill), 3 colori di pavimento. Seed 0 = arena classica.
- Accordo in rete: ogni giocatore pubblica `m` (seed), `me` (epoca), `md` (modalità). Vince la coppia (epoca, seed) più alta;
  epoca 0 = "non ancora deciso", non vince mai. "Nuova arena" = epoca + 1.

## v2 — Cattura la bandiera (CTF)
- Modalità condivisa come l'arena: tutti contro tutti (`ffa`) o cattura la bandiera (`ctf`). Cambiarla vale per tutti.
- Squadre Arancio (0, simbolo stella) e Blu (1, simbolo cuore). Ognuno sceglie la propria squadra o "Casuale"
  (va nella squadra con meno giocatori). Nel CTF il carro prende il colore della squadra.
- Accessibilità colori: coppia arancio/blu dalla palette Okabe-Ito. Differenza simulata tra le squadre ΔE ≥ 77 con
  protanopia, deuteranopia e tritanopia (prima, rosa/celeste: ΔE 18 con protanopia). Il colore non è mai l'unico segnale:
  stella/cuore su etichette dei nomi, bandiere, basi, minimappa, classifica e punteggio.
- Basi sui lati corti (150, H/2) e (W−150, H/2); il generatore tiene libera la zona delle basi.
- Bandiera: si prende toccandola (solo la bandiera avversaria). Se chi la porta viene colpito, **resta a terra** dove è caduto.
  Un giocatore della squadra proprietaria che la tocca a terra la riporta alla base. Si segna portando la bandiera avversaria
  alla propria base mentre la propria è alla base.
- Fuoco amico: disattivato (scelta di default, da confermare).
- Stato condiviso senza server: ogni giocatore ripubblica la propria vista `fl` = per bandiera `[versione, stato, x, y, portatore]`
  e `sc` = `[versione, punti Rosa, punti Celeste]`. Vince la versione più alta, a parità decide un confronto deterministico.
  Chi causa una transizione (presa, caduta, ritorno, cattura) incrementa la versione. Portatore sparito da > 2 s → bandiera a terra.
- Bot (solo quando non ci sono altri utenti): 1 compagno + avversari (difensore, attaccante). Non prendono le bandiere;
  si muovono con un flow field sulla griglia libera, difendono la base, attaccano, inseguono chi porta la loro bandiera,
  scortano il compagno umano che la porta.

## v3 — Potenziamenti, teletrasporti, partite a punti
- Bolle: 3 coppie simmetriche per arena, una per categoria (coppia 0 attacco, 1 difesa, 2 movimento); il tipo nella
  categoria dipende da (seed, coppia, generazione). Presa → generazione + 1, di nuovo disponibile dopo 15 s.
  Stato condiviso `pu` = generazioni per slot, vince la più alta.
- Attacco: Tiro rapido 8 s (6 colpi in volo, cadenza 0,12 s), Rimbalzo 10 s (3 rimbalzi), Perforante 6 s (attraversa i muri).
- Difesa: Scudo (assorbe un colpo, max 20 s; colpo remoto assorbito ignorato per 2,5 s), Invisibilità 7 s
  (altri vedono al 10%, niente etichetta né minimappa; i bot non puntano oltre 140 px).
- Movimento: Turbo 6 s (velocità ×1,6); coppia di teletrasporti generata con l'arena, cooldown 1,2 s, usata anche dai bot.
- Colpito = perdi tutti i potenziamenti. Presence: `fx` (s/g/t visibili agli altri), bit nei proiettili (1 perforante, 2 rimbalzo).
- Partite CTF al meglio di 3 / 5 / 11 (2 / 3 / 6 punti). Condiviso come arena e modalità (`bo`); a fine partita banner,
  rivincita manuale o automatica dopo 10 s (nuova arena, epoca + 1).

## v3.1 — Dimensioni dell'arena
- S 1800×1200 (area ×1), M 2200×1470 (×1,5), L 2850×1900 (×2,5), XL 4410×2940 (×6). "+50%" interpretato come area.
- Ostacoli, bolle (3 coppie × area) e coppie di teletrasporti (round(area × 0,8), etichettate A, B, …) crescono con l'area.
- Dimensione condivisa come arena/modalità (`sz`); la mappa classica esiste solo in S. Griglia libera rasterizzata per
  connettività e navigazione (generazione ~1 ms anche in XL).

## v3.2 — Bolle e difficoltà
- Con i bot (gioco da solo): coppie di bolle attive = max(1, ceil(coppie × f)), f = 1 / 2/3 / 1/3 per Facile / Normale / Difficile.
  Restano le prime coppie, quindi le categorie si perdono in ordine: prima movimento, poi difesa.
- Con altri giocatori i bot non ci sono e tutte le bolle sono attive, uguali per tutti.

## v4 — Bot di riempimento in rete, fine partita FFA
- Bolle in solitaria: per categoria max(1, round(coppie_categoria × f)); nessuna categoria sparisce (in S non cambia nulla).
- Carri minimi: S 4, M 4, L 6, XL 8; i bot coprono la differenza con le persone in gioco, entrano/escono dinamicamente.
- Un solo host simula i bot: la chiave più bassa tra i giocatori in gioco (chi è sulla schermata iniziale non conta).
  Pubblica `bt` (bot) e `bb` (proiettili dei bot). Chi subentra riprende i bot dall'ultimo `bt` visto (stesse chiavi).
- Colpi: la vittima decide (anche per i colpi dei bot, chiave `b…` in `kb`). Le uccisioni sui bot le registra chi spara
  (`bk`); l'host le vede e uccide il bot. Uccisioni bot↔bot: contatori locali dell'host (`kk`, `dd`).
- Chi spara tiene in rete per 150 ms il proiettile di un colpo previsto, invisibile in locale, così la vittima lo vede.
- CTF: i bot vanno nella squadra con meno carri; ruoli difensore/attaccante alternati; gli attaccanti scortano un compagno
  umano che porta la bandiera.
- FFA: fine partita "primo a 10 colpi" o 3/5/9 minuti (`fe`), ora di inizio condivisa (`ms`); a tempo scaduto vince chi ha
  più colpi, a pari merito pareggio. Classifica azzerata a ogni partita; a partita finita non si spara.
- Telefono: menu della barra in alto nascosti sotto i 700 px (restano nella schermata iniziale).

## v5 — Arene con codice (fase A) e atmosfera (fase C)
- Ogni arena è una stanza con nome `kw-<codice>` (`room.join`), codice di 6 caratteri senza ambigui (no 0/o/1/l/i).
  La stanza principale fa da ingresso: ognuno pubblica `{v:2, a:codice, md, sz, pl}` e l'elenco raggruppa per codice.
- All'avvio: codice dall'indirizzo (`#arena-xxxxxx`), altrimenti l'arena più popolata dopo 1,5 s, altrimenti una nuova.
  Il codice va nell'hash, quindi il link di Share porta nell'arena. "Copia codice", "Entra" con codice, "Nuova".
- Se le stanze con nome non sono disponibili: tutti nella stessa partita, senza codice (avviso).
- "Nuova arena" della barra rinominato "Nuova mappa": la mappa cambia, l'arena (codice) resta.
- Audio sintetizzato con WebAudio (nessun file): colpi, esplosioni, bolle, scudo, teletrasporto, bandiera, punto,
  vittoria, morte, rimbalzo. Volume in base alla distanza dalla telecamera. Musica generativa pentatonica su I–vi–IV–V,
  spenta di default. Interruttori Suoni/Musica salvati nel browser.
- Emote: tasti 1–4 (o pulsanti su telefono) "ciao!", "gg", "aiuto!", "evvai!"; fumetto per 2,5 s, sincronizzato con
  `em:[indice, sequenza]` in presence; non mostrato per chi è invisibile.

## v6 — Bot più bravi (fase B), versione web (fase D)
- Bot e bandiera (solo l'host decide): gli attaccanti prendono la bandiera avversaria (a casa o a terra) e la portano
  alla propria base; i difensori riportano la propria se è a terra; gli attaccanti scortano un compagno (umano o bot) che la porta.
  Bot colpito con la bandiera → bandiera a terra. Chiavi `b…` ammesse come portatori.
- Bot e bolle: deviano verso una bolla entro 140 / 320 / 520 px (Facile / Normale / Difficile) e usano tutti gli effetti.
  Effetti dei bot pubblicati in `bt` (13° campo, s/g/t); proiettili dei bot con bit perforante/rimbalzo.
  Scudo di un bot colpito da un non-host: niente uccisione, il tiratore incrementa `bs[bot]`, l'host consuma lo scudo.
- "Casuale" si ricalcola 1,5 s dopo l'ingresso nell'arena, quando si sa chi c'è.
- Messaggio esplicito se la finestra non è ammessa alla rete di Claude (link pubblico, senza accesso, account non invitato).
- Versione web: stessa pagina. Fuori da Claude usa Trystero 0.27 (pacchetto locale `trystero.js`, strategia Nostr),
  `appId: github.com/fullo/kwflag`, identità per scheda. Ogni arena = stanza Trystero `kw-<codice>`; lo stato per utente
  (`makeUserState('p')`) fa da presence. Nessun elenco delle arene: si entra dal link `#arena-codice` ("Copia link").

## v7 — "Salto 3D": vista 2.5D e piattaforme
- Il gioco base resta 2D. Nuova bolla di movimento "Salto 3D" (15 s): la propria vista passa a 2.5D e si può salire
  sulle piattaforme. La vista è personale (non condivisa); la quota sì.
- Vista 2.5D: camera prospettica dietro il giocatore, rivolta a nord; inclinazione animata da 90° (dall'alto) a 52°
  in ~0,55 s. Pavimento, basi e vortici a terra; blocchi estrusi (piattaforme 36, muri 56); carri con scafo schiacciato e
  cupola/faccina/nome dritti come i personaggi dei JRPG; bolle, colpi, bandiere ed effetti come sagome verticali.
- Piattaforme = blocchi gialli (stile 2), con strisce a gradino anche in 2D. Con l'effetto attivo non bloccano: ci si sale
  guidandoci contro (`z = 36`) e si scende uscendo dal bordo; gli altri muri bloccano anche dall'alto. A fine effetto si
  scivola giù. Morte = effetto perso.
- Colpi e quota: i colpi partono alla quota di chi spara (bit 4 nei proiettili); da terra non si colpisce chi sta in alto
  (tranne col perforante); dall'alto si colpisce chi sta in basso e si passa sopra le piattaforme. Presence `el:1` quando in alto;
  in 2D chi è in alto appare più grande con l'ombra spostata. I bot non prendono questa bolla e non puntano chi è in alto.

## v7.1 — Rampe e statistiche personali
- Ogni piattaforma ha una rampa a ciascun capo del lato lungo, dentro il suo ingombro: lunghezza `min(48, lato lungo / 3)`.
  Si sale solo dalle rampe; i lati lunghi bloccano come muri. La quota segue la rampa (`z = 36 · distanza dal capo / rampa`);
  dal lato lungo si può solo cadere giù.
- "In alto" vale da metà rampa in su (`z ≥ 18`): soglia unica per presence `el`, bit 4 dei colpi, bersagli dei bot e
  collisioni tra carri. I colpi partono a 0 o a 36. Un colpo a terra è fermato dalla piattaforma solo dove la superficie
  supera la sua quota di 10: dal basso di una rampa si spara ancora lungo la rampa.
- Rampe visibili in entrambe le viste: in 2.5D sono piani inclinati chiari con gradini e fianchi a trapezio; in 2D i capi
  sono schiariti con tre linee a gradino.
- Statistiche personali in `localStorage` (`kw.stats`, solo su questo browser): partite, vinte (il pareggio non conta),
  colpi a segno, colpi subiti, bandiere catturate. Registrate una volta a fine partita, mostrate nella schermata iniziale
  con "Azzera".

## v8 — Vista nella nebbia, radar, schermata iniziale compatta
- Vista come impostazione dell'arena (`vw`: `2d` | `fog`), scelta con la partita e uguale per tutti: viaggia nella tupla
  dell'arena insieme a modalità, dimensione e fine partita; cambiarla apre una nuova epoca. Le arene aperte la mostrano.
- Vista nella nebbia: sempre 2.5D (anche da morti), telecamera alle spalle del carro che ruota con lui (inseguimento morbido),
  punto di mira 140 avanti. Sui blocchi gialli si sale sempre dalle rampe. La bolla "Salto 3D" diventa "Faro": raggio di
  vista da 430 a 720 per la sua durata.
- Nebbia: oltre il raggio di vista il terreno si scurisce a gradini; carri, colpi e fantasmi nemici fuori raggio non si
  disegnano (in CTF i compagni sì). La minimappa mostra solo ciò che vedi. È nebbia grafica, non difesa contro chi bara:
  ogni browser riceve comunque tutte le posizioni.
- Più adrenalina: nella nebbia carri e colpi vanno a 1,35× (rotazione 1,1×); la gittata resta uguale (vita del colpo / 1,35).
  I bot non mirano oltre il raggio di vista.
- Rendering 2.5D generalizzato: rotazione del mondo attorno al punto guardato, facce dei blocchi scelte dalla posizione reale
  della telecamera, poligoni ritagliati sul piano vicino, ordinamento per profondità lungo la direzione di vista.
- Minimappa e radar (posizioni riviste in v9). Radar: raggio 950, solo carri
  e bandiere, muri come sagome; orientato col carro nella nebbia, a nord in 2D. Maiusc+M e Maiusc+R li nascondono; la
  scelta resta nel browser.
- Schermata iniziale: le impostazioni della partita sono menu a tendina in griglia (gli stessi del menu in alto),
  audio solo in alto, istruzioni in "Come si gioca" richiudibile, statistiche su una riga. Da 1146 a 716 px su desktop.

## v9 — Bandierine stile BZFlag, bot che salgono
- Le bolle diventano bandierine (asta + stendardo nel colore della categoria con la sua icona). Se ne porta **una alla volta**:
  passando su un'altra con una in mano non succede nulla ("una alla volta!"). Una buona dura finché la lasci (Q, o "lascia"
  su telefono) o al massimo 30 s; lo scudo si consuma al primo colpo. Morte = bandierina persa.
- Un terzo delle bandierine (scelta deterministica da seme, slot e generazione) è «?»: alla raccolta è buona (a caso tra tutte)
  o cattiva al 50%. Cattive, 10 s e non si staccano: Comandi invertiti (sterzo al contrario), Lumaca (velocità 0,55),
  Grilletto facile (spara da solo), Radar in tilt (niente minimappa né radar), Occhiali appannati (vista 200 nella nebbia,
  alone scuro attorno al carro in 2D).
- Una bandierina torna sempre al suo posto: lo slot è occupato finché qualcuno la porta (presence `hs`; il bot-host pubblica
  `bh` per i suoi bot) e torna disponibile 6 s dopo la raccolta o il rilascio. I bot ne portano una alla volta, con le durate
  di prima, e non prendono le «?» né il Salto 3D/Faro.
- Bot e piattaforme (solo nella nebbia): se un nemico sta in alto, il bot va al capo di rampa più vicino lungo la griglia di
  navigazione e poi sale dritto lungo l'asse lungo; in alto mira anche verso il basso. Altezza dei bot condivisa (`bt[13]`,
  bit 4 nei colpi `bb`); le regole sui colpi tra quote valgono anche per i bot. Se un bot resta incastrato 0,4 s, si divincola
  per 0,6 s prima di riprendere il percorso.
- Minimappa in alto a sinistra, radar in basso a destra (su touch più in alto e a sinistra dei pulsanti).

## v10 — Salto e bandierine BZFlag (fase 1)
- **Salto** (Tab, "salta" su telefono): velocità verticale 300, gravità 900, picco ~52 (sopra le piattaforme, sotto i muri 56).
  In aria da quota 34 in su le piattaforme non bloccano: si atterra sopra e si scende dalle rampe, anche in 2D. Chi è in aria
  sopra quota 18 conta "in alto" (non lo colpisce chi spara da terra). Presence `z` per disegnare il salto degli altri.
- Nuove bandierine: **Laser** (raggio istantaneo fino al primo muro, 1400 max, ricarica 1,1 s; bit 8, x,y → vx,vy),
  **Missile guidato** (0,75× velocità, curva fino a 2,6 rad/s verso il nemico più vicino entro 650 e ±70°; bit 16),
  **Onda d'urto** (anello che si allarga a 420/s per 0,55 s, colpisce attraverso i muri; bit 32, vx = raggio),
  **Stealth** (invisibile su radar e minimappa, visibile a schermo; l'Invisibilità ora è il contrario: si vede sul radar),
  **Identifica** (mostra cosa c'è nelle «?»). I bot non le prendono.
- Le «?» ora sono deterministiche (seme, posto, generazione): tutti e Identifica vedono la stessa cosa.
- Colpi speciali: geometria comune `shotHits` (segmento per il laser, anello per l'onda, punto per gli altri); laser e onda
  attraversano più bersagli e chi ne è colpito decide come sempre.

## v11 — Tre nuove modalità (fase 2)
- `md` ora: `ffa` | `ctf` | `koth` | `rabbit` | `waves`. `foe(a,b)`: tutti nemici in ffa/koth, altrimenti per squadra.
- **Re della collina**: una piattaforma scelta dal seme (o il centro), cerchio di 130. Guadagna tempo solo chi ci sta da solo;
  in due o più è contesa. Ognuno conta il proprio (`kh` in presence, decimi di secondo), l'host conta i bot (`bt[14]`).
  Fine: primo a 60 s, o più secondi a tempo scaduto. I bot pattugliano la collina.
- **Caccia al coniglio**: stato `rb:[v,k]` in presence, vince la versione più alta. Il coniglio è squadra 1, gli altri 0
  (i cacciatori non si colpiscono tra loro). Chi colpisce il coniglio lo diventa (`onKill` su ogni uccisione, la decide la
  vittima come sempre). Se il coniglio sparisce, l'host ne sceglie uno a caso. Punti = colpi; fine come tutti contro tutti.
  Il coniglio ha le orecchie e 🐰 in classifica.
- **Ondate di bot**: umani squadra 0 contro bot squadra 1 governati dall'host (niente bot di riempimento, niente rinascita).
  Ondata n: min(12, 1 + n + giocatori) bot, livello min(2, (n−1)/2). Nuova ondata 3,5 s dopo l'ultima eliminazione.
  Vite in comune: max(6, 3 × giocatori); i colpi subiti dagli umani le consumano. Fine a vite esaurite. L'host pubblica `wv`.
- Fascia in alto con lo stato della modalità; la colonna K della classifica diventa S (secondi) o P (punti).

## Prossime iterazioni (candidati)
- Power-up (scudo, tiro rapido, rimbalzo, invisibilità), squadre e cattura della bandiera,
  stanze multiple (`room.join`), classifica persistente (`db`), suoni.
