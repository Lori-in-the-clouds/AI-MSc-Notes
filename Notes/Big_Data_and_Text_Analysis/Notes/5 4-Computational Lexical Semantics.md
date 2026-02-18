# 5.4-Computational Lexical Semantics

Status: Done
Type: theory

La **Computational Lexical Semantics** è un’area della linguistica computazionale e del Natural Language Processing (NLP) che si occupa di sviluppare metodi computazionali per modellare il **significato delle parole**, sia singolarmente che nel contesto in cui appaiono.

Ecco alcuni esempi di aspetti considerati in questo campo:

- **Computing Word Similarity**: calcolo della similarità tra parole.
- **Computing Word Relations**: individuazione delle relazioni tra parole (es. sinonimia).
- **Word Sense Disambiguation**: disambiguazione del significato di una parola in base al contesto.
- **Semantic Role Labeling**: identificazione dei ruoli semantici delle parole in una frase.
- **Computing Word Connotation and Sentiment**: analisi della connotazione e del sentimento associato alle parole.

La lessicografia è un campo complesso, poiché deve tenere conto che:

- le parole possono assumere diverse declinazioni, mentre la loro forma base è chiamata *lemma*.
- alcune parole sono ononime, ovvero hanno più significati a seconda del contesto in cui vengono usate. Ad esempio, *mouse* può riferirsi sia a un animale che a un dispositivo informatico.

---

# Approcci per studiare il lexical semantic

## 1) Dizionari/Thesarus

Le macchine utilizzano dizionari chiamati **thesaurus**, che differiscono da quelli usati dagli esseri umani. Infatti, una macchina attribuisce significato a una parola solo quando può metterla in relazione con altre. Per questo, un thesaurus è una raccolta di parole organizzate secondo relazioni semantiche. 

Un esempio di thesaurus è **WordNet**, una base di dati lessicale:

| **Relazione** | **Anche chiamato** | **Definizione** | **Esempio** |
| --- | --- | --- | --- |
| Iperonimo | Superordinato | Dai concetti ai loro superordinati | *colazione* → *pasto* |
| Iponimo | Subordinato | Dai concetti ai loro sottotipi | *pasto* → *pranzo* |
| Iperonimo di istanza | Istanza | Dalle istanze ai loro concetti | *Austen* → *autore* |
| Iponimo di istanza | Has-Istances | Dai concetti alle loro istanze | *compositore* → *Bach* |
| Meronimo di membro | Has-Member | Dai gruppi ai loro membri | *facoltà* → *professore* |
| Olonimo di membro | Membro-Di | Dai membri ai loro gruppi | *copilota* → *equipaggio* |
| Meronimo di parte | Has-Part | Dai tutto alle loro parti | *tavolo* → *gambe* |
| Olonimo di parte | Part-of | Dalle parti ai loro tutto | *corso* → *pasto* |
| Meronimo di sostanza | - | Dalle sostanze alle loro sottoparti | *acqua* → *ossigeno* |

<aside>

**Metodi di disambiguazione del significato di una parola**

Quando una parola può avere più significati a seconda del contesto, è necessario determinare quale sia quello corretto.

Le principali tecniche utilizzate per la disambiguazione sono:

- **Supervised Machine Learning**, che sfrutta modelli addestrati su dati annotati.
- **Metodi basati su Thesaurus/Dizionari**, che utilizzano relazioni semantiche tra le parole (es. metodo di Lesk).

---

**Metodo di Lesk**

Questo approccio confronta il contesto della parola target con le definizioni dei suoi possibili significati presenti in un thesaurus. 

In particolare, il metodo esamina le *N* parole che precedono e seguono la parola target e le confronta con quelle in relazione ai 2 significati della parola target. Il significato con la maggiore sovrapposizione lessicale viene scelto come il più probabile.

Esempio:

Consideriamo la frase: *“The bank can guarantee deposits will eventually cover future tuition costs because it invests in adjustable-rate mortgage securities.”*

La parola **“bank”** può avere più significati:

1. **Banca** (istituzione finanziaria)
2. **Sponda** (di un fiume)

L’algoritmo di Lesk esamina le definizioni di ciascun significato e le confronta con le parole circostanti nella frase.

- Se la definizione di **“bank”** come **istituzione finanziaria** include parole come *depositi*, *investimenti*, *mutui* (presenti nella frase), allora questo significato avrà una maggiore sovrapposizione con il contesto e verrà scelto.
- Se la definizione di **“bank”** come **sponda di un fiume** include parole come *acqua*, *fiume*, *pesci* (assenti nella frase), allora sarà scartata.
</aside>

## 2) Vector semantics and embeddings

Si usano vettori per rappresentare il significato di una parola. Il vettore associato ad una parola si chiama **embedding**.

<aside>
💡

**Ipotesi Distribuzionale**

- Il significato di una parola è legato alla distribuzione delle parole che la circondano.
- Se due parole compaiono in contesti simili, è probabile che abbiano un significato simile.

Esempio:

Consideriamo le seguenti frasi:

![image.png](5%204-Computational%20Lexical%20Semantics/image.png)

Anche senza conoscere il significato di **tesguino**, possiamo dedurre che sia una bevanda fermentata simile alla birra, perché:

- Compare in frasi con parole come **bottle**, **drunk**, **likes**, **corn**.
- Altre parole con contesti simili (es. *beer*, *liquor*, *tequila*) confermano questa interpretazione.

</aside>

### Tecniche per costruire embeddings

1. **Rappresentazioni vettoriali sparse:** in questo caso, le parole sono rappresentate come vettori contenenti molti zeri, da cui il termine ‘sparsi’. Sono semplici da costruire, ma richiedono molta memoria.
    1. **Metodo matrice termine-documento** → le righe rappresentano i termini, mentre le colonne corrispondono ai documenti in cui tali termini appaiono. Ogni cella della matrice indica il numero di volte in cui un termine compare in un determinato documento.”
        
        ![image.png](5%204-Computational%20Lexical%20Semantics/image%201.png)
        
        Due parole sono simili se i loro rispettivi vettori sono simili.
        
    2. **Metodo matrice termine-termine** →  ****ogni riga rappresenta una parola del vocabolario, e lo stesso vale per le colonne. Nell’intersezione, indichiamo quante volte la parola della riga compare vicino alla parola della colonna all’interno del documento. Per determinare la vicinanza, si utilizza una **finestra di contesto** (**window**), la cui dimensione è generalmente compresa tra 1 e 8 parole. Una finestra più corta (± 1-3 parole) porta a una rappresentazione più sintattica, mentre una finestra più ampia (± 4-10 parole) favorisce una rappresentazione più semantica.
        
        ![image.png](5%204-Computational%20Lexical%20Semantics/image%202.png)
        
        In realtà, le celle delle matrici possono essere riempite con altro:
        
        - **Conteggio delle occorrenze**
        - **Tf-idf → $tf=\text{term frequency},\  idf=log(\frac{N}{df})\$** con $N$ numero totale di documenti nella collezione  ****→ Per ogni word $i$ nel documento **$j$: $w_{ij}=tf_{ij}\cdot idf_{ij}$**
        - **Pointwise mutual information → $I(x,y)=log_2\frac{P(x,y)}{P(x)P(y)}$** ( $P(x,y)$ = prob. che 2 parole appiano vicine nel doc , $P(x)P(y)$ = prob. che le 2 parole appaiano)
        - **Positive pointwise mutual information** → $PPMI(w,c) = max(log_2\frac{P(w,c)}{P(w)P(c)},0)$ (=versione di pointwise mutual information che elimina valori molto bassi)
2. **Rappresentazioni vettoriali dense:** le parole sono rappresentate da vettori di dimensioni più ridotte dove ogni componente contiene un numero diverso da zero. Questi vettori condensano l’informazione semantica in poche dimensioni, rendendoli più compatti e performanti. 
    1. **Tecnica di riduzione della dimensionalità →** si applicano metodi di riduzione sulle matrici ottenute con rappresentazioni vettoriali sparse, comprimendo il numero di colonne. Esempi di queste tecniche sono:
        - PCA: ruota gli assi dello spazio in cui sono rappresentati i dati, scegliendo nuove direzioni che catturano meglio la variazione nei dati.
        - SVD: la matrice $A$ viene decomposta in un prodotto di matrici: $A = U \Sigma V^T$ dove:
            - **$U$ e $V$** sono matrici ortonormali
            - $\Sigma$ è diagonale, con i valori sulla diagonale in ordine decrescente.
            
            Ponendo a zero i valori più piccoli della diagonale di $\Sigma$, non otterrò più la matrice di partenza $A$, ma una sua approssimazione $A_k = U_k \Sigma_k V_k^T$ che risulterà più **compatta** e **densa**.
            
    2. **Modelli basati su reti neurali** ad esempio l’algoritmo **Word2vec**, che presenta due versioni: skip-gram e CBO.
        
        <aside>
        💡
        
        **Definizione skip-gram**
        
        È una tecnica proposta nel 2013 che utilizza il machine learning con un modello di regressione logistica (=per vedere se due parole sono uno vicino all’altro). A differenza di altri approcci che cercano di predire la parola successiva (il che implicherebbe un numero elevato di classi), Skip-gram valuta la probabilità che una parola appartenga al contesto di un’altra, configurandosi come un problema di classificazione binaria. Dopo l’addestramento, il modello viene ignorato e si utilizzano solo i pesi per generare gli embeddings. Questi embeddings sono ‘statici’, ovvero associano a ogni parola una rappresentazione basata sul contesto senza considerare la possibilità che la parola abbia più significati.
        
        </aside>
        
        <aside>
        📌
        
        **Come viene allenato?**
        
        Ad ogni parola del vocabolario viene associato un vettore numerico. Per determinare la coerenza tra le parole, calcoliamo la distanza tra i loro vettori: se due parole appartengono allo stesso contesto, la distanza dovrà essere ridotta; altrimenti, sarà maggiore.
        
        La similarità tra due vettori viene spesso rappresentata tramite il **prodotto scalare**: più alto è il valore del prodotto tra due vettori, maggiore è la loro somiglianza. Tuttavia, invece di utilizzare direttamente questa misura di similarità (score), il modello applica la **funzione sigmoide** alla similarità per ottenere una probabilità di appartenenza al contesto:
        
        $$
        P(+|w,c) =\sigma(c\cdot w)= \frac{1}{1+e^{-c\cdot w}} \quad P(-|w,c) = 1-P(+|w,c) =\sigma (-c\cdot w) =\frac{1}{1+e^{c\cdot w}}
        $$
        
        Ora, assumendo che le parole nel contesto siano tra loro indipendenti, possiamo calcolare la probabilità che la parola $w$ appartenga al contesto $C$ semplicemente moltiplicando le probabilità individuali associate a ciascuna parola del contesto:
        
        $$
        P(+|w,c_{1:C}) = \prod_{i=1}^{C} \sigma(c_i\cdot w)= \sum_{i=1}^{C}log(\sigma(c_i \cdot w))
        $$
        
        Skip-gram in realtà memorizza due embedding per ogni parola, uno per la parola come target e uno per la parola considerata come contesto. Quindi i parametri che dobbiamo apprendere sono due matrici $W$ e $C$, ognuna contenente un embedding per ognuna delle $|V|$ parole nel vocabolario $V$:
        
        ![image.png](5%204-Computational%20Lexical%20Semantics/image%203.png)
        
        Dopo la fase di training si utilizzano come embeddings solamente la matrice $W$  (oppure $W+C)$
        
        </aside>
        
    3. **Embeddings contestualizzati:** rappresentano un’evoluzione rispetto agli embeddings statici, poiché tengono conto del contesto in cui una parola appare.
        
        A differenza dei modelli tradizionali come Word2Vec o GloVe, che assegnano a ogni parola un’unica rappresentazione fissa, gli embeddings contestualizzati generano rappresentazioni che **variano a seconda del contesto**. Questo permette di **risolvere le ambiguità lessicali** e comprendere meglio il significato di una parola in base al suo utilizzo effettivo. Ad esempio, **BERT (Bidirectional Encoder Representations from Transformers)** e **GPT (Generative Pre-trained Transformer)**.
        

### **Proprietà Semantiche degli Embeddings**

Una proprietà fondamentale degli embeddings è la loro capacità di **catturare significati relazionali** tra le parole. Si può applicare un’**analogia** tra le parole sfruttando la riduzione dimensionale con PCA. Ad esempio, la **distanza** e **l’inclinazione** tra *uomo* e *donna* risultano simili alla distanza tra *king* e *queen.*

![image.png](5%204-Computational%20Lexical%20Semantics/image%204.png)

---