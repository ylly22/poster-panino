# Panino Interattivo — Specifica di Design

## Obiettivo

Trasformare il poster `poster.svg` in una pagina web interattiva dove l'utente può trascinare gli ingredienti sul panino per costruire il proprio panino, utilizzando Paper.js.

## Elementi della pagina

### Elementi fissi (non interattivi)
- **Sfondo verde** — background della scena
- **Pane base** — la base del panino, sempre visibile e immutabile

### Elementi trascinabili
- Cipolla
- Insalata
- Hamburger
- Bacon
- Formaggio
- Pomodoro
- **Pane superiore** — la calotta superiore del panino

Ciascun ingrediente è già visibile nella sua posizione originale del poster. L'utente può scegliere quali usare e in quale ordine, senza obbligo di usarli tutti.

## Interazione

### Trascinamento
- L'utente clicca su un ingrediente e lo trascina con il mouse (o tocco su tablet)
- Durante il trascinamento, l'ingrediente segue il cursore
- L'ingrediente può essere lasciato in qualsiasi punto della pagina

### Zona di rilascio (drop target)
- La zona del pane base (la parte inferiore del panino) è l'area di rilascio
- Se l'ingrediente viene lasciato **sopra il pane base**, viene considerato "piazzato"
- Se l'ingrediente viene lasciato **fuori dal pane base**, torna alla sua posizione originale

### Ingresso degli ingredienti
- Quando un ingrediente viene piazzato sul panino, si **ingrandisce** (scalato proporzionalmente, es. 1.5x–2x)
- L'ingrediente piazzato rimane nella posizione in cui è stato lasciato sopra il panino

### Pane superiore e "Buon Appetito"
- Il pane superiore funziona come gli altri ingredienti (trascinabile, si ingrandisce se piazzato)
- **Quando il pane superiore viene piazzato:**
  1. La scritta "CREA IL TUO PANINO" (testo nella parte superiore del poster) scompare
  2. La scritta "BUON APPETITO" appare al centro della pagina

## Tecnologia

- **Paper.js** — libreria JavaScript per la gestione del canvas, del trascinamento e delle trasformazioni
- Singola pagina HTML che carica Paper.js via CDN
- Il file `poster.svg` viene importato in Paper.js con `project.importSVG()`, preservando tutte le forme, i colori e le posizioni originali
- Canvas Paper.js copre tutta l'area visibile, sostituendo la visualizzazione SVG statica
- Le scritte "BUON APPETITO" e "CREA IL TUO PANINO" esistono già nell'SVG originale. All'avvio, "CREA IL TUO PANINO" è visibile e "BUON APPETITO" è nascosto. Quando il pane superiore viene piazzato, i ruoli si invertono.

## Flusso utente

1. L'utente apre la pagina e vede il poster identico all'originale
2. Trascina gli ingredienti desiderati sul panino
3. Gli ingredienti piazzati si ingrandiscono e restano sul panino
4. Quando è soddisfatto, posiziona il pane superiore
5. "CREA IL TUO PANINO" scompare, "BUON APPETITO" appare
6. L'utente può eventualmente rimuovere il pane superiore (trascinandolo via) per modificare ulteriormente il panino

## Note tecniche

- Paper.js gestisce internamente hit testing per determinare se un ingrediente è stato rilasciato sopra il pane base
- Il ridimensionamento degli ingredienti piazzati avviene via `item.scale()` di Paper.js
- Le posizioni originali vengono salvate al caricamento per permettere il ritorno degli ingredienti non piazzati
- L'animazione di comparsa di "BUON APPETITO" può includere un leggero fade-in
