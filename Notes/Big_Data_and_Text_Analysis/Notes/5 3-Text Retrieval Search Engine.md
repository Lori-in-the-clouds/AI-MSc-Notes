# 5.3-Text Retrieval Search Engine

Status: Done
Type: theory

Un **search engine** è composto da diverse componenti che lavorano insieme per recuperare documenti rilevanti in risposta a una query:

- **Tokenizer:** si occupa di suddividere il testo di un documento in unità di base (token) che fungeranno da feature nel modello di ricerca (es. bag-of-words).
- **Indexer:** ha il compito di indicizzare una grande quantità di documenti in modo efficiente. La struttura dati principale utilizzata è l’**inverted index**, che è composto da:
    - **Lexicon**: una tabella che registra informazioni sui token, come la loro frequenza nei documenti.
        
        ![image.png](5%203-Text%20Retrieval%20Search%20Engine/image.png)
        
    - **Postings file**: contiene informazioni dettagliate sui token, inclusa la posizione in cui compaiono nei documenti.
        
        ![image.png](5%203-Text%20Retrieval%20Search%20Engine/image%201.png)
        
    
    <aside>
    📌
    
    **Come di crea l’inverted index?**
    
    1. Si applica il tokenizer ai documenti.
    2. Si crea una tabella che associa ogni token ai documenti in cui compare.
    3. Si ordina la tabella in base ai token.
    4. Si costruisce il **postings file** (= memorizza **per ogni parola**: in **quali documenti** compare, **quante volte** e **dove**)
    
    ![image.png](5%203-Text%20Retrieval%20Search%20Engine/image%202.png)
    
    ![image.png](5%203-Text%20Retrieval%20Search%20Engine/image%203.png)
    
    ![image.png](5%203-Text%20Retrieval%20Search%20Engine/image%204.png)
    
    </aside>
    
- **Scorer/Ranker:** per ogni parola di una query, il motore di ricerca utilizza l’inverted index per assegnare uno score ai documenti e classificarli in base a questo score.
    
    Algoritmo:
    
    ![image.png](5%203-Text%20Retrieval%20Search%20Engine/image%205.png)
    
- **Feedback:** meccanismo per migliorare il ranking dei documenti ( $R'(q)$) in base alle interazioni dell’utente. Può essere:
    - **Esplicito**: l’utente fornisce un feedback diretto (es. compilando un modulo).
    - **Implicito**: l’utente non comunica direttamente la rilevanza del documento, ma si usano euristiche come click-rate o tempo di lettura.
    
    I feedback vengono salvati e successivamente utilizzati per migliorare $R'(q)$ attraverso uno dei seguenti metodi:
    
    - **Modifica del peso delle parole della query**
    - **Query expansion**: aggiunta di termini rilevanti alla query iniziale.

---

## Feedback nel Vector Space: Metodo di Rocchio

L’idea del metodo di **Rocchio** è di modificare  il **vettore della query originale $q$** spostandolo più vicino ai documenti rilevanti e più lontano da quelli non rilevanti, ottenendo così una **query modificata $q_m$.** La ****rilevanza è valutata in base ai feedback del passato.

![image.png](5%203-Text%20Retrieval%20Search%20Engine/image%206.png)

La formula del metodo di **Rocchio** è: 

$$
 \vec{q}_m = \alpha \vec{q} +  \frac{\beta}{|D_r|} \sum_{\vec{d_j} \in D_r} \vec{d_j} -  \frac{\gamma}{|D_n|} \sum_{\vec{d_j} \in D_n} \vec{d_j} 
$$

Dove:

- $q_m$ è la nuova query ottimizzata
- $q$ è la query originale
- $D_r$  è l’insieme dei documenti rilevanti
- $D_n$ è l’insieme dei documenti non rilevanti
- $\alpha$, $\beta$ e $\gamma$ sono pesi che bilanciano rispettivamente l’influenza della query originale, dei documenti rilevanti e di quelli non rilevanti

**N.B.** Il fattore meno rilevante è $\gamma$, poiché ci sono molti documenti da cui allontanarsi. È più importante dare peso agli esempi positivi rispetto a quelli negativi.

---