---
layout: post
title: "Quanti esami servono davvero? La ricerca operativa applicata alla saturazione delle macchine di imaging"
categoria: Ricerca Operativa - Modellazione matematica
---

In un centro di diagnostica per immagini, le macchine sono l'investimento più pesante e il vincolo più rigido. Una risonanza o una TAC costano ogni giorno, che vengano usate o no: ammortamento, manutenzione, personale dedicato. La domanda che la direzione si pone è semplice, ma spesso senza risposta numerica: **a che livello di utilizzo il centro smette di perdere e comincia a guadagnare, e quanto margine resta da sfruttare?**

Abbiamo affrontato la questione con la ricerca operativa, partendo da un'idea semplice: prima di ottimizzare qualcosa, bisogna descriverlo con un modello matematico.

## Dalla sensazione al modello

La percezione comune era che le macchine fossero "abbastanza cariche". Ma "abbastanza" non è un dato, e senza un dato non si può decidere se aprire altre fasce orarie, rinegoziare le convenzioni o rivedere l'agenda.

Abbiamo costruito un'equazione che lega la **produttività** della giornata al **numero di servizi effettuati**. Nel modello entrano gli elementi che determinano davvero il risultato:

- il tempo di esecuzione di ciascun tipo di esame, compresi preparazione del paziente e refertazione;
- i costi fissi legati alla macchina e al personale, che pesano a prescindere dai volumi;
- il ricavo medio per servizio, al netto dei costi variabili;
- i vincoli di capacità, cioè le ore realmente disponibili in una giornata.

## Il modello

Per ogni tipo di esame *i* abbiamo descritto la giornata con pochi parametri:

- $$n_i$$: numero di esami di tipo *i* eseguiti in un giorno (la variabile da determinare);
- $$t_i$$: minuti di macchina per esame, preparazione del paziente inclusa;
- $$m_i = p_i - c_i$$: margine di contribuzione per esame (ricavo meno costo variabile);
- $$F$$: costi fissi giornalieri di macchina e personale;
- $$T$$: minuti di macchina disponibili in una giornata.

La **produttività** è la quota di tempo disponibile effettivamente utilizzata:

$$
u(n) = \frac{\sum_i t_i \, n_i}{T}
$$

Il **punto di pareggio** è il livello in cui i margini coprono i costi fissi:

$$
\sum_i m_i \, n_i = F
$$

## Che cosa ha fatto la ricerca operativa

Una volta scritto il modello, il problema diventa matematico. Abbiamo usato l'ottimizzazione vincolata per trovare i valori che rispondono a due domande concrete:

1. **Il punto di pareggio.** Qual è il numero minimo di servizi giornalieri che copre i costi, impiegando il minor tempo di lavoro possibile? Sotto quella soglia la giornata è in perdita, sopra comincia a generare margine.
2. **Il livello di produttività del 60%.** Quanti servizi servono per raggiungere un utilizzo del 60% della capacità? Una soglia di riferimento utile per capire quanto spazio c'è ancora prima di saturare la macchina.

### Il problema di ottimizzazione

Tra tutte le combinazioni di esami che raggiungono il pareggio, cerchiamo quella che impegna la macchina il minor tempo possibile:

$$
\min_{n \ge 0} \; \sum_i t_i \, n_i
\qquad \text{con} \qquad
\sum_i m_i \, n_i \ge F
$$

Con un solo tipo di esame la soluzione si scrive direttamente:

$$
n_{BE} = \frac{F}{m}
\qquad
u_{BE} = \frac{t \, F}{m \, T}
\qquad
n_{60} = \frac{0{,}6 \, T}{t}
$$

Il primo valore è il numero di esami del pareggio, il secondo la produttività a cui si raggiunge, il terzo il numero di esami che porta la macchina al 60% di utilizzo. Il confronto tra $$u_{BE}$$ e il 60% dice subito quanto margine di manovra resta.

Con più tipi di esame il risultato cambia in modo istruttivo: per coprire i costi nel minor tempo conviene dare priorità agli esami con il minor numero di minuti per euro di margine, cioè con il rapporto $$t_i / m_i$$ più basso. Nella pratica si aggiunge un limite di domanda per ogni tipo di esame ($$n_i \le d_i$$), e la soluzione riempie la giornata in quest'ordine finché i costi fissi sono coperti.

> **Approfondimento: la lagrangiana.** Associando un moltiplicatore $$\lambda \ge 0$$ al vincolo di pareggio si ottiene
>
> $$
> \mathcal{L}(n, \lambda) = \sum_i t_i \, n_i + \lambda \left( F - \sum_i m_i \, n_i \right)
> $$
>
> Le condizioni di ottimo impongono $$t_i - \lambda m_i \ge 0$$ per ogni esame, con uguaglianza per quelli effettivamente eseguiti, da cui
>
> $$
> \lambda^{*} = \min_i \frac{t_i}{m_i}
> \qquad
> W^{*} = \lambda^{*} F
> $$
>
> Il moltiplicatore è il **prezzo ombra** del vincolo: ogni euro in più di costi fissi da coprire richiede $$\lambda^{*}$$ minuti aggiuntivi di macchina, nella combinazione di esami migliore. È il numero che permette di valutare, per esempio, quanto "pesa" in tempo di lavoro un nuovo contratto di manutenzione.

### I risultati

[Inserire i valori, oppure una forma indicativa come "il punto di pareggio si colloca a circa X servizi al giorno, il 60% di produttività a circa Y"].

## Perché il tempo di lavoro conta quanto il numero di esami

Minimizzare il tempo di lavoro nel punto di pareggio significa una cosa precisa: capire **con quanto impegno della macchina e del personale si arriva a coprire i costi**. Due giornate con lo stesso numero di esami possono avere esiti molto diversi se cambia l'organizzazione dei tempi. Il modello rende questa differenza visibile e misurabile, e per questo è utile anche per valutare l'effetto di un cambio di agenda prima di applicarlo.

## Cosa cambia per chi decide

- Una soglia chiara sotto la quale una giornata non si sostiene, al posto di una stima a occhio.
- Un obiettivo di produttività concreto (il 60%) con il numero di servizi corrispondente.
- Una base oggettiva per discutere di orari, turni e convenzioni.
- La possibilità di simulare scenari: cosa succede se cambia il tempo medio di un esame, il costo di una risorsa o la tariffa.

## In sintesi

La ricerca operativa non serve a complicare le cose. Serve a trasformare una domanda gestionale ("stiamo usando bene le macchine?") in una domanda a cui si può rispondere con un numero. Il valore sta nel modello: se descrive bene il processo, l'ottimizzazione fa il resto.

Se gestite risorse costose e vincolate, come macchinari, linee o turni, e volete sapere dove si colloca il vostro punto di pareggio, possiamo costruire insieme il modello.
