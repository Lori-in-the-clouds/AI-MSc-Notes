# 9.2-LLM per risolvere task di NLP

Status: Done
Type: theory

I **Large Language Model** sono modelli di deep learning, tipicamente basati su architetture Transformer, pre-addestrati su enormi quantità di testo. Questi modelli, capaci di estrarre informazioni da testi in linguaggio naturale, possono essere perfezionati per risolvere una vasta gamma di task.

---

# LLM per text conditional generation

Un LLM è in grado di effettuare generazione di testo condizionale. Fornendo un testo di input, l’LLM può generare nuovi token coerenti con il contesto, uno alla volta. Questa capacità consente di risolvere vari problemi, come ad esempio:

- **Sentiment analysis**
    
    <aside>
    
    *Esempio:*
    
    - $P(positive\ |\ Il\ sentiment\ della\ frase\ “I\ like\ Jackie\ Chan”\ è:)$
    - $P(negative\ |\ Il\ sentiment\ della\ frase\ “I\ like\ Jackie\ Chan”\ è:)$
    
    Se la probabilità della parola ‘positivo’ è maggiore, allora il sentiment della frase è positivo; altrimenti, lo consideriamo negativo.
    
    </aside>
    
- **Question answering**
- **Text summarization**

---

# Approcci per realizzare text conditional generation

La scelta della parola successiva può avvenire con i seguenti approcci:

- **Greedy decoding** **→** il nuovo token è sempre il più probabile, ma questo approccio tende a produrre testo generico e spesso ripetitivo: $\hat{w}_t=argmax_{w \in V}P(w|w_{<t})$ dove:
    - $\hat{w}_t$ è il token generato al tempo $t$
    - $w \in V$ è un token nell’insieme di vocabolario $V$
    - $P(w | w_{<t})$ è la probabilità condizionata del token $w$ dato il contesto precedente $w_{<t}$ (cioè, tutti i token precedenti a t)
- **Top-k sampling →** una variante del greedy decoding, in cui vengono selezionati i $k$ token più probabili. Le probabilità vengono normalizzate e uno di essi viene scelto casualmente.
- **Nucleus (Top-p sampling) →** invece di selezionare un numero fisso di token $k$, si selezionano i token la cui somma di probabilità raggiunge una percentuale $p$: $\sum_{w \in V^{p}}P(w|w_{<t}) \ge p$.
    
    Dopo aver normalizzato le probabilità, uno di questi token viene scelto casualmente. È quindi simile al top-k sampling, ma con un pool di token di dimensione variabile.
    
- **Temperature sampling →** si aumenta gradualmente la probabilità delle parole più probabili e si diminuisce quella delle parole più rare. Questo viene implementato dividendo il **logit** (→ $u$ = il punteggio non normalizzato) per un parametro di temperatura $t$ prima di normalizzarlo tramite la funzione softmax: $y = softmax(\frac{u}{t})$:
    - Quando $t$  è vicino a 1, la distribuzione delle probabilità rimane quasi invariata.
    - Quando $t$ è maggiore di 1, la probabilità delle parole più rare aumenta, poiché la distribuzione si appiattisce e diminuisce la differenza tra le probabilità delle parole.
    - Quando $t$ è più basso di 1, la probabilità delle parole più probabili aumenta.

---

# Pre-training di LLM

Si fornisce al modello l’intera frase e gli si chiede di prevedere, passo dopo passo, la parola successiva. Ad ogni passo, si inserisce nell’input il token corretto, ignorando la previsione fatta dal modello (questo approccio è chiamato “teacher forcing”). L’obiettivo è minimizzare la funzione di perdita, in questo caso  come loss function utilizziamo la cross-entropy discreta:

$$
L_{CE}=-\sum_{w\in V}y_t[w]\cdot log(\hat{y_t}[w])
$$

Dove:

- $y_t[w]$ é il valore corretto per il token $w$ alla posizione $t$
- $\hat{y_t}[w]$ è la probabilità predetta dal modello per il token $w$ alla posizione $t$

---

# Finetuning

**Finetuning** è il processo che consente di specializzare un modello di linguaggio pre-addestrato (LLM) su un nuovo task. Gli approcci per realizzare il finetuning includono:

- **Continued pre-training →** si allenano tutti i parametri del modello pre-addestrato su un nuovo set di dati.
- **Parameter-efficient finetuning →** Alcuni parametri del modello vengono fissati, mentre il resto viene allenato su un nuovo set di dati.
- **Adding an extra head →** si fissano tutti i parametri del modello e si aggiunge un ulteriore head al transformer, ovvero si aggiunge un classificatore specifico per il tipo di task da risolvere.
- **Supervised finetuning →** si allena il modello su un dataset di istruzioni per insegnargli a seguire comandi specifici.

---