# 3-DataMining e Machine Learning

Status: Done
Type: theory

Machine Learning e Data Mining sono discipline strettamente correlate ma con obiettivi distinti:

- **Machine Learning**: mira a creare modelli predittivi o di classificazione ottimali, utilizzando grandi quantità di dati per l’addestramento e la valutazione dei modelli.
- **Data Mining**: si concentra sull’estrazione di informazioni e pattern nascosti nei dati, spesso impiegando tecniche di Machine Learning per analizzare e interpretare i dataset.

---

# Principio di Bonferroni

<aside>
💡

Il **Principio di Bonferroni** afferma che, quando si analizzano grandi quantità di dati alla ricerca di eventi specifici, alcuni di questi eventi possono emergere anche in dati completamente casuali. Poiché il numero di tali eventi cresce con la dimensione del dataset, è necessario applicare correzioni statistiche per evitare falsi positivi.

</aside>

### Esempio:

La CIA deve controllare se 2 persone dormono nello stesso albergo per più di una volta nello stesso periodo: 

Dati:

- $10^9$ di persone
- Ogni persona passi 1 notte su 100 in hotel
- Ci sono $10^5$ hotel e supponiamo che ciascuno possa ospitare al massimo 100 persone
- Periodo di analisi: 1000 giorni

Bonferroni dice che 2 persone potrebbero essere nello stesso hotel per caso:

$P(persona\ in \ 1 \ hotel \ in \ una \ notte)=\frac{1}{100}=10^{-2}$

$P(2\ persone\ in \ 1 \ hotel \ in \ una \ notte)= 10^{-2}\cdot 10^{-2} = 10^{-4}$

$P(2\ persone\ nello\ stesso\  hotel \ in \ una \ notte)= \frac{10^{-4}}{10^5}=10^{-9}$

$P(2\ persone\ per\ 2\ volte \ nello\ stesso\  hotel) = 10^{-9}\cdot 10^{-9}=10^{-18}$ 

Coppie di persone = $\frac{10^9\cdot(10^9-1)}{2}= 5\cdot10^{17}$

Coppie di giornate = $\frac{10^3\cdot10^3}{2}= 5\cdot10^{5}$

Quindi il numero di persone che nel periodo considerato risiedono statisticamente nello stesso hotel 2 volte sono: $5\cdot10^5\cdot5\cdot10^{17}\cdot10^{-18}=25\cdot 10^4$ → il nostro algoritmo non è così preciso, perché abbiamo tanti falsi positivi (→ **il valore randomico è superiore a quello che ci aspettiamo**).

---

# Approfondimento sui modelli di machine learning

## Scuole di pensiero sui modelli di ML

Le scuole di pensiero nell’AI si dividono in due approcci principali:

- **Data-Centric AI**: la qualità e la rilevanza dei dati sono più importanti del modello stesso. Migliorare i dati porta a prestazioni migliori.
- **Model-Centric AI**: il focus è sull’ottimizzazione del modello affinché funzioni bene anche con dati imperfetti.

## Modelli di machine Learning

### 1) Regole rudimentali:

- **One Rule:** crea una sola regola basata su un unico attributo. Si parte da una tabella, analizzando ogni istanza singolarmente e associandola a una regola con un margine di errore. L’attributo più efficace viene scelto per definire le regole.
    
    <aside>
    🚨
    
    Il rischio principale è l’**overfitting**, soprattutto con variabili numeriche (es. se si decide che a 23°C si gioca e a 24°C no). Questo si può mitigare creando intervalli di valori anziché regole troppo specifiche.
    
    </aside>
    
    <aside>
    📈
    
     ****Graficamente genera bande definite da **linee rette**.
    
    </aside>
    

![In questo caso considererò solo l’attributo Outlook](3-DataMining%20e%20Machine%20Learning/image.png)

In questo caso considererò solo l’attributo Outlook

- **Ruleset con covering approach:** i ****covering algorithms generano un insieme di regole per classificare i dati, privilegiando quelle che coprono il maggior numero possibile di istanze di una classe. Tuttavia, non è necessario coprire tutti i dati con la massima precisione: forzarlo rischierebbe di portare a fenomeni di **overfitting**.
    
    ![image.png](3-DataMining%20e%20Machine%20Learning/image%201.png)
    
    <aside>
    📌
    
    **Simple covering algorithm:**
    
    Tale algoritmo genera regole aggiungendo test che massimizzano l’accuratezza, l’obiettivo è massimizzare la precisione della classificazione.
    
    l criterio principale è il rapporto $\frac{p}{t}$, dove:
    
    - $t$ è il numero totale di istanze coperte dalla regola,
    - $p$ è il numero di esempi positivi correttamente classificati,
    - $t-p$ rappresenta gli errori della regola.
    
    Si seleziona il test che massimizza $\frac{p}{t}$, e il processo termina quando $\frac{p}{t}=1$ (nessun errore) o quando non è più possibile suddividere ulteriormente il dataset.
    
    **Esempio:** Dati sulle lenti a contatto
    
    1. **Dati di partenza** 
        
        ![image.png](3-DataMining%20e%20Machine%20Learning/image%202.png)
        
    2. **Prima Regola:** si cerca il miglior test per discriminare le istanze. Il test con la maggiore copertura è $astigmatismo = yes$, ottenendo una regola iniziale: $astigmastismo= yes \rightarrow raccomandazione=hard$
        
        ![image.png](3-DataMining%20e%20Machine%20Learning/image%203.png)
        
    3. **Miglioramento della regola**: si aggiunge il test $tear\ production\ rate = normal$, ottenendo una regola più precisa: $astigmatismo = yes\ e\ tear\ production\ rate = normal \rightarrow raccomandazione = hard$
        
        ![image.png](3-DataMining%20e%20Machine%20Learning/image%204.png)
        
    4. **Affinamento finale**: tra i test rimanenti, riparto dall’attributo che massimizza il rapporto, perciò si sceglie $spectacle\ prescription = myope$ , portando alla regola definitiva: $astigmatismo = yes\ e\ tear\  production\ rate = normal\ e\  spectacle\ prescription = myope \rightarrow raccomandazione = hard$
        
        ![image.png](3-DataMining%20e%20Machine%20Learning/image%205.png)
        
    5. **Seconda regola**: viene costruita una seconda regola per classificare altre istanze di lenti rigide: $age = young\ e\ astigmatismo = yes\ e\ tear\ production\ rate = normal \rightarrow raccomandazione = hard$
    </aside>
    
    <aside>
    📈
    
    Graficamente genera bande definite da **linee rette**.
    
    </aside>
    

### 2) Metodi parametrici:

- **Naive Bayes:** classificatore che assume che tutti gli attributi siano **equamente importanti** e che siano **indipendenti**. Anche se tali assunzioni sono spesso false, il modello è efficace in molte applicazioni. Questo modello utilizza la regola di **Bayes**:
    
    $$
    P_r[H\ |\ E]=\frac{P_r[E\ |\ H]\cdot P_r[H]}{P_r[E]} \quad \quad Prob.\ dell'evento\ dato\ l'evidenza =\frac{(Prob.\ evidenza\ dato \ l'evento )\cdot (Prob.\ evento )}{Prob.\ evidenza}
    $$
    
    **Procedimento:**
    
    1. Si parte da una tabella 
    2. Si analizza un attributo alla volta e un’istanza alla volta. Si calcolano in seguito le probabilità delle diverse condizioni.
    3. Dato un evento si calcola la probabilità che avvengano le varie condizioni
    
    ![image.png](3-DataMining%20e%20Machine%20Learning/image%206.png)
    
    <aside>
    🚨
    
    **Problema zero-frequency**: un problema di questo modello è che se un’istanza di un attributo ha sempre valore zero, non verrà mai predetto. Magari questa frequenza dipende solo dal dataset di training. Quindi si usano varie tecniche di smoothing tra cui aggiungere a tutti i dati un’unità.
    
    </aside>
    
    <aside>
    📈
    
     Graficamente genera sezioni definite da **linee curve**.
    
    </aside>
    
- **Alberi decisionali:** per la costruzione di alberi decisionali utilizziamo un approccio **top-down** con metodo ricorsivo “**divide-and-conque**r”.
    
    
    **Procedimento:**
    
    1. **Selezione dell’attributo radice:** si sceglie l’attributo più informativo come nodo radice.
    2. **Creazione dei rami:** per ogni valore dell’attributo scelto, si crea un ramo.
    3. **Suddivisione delle istanze:** i dati vengono divisi in sottoinsiemi, uno per ogni ramo creato. Tutti i dati che hanno lo **stesso valore** per l’attributo selezionato vengono assegnati al **ramo corrispondente**.
    4. **Ricorsione:** il processo si ripete per ogni sottoinsieme, considerando solo le istanze che raggiungono il nodo corrente.
    5. **Condizione di arresto:** l’algoritmo si ferma quando tutte le istanze in un sottoinsieme appartengono alla stessa classe.
    
    ![image.png](3-DataMining%20e%20Machine%20Learning/image%207.png)
    
    <aside>
    📌
    
    Un albero è fatto bene se è **compatto**, quindi con meno livelli possibili. Questo è possibile se si organizzano gli attributi in modo **polarizzante** (→ un attributo è **polarizzante** se riesce a separare in modo netto le classi, riducendo l’ambiguità).
    
    Per individuare l’attributo più **polarizzante** si può utilizzare l’**entropia** (questo nell’algoritmo **c4.5**):
    
    - Caratteristiche generali entropia:
        - $entropy = 1$ → totalmente bilanciato
        - $entropy = 0$ → totalmente sbilanciato
        
        Quindi più il nodo è puro e più l’entropia è bassa.
        
    - Per calcolare l’entropia di un’istanza:  $entropy(p_1,p_2,...,p_n)= -p_1logp_1 - p_2logp_2...-p_nlogp_n$
    - Per calcolare l’entropia di un attributo si fa combinazione lineare delle entropie delle varie istanze:
        
        ![image.png](3-DataMining%20e%20Machine%20Learning/image%208.png)
        
    
    Quindi per creare l’albero più compatto possibile:
    
    1. Si parte con il nodo radice, che rappresenta la distribuzione della classe da predire. Ad esempio, se il problema è “giocare o no a calcio”, il nodo iniziale conterà il numero di istanze per ciascuna classe (es. 9 *play* e 5 *don’t play*).
    2. Calcolo le entropie di tutti gli attributi 
    3. Calcolo $information  \ gain=(entropia\ prima\ dello \ splitting)- (entropia\ dopo\ lo \ splitting)$
    4. Scelgo l’attributo che mi fa ottenere $information\ gain$ più alto → attributo con l’entropia più bassa → attributo con valori più sbilanciati 
    
    **Problema: Attributi con molti rami →** Quando un attributo ha molti valori distinti, come un codice ID univoco, può creare problemi negli alberi decisionali. La suddivisione dei dati risulta troppo pura (in quanto ogni elemento è unico), riducendo la capacità del modello di generalizzare. Inoltre, l’Information Gain tende a favorire questi attributi, portando a overfitting. Per correggere questo effetto, si utilizza il **Gain Ratio**, che penalizza gli attributi con molti valori distinti e migliora la selezione delle suddivisioni più utili. 
    
    </aside>
    
    <aside>
    📈
    
    Gli **alberi decisionali** sono rappresentati graficamente con **riquadri.**
    
    </aside>
    
- **Linear regression:** viene usata per trovare una retta che meglio separa due classi. Il **decision boundary** è una combinazione lineare degli attributi con i relativi pesi: $x=w_0+w_1a_1+w_2a_2+...+w_ka_k$
    
    
    La retta di separazione viene determinata in due modi:
    
    - **Training**: il modello apprende dai dati di addestramento per adattare i pesi.
    - **Scelta del modello**: tecniche come **Support Vector Machine (SVM)** o **Logistic Regression** definiscono come la separazione viene effettuata.
    
    L’**obiettivo** è trovare la retta che massimizza la separazione tra le classi, riducendo errori e migliorando la generalizzazione del modello.
    
    ![image.png](3-DataMining%20e%20Machine%20Learning/image%209.png)
    
    <aside>
    📈
    
    Sono rappresentati graficamente con **riquadri.**
    
    </aside>
    
    <aside>
    📌
    
    Le **Support Vector Machines (SVM)** sono modelli di classificazione che separano i dati con una funzione lineare, cercando di massimizzare il margine tra le classi. Se i dati non sono separabili linearmente, si trasformano le caratteristiche per ottenere una separazione non lineare. Quando alcuni punti cadono sul lato sbagliato del margine, si introduce una penalità proporzionale alla loro distanza, gestita dalla hinge loss. L’obiettivo è trovare un equilibrio tra un margine ampio e un basso errore di classificazione. 
    
    </aside>
    
- **Clustering:** è un tecnica per dividere le istanze in gruppi, senza avere label associate al training set (→ non esiste una classe da prevedere).
    
    I cluster possono essere classificati in diverse categorie:
    
    - **Disgiunti vs. Sovrapposti**: un’istanza può appartenere a un solo cluster o a più cluster contemporaneamente.
    - **Deterministici vs. Probabilistici**: un’istanza può essere assegnata con certezza o con una probabilità a un cluster.
    - **Flat vs. Gerarchici**: i cluster possono essere organizzati in livelli gerarchici o rimanere su un unico livello.
    
    Step di questo metodo:
    
    1. Distribuiamo nello spazio metrico i vari elementi del dataset
    2. Si applica una strategia di clustering:
        1. **Hierarchical/agglomerative**: ogni punto inizia come un cluster separato, in seguito i cluster più vicini vengono poi uniti iterativamente (→ il concetto di “**vicinanza**” deve essere definito in base alla metrica scelta).
            
            ![image.png](3-DataMining%20e%20Machine%20Learning/image%2010.png)
            
            ![image.png](3-DataMining%20e%20Machine%20Learning/image%2011.png)
            
            Nel clustering gerarchico i dati si organizzano mediante l’utilizzo del **dendogramma** dove:
            
            - I **punti** rappresentano gli elementi
            - Le **righe** rappresentano le relazioni tra gli elementi.
            
            Quindi 2 elementi uniti da una riga appartengono allo stesso gruppo.
            
            <aside>
            
            Nel clustering gerarchico, lo **spazio euclideo** permette di rappresentare i cluster con il loro **centroide**. La vicinanza tra cluster può essere definita dalla distanza tra i centroidi o con altri criteri come la distanza minima/massima/media tra punti di cluster diversi.
            
            </aside>
            
        2. **Point assignment:** si scelgono inizialmente alcuni cluster e si assegnano i punti a quello più vicino, tipicamente utilizzando metodi come **k-means**.
            
            <aside>
            📌
            
            **Algoritmo K-means**
            
            L’algoritmo **K-means** è un metodo di clustering che suddivide i dati in **k gruppi** in base alla loro vicinanza ai centroidi dei cluster. L’algoritmo segue questi passaggi:
            
            1. Si scelgono inizialmente **k punti** che rappresentano i cluster (diversi metodi possono essere usati per questa selezione).
            2. Ogni punto del dataset viene assegnato al cluster con il **centroide più vicino**.
            3. I centroidi vengono aggiornati in base ai nuovi punti assegnati, causando un possibile spostamento.
            4. (Opzionale) Una volta terminata l’assegnazione, i centroidi possono essere fissati e tutti i punti, inclusi i k iniziali, possono essere riassegnati ai cluster finali.
            </aside>
            
    
    **N.B. Come interrompere il processo di clustering? →** Il processo di clustering può essere interrotto in base a diversi criteri: conoscenza a priori del numero di cluster, interruzione quando la fusione genera cluster non adeguati, oppure costruzione di un unico cluster restituendo un albero che rappresenta il modo in cui i punti sono stati combinati.
    

### 3) Modelli instance based learning

L’**Instance-Based Learning (IBL)** è un metodo di apprendimento che non si basa su modelli matematici con parametri predefiniti, ma confronta direttamente una nuova istanza con quelle di training. Questo approccio segue un principio simile ai motori di ricerca, valutando la **somiglianza** tra i dati.

Caratteristiche:

- **Classificazione basata sugli esempi**: le previsioni vengono fatte confrontando un nuovo esempio con quelli memorizzati e facendo inferenza in base alla somiglianza (di solito una distanza, coma la distanza euclidea).
- **Nessuna generalizzazione esplicita**: non si apprendono parametri globali per fare previsioni; la classificazione avviene direttamente dagli esempi.
- Il processo decisionale avviene “**on-demand**”, ogni volta che un nuovo punto deve essere classificato, si ricontrolla l’intero dataset.
- **Memoria e tempo computazionale**: l’IBL richiede che tutti i dati di addestramento vengano conservati. Di conseguenza, il tempo di previsione può essere più lento rispetto ad altri metodi (come la regressione logistica), poiché è necessario calcolare le distanze rispetto a molti esempi.
- **Importanza della normalizzazione**: poiché gli attributi possono avere scale diverse, è necessario normalizzarli. Se un parametro è più importante, si possono dilatare i suoi valori; se è meno importante, si possono ridurre.
    
    <aside>
    ⚠️
    
    **Esempio con e senza normalizzazione:**
    
    - **Senza normalizzazione**: il modello è dominato dalla feature con range più grande (proline), ignorando le altre.
    - **Con normalizzazione**: le feature hanno lo stesso peso e il confine decisionale è più equilibrato.
    
    ![image.png](3-DataMining%20e%20Machine%20Learning/image%2012.png)
    
    </aside>
    
- **Majority vote:** si seleziona un numero di vicini o si definisce una soglia di distanza, e al nuovo valore viene assegnata la classe più frequente tra i vicini scelti.
    
    ![image.png](3-DataMining%20e%20Machine%20Learning/image%2013.png)
    

<aside>

**Esempio di IBL: k-NN (k-Nearest Neighbors)**

- Se **k** è piccolo → rischi overfitting (ti fidi troppo dei pochi vicini).
- Se **k** è grande → hai una previsione più robusta e generalizzata.
</aside>

---

# Overfitting

<aside>
💡

Il fenomeno dell’**overfitting** si verifica quando il mio modello si adatta alla specificità del mio dataset perdendo la capacità di generalizzare.

</aside>

Il **controllo dell’overfitting** avviene testando il modello su un dataset separato (**test set** o **holdout set**). L’**overfitting** si manifesta quando il modello ottiene ottime performance sul **training set**, ma risultati significativamente peggiori sul **test set**.

Quando ciò accade, è necessario **ridurre la complessità** del modello e fermarsi a un **punto ottimale** (→ *early stopping*). L’obiettivo è trovare un **equilibrio** tra **overfitting** (modello troppo specifico, che memorizza i dati di training) e **generalizzazione** (modello capace di fare previsioni accurate su nuovi dati).

![image.png](3-DataMining%20e%20Machine%20Learning/image%2014.png)

## Overfitting nei vari modelli

- Nel **table model** per la customer churn, aumentando la dimensione della tabella, il modello memorizza sempre più dati del training set, riducendo l’errore su di esso. Tuttavia, l’errore sul test set rimane invariato (→ perché i due set non si sovrappongono), portando a overfitting e scarsa generalizzazione.
    
    ![image.png](3-DataMining%20e%20Machine%20Learning/image%2015.png)
    

- Nella **tree induction**, i modelli ad albero suddividono i dati per trovare attributi predittivi rilevanti. Se ogni istanza finisce in una foglia, il modello diventa perfettamente accurato sul training set, ma perde capacità di generalizzazione.
    
    ![image.png](3-DataMining%20e%20Machine%20Learning/image%2016.png)
    

- Nel caso delle **funzioni lineari**, il dataset Iris mostra due classi ben separate, con logistic regression e SVM che tracciano confini simili. L’introduzione di outlier causa reazioni diverse: **SVM** si adatta poco, mentre **logistic regression** modifica di più il confine decisionale. Perciò avremo la **logistic regression** meno instabile e quindi più incline all’overfitting rispetto al l’**SVM**.
    
    
    ![image.png](3-DataMining%20e%20Machine%20Learning/image%2017.png)
    
    ![image.png](3-DataMining%20e%20Machine%20Learning/image%2018.png)
    

---

# Learning curve

Le **learning curves** sono grafici che mostrano l’andamento dell’errore del modello rispetto alla quantità di dati di training. Servono per capire se il modello ha bisogno di più dati per migliorare le sue prestazioni.

<aside>

**Esempio:**

Questo esempio di learning curve mostra che la **Logistic Regression** migliora significativamente fino a circa **500 istanze**, dopo le quali l’aggiunta di ulteriori dati non porta grandi benefici. Il **Decision Tree**, invece, necessita di circa **800 istanze** per stabilizzarsi. Altri modelli, invece, possono non risentire in modo significativo dell’aumento dei dati disponibili

![image.png](3-DataMining%20e%20Machine%20Learning/image%2019.png)

</aside>

---

# Metodi per valutare il modello

### 1) Divisione del dataset in parti

     Divisione in 2 parti:

- 60/70% per il **training**
- 40/30% per il **testing**

Divisione in 3 parti:

- 60/70% percento per il **training**
- 20% per la **validation** (→ per ottimizzare il modello sulla base di alcuni parametri)
- 10% per il **testing**

### 2) Cross-Validation

1. Mescolo i dati del dataset
2. Divido randomicamente il dataset $D$ in $K$ blocchi $B_1,...,B_k$
3. 

$\forall k 

\in

 \{1,....,K\}$

  $T_e=B_k$  e  $T_r = D-B_k$  (→ rimanenti $k-1$ blocchi)
4. Addestro il modello $M$ 
 su $T_r$
5. Testo il modello su $T_e$ valutandone le performance
6. La performance viene valutata tramite la **media** dell’accuracy sui k fold, considerando anche la **deviazione standard** per capire la stabilità del modello.

![image.png](3-DataMining%20e%20Machine%20Learning/image%2020.png)

<aside>
🚧

**Cross-validation stratificato o random?**

Nella **cross-validation stratificata**, ogni fold mantiene la stessa proporzione di etichette presenti nel dataset originale, garantendo una distribuzione equilibrata. Questo evita che alcuni blocchi di dati contengano solo una classe, rendendo il modello più robusto.

Se invece la suddivisione è completamente **random**, si rischia di avere fold sbilanciati, con alcuni molto facili da predire e altri difficili. In particolare, se il dataset è **molto polarizzato**, la stratificazione diventa **obbligatoria** per evitare che il modello impari in modo distorto.

</aside>

**N.B.** Un dataset è **molto polarizzato** quando una classe è **molto più rappresentata** rispetto all’altra. Ad esempio, in un dataset per la rilevazione di frodi, potresti avere il **99%** delle transazioni legittime e solo l’**1%** fraudolente.

### 3) Leave-One-Out Cross-Validation

Variante del cross-validation dove ogni istanza è usata come test una volta mentre il resto è usato per il training. Questo metodo utilizza il massimo dei dati disponibili per l’addestramento, ma è computazionalmente oneroso.

---

# Metriche per valutare modelli di classificazione

Per valutare un modello di classificazione si utilizza la **confusion matrix**, in cui le celle sulla diagonale principale rappresentano le previsioni corrette.

![image.png](3-DataMining%20e%20Machine%20Learning/image%2021.png)

- **Accuracy → $\frac{TP +TN}{TP+TN+FP+FN}$**

Tuttavia, se il dataset è sbilanciato (→ polarizzato), l’accuratezza potrebbe non essere una metrica affidabile. In questi casi, si preferiscono:

- **TP rate (Recall) → $\frac{TP}{TP+FN}$** [positivi che prevedo rispetto ai positivi che ci sono nella realtà]
- **FP rate (Precision)→ $\frac{FP}{FP+TN}$** [quante volte ho sbagliato nel predire sì]

Queste misure sono **antitetiche →** ovvero la percentuale di sì che dovevo trovare e che trovo effettivamente **F-measure** per combinarle e bilanciarle:

- **F-measure → $F_{\beta}=\frac{(\beta^2+1)\cdot P\cdot R}{\beta^2P+R}$**
    
    Dove il valore di $\beta$ determina il peso relativo tra le due misure: se $\beta >1$, si favorisce la **recall**, mentre se $\beta < 1$, si dà più importanza alla **precision**. Quando $\beta = 1$, si ottiene la **F1-score**, che bilancia in modo equo precision e recall:
    
    $$
    \text{F1-score}=2\cdot \frac{recall\cdot precision}{recall+precision}\quad \text{(= media armonica)}
    $$
    
- **ROC (Receiver Operating Characteristic) → utilizzata per valutare un classificatore binario,** mostra il **tasso di veri positivi (TPR)** rispetto al **tasso di falsi positivi (FPR)** al variare della soglia di decisione. L’**area sotto la curva (AUC - Area Under the Curve)** rappresenta la capacità del modello di distinguere tra le due classi: più l’AUC è vicina a **1**, migliore è la separazione tra classe **A** e **B**.
    
    ![image.png](3-DataMining%20e%20Machine%20Learning/image%2022.png)
    

---

# Metriche per valutare modelli di regressione

Per valutare un modello di regressione si misura l’**errore di predizione.** Esistono diverse metriche per quantificare questo errore:

- **Mean Square Error →  $\frac{(p_1-a_1)^2+...+(p_n-a_n)^2}{n}$**   [valori target effettivi sono indicati come  $a1, a2, …, an$, mentre quelli predetti dal modello sono  $p1, p2, …, pn$ ]
- …

---

# Ensemble learning

L’**Ensemble Learning** consiste nella combinazione di più modelli per migliorare le prestazioni complessive. Le principali tecniche includono:

- **Bagging**: riduce la varianza combinando più modelli addestrati su sottocampioni dei dati, campionati con ripetizione.
- **Randomization**: introduce casualità nella creazione dei modelli per aumentarne la diversità (→ si verifica quando si utilizza lo stesso modello più volte, ma con configurazioni diverse degli iperparametri).
- **Boosting**: costruisce modelli in sequenza, dove ogni modello successivo cerca di correggere gli errori commessi dai modelli precedenti. L’algoritmo attribuisce un peso maggiore agli esempi che sono stati mal classificati, cercando così di ridurre progressivamente gli errori.