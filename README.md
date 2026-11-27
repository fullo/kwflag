# KWFlag

Clone 2D e kawaii di [BZFlag](https://www.bzflag.org/): carri armati con la faccina, colpi a stellina, arene generate,
tutti contro tutti o cattura della bandiera. Gira interamente nel browser, senza server di gioco.

**Gioca:** https://fullo.github.io/kwflag/ — per invitare qualcuno mandagli il link della tua arena
(es. `https://fullo.github.io/kwflag/#arena-ktyfog`).

## Come si gioca
- WASD o frecce per guidare, Spazio per sparare, Tab per saltare, Q per lasciare la bandierina, tasti 1–4 per le emote. Su telefono: stick a sinistra, bottone a destra.
- Modalità: **tutti contro tutti** (primo a 10 colpi o 3/5/9 minuti), **cattura la bandiera** (al meglio di 3/5/11), **quattro squadre** (★ ♥ ◆ ●, prima a 20 colpi),
  **re della collina** (60 secondi da solo nel cerchio), **caccia al coniglio** (chi colpisce il coniglio lo diventa) e
  **ondate di bot** (tutti insieme, vite in comune).
- Squadre Arancio (stella) e Blu (cuore): colori scelti per essere distinguibili anche con daltonismo.
- Bandierine stile BZFlag, una alla volta: tiro rapido, rimbalzo, perforante, laser, missile guidato, onda d'urto, scudo,
  invisibilità, stealth, identifica, turbo e **Salto 3D**, che porta la tua
  vista in 2.5D e ti fa salire sui blocchi gialli (piattaforme) dalle rampe ai due capi. Teletrasporti a vortice.
- Statistiche personali (partite, vittorie, colpi, bandiere) salvate solo nel tuo browser.
- Vista dell'arena, scelta a inizio partita e uguale per tutti: **2D dall'alto** oppure **2.5D nella nebbia**, con la
  telecamera alle spalle del carro, vista ridotta attorno a te, carri e colpi più veloci e la bolla «Faro» che allarga la vista.
- Una bandierina buona dura finché la lasci (Q) o al massimo 30 secondi. Quelle «?» possono essere cattive (comandi invertiti,
  lumaca, grilletto facile, radar in tilt, occhiali appannati) e restano attaccate 10 secondi. Ogni bandierina torna al suo posto.
- Prova guidata di un minuto, modalità spettatore, controller, tasti riassegnabili e basso consumo (dalla schermata iniziale).
- Minimappa in alto a sinistra, radar in basso a destra: Maiusc+M e Maiusc+R li mostrano o nascondono.
- Arene S, M, L, XL generate da un seme condiviso. Bot di riempimento fino a 4/4/6/8 carri, tre livelli.

## Come funziona la rete
La stessa pagina (`index.html`) funziona in due ambienti:
- **dentro Claude** (artifact): usa la "room" della piattaforma; giocano le persone con accesso e invito;
- **sul web** (GitHub Pages): usa [Trystero](https://github.com/dmotz/trystero) (MIT), incluso in `trystero.js`.
  I browser si trovano tramite relay pubblici Nostr e poi si parlano direttamente (WebRTC).

Non c'è un server autorevole: ognuno pubblica il proprio stato completo, chi viene colpito decide del colpo,
un browser alla volta comanda i bot. Dettagli e storia delle decisioni in [`SPEC.md`](SPEC.md).

Limiti noti: barare è facile (va bene tra amici), e la nebbia nasconde solo a video: ogni browser riceve tutte le
posizioni; su alcune reti aziendali o mobili la connessione diretta tra browser può non riuscire perché non c'è un server
TURN; le statistiche restano nel singolo browser; i bot salgono sulle piattaforme solo nella nebbia; su telefono minimappa e
radar non si possono nascondere.

## Contribuire
Il gioco è un solo file HTML senza build. Apri `index.html` con un server statico locale
(es. `python3 -m http.server`) e proponi le modifiche con una pull request. Prima di cambiare il protocollo di rete
leggi `SPEC.md`: i campi della presence sono condivisi con la versione dentro Claude.

## Licenza
KWFlag è software libero, distribuito con la **GNU General Public License versione 2** (`GPL-2.0-only`):
puoi usarlo, studiarlo, modificarlo e ridistribuirlo, anche modificato, purché con la stessa licenza e con i sorgenti.
Testo completo in [`LICENSE`](LICENSE).

`trystero.js` è un pacchetto di [Trystero](https://github.com/dmotz/trystero) 0.27.0, © Dan Motzenbecker,
distribuito con licenza MIT (compatibile con la GPL); l'avviso di licenza è in testa al file.
