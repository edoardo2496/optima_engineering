---
layout: post
title: "Turnover: oltre le exit interview, la probabilità di dimissioni"
categoria: Data science
---

Il turnover è uno di quei fenomeni che ogni azienda conosce e quasi nessuna riesce a descrivere con precisione. Un cliente ci ha contattato con una domanda semplice: *il turnover sta davvero crescendo, e che cosa lo spiega?* Per ragioni di riservatezza non possiamo mostrare i suoi dati. In questo articolo raccontiamo il metodo su un **gemello sintetico**: un dataset di dipendenti che abbiamo generato noi, con la stessa struttura di quello reale e con un meccanismo di uscita noto. Tutti i numeri qui sotto vengono da quel dataset e non dicono nulla sull'azienda del cliente. Servono a mostrare come ragioniamo.

## Il punto di partenza: un cliente già maturo

L'HR Director aveva già fatto molto di ciò che si dovrebbe fare. Aveva istituito **one-to-one periodici** tra responsabili di prima linea e collaboratori, per costruire una vera cultura del feedback e per evitare che le persone si sentissero un numero. Aveva introdotto le **exit interview**, per raccogliere a posteriori le ragioni di chi se ne va.

Sono due strumenti di grande valore e non ne mettiamo in discussione l'utilità. Hanno però un limite strutturale: raccontano bene *perché una persona dice di essersene andata*, ma non dicono quanto pesa ciascun fattore, né chi rischia di andarsene la prossima volta. La nostra proposta è stata di affiancare a questo lavoro qualitativo uno sguardo quantitativo.

## Passo 1: il fenomeno sta crescendo?

L'impressione dell'HR Director era che il turnover fosse in aumento. Prima di spiegarlo, abbiamo verificato se il fenomeno esistesse davvero.

Nel dataset sintetico, 1.000 dipendenti nel 2018 diventano 1.187 nel 2025. Abbiamo definito il turnover annuo come uscite nell'anno divise per organico a inizio anno.

![Turnover annuo 2018-2025](/optima_engineering/assets/turnover/turnover_trend.png)

Il tasso passa dall'**11,7% del 2018 al 24,7% del 2025**, con una crescita media di circa 1,9 punti percentuali l'anno. L'impressione era corretta: il turnover è più che raddoppiato in otto anni, e questo giustifica uno studio più approfondito.

## Passo 2: retribuzione e anzianità

Una prima ipotesi naturale è che chi resta più a lungo venga pagato di più. Abbiamo messo in relazione retribuzione mensile e anni in azienda.

![Retribuzione e anzianità](/optima_engineering/assets/turnover/income_tenure.png)

La relazione c'è ed è positiva (correlazione di Spearman 0,39). In un modello sul logaritmo della retribuzione, ogni anno di anzianità vale circa **+1,7%**. Ma l'anzianità da sola spiega solo il 17% della variabilità: quando aggiungiamo il livello di inquadramento, la quota spiegata sale all'83% e l'effetto dell'anzianità resta di circa +1,75% l'anno. In sostanza, **il livello fa la retribuzione, l'anzianità la sposta appena**. Un dipendente che resta in azienda senza cambiare livello vede crescere lo stipendio lentamente, e questo ci ha suggerito di guardare non la retribuzione assoluta ma quella *rispetto ai colleghi dello stesso livello*.

## Passo 3: fotografare il fenomeno

Prima di modellare abbiamo descritto chi se ne va, con le grandezze più semplici. Per questa fotografia abbiamo considerato le uscite del 2025 (293 su 1.187 dipendenti).

![Statistica descrittiva](/optima_engineering/assets/turnover/descrittiva.png)

Alcune osservazioni:

- **Titolo di studio.** Non c'è un andamento netto: l'uscita è più alta con licenza media e laurea triennale (29% in entrambi i casi) e più bassa tra i diplomati (19%). Non è vero, qui, che "si dimettono più i laureati" o più i meno istruiti.
- **Genere.** Le donne escono un po' di più (26,9% contro 23,2%), ma la differenza non è statisticamente solida (p = 0,17).
- **Straordinario.** Tra chi fa straordinario regolare esce il **40,7%**, contro il 21,1% di chi non ne fa.
- **Anzianità.** Il rischio è concentrato nei primi due anni: 43,7% tra 0 e 1 anno, 10% da 8 anni in su.
- **Soddisfazione.** Chi dichiara il punteggio più basso (1 su 4) esce nel 48,7% dei casi, chi dichiara il più alto nel 17,8%.

La statistica descrittiva inquadra il fenomeno, ma ha un difetto: guarda una variabile alla volta. Straordinario, età e anzianità sono intrecciati tra loro, e non sappiamo quale effetto resti quando li teniamo fermi. Per questo serve un modello.

## Passo 4: la regressione logistica

Abbiamo scelto la **regressione logistica** per due ragioni. La prima è l'interpretabilità: ogni coefficiente si traduce in un *odds ratio*, cioè in quanto cambiano le probabilità di uscita al variare di un fattore, a parità di tutti gli altri. È un linguaggio che un HR Director può discutere con la direzione. La seconda è che produce una probabilità di dimissioni per ogni persona, non un'etichetta sì/no. (Anche altri modelli, come la random forest, possono restituire probabilità: la differenza sta soprattutto nella leggibilità dei coefficienti e, spesso, nella buona calibrazione della logistica.)

Ogni dipendente è descritto da un vettore di nove variabili: età, genere, titolo di studio, distanza casa-lavoro, straordinario, soddisfazione, retribuzione relativa ai pari livello, anni dall'ultima promozione, anzianità. Il modello è stato stimato su 830 dipendenti (training) e valutato su 357 mai visti (test).

![Odds ratio](/optima_engineering/assets/turnover/odds_ratio.png)

I risultati in unità naturali, stimati sul training set:

| Fattore | Odds ratio | Intervallo di confidenza 95% | Lettura |
|---|---|---|---|
| Straordinario regolare (sì vs no) | 2,74 | 1,81 – 4,16 | le probabilità di uscita quasi triplicano |
| Soddisfazione (per punto) | 0,64 | 0,53 – 0,76 | ogni punto in più riduce le probabilità di circa un terzo |
| Anzianità (per anno) | 0,92 | 0,87 – 0,99 | il rischio cala con gli anni |
| Età (per anno) | 0,97 | 0,94 – 0,99 | i più giovani escono di più |
| Retribuzione vs pari livello (per +10%) | 0,84 | 0,72 – 0,99 | stare sotto la mediana del proprio livello pesa |
| Anni dall'ultima promozione (per anno) | 1,05 | 0,95 – 1,17 | non conclusivo |
| Distanza casa-lavoro (per km) | 1,01 | 0,99 – 1,03 | non conclusivo |
| Genere (uomo vs donna) | 0,72 | 0,50 – 1,02 | non conclusivo |
| Titolo di studio (per livello) | 1,01 | 0,86 – 1,19 | nessun effetto |

Tre cose ci sembrano degne di nota. Lo **straordinario** è il fattore dominante, e resta tale anche tenendo ferme età e anzianità. Il **titolo di studio**, che nella descrittiva sembrava dire qualcosa, a parità di tutto il resto non conta. La **retribuzione assoluta** non è nel modello per scelta: ciò che pesa è la posizione rispetto ai colleghi dello stesso livello, ed è un'informazione diversa e più utile.

Un modo concreto per leggere il modello è guardare tre profili:

- un dipendente "medio" (37 anni, 6 di anzianità, soddisfazione 3, retribuzione in linea con i pari, senza straordinario) ha una probabilità di dimissioni del **15%**;
- lo stesso dipendente con straordinario regolare sale al **33%**;
- un profilo giovane, con un anno di anzianità, straordinario, soddisfazione al minimo, retribuzione del 10% sotto i pari livello, 4 anni dall'ultima promozione e 30 km di tragitto arriva a **81%**. All'estremo opposto, un profilo di 45 anni con 9 di anzianità, soddisfazione massima e retribuzione del 10% sopra i pari scende al **5%**.

## Quanto possiamo fidarci?

Sul test set l'**AUC è 0,72** (0,77 sul training): il modello distingue nettamente meglio del caso, ma non è un oracolo. Il Brier score è 0,163 contro 0,186 di un modello che assegna a tutti la stessa probabilità. Prendendo il 20% di persone con probabilità prevista più alta, intercettiamo il **42% delle uscite** reali, con un tasso di uscita osservato del 52% in quel gruppo.

![ROC e calibrazione](/optima_engineering/assets/turnover/roc_calibrazione.png)

Il modello non va usato per etichettare le persone, ma per capire dove concentrare l'attenzione. Nel nostro caso diremmo al cliente che i one-to-one vanno intensificati prima di tutto dove si fa straordinario sistematico e nei primi due anni di permanenza. Le exit interview, a quel punto, possono verificare se i fattori emersi dal modello coincidono con le ragioni dichiarate.

## Limiti e sviluppi

- I dati sono sintetici. Il modello ritrova il meccanismo che abbiamo inserito, e questo dimostra che il metodo funziona, non che nel mondo reale i fattori siano questi.
- Una regressione logistica descrive associazioni, non cause. Lo straordinario può essere una causa di uscita, ma anche il sintomo di un reparto in sofferenza.
- Abbiamo lavorato su un solo anno di osservazione. Il passo successivo naturale è un'**analisi di sopravvivenza**, che tiene conto del *quando* le persone se ne vanno oltre che del *se*.
- Un modello va rifatto e ricalibrato: il tasso di turnover cambia nel tempo, e con esso le probabilità di base.

Il dataset sintetico e gli script che lo generano sono disponibili su richiesta, per chi volesse ripetere l'analisi.
