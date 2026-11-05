---
layout: post
title: "Quanti esami servono davvero? La ricerca operativa applicata alla saturazione delle macchine di imaging"
categoria: Ricerca Operativa - Modellazione matematica
---

In un centro di diagnostica per immagini, le macchine sono l'investimento più pesante e il vincolo più rigido. Una risonanza o una TAC costano ogni giorno, che vengano usate o no: ammortamento, manutenzione, personale dedicato. La domanda che la direzione si pone è semplice, ma spesso senza risposta numerica: **a che livello di utilizzo il centro smette di perdere e comincia a guadagnare, e quanto margine resta da sfruttare?**

Abbiamo affrontato la questione con la ricerca operativa, partendo da un'idea semplice: prima di ottimizzare qualcosa, bisogna descriverlo con un modello matematico.

## La risposta in breve

Nello scenario prudente che abbiamo simulato, un centro con una risonanza, un apparecchio radiografico e un ecografo:

- **il pareggio operativo si raggiunge tra il 58% e il 63% di saturazione** del reparto, a seconda dell'anno;
- **il 60% di saturazione non è una soglia scelta a caso**: coincide con il punto in cui i margini coprono i costi, e corrisponde a circa 48 esami al giorno;
- **il mix di esami conta quanto il volume**: nel terzo anno il centro lavora al 63% per arrivare al pareggio che il mix migliore raggiungerebbe al 58%;
- **la risonanza va riempita per prima**: ogni minuto di risonanza vale circa 4,1 euro di margine, contro i 3,1 della radiografia e i 2,8 dell'ecografia.

Il resto dell'articolo spiega come ci siamo arrivati.

## Dalla sensazione al modello

La percezione comune era che le macchine fossero "abbastanza cariche". Ma "abbastanza" non è un dato, e senza un dato non si può decidere se aprire altre fasce orarie, rinegoziare le convenzioni o rivedere l'agenda.

Abbiamo costruito un modello che lega la **saturazione** della giornata al **numero di servizi effettuati**. Nel modello entrano gli elementi che determinano davvero il risultato:

- il tempo di ciascun tipo di esame, preparazione del paziente inclusa;
- i costi fissi del centro, che pesano a prescindere dai volumi;
- il margine per servizio, cioè il ricavo al netto dei compensi ai medici e dei consumabili;
- i vincoli di capacità: le ore di ogni macchina e i minuti di lavoro dei tecnici.

### Il centro in numeri

Il dato che rende concreta tutta l'analisi è il **margine per minuto di lavoro**. Con valori arrotondati:

| Esame | Prezzo | Minuti | Margine per esame | Margine per minuto | Minuti per euro di margine |
|---|---|---|---|---|---|
| Risonanza (RMN) | 150 € | 30 | 123 € | 4,1 € | 0,24 |
| Radiografia (RX) | 40 € | 10 | 31 € | 3,1 € | 0,32 |
| Ecografia (ECO) | 90 € | 15 | 42 € | 2,8 € | 0,35 |
| Fisioterapia | 60 € | 30 | 28 € | 0,9 € | 1,06 |

La fisioterapia ha personale e spazi propri, quindi la trattiamo a parte: il suo margine riduce i costi fissi che l'imaging deve coprire.

## Il modello

Per ogni tipo di esame *i* abbiamo descritto la giornata con pochi parametri:

- $$n_i$$: numero di esami di tipo *i* eseguiti in un giorno (la variabile da determinare);
- $$t_i$$: minuti di lavoro per esame, preparazione del paziente inclusa;
- $$m_i = p_i - c_i$$: margine di contribuzione per esame (ricavo meno costi variabili);
- $$F$$: costi fissi giornalieri che l'imaging deve coprire (costi fissi del centro meno il margine della fisioterapia);
- $$T$$: minuti di lavoro disponibili al giorno nel reparto (nel nostro caso 2 tecnici per 12 ore, cioè 1.440 minuti);
- $$d_i$$: massimo giornaliero dell'esame *i*, dato dalla capacità della macchina e da un tetto di utilizzo realistico (per la risonanza, ad esempio, l'80% delle ore disponibili).

La **saturazione** è la quota di tempo disponibile effettivamente utilizzata:

$$
u(n) = \frac{\sum_i t_i n_i}{T}
$$

Il **punto di pareggio** è il livello in cui i margini coprono i costi fissi:

$$
\sum_i m_i n_i = F
$$

## Che cosa ha fatto la ricerca operativa

Una volta scritto il modello, il problema diventa matematico. Abbiamo usato l'ottimizzazione vincolata per rispondere a due domande concrete:

1. **Il punto di pareggio.** Tra tutte le combinazioni di esami che coprono i costi, qual è quella che impegna il reparto il minor tempo possibile? Sotto quella soglia la giornata è in perdita, sopra comincia a generare margine.
2. **Il livello di saturazione del 60%.** Quanti servizi servono per raggiungere un utilizzo del 60% della capacità? Una soglia di riferimento utile per capire quanto spazio c'è ancora prima di saturare il reparto.

### Il problema di ottimizzazione

$$
\min_{n} \sum_i t_i n_i
\qquad \text{con} \qquad
\sum_i m_i n_i \ge F,
\qquad 0 \le n_i \le d_i
$$

Con un solo tipo di esame la soluzione si scrive direttamente:

$$
n_{BE} = \frac{F}{m}
\qquad
u_{BE} = \frac{t F}{m T}
\qquad
n_{60} = \frac{0{,}6 T}{t}
$$

Il primo valore è il numero di esami del pareggio, il secondo la saturazione a cui si raggiunge, il terzo il numero di esami che porta il reparto al 60%.

Con più tipi di esame il risultato cambia in modo istruttivo: per coprire i costi nel minor tempo conviene dare priorità agli esami con il minor numero di minuti per euro di margine, cioè con il rapporto $$t_i / m_i$$ più basso. La soluzione riempie la giornata in quest'ordine, fino al massimo di ciascun esame, finché i costi fissi sono coperti. Nel nostro centro l'ordine è: risonanza, radiografia, ecografia.

> **Approfondimento: la lagrangiana.** Associando un moltiplicatore $$\lambda \ge 0$$ al vincolo di pareggio e $$\mu_i \ge 0$$ ai limiti di capacità si ottiene
>
> $$
> \mathcal{L}(n, \lambda, \mu) = \sum_i t_i n_i + \lambda \left( F - \sum_i m_i n_i \right) + \sum_i \mu_i \left( n_i - d_i \right)
> $$
>
> Le condizioni di ottimo dicono che un esame eseguito solo in parte soddisfa $$t_i = \lambda m_i$$; uno che arriva al suo massimo ha $$t_i \le \lambda m_i$$; uno che non entra nel mix ha $$t_i \ge \lambda m_i$$. Ne segue che
>
> $$
> \lambda^{\star} = \frac{t_k}{m_k}
> \qquad
> \frac{dW^{\star}}{dF} = \lambda^{\star}
> $$
>
> dove *k* è l'ultimo esame che entra nel mix. Il moltiplicatore è il **prezzo ombra** del vincolo: ogni euro in più di costi fissi da coprire richiede $$\lambda^{\star}$$ minuti aggiuntivi di lavoro, nella combinazione di esami migliore.

## I risultati

Abbiamo risolto il problema per ciascun anno dello scenario prudente, con 240 giorni di apertura l'anno e la risonanza usata al massimo all'80% della sua capacità:

| Anno | Costi da coprire al giorno | Esami al pareggio (mix ottimo) | Saturazione al pareggio | Saturazione nello scenario | Esito dello scenario |
|---|---|---|---|---|---|
| 2028 | 3.390 € | 52 (19 RMN + 33 RX) | 63% | 47% | in perdita di circa 970 € al giorno |
| 2029 | 3.160 € | 45 (19 RMN + 26 RX) | 58% | 63% | pareggio |
| 2030 | 3.400 € | 53 (19 RMN + 34 RX) | 63% | 68% | pareggio |
| 2031 | 3.390 € | 52 (19 RMN + 33 RX) | 63% | 76% | oltre il pareggio, circa +440 € al giorno |

Quattro letture:

1. **La soglia è stabile.** Anche se i costi cambiano di anno in anno, il pareggio si colloca sempre intorno al 60% (tra il 58% e il 63%). Per questo il 60% funziona bene come obiettivo di riferimento.
2. **Il mix migliore è corto.** L'ecografia non serve a raggiungere il pareggio: a parità di minuti rende meno di risonanza e radiografia. Questo non significa eliminarla, perché completa l'offerta e porta pazienti, ma dice dove spingere per prima l'agenda.
3. **Il mix reale costa tempo.** Nel 2030 lo scenario pareggia al 68% di saturazione, mentre il mix ottimo lo farebbe al 63%: circa 70 minuti di lavoro al giorno in più per lo stesso risultato.
4. **Se la risonanza potesse lavorare al 100%**, il pareggio scenderebbe intorno al 55-60%. È un limite teorico, ma mostra quanto pesa il tetto di utilizzo scelto.

### Quanto pesa un euro di costi fissi

Nel mix ottimo l'ultimo esame che entra è la radiografia, quindi $$\lambda^{\star} \approx 0{,}32$$ minuti per euro. Un nuovo contratto di manutenzione da 12.000 euro l'anno, cioè circa 50 euro al giorno, richiede circa 16 minuti di lavoro in più al giorno: poco più di un punto di saturazione, o un paio di radiografie.

## Perché il tempo di lavoro conta quanto il numero di esami

Minimizzare il tempo di lavoro nel punto di pareggio significa una cosa precisa: capire **con quanto impegno delle macchine e del personale si arriva a coprire i costi**. Due giornate con lo stesso numero di esami possono avere esiti molto diversi se cambia il mix. Il modello rende questa differenza visibile e misurabile, e per questo è utile anche per valutare l'effetto di un cambio di agenda prima di applicarlo.

## I limiti del modello

- La domanda massima di ogni esame è fissata dalla capacità della macchina: nella realtà dipende dai pazienti e dalle convenzioni.
- I costi fissi sono stati trattati come una cifra giornaliera; i costi a gradini (un nuovo tecnico, più ore di amministrazione) vanno riconsiderati anno per anno.
- Se le macchine sono acquistate con un finanziamento anziché in leasing, la rata entra nei costi fissi e il pareggio di cassa si sposta più avanti.
- Il modello risponde a "quanto serve", non a "come arrivarci": la qualità del servizio e i tempi di attesa restano decisioni della direzione.

## Cosa cambia per chi decide

- Una soglia chiara sotto la quale una giornata non si sostiene, al posto di una stima a occhio: circa il 60% di saturazione.
- Un obiettivo concreto con il numero di servizi corrispondente: circa 48 esami al giorno per il 60%.
- Un criterio per riempire l'agenda: prima gli esami che rendono di più per minuto.
- La possibilità di simulare scenari: cosa succede se cambia il tempo medio di un esame, il costo di una risorsa o la tariffa.

## In sintesi

La ricerca operativa non serve a complicare le cose. Serve a trasformare una domanda gestionale ("stiamo usando bene le macchine?") in una domanda a cui si può rispondere con un numero. Il valore sta nel modello: se descrive bene il processo, l'ottimizzazione fa il resto.

Se gestite risorse costose e vincolate, come macchinari, linee o turni, e volete sapere dove si colloca il vostro punto di pareggio, possiamo costruire insieme il modello.
