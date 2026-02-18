# 4.2-Trovare elementi simili

Status: Done
Type: theory

<aside>
💡

**Misura di similarità:**

Si utilizza spesso la **Jaccard Similarity**, definita come: 

$J(C1, C2) = \frac{|C1 \cap C2|}{|C1 \cup C2|}$   dove  $C1$  e $C2$  sono due insiemi di caratteristiche (features) degli item.

![image.png](4%202-Trovare%20elementi%20simili/image.png)

**Utilizzo della Similarità:**

- **Trovare documenti simili** (es. rilevamento di plagi o clustering di articoli).
- Per suggerire contenuti, trovando utenti simili o prodotti correlati (= **Filtri collaborativi**)
</aside>

---

# Pipeline per identificare items simili

![image.png](4%202-Trovare%20elementi%20simili/image%201.png)

### 1) **Shingling**

Si converte un documento in una rappresentazione basata su coppie chiave-valore, isolando le **features** distintive del testo (← estraiamo delle caratteristiche dal documento).

- Si stabilisce una finestra di dimensione $K$ e la si fa scorrere lungo il documento, generando sotto-sequenze chiamate **k-shingles** (o **n-grams** se basate su parole).
    
    ![image.png](4%202-Trovare%20elementi%20simili/image%202.png)
    
    <aside>
    📌
    
    **Scelta di K:**
    
    - **K piccolo** → Troppi elementi in comune, anche tra documenti diversi.
    - **K grande** → Poche sovrapposizioni, rendendo difficile individuare similarità.
    </aside>
    

**N.B.** Esiste anche algoritmo chiamato **bag-of-word** in cui al posto di far slittare una finestra di K caratteri, si isolano le singole parole.

### 2) **Minhashing/ Metodo della signature**

È un metodo per ridurre la dimensione della rappresentazione di un documento, mantenendo la sua similarità rispetto ad altri documenti. MinHashing trasforma ogni documento in una **firma (signature)** compatta, preservando la similarità originale (→ riduce il numero di righe).

<aside>
💡

**Idea e funzionamento del minhashing**

Vogliamo una funzione $f(D_i)$ che applicata ad un vettore mi restituisca un solo valore: $f(D_i) \rightarrow h(D_i)$

Questo valore ( $h(D_i)$) deve essere uguale al valore che ottengo per un altra colonna se i due documenti sono simili: $h(D_i)=h(D_j)$ se  $sim(D_i,D_j)$ è **elevata**.

- Stabilisco una permutazione casuale delle righe della tabella delle feature
- Per ogni colonna (documento), si prende **l’indice della prima riga** in cui compare un **1**. Questo valore rappresenta la **firma MinHash** per quella permutazione.
- Si esegue un’altra permutazione delle righe e si ripete il punto precedente.
- L’insieme di queste permutazioni mi porta ad avere una matrice ridotta che mi **preserva la similarità** della matrice originale

![image.png](4%202-Trovare%20elementi%20simili/image%203.png)

Controlliamo venga mantenuta la similarità della matrice originale:

- **Matrice originale** → $sim(D_1,D_2)=\frac{3}{6}$
- **Matrice nuova** → $sim(D_1,D_2)=\frac{2}{3}$

Notiamo che la similarità è più o meno preservata. Quindi abbiamo ottenuto che  $pr(h(D_1)=h(D_2))=sim_j(D_1,D_2)$.

**N.B.** Non esiste un numero preciso di approssimazioni della similarità di Jaccard da fare. Più permutazioni esegui, più accurata sarà l’approssimazione.

</aside>

<aside>
📌

**Dimostrazione:  $pr(h(C_1)=h(C_2))=sim_j(C_1,C_2)$**

Analizziamo due documenti $C_1$ e $C_2$ rappresentati come colonne di una matrice binaria (dove 1 indica la presenza di un elemento) e indichiamo con:

- **Riga di tipo X** → la riga avente **1** in entrambe le colonne.
- **Riga di tipo Y** → la riga avente **1** in una colonna e **0** nell’altra.
- **Riga di tipo Z** → la riga ha **0** in entrambe le colonne(non conta per la similarità).

![image.png](4%202-Trovare%20elementi%20simili/image%204.png)

La **similarità di Jaccard** (=parte destra dell’eq.) è definita come: $Sim(S₁, S₂) = \frac{|S₁ \cap S₂|}{|S₁ \cup S₂|} = \frac{x}{x + y}$

Ora, se scegliamo una **permutazione casuale** delle righe e consideriamo il primo 1 che appare in ogni colonna come la firma **MinHash**:

- Se il primo **1** appartiene a una riga di **tipo X**, entrambi i documenti riceveranno la stessa firma → contribuisce alla similarità.
- Se il primo **1** appartiene a una riga di **tipo Y**, i documenti riceveranno firme diverse → non contribuisce alla similarità.

Dato che le righe sono permutate **casualmente**, la probabilità che il primo **1** sia di **tipo X** è proprio $\frac{x}{x+y}$ , cioè la **similarità di Jaccard**.

</aside>

<aside>

**Creazione delle permutazioni delle righe**

Per velocizzare il processo di creazione delle permutazioni si può decidere di permutare solo le prime $k$ righe.

Se una permutazione genera **infiniti** (nessun 1 nelle prime k righe):

- Scarto la permutazione
- Aumento **k** per coprire più righe (aumento il k per tutte le permutazioni)

![image.png](4%202-Trovare%20elementi%20simili/image%205.png)

</aside>

<aside>
💡

**Idea e funzionamento dell’universal hashing**

L’universal hashing è una funzione del tipo: $h(x)=(ax+b)\ mod\ P$  

Dove:

- $x$  rappresenta l’indice della riga
- $a$ e $b$ sono numeri interi scelti casualmente
- $P$ è un numero primo maggiore del numero di righe

Procedimento:

1. Si inizializza una matrice in cui tutti i valori sono impostati a infinito. Le righe rappresentano le diverse funzioni di hash applicate, mentre le colonne corrispondono ai documenti.
2. Si analizza riga per riga: se nella riga compare un **1**, si aggiorna la matrice delle signature con il valore della riga, solo se questo valore è minore di quello attuale.
3. Si ripete il processo per ogni riga della tabella di partenza.

Esempio:

![image.png](4%202-Trovare%20elementi%20simili/image%206.png)

</aside>

### 3) LSH (Locality Sensitive Hashing)

L’**LSH** è un meccanismo utilizzato per ridurre il numero di confronti tra i documenti, evitando di confrontare tutte le coppie e concentrandosi solo su quelle più probabilmente simili. Permette quindi di diminuire il numero di colonne **nella matrice delle signature.**

<aside>

**Esempio:**

![image.png](4%202-Trovare%20elementi%20simili/image%207.png)

</aside>

**Funzionamento:**

1. La m**atrice delle signature** viene suddivisa in $N$ **bande** (il numero di bande è un parametro del sistema), ogni banda contiene un numero fisso di righe.
2. Viene applicata una funzione di **hashing** a ciascuna banda, inserendo nello stesso **bucket**  documenti che nella banda considerata sono uguali (ad esempio, documenti con la stessa somma delle righe).

![image.png](4%202-Trovare%20elementi%20simili/image%208.png)

<aside>
🚨

**Problemi**

Il metodo LSH può presentare due tipi di problemi a causa del comportamento della funzione di hash $h(D)$:

- **Falso positivo:** si verifica quando la funzione di hash $h(D)$ assegna a uno stesso bucket documenti che in realtà non sono simili. Tuttavia, questo errore viene individuato nel successivo controllo della similarità all’interno del bucket.
- **Falso negativo:** si verifica quando documenti simili vengono assegnati a **bucket diversi**, impedendo il loro confronto. Questo errore è più critico perché non può essere corretto: i documenti non verranno mai considerati come candidati simili.
</aside>

<aside>
📌

**Osservazione sul numero di bande:**

**Più bande ci sono** → meno righe per banda = basta una piccola somiglianza per finire nello stesso bucket → **Maggiore probabilità di falsi positivi** 

**Meno bande ci sono →** più righe per banda = è più difficile che documenti simili finiscano nello stesso bucket → **Maggiore probabilità di falsi negativi**

</aside>

<aside>
➕

**Analisi matematica della tecnica delle bande (LSH)**

Consideriamo: 

- $b$ bande
- $r$ righe per banda
- $s$ similarità di Jaccard tra due documenti

Passaggi:

- Probabilità che due documenti abbiano la stessa firma in una riga della signature matrix  è $s$, per definizione del MinHashing.
- Probabilità che due documenti abbiano la stessa firma in tutte le righe di una banda → $s^r$.
- Probabilità che due documenti NON abbiano la stessa firma in almeno una riga della banda → $1 - s^r$ .
- Probabilità che due documenti NON abbiano la stessa firma in nessuna delle bande →  $(1 - s^r)^b$ .
- Probabilità che almeno una banda abbia tutte le righe uguali (quindi che i 2 documenti vadano nello stesso bucket) → $1 - (1 - s^r)^b$

Quindi la funzione che determina la probabilità che due documenti vengano messi nello stesso bucket segue un’**S-curve** (**curva sigmoide**):

![image.png](4%202-Trovare%20elementi%20simili/image%209.png)

Esempio:

Consideriamo 20 bande di 5 righe l’una ($b=20$ e $r=5$) e supponiamo di voler trovare documenti che abbiano almeno $sim(C_1,C_2)=0.8$.

- Probabilità che $C_1$  e $C_2$  siano identici in una banda (→ finiscano quindi nello stesso bucket): $(0.8)^5=0.328$
- Probabilità che C1 e C2 non siano mai identici in nessuna banda: $(1 - 0.328)^{20} = 0.00035$

Quindi se metto nello stesso bucket elementi con $s = 0.8$ → su 3.000 coppie di documenti, avrò solo 1 falso negativo.

In base alla scelta di $r$ e $b$, la probabilità che due documenti finiscano nello stesso **bucket** cambia in modo non lineare, seguendo una **S-curve:**

![image.png](4%202-Trovare%20elementi%20simili/image%2010.png)

</aside>