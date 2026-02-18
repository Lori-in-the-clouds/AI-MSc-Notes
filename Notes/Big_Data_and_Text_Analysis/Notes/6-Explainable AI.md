# 6-Explainable AI

Status: Done
Type: theory

L’**interpretabilità** di un modello misura quanto un umano riesce a comprendere il processo decisionale del modello e a prevederne le risposte. In base all’interpretabilità, i modelli si dividono in:

- **Modelli intrinsechi** → l’interpretabilità deriva direttamente dalla struttura del modello (es. alberi decisionali, modelli lineari semplici).
- **Modelli post-hoc** → l’interpretabilità deriva da un modello semplificato che favorisce l’interpretabilità di quello più complesso.

<aside>
📌

**Perché l’interpretabilità è importante?**

L’interpretabilità è cruciale per garantire:

- **Fairness** → Evitare discriminazioni implicite o esplicite contro gruppi sottorappresentati.
- **Privacy** → Proteggere le informazioni sensibili contenute nei dati.
- **Reliability/Robustness** → Assicurarsi che piccole variazioni nei dati di input non provochino cambiamenti drastici nelle predizioni.
- **Causalità** → Verificare che il modello apprenda solo relazioni realmente causali e non semplici correlazioni.
- **Trust (Fiducia)** → Un sistema che spiega le proprie decisioni è più affidabile per gli esseri umani rispetto ad un *black box*.
</aside>

## Parametri utilizzati per interpretare un modello

L’interpretazione può avvenire considerando diversi parametri:

- **Feature summary statistics** → Valuta l’importanza di ciascuna feature nelle decisioni del modello.
- **Data point perturbation** → Analizza come cambia l’output quando si modificano leggermente i dati di input.

	*Esempio:* Se il modello nega un mutuo a *Filippo* che lavora part-time, potrebbe suggerire che con due lavori part-time otterrebbe un mutuo.

- **Model internals** → Analizza la struttura stessa del modello per valutarne il comportamento (valido solo per modelli intrinseci).

## Metodi di analisi di interpretabilità

- **Model-specific methods** → Sfruttano le caratteristiche interne del modello per analizzarlo.
    
    *Esempio:* *TreeSHAP* calcola l’importanza delle feature sfruttando la struttura degli alberi decisionali (non applicabile a modelli come le reti neurali).
    
- **Model-agnostic methods** → Applicabili a qualsiasi modello, funzionano perturbando i dati in input e osservando le variazioni dell’output.

## Una spiegazione può essere locale o globale

- **Globale**: fornisce una spiegazione dell’intero modello nella sua interezza. Indica quali sono le feature più importanti indipendentemente dai singoli punti analizzati.
- **Locale**: spiega un singolo punto di input e quali feature hanno influenzato il risultato.

Idealmente, vorremmo una spiegazione globale, ma al momento questa non è ancora sempre precisa.

<aside>
📌

**Proprietà delle spiegazioni**

- **Accuracy**: Quanto bene la spiegazione prevede dati non visti?
- **Fidelity:** Quanto bene la spiegazione approssima la predizione del modello complesso (black box)?
- **Consistency**: Quanto differiscono le spiegazioni tra modelli addestrati sullo stesso task e con previsioni simili?
- **Stability**: Quanto sono simili le spiegazioni per istanze simili?
- **Comprehensibility**: Quanto bene gli esseri umani riescono a comprendere la spiegazione?
- **Degree of Importance**: Quanto bene la spiegazione riflette l’importanza delle feature?
</aside>

---

# Modelli intrinsechi

1. **Linear Regression →** un modello che effettua una predizione come una combinazione lineare delle feature dell’input. La sua formula generale è:
    
    $$
    y=\beta_0+\beta_1x_1+...+\beta_px_p+\epsilon
    $$
    
    Dove $\epsilon$ è l’errore che rappresenta la differenza tra il valore predetto e il valore reale.
    
    Quindi ciò che ci fa interpretare il modello sono i pesi associati alle features (eventualmente normalizzati): $t_{\beta_j}=\frac{\beta_j}{SE(\beta_j)}$ (con SE → standard error).
    
    <aside>
    
    Interpretazione dei pesi:
    
    - Se una **feature numerica** $x_j$ aumenta di un’unità, l’output stimato cambia di una quantità pari al suo peso $\beta_j$.
    - Se una **feature binaria** cambia dalla categoria di riferimento all’altra categoria, l’output stimato cambia di una quantità pari al peso della feature.
    - **Intercetta ($\beta_0$)**: rappresenta il valore atteso della predizione quando tutte le feature numeriche sono zero, con:
        - **Dati non standardizzati**:  spesso non ha un’interpretazione significativa perché tutte le feature uguali a 0 potrebbe non essere un caso realistico.
        - **Dati standardizzati (media = 0, deviazione standard = 1)**: l’intercetta rappresenta il valore medio della variabile target $y$, poiché tutte le feature sono trasformate per avere media zero.
    </aside>
    
    <aside>
    
    Effect Plot
    
    Per valutare l’impatto delle singole feature sulla predizione, si può utilizzare un **effect plot**, che visualizza gli effetti combinati dei pesi e dei valori delle feature. L’effetto di una feature $j$ su un’istanza $i$ è dato da:  $effect_j^{i}=w_jx_j^{(i)}$ 
    
    Dove:
    
    - $w_j$ è il peso della feature $x_j$
    - $x_j^{(i)}$ è il valore della feature $x_j$ per l’istanza corrente (ad es. la temperatura di oggi).
    
    ![image.png](6-Explainable%20AI/image.png)
    
    </aside>
    
2. **Logistic Regression** → in questo caso il peso di una feature viene comunque calcolato come il prodotto tra la riga dei valori degli input e la colonna dei pesi, proprio come nella regressione lineare. Tuttavia, a differenza della regressione lineare, il peso $\beta_j$ non ha influenza diretta sulla predizione:
    
    $$
    logistc(\eta)= \frac{1}{1+e^{-\eta}}\ \ \ 
    $$
    
    Con 
    
    $\eta = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + … + \beta_p x_p$
    
    L’interpretabilità è meno immediata, perché modificare il peso significa modificare l’esponenziale, rendendo difficile la visualizzazione dell’effetto. Un modo per interpretarlo è calcolare il rapporto tra il valore aggiornato (con il nuovo peso) e il valore precedente:
    
    $$
    \frac{odds_{x_j+1}}{oddsx_j} \rightarrow \frac{exp(\beta_0 + \beta_1 x_1 + … +\beta_j(x_j+1)+...+ \beta_p x_p)}{exp(\beta_0 + \beta_1 x_1 + … +\beta_jx_j+...+ \beta_p x_p)}= exp(\beta_j(x_j+1)-\beta_jx_j) = exp(\beta_j)
    $$
    
3. **Decision Tree** → l’algoritmo mi permette di capire quali sono le feature più importanti: ogni volta che prendo una decisione il modello riduce l’information gain: se la diminuzione è elevata, la feature ha un impatto significativo sulla predizione. Esistono formule specifiche per quantificare l’importanza di ciascun nodo in base alla riduzione del gain.
    
    ![image.png](6-Explainable%20AI/image%201.png)
    

---

# Metodi di analisi di interpretabilità

### Model Agnostic

1. **PDP (Partial Dependence Plot):** mostra come varia la predizione media al variare di una feature, tenendo le altre costanti. Per farlo, seleziona una feature (ad esempio la temperatura), ne modifica i valori in tutta la colonna (es. aumentando di 10 gradi) e osserva come cambia la predizione, mantenendo costanti le altre variabili.
    
    **Formula del PDP: $\hat f_s(x_s)= \frac{1}{n}\sum_{i=1}^{n}\hat f(x_s,x_c^{(i)})$** dove $x_S$ rappresenta la feature di interesse mentre $x_C$  le restanti.
    
    **Limitazioni:** si assume che le feature in $C$ non siano correlate con la feature in $S$. Se invece esiste una correlazione, modificare $x_S$ influenzerebbe anche $x_C$, generando combinazioni di dati non realistiche. Un’altra limitazione è dato dallo spazio delle feature, che aumenta in maniera quadratica.
    
    **Esempio**
    
    ![image.png](6-Explainable%20AI/image%202.png)
    
2. **PFI (Permutation Feature Importance):** misura l’importanza di una feature valutando quanto peggiora l’accuratezza del modello quando i valori di quella feature vengono permutati casualmente. Se l’accuratezza **diminuisce significativamente**, la feature era importante per la predizione. [Algoritmo di Fisher, Rudin, and Dominici (2018)]
3. **LOCO (Leave One Covariate Out):** rimuovo una feature (=colonna) dal dataset e valuto come varia l’output. Se l’accuratezza diminuisce, significa che quella feature era importante.
    
    <aside>
    
    L’**importanza delle feature** **può variare** a seconda che venga calcolata sui dati di training o di test. Se un modello, come l’SVM, overfitta, potrebbe attribuire molta importanza a diverse feature nei dati di training. Tuttavia, quando l’importanza viene calcolata su dati di test non visti, le feature potrebbero risultare meno rilevanti, con valori vicini a uno, indicando una minore influenza sulla predizione.
    
    ![image.png](6-Explainable%20AI/image%203.png)
    
    </aside>
    
4. **LIME(Local Surrogate):** è un metodo post hoc che approssima un modello complesso con un modello lineare, ma solo nell’intorno di un singolo punto. Non fornisce un’interpretazione globale del modello, ma aiuta a capire il comportamento locale delle predizioni.
    
    <aside>
    
    **LIME per dati tabulari**
    
    Utilizza un modello lineare per spiegare localmente un modello complesso. Per farlo, genera nuovi dati perturbando quelli originali vicino al punto di interesse, assegnando un peso maggiore ai più vicini. Il modello lineare viene poi allenato su questi dati perturbati, permettendo di approssimare la decisione del modello complesso solo nell’intorno del punto analizzato.
    
    ![image.png](6-Explainable%20AI/image%204.png)
    
    ![image.png](6-Explainable%20AI/image%205.png)
    
    ![image.png](6-Explainable%20AI/image%206.png)
    
    ![image.png](6-Explainable%20AI/image%207.png)
    
    Vogliamo quindi un modello di explanation che, tra i vari modelli lineari $g$ che approssimano $f$, trovi quello che minimizza la seguente equazione: 
    
    $$
    explanation(x) =
    \argmin\limits_{g \in \mathbb{G}} L(f,g,\pi_x)+\Omega(g)
    
    $$
    
    Dove: 
    
    - $L$ è una funzione di loss che misura la distanza tra il modello complesso $f$ e il modello lineare $g$ in base ad $\pi_x= e^{\frac{-D(x,z)^2}{\sigma^2}}$  (con $\sigma^2 = 0.75$) che misura la distanza tra i due modelli.
    - $\Omega(g)$ rappresenta la complessità del modello $g$ e serve a penalizzare modelli troppo complessi, favorendo quelli più semplici.
    </aside>
    
5. **Controfattuali:** si applicano a singole istanze, cercando la minima variazione di una feature che cambia la predizione. In pratica, si modifica iterativamente x finché y non cambia stato (per esempio: “se $x$ fosse $tot1$, $y$ sarebbe $y_1$” → modifico il valore di $x$ finché non trovo il primo $tot_n$  che rende $y=y_2)$. 
    
    <aside>
    🚨
    
    Tuttavia, queste spiegazioni non sono generalizzabili e soffrono del “*Rashomon effect*”: esistono più modi per giustificare una variazione nella predizione, basati su cambiamenti in diverse feature. Ad esempio, sia un aumento dell’età sia un aumento del reddito possono condurre a un cambiamento nella predizione.
    
    Le spiegazioni controfattuali sono utili per l’interpretabilità umana, ma meno accurate dal punto di vista esplicativo. 
    
    </aside>
    
6. **Shapley:** è un metodo per distribuire in modo equo un guadagno tra i partecipanti in base al loro contributo. Si basa sul concetto di coalizioni, in cui i giocatori collaborano per ottenere un profitto.
    
    <aside>
    📌
    
    **Connessione tra predizioni di ML e interpretabilità**
    
    - Il **gioco** è il compito di predizione per una specifica istanza.
    - Il **guadagno** è la differenza tra la predizione del modello per quella istanza e la media delle predizioni su tutto il dataset.
    - I **giocatori** sono le feature che, collaborando, determinano la predizione.
    </aside>
    
    <aside>
    🚧
    
    **Come si calcola il valore di Shapley?**
    
    Il valore di Shapley misura il **contributo marginale medio** di una feature rispetto a tutte le possibili coalizioni di feature.
    
    1. Per ogni possibile sottoinsieme di feature (cioè tutti i sottoinsiemi del set di feature **esclusa la feature $i$**), calcoliamo la predizione (date $n$ feature, ci saranno $2^{n-1}$ possibili coalizioni che non contengono $i$).
    2. Per ogni coalizione $S$
    , calcoliamo la differenza tra la predizione **con** e **senza** la feature.
    3. Il valore di Shapley della feature $i$ si ottiene facendo la media pesata di tutte queste differenze: 
        
        $$
        
        \phi_i(N,v) = \sum_{S \subseteq N \setminus \{i\}} \frac{|S|! \cdot (|N| - |S| - 1)!}{|N|!} \cdot \left[ v(S \cup \{i\}) - v(S) \right]
        $$
        
        dove:
        
        - $N$ → insieme di tutti i giocatori (=feature)
        - $S$ → una coalizione (un sottoinsieme di feature senza $i$)
        - $|S|!$ → numero di modi in cui puoi ordinare le feature dentro $S$
        - $|N|!$ → numero di tutte le permutazioni possibili delle feature in $N$
        - $(|N| - |S| - 1)!$ → numero di modi in cui puoi ordinare le feature rimanenti, dopo aver messo $S$ e la feature $i$
        - $|S|!(|N|-|S|-1)$ → rappresenta il **peso** assegnato a ciascuna coalizione. Una feature ha un impatto maggiore se aggiunta a una coalizione piccola, perché cambia significativamente il risultato. Al contrario, se aggiunta a una coalizione grande, il suo contributo è minore, poiché il modello ha già molte informazioni.
        - $v(S)$ → valore della funzione obiettivo (ad esempio, la predizione del modello) usando solo le feature in $S$
        - $v(S \cup \{i\})$ → valore della funzione obiettivo quando aggiungiamo la feature i alla coalizione $S$
        - $\left[ v(S \cup \{i\}) - v(S) \right]$ → contributo marginale della feature $i$ dato il sottoinsieme $S$
        
        **N.B.** Per stimare il valore di coalizioni che non usano tutte le feature, possiamo **assegnare valori casuali** agli attributi mancanti.
        
    </aside>
    
    <aside>
    
    **Formulazione equivalente del valore di Shapley**
    
    1. Supponiamo di avere $N = \{P1, P2, P3, P4\}$, cioè quattro giocatori
    2. Consideriamo tutte le possibili $4!=24$ permutazioni dell’insieme $N$
    3. Per ogni permutazione $\sigma_j$, definiamo $Q(\sigma_j, i)$ come il gruppo di giocatori che precedono $i$ in quella permutazione (Esempio: se  $\sigma_j = (P1, P4, P2, P3)$ e $i = P2$, allora $Q(\sigma_j, i) = \{P1, P4\})$
    4. La formula di Shapley sarà: $\phi_i(N, v) = \frac{1}{|N|!} \sum_{\sigma_j \subseteq \sigma} [ v(Q(\sigma_j, i) \cup \{i\}) - v(Q(\sigma_j, i)) ]$
    
    Esempio:
    
    ![image.png](6-Explainable%20AI/image%208.png)
    
    </aside>
    
    <aside>
    💡
    
    **Assiomi che rendono il valore di Shapley la suddivisione di profitto più equa:**
    
    1. **Efficienza:** l’intero profitto totale della colazione $v(N)$ viene redistribuito tra tutti gli $N$ giocatori, senza lasciare niente fuori → $v(N) = \sum_{i \in N} \phi_i(N, v)$
    2. **Simmetria:** due giocatori $i$ e $j$ che contribuiscono allo stesso modo a tutte le coalizioni $S$ devono ricevere la stessa ricompensa → $\forall S \subseteq N \setminus \{i, j\}, \quad v(S \cup \{i\}) = v(S \cup \{j\}) \Rightarrow \phi_i(N, v) = \phi_j(N, v)$
    3. **Linearità:** se combiniamo due giochi diversi $(N,v_1)$ e $(N,v_2)$, i contributi di ogni giocatore devono essere la somma dei contributi nei due giochi separati → $\phi_i(N, v_1) + \phi_i(N, v_2) = \phi_i(N, v_1 + v_2)$
    4. **Giocatore nullo:** se una feature non influenza mai la predizione, il suo valore di Shapley deve essere zero → $\forall S \subseteq N \setminus \{i\},\ v(S \cup \{i\}) = v(S) \Rightarrow \phi_i(N, v) = 0$
    </aside>
    
    <aside>
    💡
    
    **Algoritmo per la stima approssimativa del valore di Shapley**
    
    1. **Scegliamo un numero di iterazioni** $M$ (più è alto, più accurata è la stima)
    2. **Selezioniamo un’istanza di interesse** $x$, l**a feature di cui calcolare il valore di Shapley** $j$, **il dataset** $X$ **e il modello** $f$
    3. **Per ogni iterazione** $m$:
        1. Estrarre un’istanza casuale $z$ dal dataset $X$
        2. Ordiniamo casualmente le colonne sia di $x$ che di $z$, in modo da simulare una coalizione → $x = xo(x(1),...,x(j),...,x(p))$ e $z = zo(z(1),...,z(j),...,z(p))$
        3. Costruiamo 2 nuove istanze: 
            - $x_{+j}$ contente la feature j originale e le altre feature considerate nella coalizione → 
            
            $x_{+j}=(x(1),…,x(j−1),x(j),z(j+1),…,z(p))$
            - $x_{-j}$ costituisce la feature $j$ con il valore corrispondente di $z$ (cioè simula l’assenza della feature nella coalizione) → 
            
            $x_{−j}=(x(1),…,x(j−1),z(j),z(j+1),…,z(p))$
        4. Sottraendo i due valori otteniamo il contributo marginale dell’iterazione $m$-esima:  
        
        $\phi_{mj} = f(x_{+j}) - f(x_{-j})$
    4. Calcoliamo la media dei contributi marginali: 
    
    $\phi_j(x) = \frac{1}{M} \sum_{m=1}^{M} \phi_{mj}$
    </aside>
    
    <aside>
    🚨
    
    **Pro e contro**
    
    I valori di Shapley permettono di descrivere il contributo di una feature nelle decisioni di un modello, tuttavia:
    
    - richiedono molta capacità computazionale (→ il valore di Shapley va calcolato per ogni attributo del dataset)
    - la regola di efficienza distribuisce il motivo della scelta (= guadagno) su tutte le features, il che porta ad una rappresentazione poco polarizzata e quindi meno significativa
    </aside>
    
    <aside>
    
    **SHapley Additive exPlanations**
    
    SHAP è un metodo basato sui valori di Shapley per spiegare l’importanza delle feature in una singola predizione. Rappresenta la predizione di un’istanza come la somma dei contributi di ciascuna feature, calcolati tramite i valori di Shapley:
    
    $$
    g(z')= \phi_0+\sum_{j=1}^M\phi_jz_j'
    $$
    
    Dove $z'_j$ è un valore booleano che mi indica se la feature ha contribuito o meno. Invece $\phi_j$  è il valore di shapley della colonna. 
    
    ---
    
    **Implementazione specifica di SHAP: KernelSHAP**
    
    Consiste nelle seguenti fasi:
    
    1. Si creano **coalizioni**, ovvero maschere binarie (0/1) che definiscono quali feature sono presenti nella coalizione. Ogni riga del dataset rappresenta una diversa coalizione.
    2. Si genera un **nuovo dataset**, in cui ogni riga varia in base alla propria maschera
        
        ![image.png](6-Explainable%20AI/image%209.png)
        
    3. Si applica il **modello di machine learning** al nuovo dataset per ottenere le predizioni → target
    4. Si calcola un **peso** per ogni coalizione $z'_i$, indicato come $w_i$
    5. Si costruisce un **modello di regressione lineare** per stimare i valori di SHAP, basato sull’espressione  $w_i x_i$.
    </aside>