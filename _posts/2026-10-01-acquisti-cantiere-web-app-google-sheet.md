---
layout: post
title: "Dal cantiere al dato: raccogliere gli acquisti senza comprare un software"
categoria: Data Engineering
---

In un'impresa edile gli acquisti nascono dove nessuno ha un computer davanti: in cantiere. Il capocantiere ordina il materiale al telefono, il fornitore consegna, la bolla finisce in una cartella o al peggio persa tra un mattone e l'altro. Quando l'amministrazione registra tutto, spesso è passato un mese, e il costo di un cantiere si scopre a lavori finiti.

Il problema non è la mancanza di un gestionale. È che **il dato nasce lontano dal sistema che dovrebbe contarlo**, e ogni passaggio manuale in mezzo è un ritardo, un errore possibile, una voce dimenticata.

## L'idea: abbassare al minimo il costo di inserire un dato

Chi lavora in cantiere non userà mai uno strumento complicato. Quindi lo strumento deve fare una sola cosa, in meno di un minuto, dal telefono: registrare un ordine.

Per questo ho costruito una **web app in HTML**, senza installazioni e senza account da gestire. Il capocantiere apre una pagina e compila pochi campi:

- fornitore
- cantiere
- materiale
- quantità
- prezzo unitario

Premendo "invia", l'ordine è registrato. Nient'altro.

<!-- TODO: aggiungi uno screenshot della web app (anonimizzato) in /assets/ e inseriscilo qui con ![Maschera di inserimento ordini]({{ '/assets/NOMEFILE.png' | relative_url }}) -->

## Un foglio Google come punto di raccolta

I dati inviati dalla web app arrivano in un **foglio Google**, che funziona da endpoint: una piccola funzione (Google Apps Script) riceve la richiesta e aggiunge una riga al foglio.

Perché un foglio e non direttamente un database?

- **Costo e manutenzione zero**: non c'è un server da tenere acceso e manutenere.
- **Visibilità immediata e facilità di utilizzo**: chi in azienda vuole controllare cosa è stato ordinato apre il foglio e lo vede, senza strumenti particolari, senza bisogno di passare per fogure intermedie che estraggano i dati dal db.
- **Un buffer sicuro**: se il resto della catena si ferma per un giorno, gli ordini sono comunque salvati e nulla va perso.

Il foglio non è l'archivio definitivo, né deve esserlo. È la **cassetta delle lettere**: raccoglie, e basta. Il lavoro vero comincia dopo, di notte, quando uno script python porta quei dati in un database ordinato (ne parlo nell'articolo dedicato al servizio di data engineering).

## Cosa ho imparato

Tre scelte che rifarei:

1. **Pochi campi obbligatori.** Ogni campo in più è un motivo per non compilare. Quello che manca si ricostruisce dopo, quello che non viene inserito no.
2. **Nessun calcolo in cantiere.** Il capocantiere inserisce quantità e prezzo; totali, ripartizioni e costi si calcolano a valle. Chi inserisce non deve fare conti.
3. **Separare raccolta ed elaborazione.** La web app e il foglio raccolgono, tutto il resto avviene altrove. Così posso cambiare il modo di elaborare i dati senza toccare ciò che usano i cantieri.

## Limiti da conoscere

Un foglio Google non è un database: non impone regole sui dati, quindi qualcuno può scrivere il nome di un fornitore in tre modi diversi. È normale, ed è per questo che serve una fase di pulizia. Non è adatto a volumi molto grandi, ma per gli ordini di un'impresa di dimensioni piccole è più che sufficiente.

---

Se la tua azienda ha lo stesso problema, cioè dati che nascono in un posto e vengono contati in un altro con settimane di ritardo, scrivimi a edoardo.crema@outlook.it. Di solito basta una call per capire da dove partire.
