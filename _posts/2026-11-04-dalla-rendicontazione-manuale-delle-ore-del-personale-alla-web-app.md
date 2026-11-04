---
layout: post
title: "Dalle foto e dai foglietti al dato in tempo reale: la rendicontazione delle ore del personale"
date: 2026-11-04
categories: ["Data Engineering & Data Optimization"]
excerpt: "Ogni mese, in molte PMI, le ore lavorate dal personale arrivano all'amministrazione per le vie più diverse: un messaggio, la foto di un foglio, un biglietto consegnato a mano. Poi qualcuno le riporta a mano in un file di calcolo, le controlla, le ricompone per il commercialista. Funziona, finché qualcosa non si perde o non si capisce."
---

Abbiamo lavorato su questo problema in un'impresa edile con più sedi e più cantieri. Il punto di partenza non era uno strumento, ma una domanda: come viaggia davvero l'informazione sulle ore, e dove si blocca?

## Mappare il processo prima di scegliere lo strumento

Abbiamo ricostruito il flusso con un diagramma a corsie (swim lane), che mostra per ogni attore cosa fa, cosa consegna e a chi. Il risultato è stato immediato: il destinatario finale, l'amministrazione, concentrava tutto il peso. Riceveva dati in formati eterogenei e doveva uniformarli a mano in un foglio di calcolo, prima di poter fare due cose distinte:

- registrare i costi del personale nella sede o nel cantiere corretto;
- comunicare al commercialista le ore di ciascun dipendente, spacchettate per sede, per la preparazione delle buste paga.

I colli di bottiglia erano nella raccolta: strumenti non tracciabili, informazioni non strutturate, nessuna convalida al momento dell'inserimento.

## Una sola raccolta per due esigenze

Abbiamo condiviso la mappa con l'amministrazione e con il commercialista. Le due esigenze sembravano diverse, ma partivano dallo stesso dato: chi ha lavorato, quando, dove, quante ore ordinarie e quante straordinarie. Se quel dato viene raccolto bene alla fonte, una sola raccolta basta per entrambi.

Abbiamo quindi progettato un'unica soluzione di raccolta, pensata per chi la usa ogni giorno: il dipendente.

## La soluzione: una web app da telefono

Il dipendente apre una pagina web dal proprio telefono, sceglie il proprio nome e il cantiere da elenchi predefiniti, indica la data e le ore (con scorciatoie per i turni più comuni) e salva con un tocco. Alla registrazione successiva il nome è già impostato, e può inserire più cantieri nella stessa giornata in pochi secondi.

Gli elenchi arrivano direttamente dal database, quindi nessuno scrive a mano il nome di un cantiere e le sigle non si moltiplicano. Ogni registrazione finisce in tempo reale in un database relazionale, con una struttura uguale per tutti.

## Dal dato al report

Con i dati ordinati, il resto si automatizza:

- **Per il commercialista**: uno script Python genera ogni mese il template standard con le ore spacchettate per sede, pronto da inviare, senza ricomposizioni manuali.
- **Per l'amministrazione**: abbiamo formato il personale amministrativo a consultare in autonomia le presenze, per dipendente, per periodo e per cantiere, così da attribuire i costi alle sedi giuste senza dipendere da nessun altro.

## Cosa è cambiato

- Un solo canale di raccolta al posto di molti, con formato unico.
- Dati disponibili in tempo reale, non a fine mese.
- Meno errori di trascrizione e meno rielaborazioni.
- Ore e costi del personale imputati correttamente a ciascuna sede.
- 25 ore risparmiate dall'amministrazione ogni mese

## Cosa ne ricaviamo

La tecnologia qui è la parte più semplice. Il lavoro vero è stato capire il processo, ascoltare chi ne usa i risultati e trovare il punto in cui un dato, raccolto una volta sola e bene, risolve più problemi insieme. È il metodo con cui affrontiamo anche altri processi aziendali: prima si mappa, poi si progetta, infine si automatizza.

Se in azienda raccogliete ancora informazioni in formati diversi, possiamo partire da una mappatura del processo.
