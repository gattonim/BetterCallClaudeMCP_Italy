# La Tana del Drago

Omaggio ai laserdisc game dei primi anni '80 — *Dragon's Lair* su tutti — scritto
in una singola pagina HTML: nessuna dipendenza, nessun asset esterno, tutta la
grafica è disegnata a runtime su `<canvas>` e i suoni sono sintetizzati con
Web Audio.

## Come si gioca

Apri `index.html` in un browser (basta un doppio clic: non serve un server).

Il cavaliere Dirk attraversa le segrete del castello di Mordroc. Ogni quadro ha
**un solo istante decisivo**: quando compare il richiamo dorato in basso, premi
il tasto indicato prima che la barra si svuoti.

| Comando | Tasti |
|---|---|
| Schiva ai lati | `←` `→` — anche `A` / `D` |
| Salta, arrampicati | `↑` — anche `W` |
| Abbassati, tuffati | `↓` — anche `S` |
| Spada (colpisci, para) | `Spazio` — anche `Z` o `Invio` |

Su schermi touch compaiono automaticamente un pad direzionale e il pulsante
*SPADA*.

## Struttura di una partita

- **8 quadri** estratti a sorte fra gli 11 disponibili (ponte crollante,
  tentacoli, sala allagata, vortice, cavaliere nero, corde in fiamme,
  pipistrelli, scala sprofondante, re lucertola, sala degli specchi, serpente di
  fuoco), poi il **drago in tre affondi**: fiammata, coda, cuore.
- **3 vite**. Tasto sbagliato o tempo scaduto: si perde una vita e si ripete il
  quadro. Finite le vite, la partita si chiude.
- Il punteggio premia la prontezza (fino a +220 per un riflesso immediato) e la
  serie di risposte esatte consecutive.
- Dopo la vittoria si può proseguire con il **Ciclo 2**, 3, … : le finestre di
  reazione si stringono del 12% a ogni ciclo e il punteggio si accumula. Il
  record resta salvato in `localStorage`.

## Note tecniche

- File unico, ~1100 righe, nessun processo di build: non fa parte dei workspace
  npm del repository e non tocca i server MCP.
- Il palco è in coordinate fisse 960×600 e viene scalato al riquadro disponibile
  tenendo conto di `devicePixelRatio`.
- `window.__tana()` espone lo stato di gioco in sola lettura (quadro, tasto
  atteso, fase, vite, punti): serve ai test automatici con browser headless.
- I font Cinzel/Inter arrivano da Google Fonts; se la rete non è disponibile il
  gioco ripiega su Georgia e sul font di sistema senza perdere nulla.
