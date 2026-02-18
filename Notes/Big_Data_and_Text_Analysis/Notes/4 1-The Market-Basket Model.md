# 4.1-The Market-Basket Model

Status: Done
Type: theory

<aside>
💡

Il **Market Basket Model** è un approccio utilizzato per individuare le **regole di associazione** tra insiemi di prodotti (baskets) e singoli articoli (items). È nato con l’obiettivo di identificare relazioni tra gli acquisti dei clienti, determinando quali prodotti vengono spesso comprati insieme.

Le **regole di associazione** sono nella formula → $if\ \ I\ \ then\ \ K$

(Dove $I$ rappresenta un insieme di **articoli acquistati**, e $K$ indica **un altro articolo che viene acquistato** con una certa probabilità quando $I$ è presente nel carrello).

Il modello assume che:

- il numero di items presenti dentro un basket sia più piccolo del numero totale di items del negozio
- il numero di basket venduti siano molti
</aside>

---

# Processo di creazione delle associations rules

1. **Trovare itemset frequenti:** un itemset può essere composto da un singolo articolo, una coppia, una tripletta e così via.  Un itemset $I$ è considerato **frequente** se: 
    
    $$
    support(I)>S\quad 
    $$
    
    Dove $S$ *è la **soglia** (scelta empiricamente).* Il **supporto** di un itemset $I$ è la probabilità di trovare $I$ nei carrelli.
    
    <aside>
    🚨
    
    Il calcolo della **frequenza** di un itemset è un’operazione **complessa** sia in termini di **memoria** che di **potenza computazionale**. Siccome itemset può essere un singolo articolo, una coppia, una tripletta e così via, il numero di combinazioni cresce esponenzialmente con il numero di articoli nel dataset.
    
    </aside>
    
2. **Creare regole di associazione a partire da itemset frequenti:** dato un itemset frequente *F*, si possono creare **regole** nella forma $I \rightarrow j$ dove $I$ è un sottoinsieme di $F$ e $j$ un elemento di $F$.
3. **Associare una misura di interesse ad ogni regola e selezionare solo le più interessanti:** a ogni regola si associa una misura di interesse chiamata **confidenza**, che rappresenta la probabilità che, dato $I$, sia presente anche $j$.
    
    $$
    conf(I \rightarrow j)= \frac{support(I \cup j)}{support(I)}
    $$
    
    Un’alta **confidenza** di una regola di associazione non implica necessariamente che la regola sia interessante. Ad esempio, se quasi tutte le persone che fanno la spesa in un supermercato acquistano una busta, la regola *“Se un cliente compra qualsiasi prodotto, allora compra anche una busta”* avrà una confidenza elevata. Tuttavia, questa associazione non fornisce informazioni utili, perché l’acquisto della busta è un comportamento quasi scontato.
    
    Per distinguere le regole veramente interessanti da quelle banali, si utilizza anche una **misura di interesse: $Interest(I \rightarrow j)=|\ conf(I\rightarrow j)-Pr[j]\ |$**
    

---

# Focus sul trovare Itemset frequenti (punto 1)

<aside>
📌

Se la soglia di **supporto minimo** è bassa, verranno identificati molti prodotti come **frequenti**, il che porta alla generazione di un numero elevato di regole. Al contrario, con una soglia più alta, solo pochi itemset saranno considerati frequenti. Di solito, si preferisce una **soglia alta** per evitare un eccessivo numero di regole, rendendo l’analisi più gestibile e significativa.

![image.png](4%201-The%20Market-Basket%20Model/image.png)

</aside>

<aside>
💡

**Principio di monotonicità**

Se un insieme di elementi è frequente, allora tutti i suoi sottoinsiemi devono essere frequenti. 

Di conseguenza, se una coppia di elementi **AB** non è frequente, non esisterà alcuna terna che la contiene (es. **ABC**) ad essere frequente. Allo stesso modo, se un singolo elemento **A** non è frequente, nessuna coppia o gruppo contenente **A** potrà esserlo.

Quindi le coppie sono più numerose delle terne, che a loro volta sono più numerose delle quaterne e così via. Per questo motivo, nella ricerca degli **itemset frequenti**, l’attenzione si concentra principalmente sulle **coppie**, dato che sono più facili da individuare e rappresentano la base per generare insiemi più grandi.

</aside>

## Procedimento naive per trovare itemset frequenti

1. **Leggo dal file basket-by-basket (salvato nel disco fisso) ←** fase che richiede più tempo
2. **Per ogni basket genero tutte le coppe degli elementi contenuti nel basket** 
3. **Per ogni coppia generata, si incrementa il contatore associato alla sua frequenza di apparizione**

**Esempio utilizzo della memoria:**

Se un algoritmo deve contare tutte le coppie di elementi e ci sono $n$ elementi distinti, allora lo spazio necessario per memorizzare i conteggi sarà: $\binom{n}{2}=\frac{n(n-1)}{2} \approx \frac{n^2}{2}$

Se ogni conteggio è memorizzato come un **intero a 4 byte**, allora il requisito di memoria totale è $4\cdot \frac{n^2}{2}=2n^2$ **byte**.

Supponendo di avere una memoria di **8 gigabyte** ($8\cdot2^{32}$→ $2^{33}$ byte), per poter gestire $n$ elementi senza superare la memoria disponibile, imponiamo: $2n^2 

\leq

 2^{33} \rightarrow n \leq 2^{16}$

<aside>
🚨

**Collo di bottiglia di questo meccanismo**

1. Utilizzo di memoria per salvare la frequenza di tutte le coppie viste in tutti i basket.
2. Computazioni da fare per creare tutte le coppie di items contenute in ogni basket.

</aside>

<aside>
✅

**Soluzione al collo di bottiglia: memoria → metodi per salvare in modo efficiente le coppie di items**

1. **Triangular matrix method:** si rappresentano gli item con indici da $1$ a $n$ e si considerano solo le coppie distinte $(i, j)$ con  $i < j$. Invece di usare una matrice bidimensionale, queste coppie vengono salvate in un **array monodimensionale**, risparmiando spazio. 
    
    La posizione k nell’array che corrisponde alla coppia (i, j) si calcola con la formula: $k = (i - 1) \cdot (n-\frac{i}{2}) + j - i$, e il suo valore rappresenta il **conteggio** di quella coppia.
    
    Riassumendo: $(i,j) \rightarrow k \rightarrow A[k]$
    
    **Pro:** molto compatta
    
    **Contro:** se alcune coppie non vengono mai utilizzate, la memoria allocata potrebbe rimane sprecata.
    
2. **Triplets method:** invece di usare una matrice o un array monodimensionale (come nel **Triangular Matrix Method**), ogni coppia $(i,j)$ viene memorizzata come una **tripla**: $(i, j, \text{count})$
    
    **Pro:** se non ho una coppia di valori non lascio spazio vuoto. Inoltre non devo rinumerare i singoli items, gli indici possono essere qualsiasi valore (anche stringhe o identificatori arbitrari).
    
    **Contro:**  utilizzo 3 interi per ogni coppia al posto di usarne solo 1.
    
</aside>

<aside>
✅

**Soluzione al collo di bottiglia: computazioni → algoritmi per calcolare il numero di coppie di items frequenti**

1. **Metodo a-priori:**
    - Creo un array di **singoletti** candidati ad essere frequenti
    - Controllo quali sono effettivamente frequenti (**elimino** quelli sotto soglia → questo perché per il **principio di monotonicità**, se un item non è frequente, nessuna coppia contenente quell’item potrà esserlo).
    - Genero array di **coppie** candidate ad essere frequenti a partire dai **singoletti** frequenti
    - Controllo quali sono effettivamente frequenti
    - ……
    
    **N.B.** Sfrutta il metodo di monotonicità ma all’**opposto** → se A non è frequente allora non esiste un insieme di elementi frequenti che comprende A.
    
    **Pro:**  riduce il numero di combinazioni da controllare (→ non genero itemset inutili).
    
    **Contro:** devo rileggere il file più volte, esattamente N volte, dove N è la dimensione massima degli itemset frequenti. (Non converge quasi mai, lentissimo)
    
    ![image.png](4%201-The%20Market-Basket%20Model/image%201.png)
    
2. **Metodo a-priori - Versione di Park, Chen e Yu (PCY):** mira a ridurre il numero di coppie candidate già dal primo passaggio, per questo il suo potenziale si esprime nella prima scansione del dataset.
    
    Durante questa fase, oltre al calcolo delle frequenze dei singoli item, si generano **le coppie effettivamente presenti nei basket** e, per ciascuna di esse, si calcola un valore hash che la assegna a un **bucket**. Ogni bucket tiene un conteggio del numero di coppie che vi finiscono.
    
    Al termine del primo passaggio, se il conteggio di un bucket è inferiore alla soglia minima di supporto, allora tutte le coppie al suo interno sono infrequenti
    
    ![image.png](4%201-The%20Market-Basket%20Model/image%202.png)
    
3. **Limited pass algorithm:** algoritmo approssimato, ovvero non trova tutti gli elementi frequenti ma solo alcuni. Questo significa che magari non trova tutte le regole, ma quelle che trova sono valide.
    - Estrae un sottoinsieme del dataset in modo tale che possa essere contenuto in memoria (es. 10%). In seguito usa qualsiasi metodo per trovare itemset frequenti nel sottoinsieme, con una soglia ridotta (es. 10% di S).
    - Controllo di non aver trovato falsi positivi (→ itemset frequenti nel sottoinsieme ma non nell’itemset totale). Nella seconda passata, verifico gli itemset trovati in precedenza aggiornando un dizionario con i loro conteggi scorrendo tutto il dataset.
        
        Non posso invece controllare di avere falsi negativi (→ quando un itemset è in realtà frequente nel dataset completo, ma l’algoritmo non lo rileva ) in quanto sto lavorando con un metodo approssimato, ma posso mitigarlo:
        
        1. Ponendo **S bassa** così individuo più candidati e poi elimino solo i falsi positivi.
        2. **Eseguendo l’analisi su più sottoinsiemi**, unendo i risultati per non perdere itemset frequenti.
4. **Algoritmo SON**
    - Divido il dataset in $N$ **chunk** per poter lavorare in parallelo. Ogni chunk ha una soglia ridotta $\frac{S}{N}$.
    - Calcolare gli **itemset frequenti** in ogni chunk. (Ogni chunk è salvabile in memoria quindi posso lavorarci facilmente).
    - Uniamo gli itemset frequenti trovati nei vari chunk.
    - Si effettua una seconda passata sui dati per calcolare la frequenza reale di ogni itemset candidato. Per ciascun chunk, si contano le occorrenze di ogni candidato e si sommano i risultati ottenuti nei diversi chunk. Se la frequenza totale supera la soglia stabilita, l’itemset viene confermato come frequente, altrimenti viene eliminato come falso positivo.
    
    **N.B.** I falsi positivi si verificano perché nella prima fase ogni chunk viene analizzato separatamente con una soglia più bassa (**S/N** invece di **S**).
    
    **N.B.** L’algoritmo garantisce di individuare tutti gli itemset realmente frequenti perché, se un itemset è frequente a livello globale, deve necessariamente esserlo anche in almeno uno dei chunk analizzati. Questo assicura che venga incluso tra i candidati. Nella seconda passata, il conteggio globale permette di confermare solo gli itemset che superano la soglia originale, evitando falsi negativi. Per questo motivo, SON è un **algoritmo esatto**.
    
    Si implementa utilizzando 2 **map-reduce**:
    
    ![image.png](4%201-The%20Market-Basket%20Model/image%203.png)
    
5. **Algoritmo di Toivonen**
    - Si estrae un sample del dataset e si identificano i candidati frequenti.
    - Si costruisce il **negative border**, cioè l’insieme di itemset che non risultano frequenti nel sample, ma i cui sottoinsiemi lo sono. (← in tutti i sottoinsiemi!)
        
        ![image.png](4%201-The%20Market-Basket%20Model/image%204.png)
        
    - Si analizza l’intero dataset per verificare la frequenza sia degli itemset candidati che di quelli nel **negative border**.
        - Se qualche itemset nel **negative border** risulta frequente, il sample non era rappresentativo e si ripete il processo con un nuovo sample.
        - Se nessun itemset del **negative border** è frequente, gli itemset trovati sono corretti.
</aside>

---