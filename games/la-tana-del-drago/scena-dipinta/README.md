# Il Ponte Levatoio — una scena completa, struttura laserdisc

Prova di un quadro singolo costruito **come erano costruiti i quadri di Dragon's
Lair**: non un gioco disegnato a runtime, ma una sequenza di segmenti filmati
con un solo istante di decisione e due diramazioni.

Apri `index.html` nel browser. Funziona anche senza le tavole dipinte: al loro
posto compare un segnaposto che dice quale file manca.

## Come era fatto Dragon's Lair (1983)

Ricerca fatta per modellare questa scena:

- **Nessun controllo diretto del personaggio.** Il giocatore non muove Dirk: ne
  comanda i riflessi. A ogni bivio sceglie una direzione o preme il tasto spada,
  e il risultato è un diverso segmento video già animato.
- **Il lampeggio è il suggerimento.** Il gioco indica la mossa facendo
  *lampeggiare* l'oggetto verso cui muoversi — una porta, una corda, una luce.
  In qualche stanza l'oggetto luminoso è una trappola.
- **Tempismo e memoria, non destrezza.** Una partita completa richiede oltre
  **200 mosse corrette**; la finestra di reazione è brevissima e sapere la
  direzione non basta, bisogna muoversi nell'istante giusto.
- **Ordine dei quadri variabile.** Alcune scene arrivano in ordine casuale nelle
  partite successive, così da non rendere il gioco una filastrocca imparata a
  memoria (e da tenere alto l'incasso per il gestore della sala).
- **Scene riusate e specchiate.** Alcuni quadri tornano più di una volta, a volte
  ribaltati orizzontalmente — e allora la mossa corretta si inverte (sinistra
  invece di destra).
- **22 minuti di animazione, 50.000 fotogrammi**, scene di morte comprese, per
  un budget di circa 3 milioni di dollari e sette mesi di lavoro dello studio di
  Don Bluth.
- **Il ponte levatoio non si giocava.** La scena esiste sul laserdisc come primo
  capitolo dopo l'attract mode, ma nessuna release americana da fabbrica la
  usava: la partita cominciava a sorte dalle corde in fiamme o dal muro di
  mattoni. L'apertura vera e propria era la sequenza non interattiva — castello
  in lontananza, Dirk che attraversa il ponte mentre crolla, tentacoli
  nell'acqua, poi dentro il castello con i cancelli che si chiudono.

Questa demo prende proprio quella sequenza d'apertura e la rende giocabile: il
quadro che il 1983 ci ha mostrato soltanto.

## Anatomia del quadro

Quattro segmenti, esattamente come i capitoli di un laserdisc:

| Segmento | Cosa fa | Durata |
|---|---|---|
| `approccio` | Dirk corre sul ponte, la macchina da presa stringe | 3,2 s |
| `pericolo` | Le tavole cedono, il tentacolo esce: **si apre la finestra** | 0,85 s |
| `riuscita` | Il salto verso il portale | 2,4 s |
| `morte` | Il fossato se lo prende | 2,8 s |

Durante `pericolo` lampeggia il **bagliore** sul portale (coordinate `richiamo`,
normalizzate 0–1 sul fotogramma): è il suggerimento originale. Il pannello con
la freccia e la barra del tempo è un'aggiunta moderna e si spegne dal titolo con
**Aiuti a schermo: no (1983)** — così resta solo il lampeggio, come in sala
giochi.

Mossa sbagliata o tempo scaduto: una vita in meno e il quadro riparte da capo,
come faceva l'originale. Tre vite esaurite, partita chiusa.

## Mettere le tavole dipinte

Salva quattro immagini in `assets/` con questi nomi esatti:

```
assets/01-approccio.png     il castello, il ponte, Dirk che corre
assets/02-pericolo.png      le tavole che cedono e il tentacolo
assets/03-riuscita.png      il salto verso il portale
assets/04-morte.png         la caduta nel fossato
```

Formato consigliato: **16:9, almeno 2560×1440**, PNG o JPEG. Il motore ritaglia
in *cover* e applica un lento movimento di macchina (zoom e carrello) definito
in `camera`, quindi un po' di margine attorno al soggetto aiuta.

## Passare alle clip video

Lo stesso motore riproduce video al posto delle immagini: basta cambiare
l'estensione in `scena.json`, il resto non cambia.

```json
"pericolo": { "file": "assets/02-pericolo.mp4", "finestra": 0.85 }
```

Per preparare le clip conviene tagliare **sui keyframe**, altrimenti il salto da
un segmento all'altro singhiozza:

```bash
# un keyframe ogni mezzo secondo, niente B-frame: salti istantanei
ffmpeg -i sorgente.mp4 -c:v libx264 -crf 18 -preset slow \
       -g 15 -keyint_min 15 -sc_threshold 0 -bf 0 \
       -pix_fmt yuv420p -movflags +faststart segmento.mp4

# taglio preciso di un segmento (dal secondo 12,40 per 2,8 s)
ffmpeg -ss 12.40 -i segmento.mp4 -t 2.8 -c copy pericolo.mp4
```

## Il copione

`scena.json` è il copione completo del quadro. Viene letto **solo se la pagina è
servita via http** (aprendo il file con doppio clic il browser blocca la lettura
dei file locali): in quel caso vince su quello incorporato in `index.html`.

```bash
cd games/la-tana-del-drago/scena-dipinta && python3 -m http.server 8080
# poi apri http://localhost:8080
```

Campi utili: `mossa` (`SU` `GIU` `SX` `DX` `SPADA`), `etichetta`, `richiamo`
(dove lampeggia), `segmenti.*.camera` (`da` e `a`, ciascuno `[centroX, centroY,
zoom]`), `segmenti.pericolo.finestra` (secondi di reazione).

## Note

- File unico, nessuna dipendenza, nessun processo di build: resta fuori dai
  workspace npm del repository.
- `window.__scena()` espone lo stato in sola lettura (segmento, fase, vite,
  punti, quali tavole sono caricate) per i test headless.
- Le immagini non sono nel repository: `assets/` è vuota di proposito.

### Fonti

- [Dragon's Lair (1983 video game) — Wikipedia](https://en.wikipedia.org/wiki/Dragon's_Lair_(1983_video_game))
- [Dragon's Lair Scene Sequencing — The Dragon's Lair Project](https://www.dragons-lair-project.com/games/related/sequence.asp)
- [Dragon's Lair — TV Tropes](https://tvtropes.org/pmwiki/pmwiki.php/VideoGame/DragonsLair)
- [Dragon's Lair — arcade-history](https://www.arcade-history.com/?n=dragons-lair&page=detail&id=702)
- [Quick time event — Wikipedia](https://en.wikipedia.org/wiki/Quick_time_event)
