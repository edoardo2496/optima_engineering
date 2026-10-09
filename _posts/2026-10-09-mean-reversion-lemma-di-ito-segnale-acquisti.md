---
layout: default
title: "Quando il prezzo del minerale di ferro suggerisce di comprare: mean-reversion, lemma di Itô e un segnale operativo per gli acquisti"
date: 2026-10-09
categories: ["Modellazione Quantitativa"]
excerpt: "I prezzi delle materie prime non si comportano come una passeggiata aleatoria: tendono a tornare verso un livello di equilibrio. Modellando il prezzo con un processo di Ornstein-Uhlenbeck e risolvendolo con il lemma di Itô si ottiene una formula esatta per il valore atteso, la varianza e uno Z-score che indica quando il prezzo corrente è statisticamente anomalo."
---

Chi compra materie prime in quantità rilevanti ha un problema strutturalmente asimmetrico: gli acquisti sono grandi e poco frequenti, ma l'esposizione al prezzo è continua. L'intuizione "compro quando costa poco" è economicamente sensata, ma operativamente vuota finché non si definisce **poco rispetto a cosa**.

Una media mobile risponde in modo approssimativo. Un modello stocastico mean-reverting risponde in modo preciso: con bande di confidenza, una velocità di ritorno calibrata sui dati e un segnale che ha un fondamento statistico invece di una soglia scelta a occhio.

Questo articolo percorre la struttura matematica del modello di Ornstein-Uhlenbeck, la sua soluzione in forma chiusa tramite il lemma di Itô, la calibrazione su vent'anni di dati mensili del minerale di ferro e la costruzione di uno Z-score utilizzabile da un ufficio acquisti.

## Perché mean-reversion e non random walk

L'ipotesi dominante nella finanza è il moto browniano geometrico (GBM): i prezzi seguono una passeggiata aleatoria con deriva, e la distanza del prezzo odierno da un livello di lungo periodo non influenza il movimento di domani. È un'ipotesi ragionevole per le azioni, dove non esiste un'àncora fondamentale che richiami il prezzo.

Le materie prime, e il minerale di ferro in particolare, obbediscono a un'economia diversa. Quando il prezzo sale, i produttori aumentano la capacità e gli acquirenti cercano sostituti. Quando crolla, i produttori marginali escono dal mercato e la domanda si riprende. Il ritorno verso un equilibrio di lungo periodo non è quindi un'ipotesi imposta ai dati: discende dai meccanismi del mercato ed è verificabile.

Il test di Dickey-Fuller aumentato (ADF) su vent'anni di dati mensili (2005-2026) rifiuta l'ipotesi di radice unitaria con $$p = 0{,}013$$. Il risultato è coerente con la stazionarietà e giustifica l'uso di un modello mean-reverting prima ancora di stimare qualsiasi parametro.

## Il modello: SDE di Ornstein-Uhlenbeck

Il processo di Ornstein-Uhlenbeck è il modello canonico a tempo continuo per le dinamiche mean-reverting. Il prezzo $$X_t$$ segue l'equazione differenziale stocastica

$$dX_t = \theta(\mu - X_t)\,dt + \sigma\,dW_t$$

dove:

- $$\theta > 0$$ è la **velocità di ritorno alla media**: quanto rapidamente il processo viene richiamato verso l'equilibrio;
- $$\mu$$ è la **media di lungo periodo**, cioè il livello di equilibrio;
- $$\sigma$$ è la **volatilità**, l'ampiezza degli shock casuali;
- $$W_t$$ è un moto browniano standard sotto la misura fisica.

Il termine di deriva $$\theta(\mu - X_t)$$ è la forza di richiamo. Se $$X_t > \mu$$ la deriva è negativa e spinge il prezzo verso il basso, se $$X_t < \mu$$ è positiva. Il parametro $$\theta$$ stabilisce quanto è energica la correzione.

## Risolvere l'SDE con il lemma di Itô

A differenza del GBM, l'SDE di Ornstein-Uhlenbeck ha una soluzione in forma chiusa. Questo permette di scrivere la distribuzione esatta di $$X_t$$ per qualsiasi istante futuro $$t$$, dato $$X_0$$.

Si introduce il cambio di variabile $$Y_t = e^{\theta t} X_t$$. Applicando il lemma di Itô a $$f(t, X_t) = e^{\theta t} X_t$$ (essendo $$f$$ lineare in $$X$$, il termine del secondo ordine è nullo):

$$dY_t = \theta e^{\theta t} X_t\,dt + e^{\theta t}\,dX_t$$

Sostituendo l'SDE per $$dX_t$$:

$$dY_t = \theta e^{\theta t} X_t\,dt + e^{\theta t}\bigl[\theta(\mu - X_t)\,dt + \sigma\,dW_t\bigr]$$

I termini in $$\theta X_t$$ si cancellano esattamente:

$$dY_t = \theta\mu\, e^{\theta t}\,dt + \sigma e^{\theta t}\,dW_t$$

Ora la deriva non dipende più dallo stato e l'equazione si integra direttamente. Integrando tra $$0$$ e $$t$$ e tornando a $$X_t = e^{-\theta t} Y_t$$:

$$X_t = X_0\, e^{-\theta t} + \mu\bigl(1 - e^{-\theta t}\bigr) + \sigma \int_0^t e^{-\theta(t-s)}\,dW_s$$

Questa è la **soluzione esatta**. È un processo gaussiano: l'integrale stocastico è normale a media nulla, quindi l'intera distribuzione di $$X_t$$ è determinata dai primi due momenti.

## Momenti in forma chiusa

Prendendo il valore atteso, l'integrale stocastico si annulla:

$$\mathbb{E}[X_t \mid X_0] = X_0\, e^{-\theta t} + \mu\bigl(1 - e^{-\theta t}\bigr)$$

Per $$t \to \infty$$ la condizione iniziale viene dimenticata a tasso esponenziale $$\theta$$ e il valore atteso converge a $$\mu$$.

La varianza, calcolata con l'isometria di Itô, è

$$\operatorname{Var}(X_t \mid X_0) = \frac{\sigma^2}{2\theta}\bigl(1 - e^{-2\theta t}\bigr)$$

Per $$t \to \infty$$ tende alla varianza stazionaria $$\sigma^2 / 2\theta$$: il processo ha una distribuzione di lungo periodo ben definita.

> Il processo di Ornstein-Uhlenbeck è, a meno di trasformazioni, l'unico processo gaussiano stazionario, markoviano e continuo. Sono queste tre proprietà a giustificarne l'uso per i prezzi delle materie prime sotto l'ipotesi di mean-reversion.

## Calibrazione dei parametri

I tre parametri $$(\theta, \mu, \sigma)$$ si stimano dalla versione discretizzata dell'SDE. Su un passo $$\Delta t$$ la distribuzione condizionata è

$$X_{t+\Delta t} \mid X_t \sim \mathcal{N}\!\left(X_t e^{-\theta\Delta t} + \mu(1 - e^{-\theta\Delta t}),\; \frac{\sigma^2}{2\theta}(1-e^{-2\theta\Delta t})\right)$$

che coincide esattamente con una regressione lineare $$X_{t+1} = a + b X_t + \varepsilon_t$$ sulla serie discretizzata. Da $$a$$, $$b$$ e dalla varianza dei residui si ricavano analiticamente $$(\theta, \mu, \sigma)$$. Sul dataset mensile di vent'anni ($$\Delta t = 1/12$$):

$$\hat{\theta} = 0{,}41,\quad \hat{\mu} = 94{,}3\ \$/\text{t},\quad \hat{\sigma} = 18{,}7$$

L'emivita è $$\ln(2)/\theta \approx 1{,}7$$ anni, circa 20 mesi: uno scostamento dall'equilibrio si dimezza in media in 20 mesi, un ordine di grandezza coerente con la durata dei cicli osservati sulle materie prime.

## Il motore Monte Carlo

Con i parametri calibrati, un motore Monte Carlo simula 5.000 traiettorie indipendenti di $$X_t$$ su un orizzonte di 36 mesi. A ogni passo il valore successivo viene estratto direttamente dalla distribuzione gaussiana condizionata esatta, senza discretizzazione di Eulero: la simulazione è quindi esatta in distribuzione qualunque sia l'ampiezza del passo.

Il risultato, per ogni data futura, è una distribuzione empirica completa dei prezzi. Non una previsione puntuale, ma un *fan chart* con bande di confidenza a livello arbitrario.

## Il segnale operativo: lo Z-score

L'output pratico è un numero. Dati il prezzo spot corrente $$X_{\text{now}}$$ e la distribuzione stazionaria $$\mathcal{N}(\mu, \sigma^2/2\theta)$$, lo Z-score è

$$Z = \frac{X_{\text{now}} - \mu}{\sigma / \sqrt{2\theta}}$$

Un valore $$Z \leq -1{,}5$$ indica un prezzo statisticamente basso rispetto all'equilibrio di lungo periodo, quindi un segnale favorevole all'acquisto. Un valore $$Z \geq +1{,}5$$ indica un prezzo statisticamente elevato, quindi un segnale per rinviare o coprirsi. La soglia non è arbitraria: corrisponde grosso modo al 7° e al 93° percentile della distribuzione stazionaria.

Il segnale non è una regola di trading. È un input strutturato per la decisione d'acquisto, che sostituisce "mi sembra che i prezzi siano bassi" con "il prezzo è 1,8 deviazioni standard sotto la media stazionaria, con un'emivita stimata di 20 mesi per il ritorno all'equilibrio".

## Limiti del modello

- **Media costante.** Il modello assume un $$\mu$$ che non evolve. In pratica, cambiamenti strutturali nella domanda cinese di acciaio o nelle politiche di decarbonizzazione possono spostare l'equilibrio in modo permanente. Un'estensione con cambi di regime (Hamilton, 1989) risolverebbe il problema, al prezzo di una calibrazione molto più complessa e di un output più difficile da comunicare.
- **Misura fisica.** Il modello opera sotto la misura fisica, non quella risk-neutral. È adatto a supportare decisioni di acquisto, non a prezzare derivati, dove valgono i vincoli di assenza di arbitraggio.
- **Code gaussiane.** L'ipotesi gaussiana implica code simmetriche, mentre il minerale di ferro mostra occasionali picchi al rialzo (shock di domanda, interruzioni portuali) che una gaussiana sottostima. Un'estensione jump-diffusion, con una componente di salto di Poisson, è il passo successivo naturale.

## Conclusioni

Il modello di Ornstein-Uhlenbeck offre ciò che pochi strumenti di previsione offrono: una caratterizzazione *analiticamente esatta* dell'incertezza futura, fondata su un'ipotesi strutturale verificabile sul processo che genera i dati. La soluzione in forma chiusa tramite il lemma di Itô non è solo eleganza matematica: rende il modello più veloce, più interpretabile e più verificabile di un'alternativa a scatola nera.

Lo Z-score che ne deriva è un solo numero, aggiornabile ogni giorno, che dice a chi decide un acquisto una cosa precisa: *dove si colloca il prezzo corrente nella distribuzione dei prezzi che il processo è destinato a visitare*.

[Dashboard dimostrativa →](https://edoardo2496.github.io/data_science_portfolio/stochastic_iro_ore_model/index.html)
