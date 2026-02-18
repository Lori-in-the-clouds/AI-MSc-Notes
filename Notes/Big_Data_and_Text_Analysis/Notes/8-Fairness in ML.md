# 8-Fairness in ML

Status: Done
Type: theory

Il machine learning si basa su dati storici, ma questo non garantisce decisioni affidabili, accurate o eque. I dati possono contenere pregiudizi, stereotipi e disuguaglianze, e i modelli rischiano di replicare queste distorsioni invece di correggerle.

Ad esempio, il sistema di consegna di Amazon ha favorito quartieri a maggioranza bianca, mostrando come le decisioni basate sui dati possano perpetuare ingiustizie.

---

# Bias

Il bias nei sistemi algoritmici si riferisce a disparità demografiche problematiche dal punto di vista sociale. Abbiamo 3 tipi di bias:

- **Preexisting bias:** deriva da pregiudizi già presenti nella società o da dati raccolti in modo distorto (→ attraverso cambiamenti nei criteri di classificazione nel tempo o l’uso di dati già influenzati da ML).
    
    <aside>
    
    Esempi:
    
    - Un articolo del *New York Times* mostra come le percentuali di studenti neri e ispanici nelle università siano cambiate nel tempo, ma ignora l’introduzione di una nuova categoria multirazziale nel 2008, distorcendo il confronto.
    - Un sistema di ML in ambito sanitario ha appreso che i pazienti asmatici avevano un rischio inferiore di polmonite, poiché ricevono cure ospedaliere più frequenti. Questo è un esempio di **feedback loop**, dove la previsione influenza l’outcome, invalidando sé stessa.
    </aside>
    
- **Technical bias** → nasce da come i dati vengono trattati. Può verificarsi quando:
    - Si riempiono i valori mancanti con il valore più frequente. Per ridurre questo effetto, si possono rappresentare le categorie in colonne separate invece di imporre una scelta binaria.
    - Nella gestione del testo, trasformare tutto in minuscolo o maiuscolo può creare ambiguità: ad esempio, “Iris” può essere sia un nome che un fiore.
    - I join e le selezioni sui dati possono escludere certi sottogruppi della popolazione. Ad esempio, basarsi su codice postale o dati clinici può alterare la rappresentazione di alcuni gruppi.
    - La mancanza di corrispondenza per nomi rari nei dataset può portare all’esclusione di determinati gruppi etnici.
    - La posizione degli elementi in una lista influenza l’attenzione ricevuta, creando disparità.
- **Emergent bias** → nasce dall’uso del modello nel tempo. Un esempio è il **feedback loop positivo**, in cui una previsione iniziale rafforza se stessa: se un modello segnala un’area come ad alta criminalità, più controlli porteranno a trovare più reati, confermando il bias iniziale (”*il ricco diventa più ricco*”). Anche gli algoritmi di raccomandazione possono amplificare certe scelte, rendendo i contenuti più popolari ancora più visibili.

---

# ML loop

Il **ML loop** descrive il ciclo in cui un modello di machine learning influenza e viene influenzato dalla realtà:

1. Si raccolgono dati dallo stato attuale del mondo.
2. Il ML costruisce un modello basato su questi dati attraverso il processo di apprendimento.
3. Il modello guida le decisioni degli individui, modificando così lo stato del mondo.
4. Nuovi dati vengono raccolti, alimentando il ciclo.

![image.png](8-Fairness%20in%20ML/image.png)

Gli individui possono essere identificati tramite dati sensibili, come etnia, sesso, religione, disabilità, ecc.

Un modello di machine learning non dovrebbe basarsi su questi dati per prendere decisioni, quindi è necessario normalizzarli. Tuttavia, questo processo non è sempre semplice: ad esempio, anche eliminando il genere da un dataset, è possibile dedurlo indirettamente attraverso altre informazioni, come lo stipendio. Inoltre, la normalizzazione deve considerare l’importanza del dato sensibile per la decisione: ad esempio, il genere può influenzare l’efficacia di un farmaco.

<aside>
🚨

**Il problema della misurazione nei modelli di ML**

La misurazione dei dati può introdurre distorsioni, poiché le categorie usate per classificare gli individui cambiano nel tempo. Ad esempio, l’introduzione di una nuova categoria in un dataset può rendere i confronti con dati storici inaccurati. Inoltre, le scelte su come raccogliere e rappresentare i dati possono alterare le proporzioni dei gruppi, generando bias nei modelli di ML.

</aside>

---

# Come si misura la fairness

L’obiettivo è creare modelli che non tengano conto di attributi sensibili nel prendere decisioni e che quindi non siano unfair in qualsiasi modo nel prendere le decisioni.

<aside>

**Notazioni:**

- $R$ →  Soglia di decisone (=valore che permette di effettuare lo scatto da 0 a 1)
- $A$ → Insieme degli attributi sensibili
- $Y$ → Insieme delle decisioni corrette
</aside>

Si misura principalmente in base a:

1. **Independence →** Il criterio per effettuare la classificazione deve essere **indipendente** dall’attributo sensibile, ovvero: $R \perp A$.  
    
    L’idea è che la probabilità di prevedere **1** (esito positivo) deve essere indipendente dalla categoria dell’individuo. In altre parole, ogni gruppo sensibile (es. etnia, genere) deve avere la stessa probabilità di ottenere un determinato esito (es. assunzione).
    
    Per verificare la **independence** in un problema di classificazione binaria, si utilizza la **demographic parity**: 
    
    $$
    P(\hat{Y}=1|A=a)=P(\hat{Y}=1|A=b)
    $$
    
    Versione rilassata, con una tolleranza $\epsilon$:
    
    $$
    P(\hat{Y}=1| A=a) \geq P(\hat{Y}=1|A=b)-\epsilon
    $$
    
    <aside>
    🚨
    
    **Contro:**
    
    Questo è il criterio di fairness più utilizzato, ma anche il più ingenuo: **pareggiare le probabilità è facile**, ma non sempre equo. Ad esempio, per garantire che uomini e donne siano assunti nella stessa proporzione, potrei dover scartare candidati altamente qualificati solo perché appartengono al gruppo in maggioranza.
    
    Un fenomeno simile è il **Glass Cliff**: uomini appartenenti a minoranze o donne vengono promossi a posizioni di leadership (es. CEO) **quando un’azienda è in crisi**, esponendoli a un alto rischio di fallimento e di successivo licenziamento.
    
    </aside>
    
2. **Separation** → impone che la capacità di predire l’output non dipenda dall’attributo sensibile, dato il valore reale della classe, ovvero: $R \perp A|Y$. Nel caso di **classificazione binaria**, la separation si verifica attraverso il criterio di **equalized odds**: 
    
    $$
    P(\hat{Y}=1|Y=1,A=a)= P(\hat{Y}=1|Y=1,A=b) \rightarrow TPR\ (=recall)\\ P(\hat{Y}=1|Y=0,A=a)= P(\hat{Y}=1|Y=0,A=b) \rightarrow FPR
    $$
    
    Questa condizione garantisce che il **modello sia equamente accurato** per tutti i gruppi:
    
    - La prima equazione assicura che la probabilità di prevedere **1** quando il valore vero è **1** sia la stessa per entrambi i gruppi.
    - La seconda equazione impone che il tasso di **falsi positivi** sia bilanciato tra i gruppi.
    
    <aside>
    🚨
    
    **Contro:**
    
    Un ostacolo pratico è che i gruppi possono avere **distribuzioni sbilanciate** nei dati. Se un gruppo è **sottorappresentato**, il modello avrà meno dati per imparare, risultando in **TPR** e **FPR peggiori** per quel gruppo. Per bilanciare i due valori tra i gruppi, si abbassano le performance del gruppo con più dati, portando a una **riduzione generale della performance del modello** (”*Se devo avere lo stesso errore sulle entrambe categorie, basta sbagliare di più in quello in cui sono più bravo”*).
    
    </aside>
    
3. **Sufficiency** → richiede che la classe vera ($Y$) sia indipendente dall’attributo sensibile ($A$), **dato il valore previsto dal modello** ($\hat{Y}$), ovvero: $Y\perp A|R$. 
    
    Nel caso di **classificazione binaria**, si verifica con le seguenti condizioni:
    
    $$
    P(Y=1|\hat{Y}=1,A=a)=P(Y=1|\hat{Y}=1,A=b) \rightarrow precision\\ P(Y=0|\hat{Y}=0,A=a)=P(Y=0|\hat{Y}=0,A=b) \rightarrow specificity
    $$
    
    Queste equazioni assicurano che:
    
    - Se il modello ha **predetto 1** (es. assunzione), la probabilità che la persona fosse effettivamente idonea deve essere la stessa per tutti i gruppi.
    - Se il modello ha **predetto 0**, la probabilità che la persona **non fosse idonea** deve essere uguale per tutti i gruppi.
    
    In pratica, significa che **la previsione è affidabile allo stesso modo per tutti i gruppi sensibili**.
    
    **N.B.** Si utilizzano principalmente le prime due misure.
    
    <aside>
    
    Esempio:
    
    ![image.png](8-Fairness%20in%20ML/image%201.png)
    
    </aside>
    

---

# Come ottenere un modello che non fa discriminazioni

Esistono 3 tecniche principali per garantire la fairness di un modello:

- **Pre-processing del dataset →** si modificano i dati prima dell’addestramento, rimuovendo attributi sensibili e riducendo le correlazioni con essi. Tuttavia, questo può comportare una perdita di accuratezza.
- **In-training →** si impongono vincoli (=constraints) durante la fase di addestramento per ottimizzare la fairness del modello.
- **Post-processing →** si ****regolano le soglie dei predittori dopo l’addestramento per bilanciare le predizioni tra le diverse categorie.

Un esempio pratico è la libreria Fairlearn, che offre strumenti per togliere correlazioni dal dataset, fare post-processing,…

---