---
layout: post
title: "Perché ho scelto un database SQLite locale per gli acquisti di cantiere"
categoria: Data Engineering
---

Quando si parla di "database aziendale" si pensa subito a un server, a una licenza, a qualcuno che lo gestisca. Per una PMI è spesso la strada sbagliata: costa troppo e richiede competenze che in azienda non ci sono.

Per gli acquisti dei cantieri di un'impresa edile ho scelto la strada opposta: **SQLite**, un database relazionale che vive in un singolo file, sul computer o sul server dell'azienda. Qui spiego perché ha funzionato e come è organizzato il flusso, dalla web app dei cantieri fino al costo per commessa.

## Il problema: gli ordini arrivano, ma non sono ancora dati utilizzabili

Gli ordini inseriti dai capicantiere finiscono in un foglio Google (ne parlo nell'[articolo precedente]({{ site.baseurl }}{% post_url 2026-10-01-acquisti-cantiere-web-app-google-sheet %})). Il foglio raccoglie bene, ma non basta per ragionare sui costi:

- lo stesso fornitore compare scritto in modi diversi;
- i numeri arrivano a volte come testo, con virgole e punti usati a caso;
- non c'è un modo semplice di incrociare gli ordini con altre informazioni, né di costruire sopra un calcolo affidabile.

Serve un posto dove i dati siano **ordinati, con regole, e interrogabili in SQL**.

## Perché SQLite e non un database server

Per questo caso d'uso SQLite ha quattro vantaggi concreti:

1. **Nessun server da installare o mantenere.** Il database è un file. Non c'è un servizio che si può fermare nel weekend.
2. **Backup banale.** Copiare un file basta. Si può farlo ogni notte con una riga di script.
3. **Costo zero.** Nessuna licenza, nessun abbonamento.
4. **È SQL vero.** Tabelle, vincoli, viste: tutto quello che si impara su un database "serio" vale anche qui. Se un giorno i volumi cresceranno, migrare verso PostgreSQL è un passaggio naturale, perché le query restano quasi identiche.

Il limite da conoscere: SQLite regge bene una sola scrittura alla volta. Per un carico notturno unico, come questo, non è un problema. Per decine di utenti che scrivono insieme durante il giorno lo sarebbe.

## Il flusso, notte dopo notte

Ogni sera uno script Python esegue gli stessi passaggi:

1. **Legge** gli ordini dal foglio Google.
2. **Pulisce**: toglie spazi e differenze di maiuscole nei nomi dei fornitori, converte quantità e prezzi in numeri, scarta le righe a cui mancano i dati essenziali.
3. **Evita i duplicati**: ogni ordine ha un identificativo, quindi se lo script gira due volte non registra due volte lo stesso acquisto.
4. **Carica** il risultato in una tabella "grezza" del database.

<!-- TODO: se vuoi, indica come lo script legge il foglio (API di Google o esportazione) e come viene pianificato (Utilità di pianificazione di Windows, cron, ecc.) -->

Questo è uno schema semplificato del passaggio di caricamento:

```python
import sqlite3
import pandas as pd

def carica_acquisti(df: pd.DataFrame, db_path: str) -> None:
    df = df.copy()
    df["fornitore"] = df["fornitore"].str.strip().str.upper()
    df["quantita"] = pd.to_numeric(df["quantita"], errors="coerce")
    df["prezzo_unitario"] = pd.to_numeric(df["prezzo_unitario"], errors="coerce")
    df = df.dropna(subset=["cantiere", "materiale", "quantita", "prezzo_unitario"])
    df = df.drop_duplicates(subset=["id_ordine"])

    with sqlite3.connect(db_path) as conn:
        df.to_sql("acquisti_appoggio", conn, if_exists="replace", index=False)
        conn.execute("""
            INSERT OR IGNORE INTO acquisti_grezzi
            SELECT id_ordine, data_ordine, cantiere, fornitore,
                   materiale, quantita, prezzo_unitario
            FROM acquisti_appoggio
        """)
```

La tabella `acquisti_appoggio` serve solo da passaggio: i dati puliti entrano in `acquisti_grezzi` e, grazie all'`INSERT OR IGNORE` sull'identificativo, un ordine già presente non viene duplicato.

## Dalla tabella grezza al costo di cantiere: una vista SQL

La tabella grezza contiene gli ordini così come sono stati registrati. Il costo per cantiere non viene calcolato a mano né in Excel, ma da una **vista SQL**: una "query salvata" che si aggiorna da sola ogni volta che arrivano nuovi dati.

Versione semplificata:

```sql
CREATE TABLE IF NOT EXISTS acquisti_grezzi (
    id_ordine       TEXT PRIMARY KEY,
    data_ordine     TEXT NOT NULL,
    cantiere        TEXT NOT NULL,
    fornitore       TEXT NOT NULL,
    materiale       TEXT NOT NULL,
    quantita        REAL NOT NULL,
    prezzo_unitario REAL NOT NULL
);

CREATE VIEW IF NOT EXISTS v_costi_cantiere AS
SELECT
    cantiere,
    strftime('%Y-%m', data_ordine)  AS mese,
    materiale,
    SUM(quantita)                   AS quantita_totale,
    SUM(quantita * prezzo_unitario) AS costo_totale
FROM acquisti_grezzi
GROUP BY cantiere, mese, materiale;
```

Il vantaggio è la **separazione delle responsabilità**: la tabella grezza non si tocca mai, mentre la logica del costing sta in un solo punto, la vista. Se cambia il modo di calcolare i costi, si modifica la vista e tutto ciò che c'è sopra (report, dashboard) si aggiorna senza altri interventi. Nella versione reale la vista è più ricca, ma l'idea è la stessa.

## Cosa ne ricavo

- **Il costo di un cantiere si aggiorna ogni notte**, invece di essere ricostruito a fine lavori.
- **Una sola fonte di verità**: i numeri che vede la direzione sono gli stessi che escono dalla vista, non da copie diverse in file diversi.
- **Una base su cui costruire**: sopra questo database si appoggia la dashboard in Power BI che racconto in un altro articolo.

---

Hai dati sparsi tra fogli, gestionali e cartelle e vorresti una base unica e affidabile? [Parliamone]({{ '/#contatto' | relative_url }}).
