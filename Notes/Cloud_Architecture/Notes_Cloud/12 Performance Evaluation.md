# 12. Performance Evaluation

Number: 5
Status: Done
Type: theory

La valutazione delle prestazioni è fondamentale per il capacity planning, la stima dei costi e per garantire la scalabilità dei sistemi, evitando errori di configurazione che possono portare a cali prestazionali significativi o costi imprevisti.

# 1. Approcci alla Valutazione

Esistono diverse metodologie per valutare un sistema:

- **Esperimenti con sistemi reali** → approccio più completo, utilizzato per **testing in produzione** e **benchmarking**, garantendo la massima accuratezza.
    
    **Contro:** costoso e spesso non disponibile, ad esempio quando l’infrastruttura è già in uso o non ancora esistente.
    
- **Esperimenti con modelli del sistema**:
    - **Modelli Fisici (Prototipi)** → sono rappresentazioni **in scala** del sistema reale, spesso realizzate tramite **ambienti virtualizzati** o piccoli cluster fisici. Si usano quando il sistema reale non è disponibile o non può essere testato direttamente. Questo tipo di modello è generalmente **più costoso** rispetto ad altri approcci e rende più difficile simulare scenari *what-if* (es. “cosa succede se il tempo di risposta passa da 50 ms a 100 ms?”), perché richiede modifiche concrete al prototipo.
        
        **Contro:** i risultati devono essere scalati al sistema reale, ma la scalabilità può essere **sub-lineare** o **super-lineare**, rendendo la stima complessa.
        
    - **Modelli Matematici** → rappresentano il sistema tramite **equazioni**, permettendo analisi rapide e simulazioni di scenari *what-if:*
        - **Modelli Analitici:** soluzioni in forma chiusa o numerica. Precisi nei calcoli, ma richiedono semplificazioni che possono trascurare parte della complessità reale. Utili per stime di ordine di grandezza.
        - **Simulazione:** software che replica il comportamento del sistema. Offre maggiore flessibilità e dettaglio rispetto ai modelli analitici, ma tempi di esecuzione elevati se troppo dettagliata.

![image.png](12%20Performance%20Evaluation/0191b434-0e88-47c6-8b45-cd9cfac0146c.png)

Il **trade-off generale** consiste nel trovare l’accuratezza sufficiente per rispondere alle proprie domande: più dettaglio aumenta i costi, meno dettaglio riduce la precisione. Ogni modello si colloca quindi in un punto diverso di questo equilibrio tra **accuratezza** e **overhead**.

---

# 2. Metriche e Curve di Carico

Le principali metriche di prestazione sono:

- **Tempo di risposta** → tempo tra l'inizio della richiesta e la fine della risposta (metrica lato utente)
- **Throughput** → volume di richieste soddisfatte nell'unità di tempo
- **Tasso di errore** → frazione di richieste non soddisfatte correttamente

Queste metriche dipendono dal carico operativo, descritto dalla **Curva di Carico**:

1. **Zona Verde (Sottoutilizzo/Normale)** → il tempo di risposta è basso e costante; il throughput cresce linearmente con il carico. L’obiettivo è mantenere il sistema in questa zona, massimizzando l’utilizzo senza entrare in saturazione. La capacità disponibile extra consente di assorbire picchi di traffico senza degradare le prestazioni.
2. **Zona Gialla (Ginocchio/Kneeling Zone)** → il sistema si avvicina alla saturazione (es. 80-90% CPU). Le richieste in eccesso iniziano ad essere accodate, il tempo di risposta cresce rapidamente e aumenta il rischio di droppaggio. La **kneeling zone** indica quanto carico aggiuntivo il sistema può tollerare prima di degradare le prestazioni. 
    
    <aside>
    📌
    
    **Capacity Planning**
    
    La posizione delle Kneeling Zone dipende dal **capacity planning**, cioè il processo con cui si decide **quanta capacità extra** (CPU, RAM, banda, nodi, server) il sistema deve avere per gestire futuri picchi di traffico senza saturarsi. Più capacità viene pianificata, più il limite della kneeling zone si sposta in avanti, permettendo al sistema di assorbire aumenti di carico senza degradare le performance. Graficamente, il capacity planning appare come uno **spostamento in avanti** del limite della kneeling zone.
    
    ![image.png](12%20Performance%20Evaluation/image.png)
    
    </aside>
    
3. **Zona Rossa (Saturazione/Thrashing)** → il throughput si appiattisce o cala, le richieste vengono largamente droppate e il sistema spende gran parte del tempo in gestione interna (context switch, I/O) anziché in lavoro utile. Alcune richieste raggiungono il **timeout**: il sistema le interrompe o scarta, causando un aumento drastico del tasso di errore.

![image.png](12%20Performance%20Evaluation/image%201.png)

---

# 3. Testing e Benchmarking

Il testing risponde a domande diverse a seconda del tipo di prova eseguita:

- **Functional Testing** → verifica che il sistema produca risultati corretti (molto usato in CI/CD). Non cattura problemi più complessi come race condition, memory leak o rallentamenti di performance, ma garantisce alta affidabilità funzionale ed è molto prezioso per il continuous integration.
- **Activity Testing** → valuta il comportamento del sistema in condizioni di carico **normale**. Misura le tempistiche **coinvolgendo spesso utenti reali**, così da raccogliere molto feedback sulla **user experience**. È utile per individuare problemi di usabilità o rallentamenti percepiti dagli utenti.
- **Endurance Testing** → test prolungati con carico **medio-alto**, per osservare degrado, memory leak, stabilità e per raccogliere dati utili al **capacity planning**.
- **Stress Testing** → porta il sistema al limite, finché non collassa. Serve per identificare il **punto di rottura** (massimo carico sostenibile) e capire quale risorsa diventa il **collo di bottiglia** (CPU, DB, rete…).
- **Benchmarking** → serve a **confrontare sistemi diversi** tramite carichi standardizzati e riproducibili. Le principali categorie sono:
    - **Toy Benchmark** → algoritmi semplici (Quicksort, 8 Regine). Veloci, ma poco rappresentativi dei carichi reali.
    - **Real Programs** → programmi veri (compilare un kernel, ray tracing). Più significativi, ma influenzati da compilatori, driver e ottimizzazioni specifiche. Per ridurre i bias, si usa una **suite** di programmi diversi, assegnando pesi adeguati ai vari programmi.
    - **Synthetic Benchmark** → suite di operazioni matematiche standard (es. **Dhrystone** per calcoli interi, **Whetstone** per calcoli in virgola mobile).
        
        <aside>
        🚨
        
        **“Abusi” di Marketing**
        
        Pur essendo molto diffusi, questi tipi di benchmark sono facilmente soggetti ad “abusi” di marketing: alcuni vendor ottimizzano hardware o compilatori appositamente per ottenere risultati eccellenti in questi test, rendendoli poco rappresentativi delle prestazioni reali.
        
        Esempi tipici:
        
        1. **Istruzioni hardware create ad hoc** per brillare in un benchmark. Esempio: nei processori **PowerPC** (IBM), venne introdotta un’istruzione *moltiplica-e-somma* molto efficiente per le operazioni matriciali, che compaiono frequentemente in Dhrystone. Nei materiali di marketing IBM mostrava prestazioni “4× migliori”, ma solo perché il processore era ottimizzato per quel benchmark. 
        2. **Cache dimensionata apposta** per contenere *interamente* il codice del benchmark, garantendo prestazioni irrealisticamente alte.
        3. **Esecuzione su macchina calda/fredda**, che può alterare significativamente i risultati a seconda dello stato della cache o del thermal throttling.
        
        **N.B.** Per questo è fondamentale leggere attentamente le **condizioni di test** e non fidarsi solo del numero riportato.
        
        </aside>
        
    - **Real Applications** → benchmark basati su scenari d’uso realistici (SPEC, TPC). Sono i più affidabili perché misurano prestazioni su workload comparabili alla produzione.
        
        <aside>
        📌
        
        **Suite di Benchmark Principali**
        
        - **SPEC & TPC:** standard industriali. *SPECWeb* e *TPC-W* simulano traffico web/e-commerce realistico.
        - **YCSB (Yahoo! Cloud Serving Benchmark):** focalizzato su database NoSQL (es. Cassandra) e scalabilità elastica.
        - **PerfKit (Google):** tool per testare le prestazioni cloud (provisioning, esecuzione, reportistica)
        </aside>
        

---

# 4. Modelli Matematici: Teoria delle Code

Se il sistema reale non è disponibile, si può usare un **modello analitico** basato sulla **teoria delle code**: ogni componente (CPU, disco, rete, nodi frontend/backend) è rappresentato come: una **coda** di richieste e un **servitore** con tempo di servizio caratteristico. Questo consente di stimare tempi di risposta e prestazioni sotto carico. 

![image.png](12%20Performance%20Evaluation/8ac9b46a-88ab-4bdd-ba4d-1f1c5dfa1bb8.png)

Più risorse possono essere concatenate per rappresentare un sistema completo (un nodo, ovvero una macchina fisica/virtuale, ad esempio):

![image.png](12%20Performance%20Evaluation/8b52e851-d64d-4437-9700-77bb91d622fd.png)

**N.B.** Modelli più dettagliati richiedono dati precisi su CPU, disco, rete, ecc.; in assenza di informazioni affidabili si devono fare stime, riducendo l’accuratezza.

<aside>
📌

**Caratterizzazione Statistica del Sistema**

Nella **teoria delle code**, il **tempo di servizio** e il **tempo di interarrivo** delle richieste sono **variabili aleatorie**, non valori deterministici. Questo richiede strumenti statistici per stimare le prestazioni, come i **tempi di risposta**, anche senza osservare il sistema reale.

Le variabili aleatorie si descrivono tramite la **funzione di densità di probabilità (PDF)**, ad esempio una Gaussiana definita da media $\mu$ e deviazione standard $\sigma$. Per una rappresentazione compatta e meno sensibile agli outlier, si ricorre spesso ai **Box Plot**, composti principalmente da:

- **Mediana (Q2)** → valore intermedio della distribuzione; lascia il 50% dei dati al di sotto e il 50% al di sopra.
- **Quartili** →  casi particolari di quantili che dividono la popolazione in quattro parti uguali:
    - 1° Quartile (Q1): 25% dei dati ≤ Q1
    - 2° Quartile (Q2): corrisponde alla mediana
    - 3° Quartile (Q3): 75% dei dati ≤ Q3
- **Baffi (Whiskers)** → sono le linee che partono dalla scatola. Di solito si estendono fino al valore minimo e massimo, oppure fino a un limite calcolato ($1,5×IQR$). Rappresentano la variabilità dei dati al di fuori della zona centrale.
- **Outliers** → eventuali punti che cadono oltre i baffi.

![image.png](12%20Performance%20Evaluation/image%202.png)

</aside>

## 4.1. Parametri Fondamentali

Per descrivere un singolo **nodo (single-core)** del sistema, si considera generalmente una **coda** e il **carico di lavoro**, caratterizzato da:

- $\lambda_i$ → tasso di arrivo delle richieste all’$i$-esima risorsa (richieste/sec), spesso modellato come variabile aleatoria
- $W_i$ → tempo medio di attesa in coda, anch’esso modellato come variabile aleatoria
- $S_i$ → tempo medio che la risorsa $i$-esima impiega a processare **una singola richiesta**. Il tasso di servizio della risorsa è $\mu=\frac{1}{S_i}$ (per esempio, se $Sᵢ = 0,1s$, la risorsa può processare in media 10 richieste/sec)

Da questi elementi si ricavano alcune metriche fondamentali:

- **$R_i$** → ****tempo di risposta medio della risorsa $i$-esima, calcolato come: $Rᵢ = S_i + W_i$
- $X_i$ → throughput della risorsa $i$-esima. Si calcola in funzione del tasso di arrivo e del tasso di servizio, ma intuitivamente:
    - Se la coda è **poco occupata** → $X_i \approx \lambda_i$ (arrivi), cioè la risorsa riesce a servire quasi tutte le richieste che arrivano.
    - Se la risorsa è **saturata** → $X_i \approx \mu$, cioè il tasso massimo che la risorsa può processare.
- $X_o$ → throughput complessivo del sistema, dominato dalla risorsa collo di bottiglia

Se si fotografa il sistema in un preciso istante, si possono osservare le seguenti grandezze volumetriche per la risorsa $i$:

- $N_{iw}$ → numero di richieste in coda
- $N_{is}$ → numero di richieste in servizio (0 o 1 per singolo servitore)
- $N_i$ → numero totale di richieste nella risorsa $i$-esima, calcolato come $Nᵢ = N_{iw} + N_{is}$

## 4.2. Leggi Fondamentali

Le relazioni tra questi parametri sono regolate da leggi universali:

1. **Legge dell’Utilizzazione** → l'utilizzazione $U_i$ rappresenta la probabilità che il servitore $i$-esimo sia occupato:
    
    $$
    U_i=X_i\cdot S_i
    $$
    
    In stato stazionario, corrisponde al rapporto tra tasso di arrivo e di servizio: $\rho=\gamma/\mu$. Se $\rho>1$, il sistema è in sovraccarico e instabile (coda infinita).
    
2. **Legge di Little** → relaziona il numero medio di job nel sistema $N$, il throughput $X$ e il tempo di permanenza $R$. È valida per qualsiasi sistema "black box", indipendentemente dalla sua struttura interna:
    
    
    $$
    N=X\cdot R
    $$
    
    Si può applicare alle singole componenti del sistema come segue:
    
    - $N_{iw} = X_i\cdot W_i$
    - $N_{is} = X_i \cdot S_i$
    - $N_i = X_i \cdot R_i$
    
    ![image.png](12%20Performance%20Evaluation/image%203.png)
    
    <aside>
    📌
    
    **Open/Closed loop models**
    
    La popolazione di job può essere modellata in due modi principali:
    
    - **Closed Loop** → popolazione finita. Ogni job alterna richieste a un tempo di inattività $Z$ prima di inviarne un’altra. In questo caso la Legge di Little si adatta a:
        
        $$
        
        R + Z = \frac{M}{X}
        $$
        
    - **Open Loop** → popolazione potenzialmente infinita. Si considerano solo le richieste in arrivo. In questo caso vale la Legge di Little standard.
    
    ![image.png](12%20Performance%20Evaluation/image%204.png)
    
    </aside>
    

## 4.3. Notazione di Kendall

I sistemi a coda si descrivono con la seguente notazione:

$$

\text{Arrival / Process / \#Servers / Queue Length}
$$

Dove: 

- $\text{Arrival}$ → distribuzione degli arrivi (es. **M** = Poissoniano/Markoviano, ovvero senza memoria).
- $\text{Process}$ → distribuzione dei tempi di servizio (es. **M** = Esponenziale, **G** = Generica).
- $\text{\#Server}$ → numero di servitori che processano le richieste.
- $\text{Queue Length}$ → lunghezza massima della coda (spesso omesso se ∞, coda infinita).

<aside>
📌

**Processo Poissoniano**

Un **processo poissoniano** è un modello matematico utilizzato per descrivere il verificarsi di eventi che avvengono in modo **casuale e indipendente** nel tempo. Un sistema *Servitore + Coda* può essere modellato tramite due processi Poissoniani:

- **Arrivi con tasso $\gamma$** → probabilità di passare da $N \rightarrow N+1$ (nuova richiesta).
- **Servizio con tasso $\mu$** → probabilità di passare da $N \rightarrow N – 1$ (richiesta completata).

![image.png](12%20Performance%20Evaluation/image%205.png)

**N.B.** Si sceglie l'approccio Poissoniano perché permette di stimare i **valori medi** (tempi di risposta e occupazione) semplificando drasticamente i calcoli. Evita, infatti, la complessità di dover caratterizzare la varianza delle distribuzioni reali, offrendo comunque una buona stima iniziale.

---

**Formule chiave:**

- **Utilizzazione: $\displaystyle\rho=\frac{\lambda}{\mu}$**  (rappresenta il grado di saturazione del sistema)
- **Probabilità di stato:** 

$P_k = (1 - \rho) \rho^k$ (= probabilità di trovare esattamente $k$ utenti nel sistema in un dato istante)
- **Tempo di risposta medio:** 

$\displaystyle T_{resp} = \frac{1}{\mu - \lambda}$ che cresce rapidamente quando $\lambda$ si avvicina a $\mu$, ovvero quando $\rho \approx 1$

![image.png](12%20Performance%20Evaluation/98cb6985-521a-49a1-bced-68069989cce6.png)

---

**Analisi del comportamento stabile**

Osservando la distribuzione di probabilità (grafico a istogrammi), possiamo dedurre che in un sistema stabile (dove $μ>λ$):

- Lo stato statisticamente più probabile è quello di sistema **"vuoto"** o con pochi utenti.
- La probabilità di avere $k$ utenti decresce geometricamente man mano che il numero di utenti aumenta.

![image.png](12%20Performance%20Evaluation/image%206.png)

</aside>

## 4.4. Modelli Principali

- **M/M/1** → il modello base. Arrivi Poisson, tempi di servizio esponenziali, un solo servitore. Il **tempo di risposta** totale è la somma del tempo trascorso in coda e del tempo di servizio effettivo:
    
    $$
    T_{\text{response}}=T_{\text{queue}}+\frac{1}{\mu}
    $$
    
- **M/G/1** → utilizzato quando i tempi di servizio seguono una distribuzione generica $G$ e non necessariamente esponenziale. Questo modello è più realistico per molti workload cloud reali. Per calcolare le prestazioni si utilizzano le **formule di Pollaczek-Khinchin**, che dimostrano matematicamente come la varianza del servizio impatti negativamente i tempi di attesa. Sapendo che:
    - $\rho = \lambda\cdot E[S]$
    - $Var(S)= E[S^2]-E[S]^2$
    - $C_v^2=\frac{Var(S)}{E[S]^2}$
    
    $$
    \left\{\begin{aligned}\mathrm{E}[W] &= \frac{\lambda \mathrm{E}[S^2]}{2(1 - \rho)} =
    \frac{\lambda\left[Var(s)+E[S]^2\right]}{2(1-\rho)}
    =
    
    \frac{\lambda\left[C_v^2\cdot E[S]^2+E[S]^2\right]}{2(1-\rho)}
    
    =
    \frac{\lambda\left[E[S]^2\cdot(1 + C_v^2)\right]}{2(1-\rho)}
    
    =
    \frac{\lambda \cdot E[S] \cdot E[S]\cdot(1 + C_v^2)}{2(1-\rho)}
    
    =
    \frac{\rho \cdot E[S]\cdot(1 + C_v^2)}{2(1-\rho)}
    
    \\[1em]\mathrm{E}[T] &= \mathrm{E}[S] + \frac{\lambda \mathrm{E}[S^2]}{2(1 - \rho)} = 
    
    \frac{2(1-\rho)E[S]+\rho E[S]\cdot(1-C_v^2)}{2(1-\rho)}
    \end{aligned}\right.
    $$
    
    Dove: 
    
    - $\mathrm{E}[W]$ = tempo medio di attesa in coda
    - $\mathrm{E}[T]$ = tempo medio di risposta totale
    
    <aside>
    📌
    
    **L'importanza del Coefficiente di Variazione $C_v$**
    
    In scenari complessi, è fondamentale conoscere il **coefficiente di variazione $C_v$**, definito come il rapporto tra deviazione standard e valore medio. Il $C_v$ indica quanto è “aperta” la campana della distribuzione di probabilità.
    
    Una **varianza elevata** (alto $C_v$) significa che la distribuzione dei tempi di servizio è più ampia, aumentando la probabilità che alcune richieste rimangano in coda a lungo e, di conseguenza, peggiorando le prestazioni complessive del sistema.
    
    </aside>
    

**N.B.** Per stimare i **tempi di risposta medi**, la teoria delle code con valori medi fornisce stime aggregate e affidabili, utili per comprendere il comportamento generale del sistema. Per analisi più dettagliate o scenari specifici, invece, è necessario ricorrere a modelli complessi che considerano la **struttura interna** e la **profondità di ogni fase del processo**.

---

# 5. Failures

## 5.1. Classificazione delle Failure

Per applicare le contromisure adeguate, è necessario classificare i guasti secondo tre criteri principali:

- **Durata:**
    - **Permanenti** → il componente si rompe in modo irreversibile (es. incendio in un data center). Richiede sostituzione fisica o ricostruzione.
    - **Recuperabili** → esiste un "safe point" a cui ritornare.
    - **Transienti** → malfunzionamenti temporanei che scompaiono da soli. Esempio tipico: durante uno *scale-up*, i nuovi server impiegano tempo per andare online, causando un sovraccarico momentaneo e risposte fuori specifica.
    - **Intermittenti** → i guasti più difficili da gestire. Il problema appare, scompare e ritorna (es. falsi contatti hardware). Sono complessi da riprodurre e diagnosticare perché spesso correlati a condizioni specifiche difficili da isolare nel "rumore di fondo" del sistema.
- **Effetto:**
    - **Funzionali** → il sistema non si comporta come dovrebbe
    - **Prestazionali** → il sistema funziona correttamente a livello logico (risposte corrette), ma **non rispetta i requisiti di performance**: tempi di risposta troppo alti, throughput insufficiente (violazione SLA).
- **Estensione:**
    - **Totali** → l'intero sistema o endpoint è irraggiungibile.
    - **Parziali** → solo un sottoinsieme di funzionalità o microservizi è affetto. Sono gestibili tramite tecniche di *graceful degradation*.

![image.png](12%20Performance%20Evaluation/image%207.png)

## 5.2. Metriche per la quantificazione

Per inserire l'affidabilità nei *Service Level Agreement* (SLA), è necessario misurarla quantitativamente utilizzando metriche statistiche basate su valori medi (assumendo che i guasti siano indipendenti e seguano distribuzioni gestibili come la gaussiana).

### 5.2.1. Definizioni Temporali

Considerando il ciclo di vita del sistema come un'alternanza di stati "Running" (verde) e "Failure" (rosso):

- **MTTF (Mean Time To Failure)** → tempo medio in cui il sistema funziona correttamente prima di incontrare un guasto.
- **MTTR (Mean Time To Repair/Recovery)** → ****tempo medio necessario per riportare il sistema in condizioni operative dopo un guasto.
- **MTBF (Mean Time Between Failures)** → tempo medio che intercorre tra due guasti successivi.
    
    $$
    MTBF=MTTF+MTTR
    $$
    

![image.png](12%20Performance%20Evaluation/image%208.png)

### 5.2.2. Availability e Reliability

- **Availability (Disponibilità - A):** la probabilità che il sistema sia disponibile in un preciso istante $t$. Si calcola come la frazione di tempo in cui il sistema è operativo:
    
    $$
    A=\frac{MTTF}{MTBF}
    $$
    
    Viene spesso espressa in "nines" (es. "3 nine" = 99.9% ≈ 8.8 ore di downtime/anno; "5 nine" = 99.999% ≈ 5 minuti di downtime/anno).
    
- **Reliability (Affidabilità - R):** la probabilità che il sistema funzioni correttamente senza interruzioni per un intero intervallo di tempo $τ$.

**N.B.** Su finestre di osservazione lunghe, le due metriche tendono a convergere qualitativamente verso lo stesso concetto.

<aside>
📌

**Calcolo dell’Affidabilità nei Sistemi**

Per dimensionare il sistema si usano le seguenti leggi probabilistiche:

- **Serie** → l'affidabilità totale è il prodotto delle singole affidabilità (basta un guasto per fermare tutto).
    
    $$
    R=\prod r_i
    $$
    
    **N.B.** Aggiungere componenti in serie abbassa drasticamente l'affidabilità.
    
- **Parallelo** → si calcola la probabilità che *tutti* falliscano $1-r_i$ e la si sottrae a $1$:
    
    $$
    R=1-\prod(1-r_i)
    $$
    
    **N.B.** Aggiungere componenti in parallelo aumenta l'affidabilità.
    
</aside>

<aside>
🔢

**Esercizi di Dimensionamento**

Es 1: Quante lame (server) con $R=0.95$ servono per avere un sistema con $R\ge 0.99$?

Siccome voglio aumentare l’affidabilità, utilizzo il sistema in parallelo: $1-(1-0.95)^N\ge 0.99 \Rightarrow 1- (0.05)^N\ge0.99 \Rightarrow 0.01 \ge (0.05)^N \Rightarrow N \ge 2$

---

Es 2: Ho componenti con $R=0.99$. Quanti posso metterne in fila senza scendere sotto $R=0.95$?

$0.99^N≥0.95 \Rightarrow N = 5$

</aside>

## 5.3. Fault Tollerance (Tolleranza ai Guasti)

La tolleranza ai guasti si basa su due pilastri fondamentali: 

- **Error Detection:**
    - **Concurrent Detection** → monitoraggio continuo durante il normale funzionamento. Permette di individuare rapidamente anomalie o guasti imminenti.
    - **Preemptive Detection** → il sistema, o parte di esso, viene messo offline per test approfonditi o stress test. Richiede ridistribuzione del carico su altre risorse.
- **Recovery:**
    - **Error Handling** → ripristina la funzionalità del sistema nel più breve tempo possibile. È la parte che incide direttamente sull’**MTTR**. Operazioni tipiche:
        - **Rollback:** ritorno all’ultimo stato stabile noto (backup, snapshot).
        - **Rollforward:** avanzamento verso un nuovo stato stabile corretto.
        - **Compensation/Graceful Degradation:** la ridondanza maschera il guasto, permettendo al sistema di continuare a funzionare, anche con risorse ridotte (es. RAID 5 con un disco guasto, cluster con N-1 nodi).
    - **Fault Handling** → serve a prevenire la ricomparsa di problemi:
        - **Diagnosi:** identificazione precisa del componente difettoso (pinpoint).
        - **Isolamento:** rimuovere il componente incriminato dal pool attivo.
        - **Riconfigurazione:** adattamento della topologia per funzionare senza il nodo guasto.
        - **Re-inizializzazione:** ripristino o sostituzione del componente e reinserimento nel sistema.

![image.png](12%20Performance%20Evaluation/image%209.png)

## 5.4. Teorema CAP

Quando si gestiscono dati replicati in un sistema distribuito, il teorema CAP (Brewer's theorem) afferma che è possibile garantire solo due delle seguenti tre proprietà contemporaneamente:

- **Consistency (C)** → tutti i nodi vedono gli stessi dati nello stesso momento (ogni lettura riceve la scrittura più recente).
- **Availability (A)** → ogni richiesta riceve una risposta (non di errore), senza garanzia che contenga i dati più recenti.
- **Partition Tolerance (P)** → il sistema continua a funzionare nonostante la perdita di messaggi o il partizionamento della rete.

In un sistema distribuito la Partizione di rete (P) è inevitabile, quindi la scelta reale è solitamente tra:

- **CP (Consistent + Partition Tolerant):** se c'è una partizione, il sistema sacrifica la disponibilità (rifiuta richieste) per mantenere i dati coerenti (es. database relazionali con ACID rigoroso).
- **AP (Available + Partition Tolerant):** il sistema risponde sempre (disponibile), ma i dati potrebbero non essere aggiornati su tutti i nodi (*Eventual Consistency*, tipico dei sistemi NoSQL).

![image.png](12%20Performance%20Evaluation/b85939cf-c4fd-4f39-a8c8-24510e0e6ea9.png)

---

# 6. Workload Forecasting

Per un buon **capacity planning** è fondamentale **prevedere il carico futuro**. Non basta conoscere il valore **medio** del traffico: occorre considerare anche la **varianza** e costruire **scenari** (ottimistico, pessimista, realistico). Il workload è **multidimensionale**: non si misura solo in richieste/secondo, ma include anche il tasso di arrivo di nuove sessioni, la loro durata, e il **mix delle operazioni** (leggere, pesanti, CPU-bound, I/O-bound).

## 6.1. Tipologie di Previsione

1. **Previsioni Qualitative** → si basa su fonti umane ed è utile per costruire scenari di riferimento quando mancano dati storici precisi.
    - **Interviste agli utenti finali:** questionari su un campione ampio per stimare bisogni e carichi previsti
    - **Opinioni degli esperti:** interviste libere per identificare trend e possibili evoluzioni del sistema
    - **Analisi dei trend tecnologici:** studio di report e ricerche di settore per anticipare cambiamenti futuri
2. **Analisi dei Dati Storici (Quantitativa)** → la previsione quantitativa si basa sull’analisi delle serie temporali storiche del carico, con l’obiettivo di identificare sia l’andamento di lungo periodo (trend lineari di crescita o diminuzione) sia eventuali componenti periodiche che si ripetono nel tempo, come la stagionalità giornaliera o settimanale.
    
    Un aspetto fondamentale è la **validazione del modello**: una parte dei dati storici viene esclusa dall’addestramento e utilizzata per verificare se la previsione, applicata al passato, è capace di ricostruire correttamente il presente. È un processo molto simile alla fase di training e validation tipica del Machine Learning.
    

## 6.2. Metriche di Errore e Regressione Lineare

Per valutare l'accuratezza di una previsione si utilizzano metriche standard di errore:

- **ME (Mean Error)** →  $ME=\frac{\sum_t(Y_t−F_t)}{n}$
- **MSE (Mean Square Error)** →  $MSE=\frac{\sum_t(Y_t−F_t)^2}{n}$
- **SSE (Sum of Square Error)** → $SSE=\sum_t(Y_t−F_t)^2$

<aside>
📌

**Regressione Lineare**

Per interpolare i dati e trovare un trend, si usa spesso la funzione di interpolazione lineare $Y=a+bX$. Per trovare la retta migliore si minimizza l'errore quadratico (metodo dei Minimi Quadrati - *Least Square Method*) calcolando i coefficienti $a$ e $b$.

</aside>

## 6.3. Calcolo della Prevedibilità: Autocorrelazione

Prima di applicare modelli complessi, è necessario capire se la serie temporale è prevedibile analizzando l'**autocorrelazione** (correlazione tra la serie $X(t)$ e se stessa traslata $X(t−lag)$. Interpretazione dei risultati:

- **Decadimento lento** → la serie è fortemente correlata al passato → **Facile da predire**
- **Decadimento rapido** → la serie è molto irregolare (randomica) → **Difficile da predire**, il passato non aiuta
- **Picchi (Spikes) a intervalli specifici** → indicano una forte **periodicità** nel comportamento del sistema

![image.png](12%20Performance%20Evaluation/image%2010.png)

---

# 7. Simulazione

La simulazione è fondamentale quando i modelli analitici diventano troppo complessi. È flessibile perché permette di rappresentare via software comportamenti difficili da descrivere con equazioni. Per le aziende è uno strumento prezioso: costa meno scoprire un problema in simulazione che in produzione. Permette inoltre di identificare bottleneck, testare scenari alternativi e “accelerare il tempo” (es. simulare una settimana in un’ora). A differenza dei modelli in forma chiusa, però, la simulazione richiede esperimenti e analisi statistica per stimare correttamente le variabili aleatorie.

Vi sono due tipi di simulazioni:

- **Simulazione Statica** → non considera lo scorrere del tempo. Viene utilizzata per analizzare stati di equilibrio o resistenza strutturale (es. calcolo agli elementi finiti per la fatica dei materiali in ingegneria civile: "il pilastro regge il carico?").
- **Simulazione Dinamica** → include esplicitamente la variabile temporale. È l'approccio principale per i sistemi informatici, dove interessa osservare l'evoluzione del sistema nel tempo.

Che possono essere:

- **Deterministiche** → i processi seguono comportamenti costanti o funzioni note senza incertezza.
- **Stocastiche** → i processi sono modellati tramite **variabili aleatorie** descritte da distribuzioni di probabilità.

I modelli possono essere:

- **Discreti** → il simulatore gestisce una lista di eventi. Il tempo non scorre uniformemente, ma "salta" all'istante del prossimo evento in coda (es. arrivo di una richiesta web, completamento di una I/O).
- **Continui** → il sistema è descritto da equazioni differenziali dove le variabili di stato cambiano con continuità. È più complesso da gestire in ambito informatico.

**N.B.** La scelta del livello di astrazione dipende dall'obiettivo: si può simulare **ogni singolo elemento** (es. auto nel traffico) per massima precisione, oppure usare una **scala aggregata**, considerando solo flussi medi, riducendo complessità e costi computazionali.

## 7.1. Componenti di un Simulatore

Un simulatore (es. Omnet++, che useremo come framework astratto) è simile a una libreria di componenti da assemblare. Gli elementi costitutivi sono:

- **Stato del Sistema** → strutture dati che descrivono la situazione corrente (variabili di stato, istante attuale $t$).
- **Coda degli Eventi** → un buffer ordinato che contiene le azioni future schedulate.
- **Data Collectors** → oggetti ottimizzati per raccogliere statistiche (medie, quantili) durante l'esecuzione senza dover salvare ogni singolo dato grezzo (anche se a volte è necessario salvare le serie temporali su disco, occupando molto spazio).
- **Funzioni di Gestione degli Eventi** → insiemi di funzioni richiamate quando si verifica un evento (es. `handleMessage`), che aggiornano lo stato e schedulano nuovi eventi.
- **Flusso di controllo (Main)** → carica la configurazione, inizializza ciascun elemento all’istante $t=0$, esegue il ciclo degli eventi e genera il report finale.

## 7.2. Processo di Simulazione

<aside>
🔑

**Pilastri di correttezza**

Quando si costruisce un modello, ci si scontra con tre problemi distinti che devono essere affrontati per garantire la qualità dello studio:

1. **Validazione (Il modello è giusto?)** → bisogna garantire che il modello concettuale rappresenti fedelmente il sistema reale.
2. **Verifica (Il software è giusto?)** → bisogna accertarsi che il simulatore (il codice) implementi correttamente il modello concettuale, senza bug o errori logici.
3. **Credibilità** → le informazioni estratte devono essere ritenute affidabili e accettabili dagli esperti del settore o dai committenti.
</aside>

Scrivere il codice è solo una parte. Un buono studio simulativo segue fasi precise per evitare il fenomeno **GIGO**(*Garbage In, Garbage Out*: dati sbagliati in ingresso producono risultati sbagliati):

1. **Definizione del Problema** → è la fase più critica. Bisogna chiarire cosa si vuole misurare, quali KPI sono rilevanti, quali parti del sistema vogliamo considerare e quali possiamo ignorare, insieme alle assunzioni che riteniamo valide.
    
    Errore comune: voler sapere “tutto” del sistema, segno che gli obiettivi non sono ancora chiari.
    
2. **Modellazione Concettuale** → si crea l'astrazione del sistema. È fondamentale documentare tutte le **assunzioni** (es. latenze, capacità). Il livello di dettaglio deve essere un compromesso: troppo dettaglio rende il modello non scalabile, troppo poco lo rende inutile 
3. **Validazione del Modello** → il modello deve essere condiviso e approvato dagli esperti del dominio prima di scrivere codice. Questo passaggio costruisce la **credibilità**: se il modello non convince gli esperti a priori, i risultati non saranno accettati.
4. **Implementazione e Verifica** → scrittura del codice e controllo della sua correttezza tramite:
    - **Unit Testing:** testare singolarmente ogni elemento o modulo del simulatore.
    - **Code Review:** far revisionare il codice da esperti esterni per individuare errori logici sfuggiti allo sviluppatore.
    
    Una volta verificato il codice, bisogna validare il comportamento del simulatore utilizzando approcci progressivi:
    
    - **Confronto con risultati noti:** Partire da scenari semplici con soluzioni analitiche note (es. coda M/M/1) per vedere se il simulatore le replica correttamente.
    - **Uso di Tracce Reali:** Se disponibili, utilizzare tracce di log reali come input e verificare se l'output è coerente con la realtà.
    - **Confronto con il Sistema Reale (Small Scale):** Un metodo molto potente è validare il simulatore lavorando in parallelo con un prototipo o una versione in piccola scala del sistema reale. Se i risultati reali e simulati coincidono su piccola scala, aumenta la confidenza nell'usare il simulatore per scenari più complessi o di larga scala.
5. **Sperimentazione e Analisi dei Dati** → si eseguono le simulazioni, si raccolgono i dati e si applicano le analisi statistiche. Una singola run fornisce solo una stima: servono quindi **più esperimenti indipendenti** per costruire intervalli di confidenza e ottenere risultati affidabili. Infine, si aggregano i dati raccolti e si risponde alle domande di valutazione iniziali.

## 7.3. Richiami Statistici per la Simulazione

La simulazione si basa intrinsecamente sulla statistica: gli input sono spesso stocastici e gli output sono campioni di variabili aleatorie da analizzare.

### 7.3.1. Concetti Fondamentali

- **Variabile Aleatoria (V.A.)** → rappresenta un fenomeno incerto. Ogni run di simulazione è un *campione* di una V.A., il cui spazio dei possibili valori può essere finito o infinito.
- **PDF (Probability Density Function)** → descrive la probabilità che la variabile assuma un certo valore. L'integrale (o la somma per valori discreti) deve valere 1.
- **Metriche Principali:**
    - Media ($\mu$) → valore atteso
    - Mediana → valore che divide la distribuzione a metà [$P(X\le x)=0.5$]
    - Varianza ($\sigma^2$) e Deviazione Standard ($\sigma$) → misurano la dispersione dei dati attorno alla media.
- **Indipendenza** → due variabili sono indipendenti se il valore dell'una non influenza l'altra. Se correlate, hanno una PDF congiunta difficile da gestire.

### 7.3.2. Analisi degli Output di Simulazione

L’output di una singola run (es. tempo di risposta medio) rappresenta solo uno dei possibili valori di una variabile aleatoria. Per ottenere risultati affidabili è necessario eseguire **più run indipendenti**, raccogliendo un insieme di campioni sufficientemente grande.

Grazie al **Teorema del Limite Centrale (CLT)**, la distribuzione delle medie campionarie tende a una gaussiana man mano che il numero di run cresce ($N \rightarrow \infty$). Questo permette di stimare **intervalli di confidenza** per le metriche di interesse, ad esempio:

- $\mu \pm \sigma$ → circa 68% di probabilità che contenga il valore vero
- $\mu \pm 3\sigma$ → circa 99% di probabilità

Per calcolare questi intervalli, è necessario stimare $\mu$ e $\sigma$ dai risultati delle run ripetute. Queste stime diventano più precise all’aumentare del numero di esperimenti indipendenti.

![image.png](12%20Performance%20Evaluation/164379f1-7311-4138-b1c0-acf829f76718.png)

### 7.3.3. Modellazione degli Input (Distribuzioni)

Gli **input del simulatore** (ad esempio il tempo tra due richieste) vengono generati a partire da **distribuzioni teoriche**, spesso ricavate da istogrammi di dati reali. Le distribuzioni più comuni includono:

- **Esponenziale** → tipica per i tempi di inter-arrivo nei processi di Poisson, grazie alla proprietà *memoryless*.
- **Gaussiana (Normale)** → problematiche per tempi o dimensioni, perché può produrre valori negativi. Si può usare la **Gaussiana troncata**, ma questo altera la media e la varianza originali.
- **Log-Normale** → molto diffusa nei sistemi informatici; corrisponde a una gaussiana su scala logaritmica, è asimmetrica e genera solo valori positivi.

![image.png](12%20Performance%20Evaluation/image%2011.png)

### 7.3.4. Generazione di Numeri Casuali (RNG)

I simulatori non usano generatori completamente casuali (RNG), ma **pseudo-casuali (PRNG)**. Questi numeri **sembrano casuali**, ma sono generati da un algoritmo deterministico a partire da un **seme (seed)**, garantendo la **ripetibilità** degli esperimenti. Quando si confrontano due scenari, si utilizzano gli stessi seed per entrambi gli esperimenti. In questo modo la variabilità casuale è identica e le differenze osservate dipendono solo dal sistema, rendendo il confronto più preciso.

È importante scegliere generatori moderni: evitare quelli obsoleti, come il **Linear Congruential**, che mostrano pattern evidenti, e preferire il **Mersenne Twister**, con periodo molto lungo e ottime proprietà statistiche.

### 7.3.5. Visualizzazione e Validazione Grafica

- **Box Plot** → rappresenta sinteticamente quartili, mediana e outlier. Può però nascondere dettagli se la distribuzione è multimodale (con più picchi). Infatti da come si denota nell’immagine diversi dataset possono generare lo stesso Box Plot:
    
    ![image.png](12%20Performance%20Evaluation/image%2012.png)
    
    **N.B.** Nella stragrande maggioranza dei casi i Box Plot forniscono informazioni affidabili e sufficienti.
    
- **Violin Plot** → combina Box Plot e funzione di densità (PDF), evidenziando meglio la forma della distribuzione e risolvendo alcuni limiti del Box Plot.
    
    ![image.png](12%20Performance%20Evaluation/0f903b2b-0378-44ba-9c7d-f97d05d6dbc6.png)
    
- **Analisi delle Correlazione:**
    - Scatter Plot →  visualizza coppie di valori $(x,y)$. Se i punti formano una nuvola diffusa o un’ellisse, le variabili sono incorrelate; se seguono una linea o curva, esiste correlazione.
- **Analisi della Distribuzione:**
    
    
    - P-P Plot (Probability-Probability) → confronta le **funzioni cumulative** (CDF) di due distribuzioni. L’asse $x$ e $y$ rappresenta i valori cumulativi, permettendo di valutare se le distribuzioni si muovono in modo simile, soprattutto al centro della distribuzione. È più sensibile ai valori tipici che compaiono frequentemente.
    - Q-Q Plot (Quantile-Quantile) → confronta i **quantili** delle due distribuzioni. L’asse $x$ e $y$ mostra i quantili normalizzati (tra 0 e 1), evidenziando meglio le differenze nelle **code** (valori estremi, testa e coda).
    
    ![image.png](12%20Performance%20Evaluation/image%2013.png)
    

## 7.4. Esecuzione: Transitorio vs Stato Stazionario

Le simulazioni possono essere:

- **Terminating** → hanno una fine naturale (es. "simula l'attività dalle 9:00 alle 18:00").
- **Non-Terminating (Steady State)** → si cerca il comportamento a regime.

In quest’ultimo caso è necessario gestire il **warm-up**, scartando il periodo iniziale in cui il sistema parte “a freddo”, e poi proseguire abbastanza a lungo da raccogliere dati affidabili. Per raccogliere i dati a regime si utilizzano due approcci:

- **Multiple run indipendenti** → si eseguono più simulazioni partendo ogni volta da zero, con **seed diversi**. In ciascuna run si scarta il warm-up e si raccoglie un singolo campione finale.
    
    Vantaggio: i campioni sono statisticamente indipendenti, semplificando l’analisi.
    
- **Single long run con batch means** → si esegue un’unica simulazione molto lunga; dopo il warm-up, l’esecuzione viene suddivisa in **lotti (batch)**, e la media di ciascun lotto è trattata come un campione.
    
    Svantaggio: possibile **autocorrelazione** tra lotti consecutivi, soprattutto se i lotti sono troppo brevi.
    

![image.png](12%20Performance%20Evaluation/image%2014.png)

---

# 8. Red Flag nella Progettazione

- **Single Point of Failure (SPOF)** → evitare colli di bottiglia o singoli componenti la cui rottura ferma l'intero sistema.
- **Over-engineering** → evitare complessità inutile. Soluzioni troppo articolate riducono l'affidabilità perché aumentano i punti di rottura e la difficoltà di sincronizzazione.
- **Centralizzazione vs Distribuzione** → a volte un controllo centralizzato (sebbene sia un potenziale SPOF) è preferibile per semplicità di gestione rispetto a complessi algoritmi distribuiti, ma va valutato caso per caso.

---