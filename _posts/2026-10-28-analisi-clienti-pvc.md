---
type: post
title: "Quanto vale davvero un cliente? Dalla classificazione per fatturato al clustering con k-means"
categoria: Data Science
---
# Quanto vale davvero un cliente?

*Dalla classificazione per fatturato al clustering con k-means: un caso studio per un produttore di tubi in PVC.*

> **Nota metodologica.** I dati di questo articolo sono **sintetici**: li abbiamo generati noi, con struttura nota, per poter mostrare il metodo che abbiamo portato al cliente senza esporre le sue informazioni commerciali. I numeri sono quindi illustrativi; ciò che conta è la soluzione ed il ragionamento che abbiamo voluto proporre

---

## 1. Il problema

Un'azienda che produce tubi in PVC ci ha chiesto di aiutarli ad analizzare la **qualità del proprio portafoglio clienti**. Fino a quel momento il cliente segmentava con criteri qualitativi o con una variabile quantitativa "debole": **il fatturato generato**. 
Dal "modus operandi" del cliente emergeva solamente un'analisi un po' piatta: i clienti *grandi* erano "buoni", quelli piccoli "meno importanti".

È una scelta comprensibile, ma implicita in essa c'è un'ipotesi mai verificata: *che il fatturato sia un buon proxy del valore del cliente*. Noi abbiamo preferito non assumerlo, ma **misurarlo** per poi trarre le dovute conclusioni.

Il percorso che abbiamo impostato per raggiungere l'obiettivo si divide in tre passi:

1. **Gini** → il portafoglio è concentrato? L'indice Gini con la curva di Lorenz ci ha aiutato a capirlo
2. **Pareto** → con quanti clienti si costruisce la maggior parte del fatturato?
3. **K-means** → esistono *tipi* di cliente diversi, che il fatturato da solo non distingue?

## 2. Il dataset sintetico

Al fine di poter raccontare in questo articolo il lavoro svolto abbiamo generato un dataset sintetico che però avesse le medesime caratteristiche di quello su cui abbiamo effettivamente lavorato.
400 clienti, circa 20,8 M€ di fatturato annuo complessivo. Per ogni cliente, 7 variabili (queste ultime sono state individuate dopo un'EDA inziiale sui dati):

| Variabile | Significato |
|---|---|
| `fatturato` | Fatturato annuo (€) |
| `margine_pct` | Margine lordo percentuale sul venduto |
| `giorni_incasso` | Giorni medi di incasso (DSO) |
| `tasso_reclami` | Quota di fatturato oggetto di reclamo/reso |
| `ordini_anno` | Numero di ordini all'anno |
| `cv_ordini` | Variabilità degli ordini (coefficiente di variazione) |
| `quota_prodotti_alto_margine` | Peso dei prodotti ad alto margine nel mix |

Per valutare la "vera" redditività abbiamo definito un **margine netto** per cliente:

$$
M_{netto} = \underbrace{F \cdot m}_{\text{margine lordo}} - \underbrace{F \cdot r \cdot \frac{DSO}{365}}_{\text{costo del credito}} - \underbrace{1.5 \cdot F \cdot \rho}_{\text{costo dei reclami}}
$$

con $$F$$ fatturato, $$m$$ margine lordo %, $$r$$ = 8% costo annuo del capitale, $$\rho$$ tasso di reclami. (Il fattore 1,5 riflette il fatto che un reclamo costa più del suo valore nominale: reso, logistica, assistenza.)

Il dataset è stato costruito con quattro profili di comportamento *nascosti*. **L'analisi non li conosce**: servono solo, alla fine, per verificare se il metodo li ritrova. In breve abbiamo voluto verificare se l'analisi che abbiamo condotto fosse veramente in gardo di "scovare" questi comportamenti d'acquisto dei clienti sulla base dei quali si realizzasse la segmentazione.

---

## 3. Passo 1 – Il portafoglio è concentrato? L'indice di Gini

Ordinando i clienti dal più piccolo al più grande, la **curva di Lorenz** mostra quale quota di fatturato è generata da una data quota di clienti. L'indice di Gini è l'area tra la curva e la diagonale di equidistribuzione (normalizzata):

$$
G = \frac{n + 1 - 2 \cdot \frac{\sum_{i=1}^{n} (n+1-i) x_{(i)}}{\sum_{i=1}^{n} x_{(i)}}}{n}
$$

![Curva di Lorenz del fatturato](https://edoardo2496.github.io/optima_engineering/assets/01_lorenz.png)

**Risultato: G ≈ 0,67.** Un valore alto: il portafoglio è fortemente concentrato. C'è una vera *customer concentration*, con il rischio che comporta (dipendenza da pochi clienti, potere contrattuale sbilanciato).

Fin qui, il quadro conferma l'intuizione del cliente. Ma il Gini ci dice **quanto** è concentrato il fatturato, non **chi** vale. In poche parole con l'indice di Gini abbiamo messo in luce, in maniera razionale, una forte dipendenza commerciale da pochi "grossi" cliente, magari a volte poco profittevoli.

## 4. Passo 2 – Analisi di Pareto

Quanti clienti servono per costruire una data quota di fatturato?

![Curva di Pareto](https://edoardo2496.github.io/optima_engineering/assets/02_pareto.png)

| Quota di fatturato | Clienti necessari | % dei clienti |
|---|---|---|
| 20% | **10** | 2,5% |
| 50% | 32 | 8% |
| 80% | 108 | 27% |

Il 20% *dei clienti* (80 su 400) genera circa il **75%** del fatturato: una distribuzione tipica "alla Pareto", e persino più estrema del classico 80/20 sul vertice (dieci clienti valgono da soli un quinto del business).

A questo punto la tentazione è naturale: *concentriamoci su quei clienti*. È esattamente ciò che fa la classificazione ABC per fatturato del cliente:

- **Classe A**: clienti che costruiscono l'80% del fatturato → 108 clienti
- **Classe B**: il successivo 15% → 163 clienti
- **Classe C**: l'ultimo 5% → 129 clienti

Questa "segmentazione ABC", tipica quando si utilizza Pareto, è cieca infatti di fronte a tutta la profondità che invece ha approccio multi-fattoriale. Il fatturato, preso da solo, non fa altro che distribuire i clienti lungo una "retta ideale", che "ranka" i clienti solamente per una variabile di per sé muta.

## 5. Mettiamo alla prova la classificazione per fatturato

Con questa analisi abbiamo voluto "in primis" mettere alla prova la classica classificazione per fatturato.
Guardiamo infatti cosa succede ai **margini** dentro le classi ABC (quelle che solitamente su usano per fare segmentazione basandosi sui "cut-off" di Pareto):

| Classe | Clienti | % Fatturato | Margine netto / fatturato | Giorni di incasso medi |
|---|---|---|---|---|
| A | 108 | 80,0% | 14,9% | 48 |
| B | 163 | 15,0% | **21,4%** | 43 |
| C | 129 | 5,0% | 13,3% | 54 |

La classe B, che il cliente considerava "di secondo piano", ha una **redditività unitaria superiore alla classe A**. Ma il dato più rivelatore è un altro:

> **Tutti i 17 clienti a margine netto negativo si trovano nella classe A.**

![Fatturato vs margine netto per classe ABC](https://edoardo2496.github.io/optima_engineering/assets/03_abc_vs_margine.png)

Nel grafico (scala logaritmica sull'asse x usata per facilitare il confronto tra clienti con fattrati di ordini differenti) si vede il problema. 
**La correlazione tra (log-)fatturato e margine netto percentuale è praticamente nulla (≈ 0,06)**. Sapere quanto fattura un cliente non dice quasi nulla su quanto *guadagna* l'azienda con lui. Una sola dimensione, il fatturato, sta togliendo capacità di analisi portando tutto su un piano "asettico".

Quello che volevamo evidenziare era sopratutto che la classificazione ABC non fosse in grado di restituire una "vera" redditività dei clienti.
Seguendo il modello ABC, infatti, i clienti che avremmo dovuto valutare come *top-customers*, con fatturati alti quindi, erano in realtà quelli che, nel dataset del nostro cliente, presentavano una reddititvità più bassa. 

## 6. Passo 3 – L'approccio ingegneristico: clustering con k-means

### 6.1 Ogni cliente è un vettore

Il cambio di prospettiva è questo: un cliente non è un numero (il fatturato), ma un **vettore** in $$\mathbb{R}^7$$:

$$
\mathbf{x}_i = (\log F_i, m_i, DSO_i, \rho_i, \log N_i, cv_i, q_i) \in \mathbb{R}^7
$$

Le variabili hanno unità e ordini di grandezza diversi (euro, giorni, percentuali), quindi:

- applichiamo il **logaritmo** a fatturato e numero di ordini (distribuzioni molto asimmetriche);
- **standardizziamo** ogni colonna (media 0, deviazione standard 1), così che nessuna dimensione domini le distanze solo per la sua scala.
In pratica normalizziamo i dati affinchè il nostro modello di "Machine Learning" non abbia allucinazioni.

Otteniamo la matrice $$Z \in \mathbb{R}^{400 \times 7}$$: ogni riga è un cliente, ogni colonna una caratteristica.
Tale matrice non è altro che il dataset di partenza con i dati normalizzati.

### 6.2 K-means come problema di ottimizzazione

K-means cerca $$k$$ gruppi $$C_1, \dots, C_k$$ con centroidi $$\boldsymbol{\mu}_j$$ che minimizzino la varianza interna ai gruppi:

$$
\min_{C_1,\dots,C_k} \sum_{j=1}^{k} \sum_{\mathbf{z}_i \in C_j} \Vert \mathbf{z}_i - \boldsymbol{\mu}_j \Vert^2
$$

Il metodo procede per iterazioni: assegna ogni cliente al centroide più vicino, ricalcola i centroidi come media dei punti assegnati, e ripete fino a convergenza. È geometria euclidea pura: ciò che conta è la **distanza tra vettori**. In altre parole il centroide è il cuore geometrico del gruppo. Vettori la cui distanza è simile da un centroide possono essere *classificati" come simili e quindi appartenenti alla stessa categoria, allo stesso segmento.

### 6.3 Quanti cluster?

![Scelta di k](https://edoardo2496.github.io/optima_engineering/assets/04_scelta_k.png)

Il gomito dell'inerzia e la silhouette (i metodi più diffusi per rispondere alla domanda circa quanti siano i gruppi) indicano la zona **k = 3–4**. 
La silhouette massima è per k = 3 (0,63), ma k = 4 resta ottima (0,56) e crolla solo da k = 5 in poi (0,32). 
In pratica questi due metodi ci dicono l'affidabilità che otteniamo utilizzando un determinato numero di gruppi.
Abbiamo scelto **k = 4**: la differenza di silhouette è piccola e k = 4 separa un gruppo (i clienti strategici da quelli "piccoli e fedeli") che sul piano del business ha politiche commerciali molto diverse.

> *Onestà intellettuale*: la scelta di k non è mai "automatica". Gli indicatori restringono il campo, il dominio decide. Con k = 3 le conclusioni qualitative non cambierebbero. Abbiamo optato per 4 gruppi in quanto la differenza di significatività tra 3 e 4 era trascurabile.

### 6.4 Visualizzare 7 dimensioni: PCA via decomposizione ai valori singolari

Per *vedere* i cluster proiettiamo $$Z$$ su due dimensioni con la **PCA**, ottenuta dalla **decomposizione ai valori singolari (SVD)**:

$$
Z = U \Sigma V^{T}
$$

Le colonne di $$V$$ sono le direzioni principali; i valori singolari $$\sigma_i$$ misurano quanta varianza cattura ciascuna. Qui le prime due componenti spiegano **~87%** della varianza totale (PC1: 65%, PC2: 21%).

Interpretiamo le componenti leggendo i *loadings*:

- **PC1** (65%) è un asse di **"qualità commerciale"**: pesa positivamente margine, mix ad alto margine, regolarità degli ordini; negativamente giorni di incasso, reclami, variabilità degli ordini.
- **PC2** (21%) è soprattutto un asse di **dimensione** (fatturato, volume di ordini, con segno negativo).

Il fatturato, da solo, vive su un asse *secondario* e ortogonale alla qualità del cliente: è la dimostrazione geometrica di ciò che la correlazione ≈ 0 ci aveva già suggerito.

![Cluster k-means vs classi ABC nello spazio PCA](https://edoardo2496.github.io/optima_engineering/assets/05_pca_cluster_vs_abc.png)

A sinistra, i cluster di k-means formano **quattro nuvole nettamente separate**. A destra, **gli stessi identici clienti** colorati secondo le classi ABC (quelle derivate dall'approccio di Pareto): le classi si sovrappongono e si mescolano, tagliando le nuvole "a fette" lungo una sola direzione.

## 7. Chi sono i quattro cluster

| Cluster | Clienti | % Fatturato | Fatturato medio | Margine lordo % | Giorni incasso | Reclami | Margine netto / fatturato | % del margine netto totale |
|---|---|---|---|---|---|---|---|---|
| **Strategico** | 26 | 37,0% | ~296 k€ | 27,8% | 37 | 0,8% | **25,4%** | **59,6%** |
| **Grande ma non profittevole** | 30 | 32,6% | ~226 k€ | 11,0% | 78 | 6,7% | **−0,6%** | **−1,3%** |
| **Piccolo fedele** | 139 | 19,1% | ~29 k€ | 29,6% | 33 | 1,3% | **26,9%** | 32,5% |
| **Occasionale** | 205 | 11,4% | ~12 k€ | 18,6% | 55 | 3,0% | 12,8% | 9,2% |

Le implicazioni sono molto concrete:

- **Il cluster "Grande ma non profittevole"** vale il **33% del fatturato** e quasi **zero margine netto** (anzi, leggermente negativo): ordini grandi, ma con margini compressi (11%), DSO lunghi (incassi a 78 giorni) e reclami al 6,7%. Nel sistema ABC sono tutti in classe A, cioè trattati come i migliori clienti, quando in realtà sono solamente i "top spender" e non i "top customer".
- **I "Piccoli fedeli"** hanno un fatturato medio dieci volte inferiore ai grandi, ma una redditività netta del 27%, la più alta del portafoglio, e generano un terzo del margine totale. Oltre il 60% finisce in classe B e solo il 35% in classe A: il fatturato non li riconosce. L'analisi ABC li individuerebbe come quelli che apportano un contributo marginalmente piccolo al fatturato complessivo, non mettendo in luce la tridimensionalità di un aspetto strategico come la redditività di quel fatturato.
- **Gli "Strategici"** sono solo 26 clienti, ma producono **quasi il 60% del margine netto**. Questo è il nucleo da proteggere davvero.

### Il confronto che conta

Quanto bene ciascuna segmentazione ricostruisce i profili di comportamento reali? Usiamo l'*Adjusted Rand Index* (ARI: 1 = coincidenza perfetta, 0 = casuale) e la silhouette (coesione dei gruppi nello spazio a 7 dimensioni):

| Metodo | ARI vs profili reali | Silhouette |
|---|---|---|
| Classi ABC per fatturato | 0,30 | 0,10 |
| Quartili di fatturato | 0,27 | – |
| **K-means (k = 4)** | **≈ 1,00** | **0,56** |

Segmentare per fatturato significa tracciare confini arbitrari in un'unica dimensione. Banalmente è come prendere la retta y=x e divere il piano cartesiano in due metà, classificando come "ottimi" i valori di y positivi e "cattivi" quelli inferiori a zero.
k-means trova i confini che *esistono davvero nella geometria dei dati*, valutando appunto il cliente come un vettore (in questo caso a 7 componenti) e non come un punto (monodimensionale). L'approccio quantitativo del cliente, segmentare per fatturato (quindi usare il clustering ABC), falliva invece quindi proprio in questo. Trattava i clienti come punti e li disponeva lungo una retta, decidendo buoni e cattivi sulla base di una sola dimensione.

---

## 8. Validazione: i cluster sono reali o sono un artefatto?

Un risultato di clustering è convincente solo se **sopravvive a un cambio di campione, di inizializzazione e di modello**. Abbiamo eseguito tre verifiche.

### 8.1 Stabilità via bootstrap

Abbiamo estratto 200 clienti casuali con reinserimento sul bacino originale di 400 clienti, rilanciato k-means su ciascuno e confrontato la partizione ottenuta con quella originale tramite ARI. In pratica siamo partiti da una così detta classificazione a priori (o analitica) e l'abbiamo confrontata con quella che, a posteriori, emergeva su campioni random. L'idea era quella di vedere, rilanciando k-means sui clienti casualmente estratti, se la classificazione a posteriori combaciasse con quella a priori.

![Stabilità bootstrap e rimozione di una variabile alla volta](https://edoardo2496.github.io/optima_engineering/assets/06_bootstrap_lofo.png)

- **ARI medio = 0,99** con k = 4 (in quasi tutti i ricampionamenti la partizione è identica a quella originale; nel caso peggiore l'ARI scende a 0,62).
- La stabilità *per singolo cluster*, misurata con l'indice di Jaccard medio, è altissima: 1,00 per i "Grandi non prodittevoli", 0,99 per "Occasionali" e "Piccoli fedeli", **0,98 per gli "Strategici"**, che sono il gruppo più piccolo (26 clienti) e quindi il più sensibile al ricampionamento.
- Con k = 3 la partizione è addirittura invariante (ARI = 1,00 in tutti i ricampionamenti): i due modelli sono entrambi stabili, quindi la scelta tra 3 e 4 resta una scelta di *business* più che statistica.

**Nota metodologica sull'inizializzazione.** Lanciando k-means **una sola volta** con centroidi casuali, solo il 63% delle esecuzioni (su 100 seed) converge alla soluzione corretta; le altre si bloccano in minimi locali (ARI fino a 0,61). Il risultato dell'articolo usa 50 inizializzazioni e tiene la migliore: non è un dettaglio tecnico, è ciò che rende affidabile l'algoritmo.

### 8.2 Quanto conta ciascuna variabile? Rimozione di una variabile alla volta

Rilanciando k-means dopo aver tolto una variabile per volta, la partizione resta praticamente identica (ARI ≥ 0,996) per **sei variabili su sette**. Significa che molte dimensioni sono ridondanti e portano la stessa informazione di "qualità del cliente" (margine, incassi, reclami, mix sono fortemente correlati tra loro: è anche ciò che emerge da PC1).

L'unica eccezione è istruttiva: **rimuovendo il fatturato l'ARI scende a 0,62**. Il fatturato, quindi, non è inutile: serve per distinguere gli "Strategici" dai "Piccoli fedeli", che hanno qualità simile ma dimensione molto diversa. Il punto della nostro lavoro, quindi, non è "il fatturato non conta", bensì:

> **il fatturato è necessario ma non sufficiente**: da solo ordina i clienti lungo un'unica retta, ma l'informazione utile sta nella combinazione con le altre sei dimensioni.

### 8.3 Cambiare modello: Gaussian Mixture

Come controllo indipendente abbiamo adattato un **Gaussian Mixture Model**, che a differenza di k-means ammette cluster ellittici di dimensione diversa e assegna a ogni cliente una **probabilità** di appartenenza anziché una distanza dall'orgine.
Per fare un'analogia geografica, **K-means** è come assegnare ogni abitante al municipio più vicino in linea d’aria: i confini sono netti e, implicitamente, ogni comune è un cerchio di dimensione simile. Un **Gaussian Mixture** ammette invece comuni di forma e dimensione diverse (uno compatto, uno allungato lungo una valle) e, per chi vive vicino al confine, non obbliga a scegliere: dice che quella persona appartiene al comune A al 70% e al comune B al 30%. Per i clienti vale lo stesso: quelli “di confine”, che stanno migrando da un tipo all’altro, diventano visibili.

![BIC al variare del numero di componenti](https://edoardo2496.github.io/optima_engineering/assets/07_gmm_bic.png)

- Con covarianza completa o diagonale il **criterio BIC è minimo a k = 4**. Il vantaggio su k = 3 è netto con covarianza diagonale (BIC 3.703 contro 3.916) ma **modesto con covarianza completa** (4.108 contro 4.115): in quel caso la preferenza per k = 4 è solo debole.
- Con covarianza sferica, cioè l'assunzione implicita di k-means, il BIC indicherebbe k = 6: un segnale che le nuvole reali non sono perfettamente sferiche e che k-means va usato con consapevolezza.
- Il GMM a 4 componenti ritrova **gli stessi gruppi di k-means** (ARI tra i due modelli = 0,996) e **ricostruisce esattamente i profili sottostanti** (ARI = 1,00).
- **Nessun cliente è ambiguo**: la probabilità di appartenenza al cluster assegnato è sempre superiore al 99,99%.

**Come leggere questi risultati.** Probabilità e stabilità così alte dipendono dal fatto che il dataset è sintetico e costruito con gruppi ben distanti. Sui dati reali del nostro cliente abbiamo ottenuto ottimi risultati ma i clienti erano molto più "di confine", con probabilità intermedie, ed è proprio lì che il GMM aggiunge valore rispetto a k-means: permette di segnalare i clienti che stanno migrando da un tipo all'altro. La **procedura di validazione**, però, è quella che abbiamo eseguito sui dati veri del cliente.

---

## 9. Cosa cambia per il cliente

La segmentazione per fatturato, che il cliente aveva impostato, ha prodotto una **decisione sbagliata in modo sistematico**: trattare come "premium" un gruppo di clienti che non genera margine, e trascurare quelli più redditizi per unità di fatturato. Con la segmentazione per cluster, le politiche invece si differenziano:

| Cluster | Azione suggerita |
|---|---|
| **Strategico** | Protezione e fidelizzazione: contratti pluriennali, priorità di servizio, accordi di fornitura. |
| **Grande non profittevole** | Rinegoziazione: listini, scontistica, condizioni di pagamento (target: da 78 a ~45 giorni), penali o premi legati ai reclami. Valutare il distacco se il margine non migliora. |
| **Piccolo fedele** | Crescita: cross-selling sul mix ad alto margine, servizio efficiente a basso costo (e-commerce, riordini programmati). |
| **Occasionale** | Servizio a basso costo, ordini minimi, prezzi di listino; nessun investimento commerciale dedicato. |

## 10. Limiti e prossimi passi

- **I dati sono sintetici e costruiti con cluster ben separati.** Con dati reali del cliente i gruppo si sono rivelati tipicamente più sfumati, e un ARI vicino a 1 sarebbe sospetto (quello realmente ottenuto nel dataset del nostro cliente era infatti ben più basso).
Il valore dimostrativo sta nel *confronto* tra i metodi, non nel valore assoluto degli indici. Questo articolo vuole mostrare la "ratio" applicata al problema che il nostro cliente ci ha portato all'attenzione.
- **K-means assume cluster "sferici" e di dimensione simile.** Abbiamo verificato la robustezza con bootstrap e Gaussian Mixture (sezione 8); con dati reali abbiamo aggiunto anche il clustering gerarchico ed abbiamo ripetuto i test, perché i gruppi erano ovviamente meno separati.
- **Il margine netto è un modello.** I coefficienti (costo del capitale, moltiplicatore dei reclami) vanno calibrati sui costi aziendali reali.
- **La fotografia è statica.** Il passo successivo è seguire i clienti nel tempo: migrazioni tra cluster, early-warning sul deterioramento dei pagamenti, stima del *Customer Lifetime Value*.

## 11. Conclusione

Il fatturato è una variabile facile da misurare e facile da comunicare, che di primo acchito può dare l'idea di un approccio quantitativo. Questa metrica nasconde però un limite enorme: **ordina i clienti lungo un asse che, nei nostri dati, è quasi indipendente dal loro valore economico**. Il fatturato ha infatti il grande limite di trattare ogni cliente come un punto nello spazio delle sue caratteristiche. Si deve quindi ricorrere ad un po' di algebra lineare e geometria analitica per rivelare la struttura più profonda del comportamento d'acquisto dei clienti.
Questo approccio è ciò che distingue un'analisi quantitativa da una classificazione per abitudine.
