# 2-Map Reduce

Status: Done
Type: theory

L’analisi dei **Big Data** richiede l’elaborazione di grandi quantità di informazioni, come nel caso del ranking delle pagine web o della link analysis nei social network. Questa necessità ha portato allo sviluppo di **filesystem distribuiti** e di **nuovi algoritmi** per gestire l’enorme volume di dati.

Per l’elaborazione si potrebbero utilizzare due approcci:

1. Utilizzare un **supercomputer** molto potente.
2. Usare **più computer normali in parallelo**.

La seconda soluzione si è rivelata più efficace: l’ingresso dei dati, infatti, costituisce un *collo di bottiglia*, e suddividere il carico di lavoro su più macchine rende il processo più efficiente e facilmente scalabile.

---

# File System distribuito

Un **filesystem distribuito** è formato da un **cluster di computer** organizzati in una struttura ad albero, con diversi **compute nodes** e un **master node** (detto anche *namenode* o *Hadoop*).

Il **master node** ha il compito di:

- Gestire la struttura dell’albero, i metadati e le cartelle del filesystem.
- Eseguire ping periodici per monitorare lo stato dei nodi.
- Riassegnare i processi di *map* e *reduce* ad altri nodi in caso di malfunzionamenti.

![image.png](2-Map%20Reduce/image.png)

<aside>
🚨

Poiché un numero elevato di nodi aumenta la probabilità di guasti, viene introdotta la **ridondanza**: i file vengono suddivisi in blocchi (*chunk*), e ogni blocco è **replicato** su compute nodes differenti.

Le copie non sono soltanto distribuite su più computer, ma vengono anche salvate su **dischi fisici diversi** e su **tracce differenti** all’interno dei dischi, per ridurre il rischio che un singolo guasto hardware (come un danno a un disco o a una sua parte) provochi la perdita definitiva dei dati.

Anche il **master node** viene replicato, per garantire la massima affidabilità dell’intero sistema.

</aside>

---

# Map-reduce (→ paradigma di programmazione)

Per lavorare con un **filesystem distribuito** è necessario un paradigma di programmazione come il **Map-Reduce**, progettato per elaborare grandi quantità di dati testuali suddivisi in **chunk** e distribuiti su più **compute nodes**.

## Fasi del Map-reduce

Il Map-Reduced si articola in 3 fasi:

1. **Map**: il mapper riceve dati in formato *chiave-valore* e li mappa in nuove coppie *chiave-valore*.
2. **Group-by key**: i valori associati alla stessa chiave vengono raggruppati. Questa operazione avviene mentre i dati vengono **ridistribuiti dai mapper ai reducer.**
3. **Reduce**: il reducer applica un’operazione di riduzione sui valori raggruppati per ciascuna chiave.

<aside>

**Esempio:**

Supponiamo di voler leggere tutte le parole presenti in un file e contarne la frequenza di comparsa:

![image.png](2-Map%20Reduce/image%201.png)

L’esecuzione di **MapReduce** si divide in due fasi principali:

- **Map:** vengono avviati uno o più processi di *map* per macchina
- **Reduce:** per ogni chiave viene eseguito un processo di *reduce*, con la possibilità di far girare più *reducer* sulla stessa macchina

Per **bilanciare i tempi di esecuzione** ed evitare che alcuni nodi completino il lavoro molto prima di altri, è utile assegnare **più processi Map-Reduce** nello stesso task.

Questo aiuta a ridurre lo **skew** (*sbilanciamento*) tra i diversi processi.

</aside>

<aside>
⛔

**Collo di bottiglia: trasferimento dei dati**

Uno dei problemi principali è **il trasferimento dei dati tra mapper e reducer**:

1.	Ogni **mapper** legge i dati dal **server più vicino** per ridurre i tempi di accesso.

2.	L’output del mapper può:

1. **Essere scritto sul filesystem distribuito,** introducendo una latenza dovuta al tempo necessario per l’aggiornamento.
2. **Essere mantenuto in locale**, evitando latenza aggiuntiva.
1. Il **reducer** raccoglie i dati dai nodi che hanno generato una determinata chiave (← COLLO DI BOTTIGLIA). Il **master node** coordina questo processo, indicando al reducer da quali macchine recuperare i dati necessari. Durante questa fase, avviene anche il **group-by key**, in cui tutti i valori associati alla stessa chiave vengono raggruppati prima di essere elaborati.
</aside>

<aside>
✅

**Misure per migliorare il problema di trasferimento dati**

- Il master node può ottimizzare il processo 3 istanziando il reducer sullo stesso node su cui sono presenti **la maggior parte dei dati per quella chiave.** I **mapper** che generano più coppie chiave-valore possono mantenere i dati in locale, evitando trasferimenti inutili. Inoltre, è possibile istanziare **reducer aggiuntivi** su altri nodi per distribuire meglio il carico di lavoro e bilanciare l’elaborazione.
- **Combiner**: se l’operazione di riduzione è commutativa e associativa, si può eseguire sul nodo del mapper. In questo modo si riduce il traffico di rete, migliorando l’efficienza.
    
    ![image.png](2-Map%20Reduce/image%202.png)
    
</aside>

---

# Algoritmi con Map-reduced

- **Moltiplicazione matrice per vettore:** supponiamo di avere una matrice $M$ di dimensioni $n \times n$ e un vettore $V$ di dimensione $n$. L’obbiettivo è calcolare il prodotto matrice-vettore, che si esprime come: $\mathbf{M} \times \mathbf{V} = \sum_{j=1}^{n} m_{ij} \cdot v_j$
    1. Per poter distribuire la matrice su più nodi nel **Filesystem Distribuito (DFS)** e processarla con **MapReduce**, la matrice $M$ e il vettore $V$  vengono rappresentati rispettivamente nel formato: $(i,j,m_{ij}) \ \ (j,v_j)$
    2. Ogni task di **Map** prende una tripla  $(i, j, m_{ij})$ dalla matrice $M$ e calcola $m_{ij} \cdot v_j$, creando una coppia $(i, m_{ij} \cdot v_j)$
    3. La funzione **Reduce** riceve tutte le coppie  $(i, m_{ij} \cdot v_j)$ che condividono lo stesso indice  $i$, e somma tutti i valori associati a quel $i$: $x_i = \sum\limits_{j=0}m_{ij}v_j$
        
        $$
        \begin{bmatrix}m_{11} & m_{12} & m_{13} \\m_{21} & m_{22} & m_{23} \\m_{31} & m_{32} & m_{33}\end{bmatrix} \cdot 
        
        \begin{bmatrix}
        v_1 \\
        v_2 \\
        v_3
        \end{bmatrix} = \begin{bmatrix}
        m_{11}v_1+ m_{12}v_2+m_{13}v_3 \\
        m_{21}v_1+ m_{22}v_2+m_{23}v_3 \\
        m_{31}v_1+ m_{32}v_2+m_{33}v_3
        \end{bmatrix}
        $$
        

<aside>
🚨

**Cosa succede se il vettore $V$ è troppo grande per la memoria?**

Se il vettore $V$ è troppo grande per essere caricato interamente in memoria principale, il sistema dovrà accedere molte volte al disco, rallentando il calcolo. Tuttavia, osservando il processo di **Map**, si nota che gli elementi di ciascuna colonna della matrice $M$ interagiscono esclusivamente con il corrispondente elemento del vettore $V$. Sfruttando questa proprietà, è possibile suddividere sia la matrice che il vettore in **sottoinsiemi** (divisione in bande), riducendo la necessità di accessi ripetuti al disco e migliorando l’efficienza del calcolo.

![image.png](2-Map%20Reduce/image%203.png)

E in seguito tali sottoinsiemi verranno **salvati nello stesso filesystem** in modo tale  da ridurre al minimo il numero di accessi al disco e ottimizzare la fase di **Map**.

</aside>

- **Select** $\sigma_C(R)$:
    - **Map**: per ogni tupla $t$ in $R$, se $t$ soddisfa $C$ → il Mapper emette la coppia $(t,t)$.
    - **Reduce**: è una funzione **identità**, il che significa che passa semplicemente i valori ricevuti come output senza modificarli.
        
        <aside>
        📌
        
        **Esempio:**
        
        Selezioniamo le tuple con segno zodiacale = G
        
        | Name  | Age | Zodiac |
        | --- | --- | --- |
        | F | 30 | G |
        | B | 23 | G |
        | L | 60 | P |
        
        1. **map →** $((F,30,G),(F,30,G)) \quad ((B,23,G),(B,23,G))$ 
        2. **reduce:**
            
            
            | Name  | Age | Zodiac |
            | --- | --- | --- |
            | F | 30 | G |
            | B | 23 | G |
        </aside>
        
- **Proiezione $\pi_S(R)$:**
    - **Map:** per ogni tupla $t$ in $R$ → il Mapper emette la coppia $(t',t')$, dove $t'$ è una tupla ottenuta rimuovendo gli attributi che non fanno parte dell’insieme $S$ (= insieme di interesse).
    - **Reduce:** Per ogni chiave $t'$, emette una sola volta $(t',t')$, eliminando eventuali ripetizioni.
        
        <aside>
        📌
        
        **Esempio:**
        
        | Name  | Age | Zodiac |
        | --- | --- | --- |
        | F | 30 | G |
        | B | 23 | G |
        | L | 60 | P |
        
        1. **map →** $(G,G) \quad (G,G) \quad (P,P)$ 
        2. **reduce →  $(G,G)\quad (P,P)$**
        
        </aside>
        
- **Unione(R,S):**
    - **Map:**  per ogni tupla $t$ proveniente da $R$ o $S$, il Mapper emette la coppia $(t,t)$.
    - **Reduce:** emette una sola volta $(t,t)$, garantendo l’eliminazione dei duplicati.
        
        <aside>
        📌
        
        **Esempio:**
        
        | Name  | Age | Zodiac |
        | --- | --- | --- |
        | F | 29 | G |
        | L | 60 | L |
        
        | Name | Age | Zodiac |
        | --- | --- | --- |
        | F | 29 | G |
        | B | 23 | B |
        
        1. **map →** $((F,29,G),(F,29,G))\ ((L,60,L),(L,60,L))\ ((F,29,G),(F,29,G)) \  ((B,23,B),(B,23,B))$ 
        2. **reduce →  $(F,29,G)\quad (L,60,L)\quad (B,23,B)$**
        
        </aside>
        
- **Intersezione(R,S):**
    - **Map:**  per ogni tupla $t$ proveniente da $R$ o $S$, il Mapper emette la coppia $(t,t)$.
    - **Reduce:** per ogni chiave ricevuta, il reducer controlla quante volte appare. Se la chiave compare due volte, significa che è presente sia in  $R$ che in $S$, quindi viene emessa. Se invece compare solo una volta, vuol dire che appartiene a un solo dataset e viene ignorata.
- **Differenza(R,S):**
    - **Map:**
        - ogni tupla $t$ di $R$ viene messa con l’etichetta di $R$ → $(t,R)$
        - ogni tupla $t$ di $S$ viene messa con l’etichetta di $R$ → $(t,S)$
    - **Reduce:** per ogni chiave $t$, si verifica l’origine dei valori associati: se tutti provengono da $R$ , la chiave viene emessa; altrimenti, se è presente almeno un valore proveniente da $S$, la chiave viene ignorata.
- **Natural Join:**
    - **Map:**
        - per ogni tupla $(a,b)$ in $R$ → $(b,(a,R))$
        - per ogni tupla $(b,c)$ in $S$ → $(b,(c,S))$
    - **Reduce:** raccoglie tutte le coppie che hanno la stessa chiave $b$. Successivamente, genera tutte le combinazioni possibili tra gli elementi di $R$ e quelli di $S$ che condividono lo stesso valore di $b$. Il risultato finale è una nuova relazione formata da tuple della forma $(a,b,c)$.
- **Raggruppamento e Aggregazione:** Data una relazione $R(A,B,C)$, l’operazione $\gamma_{A,\theta(B)}(R)$ raggruppa i dati in base ad $A$ e applica la funzione $\theta$ sugli elementi di $B$.
    - **Map:** ogni tupla $(a,b,c)$ viene trasformata in una coppia chiave-valore $(a,b)$, dove $a$ è il valore su cui raggruppare e $b$ è il valore da aggregare.
    - **Reduce:** Per ogni chiave $a$, si ottiene la lista dei valori associati $[b_1, b_2, …, b_n]$ e si applica la funzione $\theta$ per ottenere il risultato finale  $(a,x)$.
- **Moltiplicazione tra matrici:** può essere implementata utilizzando due approcci: **a due fasi** o **a una fase**.
    
    <aside>
    
    **Metodo a 2 fasi:**
    
    - **1° fase:**
        - **Map:**
            - Per ogni elemento $m_{ij}$, si mette $(j, (M, i, m_{ij}))$
            - Per ogni elemento $n_{jk}$, si emette $(j, (N, k, n_{jk}))$
        - **Reduce:**
            - Per ogni chiave $j$, si combinano gli elementi di $M$ e $N$ aventi lo stesso valore di $j$.
            - Si generano coppie  $((i, k), m_{ij} n_{jk})$, che rappresentano i contributi ai valori finali della matrice risultante.
    - **2° fase:**
        - **Map:** passa i dati così come sono (→ identità)
        - **Reduce:** per ogni coppia $(i, k)$, somma tutti i valori associati per ottenere l’elemento finale della matrice risultante $(i, k)$.
    
    ---
    
    Esempio:
    
    $$
    \begin{bmatrix}
    m_{11} & m_{12} \\
    m_{21} & m_{22}
    \end{bmatrix} \cdot \begin{bmatrix}
    n_{11} & n_{12} \\
    n_{21} & n_{22}
    \end{bmatrix} = \begin{bmatrix}
    v_{11} & v_{12} \\
    v_{21} & v_{22}
    \end{bmatrix}
    $$
    
    ![image.png](2-Map%20Reduce/image%204.png)
    
    </aside>
    
    <aside>
    
    **Metodo a 1 fase:**
    
    - **Map:**
        - Per ogni elemento $m_{ij}$ di $M$, si emette $((i,k), (M, j, m_{ij}))$ per ogni $k$.
        - Per ogni elemento $n_{jk}$ di $N$, si emette $((i,k), (N, j, n_{jk}))$ per ogni $i$.
    - **Shuffle:** Raggruppa tutti i valori aventi la stessa chiave $(i, k)$.
    - **Reduce:**
        - Per ogni chiave $(i, k)$, si associano gli elementi di $M$ e $N$ che hanno lo stesso valore di $j$.
        - Si calcola $v_{ik}$ come la somma dei prodotti $m_{ij} n_{jk}$.
    
    ---
    
    Esempio:
    
    $$
    \begin{bmatrix}
    m_{11} & m_{12} \\
    m_{21} & m_{22}
    \end{bmatrix} \cdot \begin{bmatrix}
    n_{11} & n_{12} \\
    n_{21} & n_{22}
    \end{bmatrix} = \begin{bmatrix}
    v_{11} & v_{12} \\
    v_{21} & v_{22}
    \end{bmatrix}
    $$
    
    ![image.png](2-Map%20Reduce/image%205.png)
    
    **N.B.** Molto interessante in quanto per ogni elemento, mi genera un numero di coppie (chiave,valore) pari al numero di colonne [In tutti gli altri algoritmi da ogni elemento si genera una sola coppia (chiave,valore)].
    
    </aside>
    

---

# Teoria della complessità per gli algoritmi map-reduce

La complessità di un algoritmo map-reduce può essere valutato utilizzando due parametri:

- **Reducer size (q) →** numero massimo di valori che possono essere associati ad una chiave.
    - Se si usa un **combiner**, allora **q = 1**, perché il combiner aggrega i valori localmente prima di inviarli al reducer.
    - Senza **combiner**, **q** può crescere molto in base ai dati (es. nel conteggio parole, sarà pari al numero massimo di occorrenze di una parola).
- **Replication rate (r) →** numero di coppie chiave-valore generato da ogni elemento letto.
    - Sempre a **1** in quasi tutti gli algoritmi MapReduce che abbiamo visto, tranne nel caso del **prodotto tra matrici in un solo step**, dove **r = N** (numero di colonne della matrice).

**N.B.** Questi due valori sono **inversamente proporzionali** → se ci sono poche coppie (chiave-valore) [replication rate bassa] allora ci saranno vari valori associati a quelle chiavi [reducer size alta]

<aside>
💡

**Trade-off tra r e p nel Prodotto di Matrici a 1 fase:**

Nel **prodotto tra matrici con un solo passaggio** (n × n):

- **r = n** → Ogni elemento di entrambe le matrici genera $n$ coppie chiave-valore.
- **q = 2n** → Ogni chiave $(i,k)$ ha $n$ valori dalla prima matrice e $n$ dalla seconda.
- **Trade-off**: $qr \geq 2n^2$
</aside>

---

## Esempi sul map-reduce

<aside>

### Esempio 1

![image.png](2-Map%20Reduce/image%206.png)

</aside>

<aside>

### Esempio 2

![image.png](2-Map%20Reduce/image%207.png)

</aside>