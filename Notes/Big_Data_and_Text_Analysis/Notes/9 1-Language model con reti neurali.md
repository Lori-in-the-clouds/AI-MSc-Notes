# 9.1-Language model con reti neurali

Status: Done
Type: theory

# Ripasso

<aside>

### **Neurone**

Le reti neurali sono costituite da unità computazionali che ricevono input numerici, eseguono calcoli e producono un output. Ogni unità è caratterizzata da pesi e da un bias, e la somma pesata degli input viene trasformata attraverso una funzione di attivazione, come la **sigmoide** ( **$y=\frac{1}{1+e^{-z}}$**), la **tangente iperbolica** ( $y=\frac{e^z-e^{-z}}{e^z+e^{-z}}$) o la **ReLU** ( $y=max(x,0)$).

$$
y=\sigma(\omega\cdot x+b)=\frac{1}{1+e^{-(\omega\cdot x+b)}}
$$

![image.png](9%201-Language%20model%20con%20reti%20neurali/image.png)

</aside>

---

# **Reti neurali feed-forward (FFNN) per NLP**

Le **reti neurali feed-forward (FFNN)** sono reti neurali multilayer senza cicli, in cui **ogni neurone di un livello è connesso a tutti i neuroni del livello successivo**. Sono spesso utilizzate nel **Natural Language Processing (NLP)** per compiti come classificazione del testo e modellazione del linguaggio.

Tuttavia, le reti neurali **non possono lavorare direttamente con il testo**, quindi le parole devono essere trasformate in numeri. Questo si ottiene tramite tecniche di **embedding**, che convertono le parole in vettori numerici.

![image.png](9%201-Language%20model%20con%20reti%20neurali/image%201.png)

<aside>

### **Embedding dell’input**

- Ogni parola viene inizialmente rappresentata tramite un **vettore one-hot** di dimensione  $|V|$, dove $V$ è il numero di parole nel vocabolario.
    
    *Ad esempio, in un vocabolario di **10.000 parole**, ogni parola viene rappresentata da un vettore binario di **10.000 dimensioni**, con un unico valore pari a **1** nella posizione corrispondente alla parola.*
    
- Costruiamo la matrice degli embedding con il ***wordToVec** c*he riduce la dimensionalità e cattura le relazioni tra parole. La **matrice di embedding** ha dimensione $(N \times |V|)$ con:
    - $V$ colonne → numero di parole nel vocabolario
    - $N$ righe → dimensione desiderata del word embedding, tipicamente molto più piccolo di $|V|$
- Per ottenere l’embedding di una parola, si moltiplica il **vettore one-hot** per la **matrice di embedding**, ottenendo un vettore numerico compatto.

![image.png](9%201-Language%20model%20con%20reti%20neurali/image%202.png)

</aside>

<aside>

### **Gestione della variabilità della lunghezza delle frasi**

Un problema tipico nel NLP è che le frasi hanno lunghezza variabile, mentre una rete neurale richiede un input di dimensione fissa. Per risolvere questo problema, si utilizzano tecniche di **pooling**, come:

- **Mean pooling**: calcola la media degli embedding delle parole
- **Max pooling**: seleziona il valore massimo tra gli embedding

Queste tecniche producono un **vettore di dimensione fissa**, indipendentemente dalla lunghezza della frase.

</aside>

## **Struttura della rete**

Una FFNN è composta da tre livelli principali:

1. **Strato di input →** riceve i dati sotto forma di vettore numerico, ottenuto dopo il **pooling**  degli embedding. 
2. **Strati nascosti →** composti  da neuroni completamente connessi con funzioni di attivazione non lineari (ReLU, Tanh, Sigmoide). Per ogni singolo strato nascosto: $h_j = \sigma(\sum_{i=1}^{n_0}W_{ji}x_i+b_j)$
3. **Strato di output:**
    - Per la **classificazione del sentiment** (positivo, negativo, neutro) si utilizzano **3 neuroni in uscita**.
    - Per problemi con **molte classi** (es. previsione della parola successiva in un vocabolario da 5000 parole), si avranno **5000 neuroni in uscita**
    
    L’output è ottenuto come $z = Uh$, con $U$ come matrice dei pesi dello strato finale. Per normalizzare le probabilità tra le classi, si utilizza la **funzione softmax**: $sofmax(z_i)=\frac{e^{z_i}}{\sum_{j=1}^{d}e^{z_j}}$  con $1 \le i \le d$.
    

<aside>
💡

**Una delle principali potenzialità delle FFNN** è la loro capacità di funzionare con un numero **molto elevato di classi in output**. Questa caratteristica le rende particolarmente potenti, poiché consentono di classificare fino a **migliaia di categorie** senza compromettere l’efficacia del modello.

</aside>

<aside>
🚨

### Problema

Le reti feed-forward non gestiscono bene le dipendenze a lungo raggio in quanto considerano solo un numero fisso di parole precedenti. Questo significa che, se dobbiamo prevedere una parola basandoci sul contesto, il modello potrebbe ignorare informazioni cruciali che si trovano più lontano nella frase, rendendo difficile la comprensione di strutture linguistiche complesse.

</aside>

---

# Transformers

I transformers sono un’architettura di rete neurale composta da due moduli:

- **Encoder** → prende un input e genera una serie di embeddings
- **Decoder** → prende gli embeddings e genera una serie di parole

![image.png](9%201-Language%20model%20con%20reti%20neurali/image%203.png)

## Architetture dei transformers

Esistono **tre principali architetture** di Transformer:

- **Encoder-only** → utilizzati per la comprensione del linguaggio, ad esempio per analizzare il significato dei token o classificare il testo. Questi modelli elaborano l’input tutto in una volta, senza produrre nuove parole.
    
    Un esempio noto è **BERT (Bidirectional Encoder Representations from Transformers):**
    
    ![image.png](9%201-Language%20model%20con%20reti%20neurali/27dbc226-e410-4b01-b4df-48290f32d74b.png)
    
- **Decoder-only →** progettati per generare testo in modo autoregressivo, il che significa che generano una parola alla volta. Dopo aver prodotto una parola, questa viene aggiunta all’input, consentendo al modello di prevedere la successiva.
    
    Questo tipo di approccio è utilizzato in modelli come GPT (**Generative Pre-trained Transformer**), che è molto popolare per applicazioni come la generazione di testo, i chatbot, il completamento automatico e la creazione di codice. Il modello si basa esclusivamente sulla parte di decoder dell’architettura Transformer, senza utilizzare un encoder.
    
    ![image.png](9%201-Language%20model%20con%20reti%20neurali/image%204.png)
    
- **Encoder-Decoder** →  utilizzati per compiti che richiedono sia la comprensione che la generazione del linguaggio. In questi modelli, l’encoder elabora l’input e lo trasforma in una rappresentazione compatta, che viene poi passata al decoder. Il decoder utilizza questa rappresentazione per generare l’output sequenziale. Un esempio di questo tipo di architettura è T5 (Text-to-Text Transfer Transformer), che viene impiegato in compiti come la traduzione automatica, la creazione di riassunti.

## Architettura dei transformers decoder-only

Il transformer del decoder only è costituito da 3 moduli:

1. **Tokenizer** → si occupa di trasformare il testo grezzo in una sequenza di token numerici. 
    1. Il tokenizer crea un vocabolario in cui ogni token è associato a un **token ID** ($[token,ID\_token]$). 
    2. Si utilizza questo vocabolario per passare dal testo originale a una sequenza di token numerici.
    3. I token numerici vengono poi mappati negli **embeddings** tramite una matrice di embedding $W_e$, che ha dimensione $(N \times  V)$, dove $V$ è il numero di token nel vocabolario e $N$ è la dimensione dell’embedding. La matrice $W_e$ viene inizialmente creata con valori casuali e successivamente aggiornata durante l’addestramento del modello.
    4. Ogni **embedding** rappresenta una coordinata in uno spazio, e i token con significato simile tendono a trovarsi in posizioni vicine nello spazio di embedding. Questo permette al modello di cogliere relazioni semantiche tra i token.
    
    <aside>
    💡
    
    **Context size:** numero di token che è in grado di gestire il transformer. 
    
    </aside>
    

![image.png](9%201-Language%20model%20con%20reti%20neurali/image%205.png)

![image.png](9%201-Language%20model%20con%20reti%20neurali/e4c0e690-5d49-417a-afe1-68edd2348f3c.png)

1. **Stack of transformer blocks** → questo stadio si occupa di processare in parallelo gli embedding attraverso i **transformer block.**
    
    ![image.png](9%201-Language%20model%20con%20reti%20neurali/image%206.png)
    
2. **LM head** → prende l’embedding dell’ultimo blocco del Transformer e lo moltiplica per la matrice di unembedding $U$. Da questo otteniamo un vettore di dimensione $(1 \times V$), dove $V$ è il numero di token nel vocabolario. Applichiamo la **softmax** su questo vettore di dimensione $(1 \times V)$ per ottenere una distribuzione di probabilità sui token del vocabolario, quindi selezioniamo il token corrispondente alla probabilità più alta. Questo token rappresenta la parola generata.
    
    ![image.png](9%201-Language%20model%20con%20reti%20neurali/image%207.png)
    

## Approfondimento: embeddings

Per ogni input si possono calcolare due tipi di embedding:

- **Token embedding**: Il token embedding è la rappresentazione iniziale di ogni token, che poi viene aggiornata in base al contesto che lo circonda. Il Transformer avrà una matrice di embedding dei token, con tante righe quanti sono i token conosciuti dal modello e tante colonne quante sono le dimensioni degli embedding. Questa matrice è inizializzata in modo casuale e poi viene appresa durante l’allenamento.
- **Positional embedding**: Siccome le operazioni sui token avvengono in parallelo dobbiamo memorizzare la loro posizione nella frase.
    
    *Esempio*: “*il topo mangia il gatto*” e “*il gatto mangia il topo*” sarebbe la stessa frase senza il positional.
    
    Il positional embedding rappresenta la posizione di un token all’interno di una sequenza. Il problema dei positional embedding è che ci sono meno dati per i token nelle posizioni finali di una sequenza di lunghezza massima, il che può ridurre l’accuratezza per questi token. 
    
    *Esempio: s*upponiamo di avere una frase molto lunga composta da 1000 parole e che il modello stia cercando di prevedere una parola alla posizione 1000 della sequenza → durante l’allenamento, il modello potrebbe aver visto solo un numero limitato di sequenze lunghe, per cui il modello non ha visto tante **parole nella posizione 1000.**
    
    Il Transformer avrà una matrice di embedding posizionali, con tante righe quante sono le possibili posizioni all’interno di una sequenza e tante colonne quante sono le dimensioni degli embedding. Anche questa matrice è inizializzata casualmente e poi appresa.
    

Solitamente, per ogni token di input → $\text{embedding}=\text{token embedding}\ +\ \text{positional embedding}$

## Approfondimento: **Transformer block**

Il transformer block è composto da 3 componenti:

- **Self-attention**: permette al modello di focalizzarsi sui token più rilevanti del contesto, calcolando quanto ogni token sia correlato agli altri. A differenza delle RNN, i transformer non dipendono sequenzialmente dall’output precedente, permettendo calcoli in **parallelo** e riducendo i tempi di elaborazione.
    
    <aside>
    
    **Single-Head Self Attention**
    
    1. Il trasformatore introduce 3 matrici $W^Q$, $W^K$, $W^V$ calcolate durante l’allenamento che sono utilizzate per generare 3 tipi di embeddings:
        - **Key** ($K$) → rappresenta gli input passati usati per calcolare l’attenzione: $k_i=x_i\cdot W^K$
        - **Query** ($Q$) → rappresenta l’input corrente: $q_ j= x_j\cdot W^Q$
        - **Value** ($V$) → valore utilizzato per il calcolo dell’output corrente: $v_i=x_i \cdot W^V$
        
        **N.B**. $q_j$ e $k_i$ non sono altro che rappresentazioni di $x_i$ in uno spazio vettoriale di dimensione inferiore.
        
    2. Calcoliamo la similarità tra le query e tutte le key con il **dot product normalizzato** (con $d_k$ dimensione della matrice)**: $score(x_i,x_j)= \frac{q_j\cdot k_i}{\sqrt{d_k}}$**  → (Maggiore è lo score, più i due token sono correlati).
    3. Convertiamo gli score in probabilità per ottenere l’importanza di ogni token rispetto alla query tramite softmax: $\alpha_{ij}=softmax(score(x_i,x_j)) \ \ \forall i \le j$
    
    4. I pesi ottenuti vengono moltiplicati per i value corrispondenti e sommati per generare l’output finale: $a_j= \sum_{i \le j}\alpha_{ij}v_i$
    
    **N.B.**  ****Il calcolo viene effettuato solo per $\forall i \leq j$ perché si considerano esclusivamente le parole precedenti. Se l’obiettivo è generare testo, non si deve avere accesso alle parole future.
    
    Esempio:
    
    ![image.png](9%201-Language%20model%20con%20reti%20neurali/image%208.png)
    
    </aside>
    
    <aside>
    💡
    
    **Multi-Head Self-Attention**
    
    Nel meccanismo **Multi-Head Self-Attention**, più strati di **self-attention** vengono calcolati in parallelo, ciascuno con pesi diversi. Ogni singolo strato è chiamato **head** e cattura informazioni diverse dalla sequenza di input. 
    
    ![image.png](9%201-Language%20model%20con%20reti%20neurali/image%209.png)
    
    </aside>
    
    <aside>
    
    Parallelizzazione:
    
    - L’intero processo di self-attention può essere parallelizzato raggruppando gli embedding in una matrice $X$ di dimensione $(N \times d)$, dove ogni riga rappresenta un token.
    - Le matrici dei pesi $W^Q$, $W^K$, $W^K$ trasformano $X$ nelle matrici $Q$, $K$, $V$.
    - La matrice $QK^T$ calcola le correlazioni tra tutti i token, ma nei **modelli generativi** si annullano le parti superiori della matrice per evitare di usare informazioni future.
    </aside>
    
- **Rete feed forward:** si occupa di estrapolare una nuova informazione dall’embedding.
- **Normalization layer:** conferisce ai valori dell’output un intervallo più stabile.
    1. Ogni attivazione del neurone viene normalizzato: 
    
    $\hat{x} = \frac{x - \mu}{\sigma}$
    2. Si applicano due parametri di **scaling** e **shifting: 
    
    $y = \gamma \hat{x} + \beta$**

<aside>
💡

**Residual Connection**

A ogni strato non si passa solo l’output elaborato, ma anche l’input originale, sommando i due valori. Questo aiuta a **preservare le informazioni** anche quando il modello diventa molto profondo, evitando la degradazione del segnale durante l’addestramento:

$$

\text{Output} = \text{Layer}(\text{Input}) + \text{Input}
$$

(Evita il *vanishing gradient*)

</aside>

<aside>

**Riassunto funzionamento:**

$$
O=\text{LayerNorm}(X+\text{SelfAttention(X)}) \\ H = \text{LayerNorm(O+FFN(O))}
$$

![image.png](9%201-Language%20model%20con%20reti%20neurali/8c7cc339-003f-46a8-a28a-90181a6ad58b.png)

</aside>

---