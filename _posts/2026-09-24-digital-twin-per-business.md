---
layout: post
title: "Dimensionare un centro fitness con la ricerca operativa: break-even, churn e un gemello matematico del business"
categoria: Ricerca operativa
---

Un operatore del settore fitness & Wellness ci ha contattati per affiancarlo nell'apertura di un nuovo centro in una zona strategica. La domanda iniziale con cui il cliente si è presentato da noi era quella di ogni investimento: *conviene?*  Quella che invece è emersa lavorando insieme è stata più interessante: *quanti abbonati, di quale tipo e in quali spazi servono perché il centro regga, e che cosa succede se le ipotesi cambiano?*

In questo articolo ripercorriamo il percorso: dallo studio di fattibilità economica alla costruzione di un modello che legasse abbonamenti, spazi e conto economico in un unico sistema coerente.

## 1. Il punto di partenza: lo studio di fattibilità

Il primo passo è stato lo studio di fattibilità economica. Abbiamo costruito per il nostro al cliente un piano degli investimenti (macchinari e allestimento), una struttura dei costi, un conto economico previsionale su tre esercizi ed un modello di cahsflow e fonti di finanziamento.

L'offerta, individuata dal cliente, si articolava su due formule di abbonamento:

- **Smart**, la formula base, a 49 € al mese;
- **Premium**, con servizi aggiuntivi, a 69 € al mese.

A queste si somma una terza fonte di ricavo, meno scontata: l'**affitto degli spazi ai personal trainer** indipendenti, che pagano un canone mensile fisso per operare nella struttura.

Uno studio di fattibilità tradizionale si ferma qui. Si ipotizza un numero di iscritti, si compila il conto economico e si guarda il risultato. Il limite di questo approccio è che ogni singolo numero presente in conto economico è un'ipotesi *inserita a mano*. Noi volevamo che fosse un *risultato di un'equazione matematica*.

## 2. Il problema: dimensionare abbonamenti e spazi

Due domande, strettamente collegate:

1. **Qual è il numero minimo di abbonamenti, tra Smart e Premium, che porta il centro al pareggio?**
2. **Quanti iscritti può ospitare la struttura prima che la saturazione degli spazi degradi il servizio?**

La prima fissa un limite inferiore, la seconda un limite superiore. Il margine di manovra del business sta in mezzo, ed è lì che serve un modello.

## 3. Il break-even come equazione

Indichiamo con $$S$$ e $$P$$ il numero di iscritti Smart e Premium, con $$p_S$$ e $$p_P$$ le tariffe mensili, con $$n_T$$ il numero di trainer e con $$c_T$$ il canone mensile che ciascuno paga. I ricavi annui sono:

$$
R(S,P) = 12\,(p_S\,S + p_P\,P) + q\,(S_{nuovi} + P_{nuovi}) + 12\,n_T\,c_T
$$

dove $$q$$ è la quota di iscrizione pagata dai nuovi iscritti. I costi si dividono in una componente variabile, proporzionale ai ricavi (le commissioni sugli incassi elettronici, con aliquota $$\tau$$) più il costo dei trainer $$C_T$$, e in una componente fissa $$F$$ (segreteria, affitto, utenze, manutenzione, pulizie, consulenze, assicurazione, gestionale):

$$
C(S,P) = \tau\,R(S,P) + C_T + F
$$

Il punto di pareggio è lo **zero matematico** del margine operativo:

$$
\Pi(S,P) = R(S,P) - C(S,P) = 0
\quad\Longleftrightarrow\quad
(1-\tau)\,R(S,P) = C_T + F
$$

Nell'ipotesi di regime, in cui il numero di abbonati è stabile, i nuovi iscritti servono solo a rimpiazzare chi lascia ($$S_{nuovi} = \gamma_S S$$, $$P_{nuovi} = \gamma_P P$$, con $$\gamma$$ tasso di abbandono). L'equazione diventa una retta nel piano $$(S,P)$$:

$$
a_S\,S + a_P\,P = C_T + F - (1-\tau)\,12\,n_T\,c_T
$$

con $$a_S = (1-\tau)(12\,p_S + q\,\gamma_S)$$ e $$a_P = (1-\tau)(12\,p_P + q\,\gamma_P)$$, cioè il **contributo annuo netto** di un iscritto Smart e di un iscritto Premium.

Il pareggio non è quindi un numero, ma una *famiglia* di combinazioni Smart/Premium. Per ottenere un punto preciso si fissa il mix commerciale: con la struttura di costi del secondo anno e una quota Premium tra il 60% e il 65% (il desiderata del cliente come composizione di clientela), il pareggio operativo si colloca quindi intorno a **630–640 iscritti complessivi**; se si vuole coprire anche ammortamenti e interessi (pareggio a livello di utile ante imposte), la soglia sale a circa **780–790**.

È un'informazione che cambia il modo di leggere il progetto: sappiamo esattamente quanto vale, in termini di abbonati, ogni euro di costo fisso.

## 4. Dal conto economico "stimato" al conto economico "modellato"

Il secondo passo è quello che, a nostro avviso, distingue un lavoro ingegneristico da una buona stima.

In un business plan il numero di iscritti di ogni anno viene di solito scritto direttamente nel conto economico. Noi abbiamo invece modellato il fenomeno che lo genera: il **churn**. A ogni fine anno una parte degli iscritti non rinnova. Per ciascuna formula:

$$
S_t = (1-\gamma_S)\,S_{t-1} + N^S_t
\qquad
P_t = (1-\gamma_P)\,P_{t-1} + N^P_t
$$

Il primo termine è l'**organico che sopravvive** all'anno successivo; il secondo, $$N_t$$, sono i **nuovi iscritti** che servono per raggiungere l'obiettivo. Le variabili decisionali del modello sono proprio i nuovi iscritti, e il vincolo è il raggiungimento del target di iscritti totali di ogni anno.

Con tassi di abbandono ipotizzati sulla base di benchmark di settore (55% per lo Smart, 35% per il Premium, più fidelizzato), il modello restituisce:

| | 2027 (4 mesi) | 2028 | 2029 |
|---|---:|---:|---:|
| Smart riconfermati | – | 95 | 198 |
| Premium riconfermati | – | 318 | 430 |
| Nuovi Smart | 211 | 344 | 292 |
| Nuovi Premium | 489 | 343 | 480 |
| **Iscritti totali Smart** | 211 | 439 | 490 |
| **Iscritti totali Premium** | 489 | 661 | 910 |
| **Totale** | 700 | 1.100 | 1.400 |

Questi numeri alimentano direttamente il P&L. Il risultato è un conto economico i cui ricavi non sono più stime o semplici operazioni di somma e prodotto, ma l'**output di un modello**. I ricavi da abbonamenti, quelli dai canoni dei trainer e quelli derivanti dalle quote di iscrizione sono tutti appartenenti alla stessa dinamica degli iscritti. Il margine operativo ante interessi, tasse e ammortamenti (EBITDA) passa quindi da circa il 6% del primo esercizio, che è parziale e sconta l'avviamento, a circa il 40% nel secondo e oltre il 50% nel terzo.

Il modello mette quindi in luce quanto **costi mantenere** un livello di iscritti. Con un churn del 55%, per tenere fermo il numero di abbonati Smart bisogna rimpiazzarne più della metà ogni anno. È un dato commerciale, non solo finanziario.

## 5. Gli spazi e la saturazione

La struttura dispone di una superficie utilizzabile limitata (una parte dei metri quadrati complessivi è occupata da servizi e spazi non destinati all'allenamento). Abbiamo quindi introdotto un indice di afflusso $$\rho$$, cioè la quota di iscritti presenti contemporaneamente nei momenti di picco, e calcolato i **metri quadrati disponibili per utilizzatore**:

$$
m(N) = \frac{M}{\rho\,N}
$$

con $$M$$ superficie utilizzabile e $$N$$ iscritti totali. Nel piano, questo indicatore scende progressivamente al crescere degli iscritti: da circa 20 m² per utilizzatore nel primo anno a circa 13 nel secondo e circa 10 nel terzo.

Fissata una soglia minima di comfort $$m_{min}$$, la capacità massima del centro diventa un vincolo:

$$
N \le \frac{M}{\rho\,m_{min}}
$$

Ecco perché le due domande di partenza sono la stessa domanda: il break-even dà il limite inferiore sugli iscritti, la saturazione il limite superiore. Il dimensionamento di abbonamenti e spazi è il problema di stare, con il miglior risultato economico, dentro questa finestra.

## 6. I moltiplicatori di Lagrange: quanto vale un iscritto in più

Trovato il pareggio, la domanda successiva è: *quanto cambia il risultato se aggiungiamo un iscritto?* Formalmente, il problema è un'ottimizzazione vincolata:

$$
\max_{S,P}\ \Pi(S,P)
\quad\text{s.t.}\quad
S + P = N,\quad
q_{min} \le \frac{P}{N} \le q_{max},\quad
N \le \frac{M}{\rho\,m_{min}}
$$

e la lagrangiana associata è

$$
\mathcal{L} = \Pi(S,P) - \lambda\,(S+P-N) - \mu\,(\text{vincoli sul mix}) - \nu\,\Big(N - \tfrac{M}{\rho\,m_{min}}\Big)
$$

Il moltiplicatore $$\lambda^*$$ ha un'interpretazione economica diretta, quella di **prezzo ombra**: è la variazione del risultato ottimo al variare di una unità del vincolo,

$$
\lambda^* = \frac{\partial \Pi^*}{\partial N}
$$

cioè *quanto vale, in termini di ricavi, un iscritto aggiuntivo*. Con le tariffe del progetto e un mix al 60–65% di Premium, un iscritto in più porta circa **730–745 € di ricavi annui da abbonamento**, poco più di 60 € al mese, a cui si aggiunge la quota di iscrizione. Il moltiplicatore sul vincolo di mix racconta invece quanto convenga spostare la base verso il Premium: ogni iscritto che passa da Smart a Premium vale circa **220 € netti all'anno** in più, ed è per questo che l'ottimo si colloca sul limite superiore della quota Premium.

Il terzo moltiplicatore, quello sulla capacità, collega economia e spazi: il valore di un metro quadrato in più è il valore di un iscritto, diviso per i metri quadrati che quell'iscritto "consuma" in ora di punta, $$\lambda^*/(\rho\,m_{min})$$. Quando il vincolo di capacità è attivo, risponde a una domanda concreta: *quanto sono disposto a pagare per ampliare gli spazi?*

## 7. Un gemello matematico per gli scenari what-if

Il risultato finale non è un singolo numero, ma uno **strumento**. Abbiamo quindi costruito un gemello matematico del business in cui tariffe, tassi di abbandono, mix, numero di trainer, superficie e obiettivi di crescita sono parametri, e ogni scenario ricalcola in modo coerente iscritti, conto economico e saturazione degli spazi.

Qualche domanda a cui ora si può rispondere in pochi minuti:

- Se il churn dello Smart passa dal 55% al 45%, quanti nuovi iscritti servono in meno e che effetto ha sul margine?
- Aumentare di 5 € la tariffa Premium compensa una crescita più lenta della base?
- Quanto pesa, sul pareggio, un ritardo di due mesi nell'apertura?
- Fino a quale numero di iscritti possiamo spingere la crescita prima che la saturazione diventi un problema per la qualità percepita?

La differenza rispetto a un foglio di calcolo costruito a mano è la tenuta: gli scenari non sono variazioni arbitrarie di celle, ma soluzioni di un modello in cui i vincoli restano sempre rispettati.

## Conclusioni

Il lavoro mostra come strumenti della ricerca operativa, cioè modello di dinamica degli iscritti, ottimizzazione vincolata e analisi di sensitività tramite moltiplicatori, rendano un business plan qualcosa di più di una proiezione: un sistema che si può interrogare. Il cliente ha ottenuto una soglia di pareggio espressa in abbonati, una finestra di dimensionamento definita dagli spazi e una misura del valore marginale di ogni scelta commerciale.

Se stai valutando l'apertura di un nuovo punto vendita, o vuoi capire come dimensionare un'attività esistente in funzione della sua saturazione, [scrivici](/optima_engineering/#contatti): partiamo volentieri dai tuoi numeri.
