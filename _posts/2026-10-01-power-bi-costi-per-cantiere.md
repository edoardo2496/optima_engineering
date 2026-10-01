---
layout: post
title: "Una dashboard Power BI per vedere i costi di ogni cantiere mentre accadono"
categoria: BI e Analytics
---

Chi dirige un'impresa edile ha una domanda semplice: *quanto mi sta costando questo cantiere, adesso?* La risposta di solito arriva tardi e a pezzi: un file Excel da una persona, un estratto del gestionale da un'altra, qualche stima a voce.

Per un cliente del settore costruzioni ho costruito una **dashboard in Power BI** che risponde a quella domanda in pochi secondi, con dati aggiornati ogni notte. Qui racconto come è pensata e come si collega al database degli acquisti.

## Da dove arrivano i numeri

La dashboard non legge file sparsi. Si appoggia a una sola fonte: un database SQLite in cui ogni sera vengono caricati gli ordini dei cantieri (il flusso è descritto negli articoli sulla raccolta degli acquisti e sulla [pipeline notturna]({{ site.baseurl }}{% post_url 2026-10-01-sqlite-pipeline-notturna-costing %})).

Sopra quel database c'è una **vista SQL** che calcola il costo per cantiere, per mese e per materiale. Power BI legge direttamente quella vista. È una scelta precisa: **la logica del calcolo sta nel database, non nella dashboard.** Se cambia il modo di calcolare un costo, si corregge in un solo punto e tutti i report restano coerenti.

## Collegare Power BI a SQLite

Power BI non ha un connettore nativo per SQLite. Si usa un **driver ODBC per SQLite**, installato sul computer, e poi si sceglie *Recupera dati → ODBC* indicando il file del database.

Due aspetti pratici da tenere presenti:

- **Aggiornamento dei dati.** Con Power BI Desktop basta premere "Aggiorna". Per pubblicare il report online e farlo aggiornare in automatico serve un *gateway dati locale*, perché il database sta su una macchina aziendale e non nel cloud.
- **Importazione dei dati.** Per i volumi di un'impresa di medie dimensioni la modalità "Importa" è la scelta più semplice e veloce: i dati vengono caricati nel report a ogni aggiornamento.

## Come ho pensato la dashboard

La regola che mi do è che **una dashboard deve rispondere a domande, non mostrare dati**. Le domande di chi dirige un'impresa sono poche:

- Quanto abbiamo speso, in totale e per ciascun cantiere?
- Quali materiali pesano di più?
- Come cambia la spesa di mese in mese?

Per questo ho evitato decine di grafici. Ogni elemento della pagina risponde a una di queste domande, e un filtro per cantiere permette di passare dal quadro generale al dettaglio senza cambiare schermata.

<!-- TODO: descrivi qui le pagine e i grafici reali della dashboard (ad esempio: costo per cantiere, andamento mensile, peso per materiale) e aggiungi uno screenshot anonimizzato in /assets/ con ![Dashboard costi per cantiere]({{ '/assets/NOMEFILE.png' | relative_url }}) -->

## Le misure: poche, ma chiare

Nel modello le misure sono volutamente semplici. Due esempi in DAX, il linguaggio di calcolo di Power BI:

```dax
Costo totale = SUM(v_costi_cantiere[costo_totale])

Incidenza materiale % =
DIVIDE(
    [Costo totale],
    CALCULATE([Costo totale], ALL(v_costi_cantiere[materiale]))
)
```

La prima somma il costo nel contesto selezionato (un cantiere, un mese). La seconda dice quanto pesa un materiale sul totale dello stesso contesto, e permette di vedere subito dove si concentra la spesa.

## Cosa cambia per chi decide

- **Meno tempo a cercare i numeri.** Il costo di un cantiere è a una pagina di distanza, già aggiornato alla notte precedente.
- **Un confronto possibile tra cantieri.** Con gli stessi criteri di calcolo ovunque, i cantieri si possono confrontare davvero.
- **Un punto di partenza.** Una volta che il dato è affidabile, si può costruire sopra: confronti con il preventivo, indicatori di scostamento, previsioni.

---

Se vuoi vedere i costi della tua azienda in una dashboard simile, [scrivimi]({{ '/#contatto' | relative_url }}): si parte dai dati che hai già.
