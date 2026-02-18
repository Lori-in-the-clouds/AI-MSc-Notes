# 5.2-Text Retrieval

Status: Done
Type: theory

<aside>
💡

Il **Text Retrieval** è un sottoinsieme dell’**Information Retrieval** che si occupa esclusivamente di documenti testuali, mentre l’Information Retrieval include anche immagini, video e altri tipi di dati.

</aside>

Un sistema di text retrieval (come il search engine) offre il supporto per eseguire query su una collezione di documenti. Tuttavia, restituire risultati pertinenti è complesso in quanto:

- L’utente spesso non sa esattamente cosa sta cercando.
- Le query sono brevi e i documenti contengono molte parole, rendendo difficile individuare le parole chiave più rilevanti.
- Non esiste una “verità assoluta” per determinare l’importanza delle parole: solo l’utente finale può valutare la pertinenza dei risultati.

Al contrario, fare query su un database è più semplice perché:

- I dati sono strutturati in colonne ben definite.
- I significati delle parole sono chiari grazie al contesto strutturato (es. “Guerra” è un cognome e non un evento storico).

---

# Formalizzazione del problema

L’obiettivo dell’information retrieval è, data una collezione di documenti, trovare in risposta a una query dell’utente un sottoinsieme di documenti che rispondano nel modo più preciso possibile al suo bisogno informativo.

- **Vocabulario**: $V = \{w_1, w_2, …, w_N\}$ è l’insieme delle parole presenti nei documenti.
- **Query**: $q = (q_1, q_2, …, q_m)$  è una sequenza di parole con $q_i \in V$ (→ la query è più corta del documento: $m < s$).
- **Documento**: $d_i = (d_{i1}, d_{i2}, …, d_{is})$ è una sequenza di parole con $d_{ij} \in V$.
- **Collezione di documenti**:  $C = \{d_1, d_2, …, d_M\}$.
- **Documenti rilevanti**: $R(q) \subset C$  è l’insieme dei documenti effettivamente rilevanti per la query $q$ .

<aside>
🚨

**Problema:**

- **Ambiguità semantica**: utenti diversi possono usare la stessa query con intenzioni diverse.
- **Approssimazione**: il sistema non può identificare perfettamente $R(q)$, quindi restituisce un insieme stimato $R'(q)$ , che è un’approssimazione dell’insieme di documenti rilevanti.
</aside>

---

# Come calcolare R’(q)

Ci sono principalmente 2 tecniche:

- **Document selection:** si crea una funzione che, data una query e un documento, restituisce:
    - **1** se il documento è rilevante
    - **0** se il documento non è rilevante
    
    Non ottimale in quanto questo metodo mette sullo stesso piano i vari documenti, senza premiare quelli più rilevanti.
    
- **Ranking:** ogni documento riceve un punteggio tra 0 e 1 in base alla sua rilevanza rispetto alla query, e viene ordinato in ordine decrescente. È possibile applicare un cutoff per mostrare solo i primi documenti più rilevanti.
    
    Questo metodo è più efficace rispetto alla selezione binaria e risulta teoricamente più efficiente, a condizione che:
    
    - l’utilità di un documento non influenzi quella degli altri
    - l’utente consulti i risultati in modo sequenziale

---

# Concetto di rilevanza (Modelli di Ranking)

Per valutare la rilevanza di un documento rispetto a una query si possono usare diversi modelli:

- **Vector Space Model** afferma che un documento è tanto più rilevante quanto più è simile alla query, ma questo è complesso perché le dimensioni di un documento e di una query sono molto diverse.
- **Probabilistic Retrieval Model** si basa invece sull’analisi delle query passate e delle interazioni degli utenti con i documenti. Se un documento ha soddisfatto molti utenti per una certa query, verrà riproposto con maggiore probabilità.

---

# **Preparazione dei Dati per il successivo calcolo della Similarità nei Modelli di Ranking**

1. **Bag-of-Words:** i documenti e le query vengono rappresentati come vettori numerici basati sulla frequenza delle parole, senza considerare il loro ordine. Durante questo processo, si ignorano le **stop words**, ovvero parole molto comuni che non aggiungono significato rilevante (es. *it, the, and*).
2. Un modo più intelligente di riempire la bag of words è rappresentato da **TF-IDF** ( $TF\cdot IDF$ ):
    - Term Frequency (TF): quantifica quante volte una parola appare in un documento.
    - Document Frequency (DF): indica il numero di documenti in cui compare una parola.
    - Inverse Document Frequency (IDF**)**: indica quanto è rara o comune una parola in tutta la collezione di documenti. È calcolato come:
        
        $$
        IDF(\omega)=log(\frac{M+1}{df(\omega)}) \quad \text{}
        $$
        
        dove:
        
        - $M$  è il numero totale di documenti
        - $df(\omega)$  è il numero di documenti in cui la parola $\omega$ appare
        
        **N.B.** Si tiene anche conto della **lunghezza del documento**, per evitare di favorire documenti lunghi solo perché contengono più parole.
        
    
    <aside>
    📌
    
    Quindi:
    
    - se **TF è alto** → la parola è **frequente nel documento**
    - se **IDF è alto** → la parola è **rara nei documenti** (→ molto informativa)
    
    Allora $TF-IDF=\text{molto alto}$ → **La parola è molto rappresentativa di quel doumento.**
    
    </aside>
    

---

# Vector Space model (VSM)

L’VSM è un modello generico utilizzato per ottenere  una funzione di ranking basata sulla similarità degli items. 

1. **Si crea una rappresentazione vettoriale sia dei documenti che della query**, in particolare si crea una dimensione per ogni parola precedentemente individuata con la bag-of-words e si definiscono le coordinate del vettore come la frequenza delle parole nel documento o nella query.
    
    ![image.png](5%202-Text%20Retrieval/image.png)
    
2. **Si calcola la similarità tra il vettore query e quello del documento,** i metodi per calcolare la similarità tra due vettori sono vari:
    - Similarità del coseno: si valuta la differenza di angolo dei due vettori. Il coseno dell’angolo tra due vettore si calcola come: 
    
    $\cos(\theta) = \frac{\vec{q} \cdot \vec{d}}{||\vec{q}|| \cdot ||\vec{d}||}$
    - Moltiplicazione riga per colonna: maggiore è il valore del prodotto, più i due vettori sono simili.
    

<aside>
🚧

**Possibili implementazioni:**

- **Implementazione booleana:**
    
    ![image.png](5%202-Text%20Retrieval/image%201.png)
    
    Nell’**implementazione booleana**, un documento ottiene un punteggio in base alla presenza delle parole della query, senza considerare la loro frequenza. Il punteggio massimo corrisponde al numero di parole della query, se un documento le contiene tutte, ha il valore più alto. 
    
    **CONTRO:** questo approccio è rigido, poiché non distingue tra documenti con più o meno occorrenze delle parole e non assegna pesi diversi ai termini più significativi. 
    
- **Implementazione con Term Frequency (TF)**:
    
    ![image.png](5%202-Text%20Retrieval/image%202.png)
    
    Tale implementazione assegna un punteggio ai documenti in base alla frequenza con cui le parole della query compaiono in essi. Maggiore è il numero di occorrenze di una parola, maggiore sarà il peso attribuito al documento, rendendo il ranking più raffinato rispetto al modello booleano, che considera solo la presenza o assenza dei termini. 
    
    **CONTRO:** può dare troppo peso alle parole molto frequenti, senza considerare la loro rilevanza effettiva.
    
- **Implementazione con TF + IDF:**
    
    ![image.png](5%202-Text%20Retrieval/image%203.png)
    
- **Implementazione con TF trasformato + IDF**:  la componente della frequenza del termine (TF) può essere trasformata prima di essere moltiplicata per l’IDF. Infatti, il TF cresce linearmente con il numero di occorrenze di una parola in un documento: ad esempio, se una parola appare 10 volte in un documento e una sola volta in un altro, il primo documento avrà un punteggio 10 volte maggiore. Tuttavia, questo può amplificare eccessivamente l’importanza di termini molto ripetuti. Per attenuare tale effetto, è comune applicare una trasformazione al TF, ad esempio utilizzando una funzione logaritmica.
    
    ![image.png](5%202-Text%20Retrieval/image%204.png)
    
- **Implementazione con term frequency(trasformato) + IDF + pivoted length normalization:** si desidera bilanciare la lunghezza dei documenti, penalizzando quelli più lunghi e favorendo quelli più brevi. A tale scopo, si introduce un parametro $b$ che regola l’importanza della normalizzazione: se $b$ è basso, la lunghezza del documento incide poco sul ranking, mentre valori più alti amplificano l’effetto della penalizzazione o dell’agevolazione.
    
    $$
    normalizer = 1-b+b\frac{|d|}{avdl}\quad con b \in [0,1]
    $$
    
    ![image.png](5%202-Text%20Retrieval/image%205.png)
    

Quindi, per ricapitolare, il punteggio finale di un documento d per una query q utilizzando TF (trasformato) + IDF + pivoted length normalization è:

$$
\text{Score}(q, d) = \sum_{W_i \in q \cap d} \frac{\text{TF}(W_i, d) \times \text{IDF}(W_i)}{(1 - b) + b \times \frac{|d|}{avgdl}}
$$

</aside>

---

# Tecniche di linguistica computazionale (propedeuticità al probabilistic retrieval model)

## 1) Legge di Zipf

In qualsiasi vocabolario, la distribuzione delle parole segue un andamento **logaritmico**: poche parole hanno una frequenza molto alta, mentre la maggior parte delle parole appare raramente.

La **legge di Zipf** afferma che il prodotto tra il ranking di una parola (ovvero la sua posizione nella classifica delle parole più frequenti) e la sua frequenza è **costante:** 

$$
r\cdot f = k
$$

(Con $r = ranking$ → posizione nella classifica delle parole più frequenti).

Un altro modo per esprimere questa legge è tramite la relazione: 

$$

r \cdot P(r) = c
$$

Dove $P(r)$ è la probabilità di occorrenza della parola con rank $r$, e $c$ è una costante che per la lingua inglese vale circa **0.1**.

![image.png](5%202-Text%20Retrieval/image%206.png)

## 2) Language Model

Un **modello linguistico** è un modello matematico che apprende le strutture e i modelli statistici del linguaggio basandosi su grandi quantità di testo. In altre parole, un modello linguistico può:

1. Data una sequenza di parole $(w_1,w_2,w_3,...)$ ed un contesto, calcolare la probabilità di questa sequenza di parole → 

$P(W) = P(w_1, w_2, w_3, …, w_n)$
2. Data una sequenza di parole $(w_1,w_2,w_3,...)$ ed un contesto, trovare la prossima parola → 

$P(w_n | w_1, w_2, …, w_{n-1})$

### Come funziona il modello linguistico

L’obiettivo è calcolare la probabilità di una parola $w$ dato un contesto precedente $h$, ovvero: $P(w|h)$. Ad esempio, dato:

- $h$ = “its water is so transparent that”
- $w$ = “the”

Un metodo intuitivo per calcolare questa probabilità è usare la frequenza relativa: 

$P(w | h) = \frac{\text{Count}(h, w)}{\text{Count}(h)} = \frac{Count(\text{its water is so transparent that the})}{Count(\text{its water is so transparent that})}$

<aside>
🚨

**Limiti di questo metodo**

- **Frasi simili ma non identiche al contesto** non verrebbero considerate nel calcolo della probabilità  $P(w \mid h)$, anche se esprimono lo stesso significato.
    
    Una possibile soluzione è **utilizzare il lemma delle parole**, ovvero **la loro forma base**, così da rendere il modello più generalizzabile.
    
- Per poter stimare correttamente  $P(w \mid h)$ , servirebbero miliardi di frasi reali. Frasi identiche al contesto potrebbero essere molto rare o assenti.
</aside>

<aside>
✅

**1° sol. →** usare la catena di probabilità

$$
P(w_1, w_2, ..., w_n) = \prod_{i=1}^{n} P(w_i | w_1, w_2, ..., w_{i-1})
$$

Ad esempio: $P(\text{"its water is so transparent that"})=P(\text{its})\cdot P(\text{water}|\text{its})\cdot P(\text{is}|\text{its water})\cdot P(\text{so}|\text{its water is)}\cdot P(\text{tranparent}|\text{its water is so})\cdot P(\text{that}|\text{its water is so transparent})$

**CONTRO:** difficoltà nel calcolare le probabilità condizionate per frasi lunghe.

</aside>

<aside>
✅

**2° sol.**→ usare la catena di probabilità considerando solo le ultime N parole (**N-gram**)

Impostando il valore di $N$ come il numero di token da considerare in sequenza, la probabilità della frase può essere approssimata considerando solo le ultime $N-1$ parole precedenti:

$$

P(w_i| w_1, w_2, …, w_{i-1}) \approx P(w_i | w_{i-N+1}, …, w_{i-1})

$$

Ecco alcuni esempi:

- **Bigramma ($N=2$)**: La probabilità di $w_i$ dipende solo dalla parola precedente $w_{i-1}$ , quindi $P(w_i | w_{i-1})$ .
- **Trigramma ($N=3$)**: La probabilità di $w_i$ dipende dalle due parole precedenti  $w_{i-2}, w_{i-1}$ , quindi  $P(w_i | w_{i-2}, w_{i-1})$ .

---

Esempio concreto

Jurafsky, un professore, insieme ad alcuni studenti ha registrato le frasi pronunciate in un ristorante e, basandosi su di esse, ha costruito un bigramma:

- Ha raccolto 9222 frasi
    
    ![image.png](5%202-Text%20Retrieval/image%207.png)
    

- Unigramma (probabilità della parola nella collezione)
    
    ![image.png](5%202-Text%20Retrieval/image%208.png)
    

- Ristato finale:
    
    ![image.png](5%202-Text%20Retrieval/image%209.png)
    

---

**CONTRO:**

- Non è in grado di catturare dipendenze a lungo raggio, poiché considera solo le ultime  N-1  parole e ignora il resto della frase. Questo può portare a previsioni errate quando il contesto richiede informazioni più distanti.
- Con un dataset di addestramento ridotto, il modello rischia di andare in **overfitting**, imparando sequenze specifiche senza riuscire a generalizzare.
- Se una sequenza di parole non è presente nel training set, la sua probabilità viene calcolata come zero, annullando di conseguenza la probabilità totale della frase →  La soluzione a questo problema è applicare lo **smoothing.**

---

N-gram con smoothing

Ci sono varie tecniche di smoothing:

- Una sarebbe quella della **Laplace Smoothing** (Add-One Smoothing), che consiste nell’aggiungere 1 a tutti i conteggi delle sequenze di parole, garantendo che nessuna probabilità sia mai esattamente zero. Tuttavia, questa tecnica può distorcere eccessivamente le probabilità reali.

Lo smoothing in generale aiuta a evitare probabilità nulle (valori = a 0) nella **chain probability**, ma modifica leggermente le distribuzioni delle parole effettivamente osservate nel training set.

Per quantificare questa modifica, è stato introdotto il **fattore di discount**, definito come il rapporto tra la frequenza di una parola dopo lo smoothing e la sua frequenza prima dello smoothing: $d = \frac{c^*}{c}$.

</aside>

---

# Probabilistic retrieval Models

L’idea di base è stimare la rilevanza di un documento rispetto a una query basandoci su dati passati.

$$
p(R=1\ |\ d,q)\quad R \in\{0,1\}
$$

Questo approccio si fonda su una tabella di rilevanza che associa una query a un documento, indicando il livello di rilevanza secondo l’utente.

![image.png](5%202-Text%20Retrieval/image%2010.png)

<aside>
🚨

**Limiti di questo approccio:**

- Un documento nuovo non avrà dati in tabella e quindi rischia di non essere mai considerato.
- Lo stesso problema si presenta per query mai poste prima.
- Anche disponendo di dati per tutte le possibili combinazioni di documenti e query, la quantità di informazioni da gestire sarebbe eccessiva.

Di conseguenza, questo metodo risulta **inefficace**.

</aside>

## Cambiamento approccio

Invece di stimare direttamente la rilevanza di un documento $d$ data una query $q$, si passa a stimare la probabilità che una certa query venga effettuata per ottenere un dato documento.

$$
p(R=1|d,q)

\approx

 p(q|d,R=1)
$$

Per calcolare $p(q|d)$, si utilizza l’**unigramma**, che assume che ogni termine della query sia generato in modo indipendente dal documento.

Il modello si basa sulle seguenti ipotesi:

- Ogni parola della query è indipendente dalle altre.
- Ogni parola è ottenuta da un documento ideale che soddisfa il bisogno informativo dell’utente.

La probabilità totale della query viene calcolata come il prodotto delle probabilità dei singoli termini, dove la probabilità di ogni parola è data dalla sua frequenza relativa nel documento:

$$
p(q|d)=p(w_1|d)\cdot....\cdot p(w_n|d) \quad \text{con}\ q=w_1w_2...w_n
$$

Questo porta a una nuova **formula** effettiva **per il ranking** dei documenti, per facilitare il calcolo ed evitare problemi di underflow numerico, utilizziamo il logaritmo (in modo che la produttoria diventi una sommatoria):

$$
f(q,d)=log\ p(q|d)= \sum_{i=1}^{n}log\ p(w_i|d) = \sum_{w \in V}c(w,q)log\ p(w|d)
$$

Dove:

- $c(w,q)$ → indica quante volte la parola $w$ appare nella query
- $lop\ p(w|d)$ → è ricavato con il **document language model**

<aside>
🚨

Tuttavia, alcune probabilità potrebbero risultare **nulle**, quindi è necessario applicare una tecnica di **smoothing** per gestire i termini non presenti nel documento.

</aside>

### Ranking function with smoothing

Per le parole presenti nel documento vorrei stimare la probabilità con linguistic model, invece per i termini **non presenti** nel documento, è necessario stimare una probabilità che vada verso zero in modo graduale, senza annullarsi completamente.

Un metodo per fare lo smoothing della parola è il seguente:

$$
p(w|d)=\begin{cases}p_{Seen}(w|d)\quad se \ w \  è\ in\ d\\ \alpha_dp(w|C)\quad altrimenti\end{cases}
$$

Dove:

- Per le parole **presenti** nel documento, si utilizza la probabilità empirica $p_{\text{Seen}}(w|d)$.
- Per le parole **non presenti**, si introduce un parametro $\alpha_d$ che regola il contributo della probabilità della parola nell’intera collezione $p(w|C)$, garantendo che il valore sia positivo ma tendente a zero.

![image.png](5%202-Text%20Retrieval/image%2011.png)

La **ranking function** aggiornata con lo **smoothing** diventa:

![image.png](5%202-Text%20Retrieval/image%2012.png)

<aside>

**Correlazione tra Probabilistic Retrieval e Vector Space Model**

Anche adottando un **approccio probabilistico**, alla fine si riconduce sempre a un modello basato sulla **similarità** tra la query e il documento.

![image.png](5%202-Text%20Retrieval/image%2013.png)

</aside>

---