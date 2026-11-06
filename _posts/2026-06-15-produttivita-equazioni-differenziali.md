# Quando i fogli di calcolo non bastano: modellare la produttività con le equazioni differenziali

*Un caso di studio di Optima Engineering. Il contesto è ricostruito e i dati sono interamente sintetici: il caso reale resta riservato.*

## La domanda

Un'azienda manifatturiera ci ha posto una domanda solo in apparenza semplice: **se in una giornata arriva più lavoro, cosa succede alla produttività? E la produzione riesce a "digerirlo"?**

Il CFO, che ad interim svolgeva anche il ruolo di ingegnere gestionale, aveva un timore fondato. La forza commerciale era brava ed entusiasta, e forse troppo avanti rispetto alle operations. Per verificarlo aveva costruito dei fogli di calcolo che bilanciavano capacità delle macchine e degli operatori. Era un lavoro di grande fattura, ma non rispondeva alla domanda.

## Perché il foglio di calcolo non risponde

Il foglio confronta due medie: il lavoro medio giornaliero e la capacità media giornaliera. Nel nostro dataset sintetico, che riproduce la struttura del caso reale, il reparto più carico (il **trattamento**) risulta occupato al **79%** della sua capacità. Per un foglio di calcolo è una situazione sana.

Il confronto tra medie ignora però il **quando**. Il lavoro non arriva spalmato in modo uniforme sulla giornata, e la capacità non si recupera: un'ora in cui gli operatori sono fermi non si può usare per smaltire il lavoro che arriverà dopo.

## Passo 1: che natura hanno gli arrivi?

Prima di scegliere il modello abbiamo testato la natura degli arrivi. Se fossero casuali e indipendenti (un processo di Poisson), la teoria delle code sarebbe la strada naturale. Abbiamo contato gli ordini in finestre da 15 minuti e calcolato l'**indice di dispersione** (varianza/media), che per un processo di Poisson vale 1.

Sul dataset sintetico (20 giornate, 1.484 ordini) l'indice vale **circa 6,7**. Gli arrivi non sono affatto spontanei: arrivano a ondate, agganciate ai ritiri dei corrieri (mattina presto, metà mattina, primo pomeriggio, fine pomeriggio), con qualche urgente fuori finestra. Con arrivi così strutturati le ipotesi dei modelli a coda non reggono, e abbiamo scelto un'altra strada.

## Passo 2: il modello, un conto in banca per la produzione

Abbiamo descritto ogni reparto con un'equazione differenziale sul **backlog** B(t), cioè i minuti di lavoro arrivati e non ancora smaltiti:

$$
\frac{dB}{dt} = \lambda(t) - \mu(t), \qquad B(t) \ge 0
$$

- **λ(t)** è il lavoro che entra, in minuti di lavoro standard, ricostruito dagli orari reali di arrivo degli ordini.
- **μ(t)** è il lavoro che esce: FTE × efficienza, pari a zero durante la pausa e a fine turno.
- **B(t)** è il saldo del conto: in ogni istante dt sappiamo quanto lavoro è in attesa.

È la logica del conto in banca. Se le entrate superano le uscite, il saldo cresce e la produzione **soffoca**. La condizione B ≥ 0 esprime il fatto che la capacità non si accumula: se oggi lavoriamo meno di quanto potremmo, non recuperiamo domani.

In forma discreta, con passo di un minuto:

$$
B_{t+1} = \max\big(B_t + \lambda_t - \mu_t,\ 0\big)
$$

Il passo di un minuto permette di vedere cose che una media giornaliera nasconde: l'ora del picco, la sua altezza, e il residuo che resta in coda alla chiusura.

## Passo 3: cosa ci dice il modello

### La giornata tipica

Il trattamento, con **2 FTE**, è al 79% in media. Eppure nel modello **in 17 giornate su 20 chiude con lavoro non smaltito**: la mediana del residuo a fine turno è di circa **57 minuti di lavoro**. Il motivo sta nella forma dell'onda. Il grosso dell'ultima ondata arriva verso le 15:30, quando restano pochi minuti di capacità, e il picco di backlog si raggiunge intorno alle 15:40 (circa 176 minuti di lavoro in coda).

![Backlog del reparto trattamento nel corso della giornata](trattamento_backlog.png)

### Lo stress test: la giornata post-promozione

La giornata 14 simula l'effetto di una spinta commerciale: 92 ordini contro una media di 74 (+24%). Il trattamento arriva al **100%** di utilizzo teorico. Nel modello il picco sale a circa **249 minuti** di lavoro in coda e **133 minuti** restano a fine giornata, da scaricare sul giorno dopo.

### Cosa succede se il lavoro aumenta (what-if)

Abbiamo scalato il lavoro di tutte le 20 giornate di un fattore e misurato in quante giornate il reparto chiude senza residuo.

| Aumento del lavoro | Sagomatura | Trattamento | Montaggio | Controllo |
|---|---|---|---|---|
| +0% | 100% | **15%** | 100% | 100% |
| +10% | 100% | **5%** | 100% | 100% |
| +20% | 95% | **0%** | 100% | 100% |
| +30% | 80% | **0%** | 100% | 100% |
| +40% | 70% | **0%** | 100% | 100% |

*Quota di giornate chiuse senza lavoro residuo, per reparto e per scenario.*

Il sistema non è pieno ovunque: ha **un solo collo di bottiglia**. Il resto della produzione ha margine (montaggio e controllo non soffrono nemmeno con +40%). Assumere in montaggio per "aiutare" la produzione non risolverebbe nulla.

### Quanti FTE servono davvero?

Per ogni reparto abbiamo cercato il numero minimo di FTE che chiude ogni giornata senza residuo.

| Reparto | FTE attuali | Minimi (giornata mediana) | Minimi (90° percentile) | Minimi (giornata 14) |
|---|---|---|---|---|
| Sagomatura | 3 | 2 | 3 | 3 |
| Trattamento | 2 | **3** | **4** | 3 |
| Montaggio | 4 | 2 | 2 | 2 |
| Controllo | 2 | 1 | 2 | 2 |

L'organico è **sbilanciato**: il trattamento è sotto-dimensionato rispetto al profilo degli arrivi, mentre il montaggio ha capacità in eccesso. Prima di pensare a nuove assunzioni conviene valutare una riallocazione di risorse tra reparti.

## La risposta al CFO

1. **Il timore era fondato, ma localizzato.** La forza commerciale non è sovradimensionata rispetto a tutta la produzione: lo è rispetto a un reparto, il trattamento.
2. **L'utilizzo medio inganna.** Un reparto al 79% può chiudere in ritardo quasi ogni giorno, perché conta quando arriva il lavoro, non solo quanto.
3. **Si può agire sul tempo, non solo sulla capacità.** Spostare o distribuire gli orari di ritiro dei corrieri riduce il picco di λ(t) senza assumere nessuno. Il modello permette di testare anche questa leva prima di toccare i ritiri.

## Limiti del modello (e come li abbiamo gestiti)

- **Reparti in parallelo.** Nella versione mostrata ogni reparto riceve il suo lavoro all'istante di arrivo dell'ordine. In realtà c'è un flusso in serie: l'uscita da un reparto è l'ingresso del successivo. Il modello si estende a una cascata di equazioni accoppiate, ed è il passo successivo più naturale.
- **Giornate indipendenti.** Il residuo di fine giornata non viene riportato al giorno dopo. Includerlo peggiora i risultati dei reparti già critici.
- **Efficienza costante.** FTE × efficienza è una semplificazione: assenze, cambi di mix e curve di apprendimento si possono aggiungere come parametri.
- **Dati sintetici.** Numeri e percentuali di questo articolo vengono da un dataset generato per illustrare il metodo, non da dati del cliente.

## Il metodo in sintesi

1. **Testare** la natura degli arrivi prima di scegliere il modello.
2. **Modellare** il backlog come un bilancio istante per istante.
3. **Stressare** il modello con scenari di aumento del lavoro.
4. **Dimensionare** gli FTE minimi reparto per reparto.

Un foglio di calcolo ti dice se i conti tornano a fine giornata. Un modello dinamico ti dice **cosa succede durante la giornata**, che è dove la produzione soffoca o respira.

*Optima Engineering*
