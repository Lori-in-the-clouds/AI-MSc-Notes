# 9.3-BERT e GPT

Status: Done
Type: theory

# BERT

<aside>
💡

**BERT (Bidirectional Encoder Representations from Transformers)** è un modello basato sull’**encoder** di un Transformer che genera **embedding contestuali**. A differenza di modelli precedenti come **Word2Vec**, che rappresentano ogni parola come un’entità indipendente dal suo contesto (**context-free**), **BERT** è progettato per comprendere come il significato di una parola possa cambiare in base alle altre parole che la circondano.

</aside>

## **Caratteristiche principali:**

- **Context-based**: l’embedding di un token non rappresenta solo il token stesso, ma anche il suo **significato** nel contesto in cui appare.
    
    *Esempio:* Python può rappresentare il linguaggio di programmazione oppure il serpente → BERT è in grado di cogliere questa differenza. 
    
- **Bidirezionale:** a differenza di modelli unidirezionali, BERT considera sia il contesto precedente che quello successivo di un token.
    
    ![image.png](9%203-BERT%20e%20GPT/image.png)
    

## Struttura

![image.png](9%203-BERT%20e%20GPT/image%201.png)

## Pre-training

BERT è pre-addestrato utilizzando un approccio **denoising**, ovvero si introduce del rumore nell’input e si addestra il modello a ripristinare l’informazione originale. Questo pre-training avviene principalmente tramite 2 tecniche:

- **Masked Language Modeling (MLM) →**  alcune parole della frase vengono mascherate e si chiede al modello di prevederle. In particolare:
    1. Si seleziona casualmente il **15% dei token** in una frase.
    2. Per ogni token selezionato, si applica una delle seguenti operazioni:
        - **80% delle volte**, il token viene sostituito con il token speciale [MASK]
        - **10% delle volte**, il token viene sostituito con un altro token casuale del vocabolario
        - **10% delle volte**, il token rimane invariato

![image.png](9%203-BERT%20e%20GPT/2c45bbb7-0269-4fd5-b45b-fcba0b996021.png)

- **Next Sentence Prediction (NSP) →** al modello vengono presentate coppie di frasi e deve predire se sono consecutive o meno. In particolare:
    
    
    1. Al modello vengono fornite coppie di frasi, che possono appartenere allo stesso documento oppure una frase da un documento e l’altra da un documento random.
    2. Vengono inseriti due token speciali:
        - **[CLS]** all’inizio della prima frase, contiene una rappresentazione condensata del significato di tutta la frase
        - **[SEP]** tra le due frasi per separarle
    3. La rappresentazione finale del token [CLS] rappresenta la predizione.
    
    ![image.png](9%203-BERT%20e%20GPT/81b4197d-0bca-4b43-bd0c-0f24e195f5bd.png)
    

**N.B.** Quindi il modello viene allenato usando i due approcci. Con il primo impara il significato contestuale dei token e con il secondo il rapporto reciproco delle frasi.

## Tokenizer utilizzato da BERT

BERT utilizza un tokenizer chiamato **WordPiece**. Il processo di tokenizzazione avviene nel seguente modo:

- Se la parola è presente nel vocabolario → viene considerata come un unico token.
- Se la parola non è presente nel vocabolario → viene suddivisa in sottoparti e si applica una ricerca ricorsiva per trovare le sottosequenze più lunghe presenti nel vocabolario.

*Esempio:* 

La parola “pretraining” viene suddivisa in: “pre”, “##train”, “##ing” (dove il simbolo ## indica che il token è una sotto-parte di una parola più grande).

---

# Prompting, Instruction Finetuning, and PPO/RLHF (facoltativa)

## GPT (Generative Pre-trained Transformer)

GPT è un modello generativo di linguaggio, progettato per produrre testo. La sua architettura si basa sul principio dell’autoregressione, ovvero genera il testo token dopo token: ogni nuovo token prodotto viene aggiunto al testo di input per prevedere il successivo. È composto da:

- **Tokenizer**
- **Transformer**
- **ML Head**

![image.png](9%203-BERT%20e%20GPT/image%202.png)

<aside>

**Evoluzione delle versioni di GPT:**

- **GPT-1:** Ha dimostrato che l'addestramento del modello linguistico su larga scala con auto-supervisione, seguito da un fine-tuning specifico per il compito, può migliorare significativamente le prestazioni in vari compiti a valle.
- **GPT-2:** L'architettura è la stessa di GPT-1, ma con un aumento significativo della quantità di dati di addestramento (40 GB) e del numero di parametri (da 100 milioni a 1,5 miliardi). Un risultato importante è stato che, grazie alle dimensioni maggiori, il modello è diventato capace di ottenere buone prestazioni con il solo pre-training, riducendo o eliminando la necessità del fine-tuning. Questo ha portato all'approccio “***zero-shot”*** in molte applicazioni.
- **GPT-3:** Anche in questo caso, l'architettura rimane simile, ma con un ulteriore aumento dei parametri (175 miliardi) e dei dati di addestramento (oltre 600 GB). Si è osservato che le prestazioni di questi modelli possono essere ulteriormente migliorate fornendo esempi nel prompt di input, una tecnica chiamata "***few-shot learning***".
</aside>

## Prompting

Il **prompting** implica l’utilizzo di un input strutturato per ottenere un output desiderato senza modificare il modello sottostante. Esistono diverse tecniche di prompting:

- **Zero-shot →** si fornisce al modello una richiesta senza esempi aggiuntivi. Il modello deve rispondere basandosi esclusivamente sulle conoscenze acquisite durante il pre-training. Questa tecnica è applicabile a tutti i modelli linguistici.
- **One-shot →** si fornisce un singolo esempio, seguito dalla domanda principale. Anche questa tecnica è generalmente applicabile a tutti i modelli.
- **Few-shot →** si forniscono molteplici esempi prima della domanda principale. L'efficacia di questa tecnica aumenta significativamente con modelli di dimensioni maggiori (con molti parametri).
- **Chain of thought (CoT) →** si richiede al modello di mostrare il processo di ragionamento passo dopo passo, invece di fornire direttamente la risposta finale. Anche questa tecnica tende a funzionare meglio con modelli più grandi.
- **Self-consistency →** si chiede al modello di generare diverse risposte alla stessa domanda e poi si seleziona quella migliore in base alla coerenza.
- **Least-to-most →** si chiede al modello di scomporre un problema complesso in sotto-problemi più semplici, risolvendoli in ordine di difficoltà crescente.
- **Prompting tuning →** Invece di modificare direttamente i parametri del modello, si ottimizzano gli "embedding" utilizzati per rappresentare il prompt. Si aggiungono al prompt dei token speciali, chiamati "soft prompt", che non corrispondono a token di parole reali. Questi soft prompt vengono ottimizzati durante l'addestramento, mantenendo fissi gli altri parametri del modello. Questo approccio si colloca tra il fine-tuning completo e il prompt design manuale.
    
    <aside>
    
    Esempio:
    
    Immagina di avere una frase come: "*Il cielo è [MASK] oggi.*"
    
    In un prompt tradizionale, sostituiremmo ‘*[MASK]*’ con una parola del vocabolario, come ‘*blu*’ o ‘*nuvoloso*’. 
    
    Con il prompt tuning, invece, aggiungiamo dei soft token al prompt. Ad esempio, potremmo inserire due soft token prima di ‘[MASK]’:
    
    ‘*SoftToken1 SoftToken2 Il cielo è [MASK] oggi.*’
    
    Durante l’addestramento, i valori di ‘SoftToken1’ e ‘SoftToken2’ vengono ottimizzati tramite backpropagation, in modo da aumentare la probabilità che il modello preveda ‘*blu*’ se vogliamo che parli del colore del cielo, o ‘*nuvoloso*’ se vogliamo che si riferisca al meteo.
    
    </aside>
    

## Instruction Fine-Tuning

È un processo simile al fine-tuning tradizionale, ma con un focus specifico sull'addestramento del modello su un dataset contenente $coppia(\text{istruzione},\text{risposta desiderata})$. 

<aside>
💡

**Self-Instruct**

Consiste nella generazione di nuovi dati di addestramento a partire da un piccolo set iniziale di domande scritte dall’essere umano. Il modello genera risposte ai nuovi dati, che vengono valutate. Successivamente, sia le istruzioni che le risposte vengono aggiunte al dataset per un ulteriore fine-tuning.

</aside>

## Retrieval-Augmented Generation (RAG)

RAG combina il recupero di informazioni con la generazione testuale. Funziona così:

1. Data una domanda, il modulo *retrieval* confronta la query con segmenti di documenti (chunk).
2. Recupera i chunk più rilevanti.
3. Fornisce questi documenti al modello, che li utilizza per generare una risposta contestualizzata.

![image.png](9%203-BERT%20e%20GPT/image%203.png)

## **Reinforcement learning with human feedback (RLHF)**

Si addestra un modello di "reward" utilizzando dati che includono feedback umano sulla qualità delle risposte. Il modello principale viene poi ottimizzato per massimizzare il punteggio fornito dal modello di reward, in modo da generare risposte che siano considerate di alta qualità dagli esseri umani.

---