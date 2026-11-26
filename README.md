# KWFlag

Clone 2D e kawaii di [BZFlag](https://www.bzflag.org/): carri armati con la faccina, colpi a stellina, arene generate,
tutti contro tutti o cattura della bandiera. Gira interamente nel browser, senza server di gioco.

**Gioca:** https://fullo.github.io/kwflag/ — per invitare qualcuno mandagli il link della tua arena
(es. `https://fullo.github.io/kwflag/#arena-ktyfog`).

## Come si gioca
- WASD o frecce per guidare, Spazio per sparare, tasti 1–4 per le emote. Su telefono: stick a sinistra, bottone a destra.
- Modalità: **tutti contro tutti** (primo a 10 colpi o 3/5/9 minuti) e **cattura la bandiera** (al meglio di 3/5/11).
- Squadre Arancio (stella) e Blu (cuore): colori scelti per essere distinguibili anche con daltonismo.
- Bolle di potenziamento: tiro rapido, rimbalzo, perforante, scudo, invisibilità, turbo e **Salto 3D**, che porta la tua
  vista in 2.5D e ti fa salire sui blocchi gialli (piattaforme) dalle rampe ai due capi. Teletrasporti a vortice.
- Statistiche personali (partite, vittorie, colpi, bandiere) salvate solo nel tuo browser.
- Arene S, M, L, XL generate da un seme condiviso. Bot di riempimento fino a 4/4/6/8 carri, tre livelli.

## Come funziona la rete
La stessa pagina (`index.html`) funziona in due ambienti:
- **dentro Claude** (artifact): usa la "room" della piattaforma; giocano le persone con accesso e invito;
- **sul web** (GitHub Pages): usa [Trystero](https://github.com/dmotz/trystero) (MIT), incluso in `trystero.js`.
  I browser si trovano tramite relay pubblici Nostr e poi si parlano direttamente (WebRTC).

Non c'è un server autorevole: ognuno pubblica il proprio stato completo, chi viene colpito decide del colpo,
un browser alla volta comanda i bot. Dettagli e storia delle decisioni in [`SPEC.md`](SPEC.md).

Limiti noti: barare è facile (va bene tra amici); su alcune reti aziendali o mobili la connessione diretta tra browser
può non riuscire perché non c'è un server TURN; il punteggio non è salvato tra una sessione e l'altra.

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
