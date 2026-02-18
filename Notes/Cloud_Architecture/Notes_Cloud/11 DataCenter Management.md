# 11. DataCenter Management

Number: 3
Status: Done
Type: theory

# 1. Introduzione

La gestione di un data center moderno non si limita al semplice mantenimento dell'operatività, ma rappresenta un complesso problema di **ottimizzazione**. L'obiettivo principale è sfruttare al meglio le risorse disponibili, decidendo ad esempio come mappare le macchine virtuali (VM) sui nodi fisici e quali nodi mantenere accesi o spenti.

Tipicamente, si cerca di massimizzare il profitto (*revenue*) servendo il maggior traffico possibile, e contemporaneamente di minimizzare i costi operativi, legati principalmente al consumo energetico e alla manutenzione. Per affrontare queste sfide si ricorre a tecniche di **Ricerca Operativa**, definendo una **funzione obiettivo** da ottimizzare soggetta a determinati **vincoli.**

---

# 2. Key Performance Indicators (KPI)

<aside>
💡

I **KPI** sono **metriche o indicatori** che misurano le prestazioni di un sistema. Possono essere usate come variabili della funzione obiettivo o come vincoli.

</aside>

 Tipicamente, le metriche prese in considerazione sono riguardo:

- **Consumo Energetico** → esistono modelli specifici per descrivere il consumo a livello di componenti (CPU, storage, rete):
    - **Modello del Server:** si assume che il consumo degli altri componenti sia in funzione del consumo del server. Un modello lineare comune descrive il consumo come la somma di una potenza base in stato di *idle* e una parte variabile proporzionale all'utilizzo della CPU.
    - **Fonti Energetiche:** è rilevante distinguere tra **Brown Energy** (da fonti fossili, disponibile e costante) e **Green Energy** (da fonti rinnovabili, spesso intermittente e non uniforme nel tempo). Le politiche di gestione mirano a massimizzare l'uso di energia rinnovabile quando disponibile. Questo abilita anche politiche di allocazione del workload:
        - i workload **real-time** (es. web server) non possono essere posticipati;
        - i **job batch** possono invece essere schedulati nei periodi di maggiore disponibilità di energia verde, riducendo costi e impatto ambientale.
    - **PUE (Power Usage Effectiveness):** è un indice che misura l'efficienza dell'infrastruttura, calcolato come rapporto tra l'energia totale assorbita dal data center e l'energia utilizzata dai soli apparati IT.
- **Service Level Agreement (SLA)** →definiscono i livelli di servizio garantiti ai clienti (es. disponibilità al 99.999%, tempi di risposta minimi). Il mancato rispetto degli SLA comporta penalità economiche (*penalty cost*), che devono essere incluse nella funzione di ottimizzazione come costi da minimizzare. Lo SLA, in particolare sul tempo di risposta, è una **composizione di indicatori** e viene misurato come un'**aggregazione di valori**. Questi valori formano una **distribuzione di probabilità:**
    
    ![image.png](11%20DataCenter%20Management/9048a956-2ee1-4974-b843-2bbde6fa1875.png)
    
    Per descrivere l'andamento della **maggior parte della misura** si utilizzano metriche come il **90-percentile** (= valore sotto il quale si trova il 90% delle osservazioni), evitando di basarsi su metriche fuorvianti come la media. Riguardo questa distribuzione di valori, è fondamentale che l'**observation window** (la finestra temporale di osservazione) sia **rappresentativa** di scenari reali.
    
    <aside>
    🚨
    
    **Criticità: Crescita Non Lineare del Tempo di Risposta**
    
    La definizione di un **SLA** è legata allo **scenario di utilizzo** (volume delle richieste). Il **tempo di risposta** del server cresce in modo **non lineare** con l'utilizzazione:
    
    ![image.png](11%20DataCenter%20Management/image.png)
    
    Quando il server si avvicina alla **saturazione** (utilizzo $\rho\approx1$), un piccolo incremento del tasso di arrivo delle richieste può portare all'**esplosione del tempo di risposta** (triplicando o raddoppiando). Pertanto, per offrire garanzie con alta probabilità, il livello di utilizzo del server deve essere mantenuto sotto una soglia critica, tipicamente **0.6 o 0.7 (60-70%)**.
    
    </aside>
    
- **Cicli Termici delle Risorse** → è possibile ottimizzare anche in base alla **temperatura media delle CPU e delle apparecchiature**. Alcuni studi introducono l’**acceleration factor**, una metrica che misura **quanto un componente invecchia** in funzione dei cicli di accensione e spegnimento.
    
    Questo è utile perché, se si vuole mantenere una certa temperatura in una zona del data center, non è sufficiente “spegnere le CPU più calde” e riaccenderle in seguito: i cicli on/off accelerano l’usura dell’hardware. L’ottimizzazione termica deve quindi trovare un equilibrio tra **controllo della temperatura** e **minimizzazione dell’invecchiamento dei componenti**.
    
- **Utilizzazione delle risorse** → la frazione di tempo o capacità di una risorsa che viene effettivamente utilizzata. Viene misurata in dettaglio per:
    - CPU Utilization: tipicamente definita come il rapporto tra il tempo di risorsa effettivamente utilizzato e l’intervallo di osservazione. Dal punto di vista della **teoria delle code**, l’utilizzazione rappresenta la probabilità che la CPU sia occupata:
        
        
        $$
        \rho=\frac{\lambda}{\mu}
        $$
        
        Dove 
        
        - $\lambda$ → **tasso di arrivo** delle richieste alla risorsa
        - $\mu$ → **tasso di servizio**, cioè quante richieste la risorsa può servire per unità di tempo
        
        ![image.png](11%20DataCenter%20Management/image%201.png)
        
    - CPU Load: la lunghezza della coda dei processi pronti per l'esecuzione (*runqueue*). Quando il CPU load supera il numero di core, il sistema entra in una condizione di **potenziale sovraccarico**.
    - Network Utilization: la frazione di tempo in cui la rete è occupata nel trasferire dati. Può essere misurata rispetto alla **banda disponibile** o al **throughput corrente**. È importante notare che la banda non raggiunge mai il 100% di utilizzo: a causa della natura algoritmica delle reti (CSMA/CD), oltre un certo tasso di utilizzo (~80%) aumentare ulteriormente la domanda provoca **più collisioni**, riducendo drasticamente la banda effettiva.
- **Reliability & Availability** → questi KPI misurano la capacità del sistema di operare **in modo continuo e senza guasti**:
    - **Availability (Disponibilità):** probabilità che il sistema sia operativo e pronto all’uso in un dato istante, spesso legata all’**uptime**.
    - **Reliability (Affidabilità):** probabilità che il sistema funzioni correttamente per un periodo di tempo specificato senza guasti.
    
    L’availability complessiva dipende da come sono collegati i sottosistemi:
    
    - **Sistemi in Serie** → il sistema fallisce se fallisce un qualsiasi componente della catena → l'Availability complessiva è **ridotta** (è inferiore o uguale a quella del componente meno disponibile)
    - **Sistemi in Parallelo** → il sistema continua a funzionare finché almeno un componente ridondante è attivo → l'Availability complessiva è **massimizzata** (è maggiore di quella del singolo componente).
    
    ![image.png](11%20DataCenter%20Management/4a2620f5-e70d-48ab-b9e6-d5cde5d714b0.png)
    

---

# 3. Modelli di Performance

<aside>
💡

I **modelli di performance** sono rappresentazioni matematiche o simulazioni del sistema che permettono di prevedere il comportamento dei KPI e, allo stesso tempo, di utilizzare questi KPI come vincoli per guidare le decisioni operative o progettuali.

</aside>

## 3.1. Modelli Energetici

L’obiettivo principale dei modelli energetici è ridurre il **consumo dei server** e, di conseguenza, l’energia necessaria per il raffreddamento del data center:

- **Modello Lineare** → considera la dissipazione di calore proporzionale all’utilizzo della CPU, la potenza totale è data dalla somma di una componente costante in *idle* e di una variabile proporzionale al carico.
    
    ![image.png](11%20DataCenter%20Management/52683f59-4b1c-4c6e-8a97-8f799b3c5963.png)
    
- **Modello basato su Dynamic Voltage Frequency Scaling (DVFS) →** un approccio più avanzato si basa sul **Dynamic Voltage Frequency Scaling (DVFS)**. Le CPU moderne possono adattare la frequenza del clock in base al carico di lavoro: frequenze più alte comportano maggior consumo perché aumentano la corrente, mentre cicli “vuoti” ad alta frequenza sono energeticamente inefficaci. I modelli di consumo più accurati esprimono il consumo in funzione del **cubo della frequenza**, catturando l’impatto dei cambi di stato della CPU.

<aside>
📌

**Altri possibili Componenti:**

- **Apparati di Rete Dedicati:** il consumo energetico di uno switch o router dedicato è **sostanzialmente costante** (costo fisso), indipendentemente dal throughput effettivo (uno switch a 10 Mbps o 1000 Mbps consuma circa la stessa energia).
- **Rete Software-Defined:** la situazione è **completamente diversa** nelle reti *software-defined* (SDDC), dove il **traffico influisce in modo significativo** sul consumo energetico, rendendolo un fattore variabile nell'ottimizzazione.
- **Dischi/Storage:** si possono includere modelli per il consumo di dischi.
- **Cooling:** I costi di raffreddamento sono spesso **scartati o considerati proporzionali** al consumo dei server, data la correlazione diretta tra l'energia consumata dal calcolo e il calore dissipato.
</aside>

## 3.2. Modelli di Tempo di Risposta

Per modellare i tempi di risposta dei server e stimarne le prestazioni, si utilizzano i **sistemi a coda**. Questo approccio permette di capire quanto tempo una richiesta trascorre in attesa e quanto in elaborazione (spiegato più approfonditamente al capitolo su Performance Evaluation).

**N.B.** Non possiamo usare i server al 100% a causa della natura non lineare delle code.

## 3.3. Modelli Economici

Quando si prendono decisioni di gestione in un data center, spesso si utilizzano **Key Performance Indicators (KPI)** di natura economica. I modelli economici servono a bilanciare obiettivi opposti:

- **Cloud Provider** → massimizzare la *revenue* (vendere le risorse inutilizzate, anche ad altri provider).
- **Utente** → minimizzare i costi a parità di prestazioni.

<aside>
🔑

**Tipologie di Istanze VM**

I costi e le garanzie delle risorse variano drasticamente in base al tipo di contratto (commitment). L'esempio tipico è il modello di pricing di AWS:

- **Reserved/Dedicated** → costo basso, impegno a lungo termine, alta affidabilità. Utili per gestire del **workload di base costante** (base load) che non subisce variazioni
- **On-Demand** → costo medio-alto, impegno a breve termine (orario), sempre disponibili. Utili per gestire dei picchi di traffico imprevisti o carichi variabili che eccedono la capacità riservata.
- **Spot** → costo variabile (asta), basso, nessuna garanzia di disponibilità (possono essere terminate dal provider). Utili per job **differibili** nel tempo (es. analisi dati batch, training ML) che possono essere interrotti e ripresi senza danni.
</aside>

---

# 4. Modellazione dei Problemi

In questa fase si analizzano i problemi reali di gestione di un Data Center, definendo variabili decisionali, funzioni obiettivo e vincoli per l'ottimizzazione:

- **VM Placement Problem** → è il problema classico di un Cloud Provider IaaS: dato un set di nuove richieste di VM con requisiti specifici, bisogna decidere **dove allocarle** (su quali server) e se è necessario accendere nuovi server, garantendo gli SLA.
    
    <aside>
    🔢
    
    **Modello Matematico: Bin Packing**
    
    Il problema è modellato come un **Bin Packing Problem**:
    
    - **Bin (Contenitori)** → i server fisici, ciascuno con capacità limitata.
    - **Items (Oggetti)** → le VM da posizionare, con l’obiettivo di minimizzare il numero di server attivi (o massimizzare l’utilizzo dei server disponibili).
    
    Il Bin Packing in ambito cloud presenta due forme di complessità:
    
    1. **Multidimensionale:** ogni richiesta è un vettore di risorse (CPU, RAM, Disco), non un singolo scalare. Bisogna avere spazio disponibile su tutte le dimensioni.
    2. **Temporale:** bisogna considerare l'utilizzo delle risorse su più istanti temporali (es. utilizzo fra 2 minuti o 2 ore), basandosi sull'utilizzo effettivo o sugli *hard limit* imposti. Questo rende il problema dinamico: una allocazione ottimale ora potrebbe non esserlo tra 10 minuti.
    
    ![image.png](11%20DataCenter%20Management/image%202.png)
    
    </aside>
    
    <aside>
    ✅
    
    **Soluzione: Class-Based Placement (CBP)**
    
    Poiché il problema è NP-Hard ed esplode con il numero di VM, si semplifica considerando le VM non come oggetti unici ma come **istanze di classi** (ad esempio cluster di macchine identiche).
    
    La soluzione consiste nel risolvere il problema allocando i “rappresentanti” delle classi, riducendo così la dimensionalità del problema (ad esempio da 10.000 VM a 1.000 gruppi). Una volta determinata l’allocazione dei rappresentanti, si gestisce poi a parte le VM che non rientrano perfettamente nei gruppi.
    
    </aside>
    
- **VM Migration Problem** →la **migrazione delle VM** serve a identificare server **sottoutilizzati** (carico ≤20%) e trasferirne le macchine virtuali su altri nodi, spegnendo poi i server vuoti per risparmiare energia, operazione nota come **Server Consolidation**.
    
    Il problema si modella in **time slot**, durante i quali si decide **quali VM migrare**, su quali server allocarle e lo **stato dei server** (on/off). Poiché la migrazione comporta un costo in termini di trasferimento dati e banda, l’ottimizzazione si basa su una valutazione economica: il risparmio energetico deve giustificare il costo della migrazione.
    
- **Network Optimization Problem** → si concentra sull’**ottimizzazione del trasferimento dati e dell’uso della rete**, mirando a ridurre consumo energetico, colli di bottiglia e numero di hop tra nodi.
    
    La strategia principale è il **clustering**: raggruppare VM “ben connesse” sullo stesso rack o server fisico per mantenere il traffico locale e alleggerire la rete core.
    
- **Orchestrazione** → questo problema riguarda l’**utente Cloud** che deve gestire applicazioni complesse. L’input è un insieme di **microservizi** con workload e requisiti SLA, modellati come un **grafo (DAG)** che rappresenta dipendenze e comunicazioni tra i servizi.
    
    L’obiettivo è individuare il **deployment ottimale**, minimizzando costi e rispettando SLA, scegliendo provider, tipo e dimensione delle VM, e considerando l’**elasticità** del carico nel tempo. Una corretta rappresentazione delle dipendenze tra microservizi è essenziale per un deploy efficiente.
    

---

# 5. Some Management Strategies

1. **Utilization-Based (Regola 80-20)** → una delle strategie più diffuse nei data center è la **regola dell’80–20**: se l’utilizzazione della CPU supera l’80%, si migrano alcune VM per evitare il sovraccarico; se scende sotto il 20%, si spostano tutte le VM altrove e si spegne il server per risparmiare energia (**server consolidation**).
    
    Il controllo è spesso affidato a un **controller centrale** o a un orchestratore che monitora i server e decide le migrazioni. Questo approccio può però causare fenomeni di **instabilità (ping-pong)**: se un server oscilla, ad esempio, tra il 79% e l’81% di utilizzo, le VM rischiano di essere migrate continuamente senza reali benefici.
    
2. **EcoCloud (Approccio Stocastico)** → utilizza un **algoritmo decentralizzato e probabilistico**: ogni server decide in modo **locale**, basandosi solo sul proprio livello di carico, se **accettare** nuove VM o **migrarle** altrove.
    - La probabilità di **accettare VM** è massima nella zona di carico ottimale (60–70%) e scende sia quando il server è troppo scarico, sia quando è sovraccarico (oltre il 90% diventa zero):
        
        ![image.png](11%20DataCenter%20Management/image%203.png)
        
    - La probabilità di **scaricare VM** aumenta invece quando il server è molto scarico (per spegnersi) o molto carico (per ridurre la pressione locale):
        
        ![image.png](11%20DataCenter%20Management/image%204.png)
        
    
    Questo comportamento stocastico permette di ottenere un **bilanciamento globale** efficace senza modelli complessi e con **meno migrazioni** rispetto ad algoritmi deterministici. L’approccio converge rapidamente ed è semplice da implementare.
    
    ![image.png](11%20DataCenter%20Management/image%205.png)
    
3. **MBFD (Modified Best Fit Decreasing)** → estende il *best-fit decreasing* (valido solo per VM uniformi) per gestire **VM eterogenee**, con l’obbiettivo di **minimizzare il consumo energetico** (modellato linearmente). L’algoritmo opera in due fasi:
    - **Placement:** le VM vengono ordinate per utilizzo decrescente e assegnate al server che comporta il minor aumento di consumo energetico (*best fit*). Usato per l’ingresso di nuove VM.
    - **Consolidation:** per host sovraccarichi o sottoutilizzati si selezionano le VM da migrare secondo diverse strategie:
        - **Minimum Potential Growth:** si rimuovono prima le VM più grandi → facilita lo spegnimento del server.
        - **Highest Potential Growth:** si rimuovono prima le VM più piccole → massima granularità e vicinanza alla soglia di utilizzo.
        - **Random Choice:** scelta casuale, usata come baseline di confronto.

1. **Approccio Gerarchico (Multi-Layer)** → quando un’applicazione è distribuita globalmente, la gestione delle risorse diventa **gerarchica**, con decisioni prese su due livelli:
    - **Livello Alto: Multi Data Center** → decide come distribuire il traffico totale tra regioni o provider cloud, rispettando vincoli di **latenza**, **costo** e **domanda prevista**. Le decisioni hanno una **scala temporale lenta (ore)**, in linea con il billing orario dei cloud provider, e richiedono **predittori del workload** per stimare la richiesta futura. Qui si considerano sia la località dell’utente sia le differenze di costo tra regioni.
    - **Livello Basso: Single Data Center** → gestisce la quota di traffico assegnata dal livello globale. Le decisioni sono rapide (secondi o minuti) per reagire ai burst non previsti. Il controller locale si occupa di **scaling**, **allocazione delle VM**, gestione degli errori del livello superiore e **coerenza dei dati**, garantendo efficienza e stabilità interna.
    
    <aside>
    💡
    
    **Load Balancer Manager**
    
    L’architettura complessiva prevede un **Load Balancer Manager** centrale che smista il carico ai vari provider. Ogni provider dispone poi di un gestore locale che assegna le VM ai server fisici, gestisce lo storage e compensa le variazioni di traffico.
    
    </aside>
    
    La strategia complessiva distribuisce il traffico su base oraria (long-term) e ottimizza l’allocazione locale su base di minuti (short-term), minimizzando i costi delle VM già attivate e limitando le migrazioni. Questa combinazione consente di gestire applicazioni distribuite globalmente in modo **efficiente, reattivo e orientato ai costi**.
    

![image.png](11%20DataCenter%20Management/3a65c0b0-4371-4257-bf03-d0f98a7777e9.png)

1. **VM Planner** → ottimizza il traffico di rete nei data center, considerando l’impatto del **placement delle VM** sulla rete. L’obiettivo è **raggruppare VM che comunicano intensamente** e posizionarle vicine (stesso rack) per ridurre l’uso dei link core, degli switch e il consumo energetico, permettendo di **spegnere parti della rete non necessarie**. Essendo un problema **NP-hard**, viene suddiviso in sottoproblemi più semplici da risolvere separatamente:
    1. Le VM sono raggruppate per pattern di traffico
    2. I gruppi di traffico vengono mappati sui vari **rack**
    3. Si mappa il traffico tra gruppi (Multi-Commodity Flow) per minimizzare il carico complessivo
    
    Per accelerare i calcoli si utilizzano **euristiche e stime**, permettendo di gestire grandi data center con una leggera perdita di precisione a favore delle prestazioni.
    

![image.png](11%20DataCenter%20Management/2f261d32-29a4-4f2f-8b4b-da22585fb49e.png)

1. **Adaptive Infrastructure** → gestisce in modo dinamico la rete dei data center, distinguendo tra due classi principali di traffico: **elephant** e **mice**. In base ai flussi tra le VM, la rete può essere **riconfigurata on demand** tramite una matrice gestita da un controller del traffico. Questo permette di ottimizzare le risorse e ridurre congestioni. La rete diventa quindi **bipartita**:
    - una parte dedicata ai flussi **grandi** (Elephants) → **circuit switching:** percorsi dedicati ad alta banda; la latenza elevata è tollerabile.
    - una parte dedicata ai flussi **piccoli** (Mice) → **packet switching:** percorsi condivisi ma rapidi; la latenza deve rimanere bassa.

![image.png](11%20DataCenter%20Management/image%206.png)

1. **JCDME (Joint Optimization)** → modello complesso che ottimizza congiuntamente l’**energia di calcolo**, l’**energia di trasmissione dati** e il **costo di migrazione delle VM**.
    
    Il fattore **γ** indica la “convenienza” della migrazione: una VM viene migrata solo se il risparmio energetico futuro copre il costo immediato della migrazione (trasferimento dati + overhead CPU) entro un certo orizzonte temporale. Il modello considera anche il **costo di rete**, poiché collocare VM comunicanti su nodi diversi aumenta il consumo complessivo.
    
    - $\gamma = 0$ → migrazione facile
    - $\gamma = 1$ → migrazione difficile
    
    Un limite del modello è l’uso del **single time-step**: considerando solo il risparmio nel prossimo intervallo, molte migrazioni non vengono effettuate perché il costo immediato non viene compensato subito. Per superare questo problema si introduce un **peso temporale**, che “spalma” il costo della migrazione su più time slot futuri, valutando il beneficio energetico complessivo e permettendo decisioni più realistiche.
    
    Per stimare $\gamma$ sono stati confrontati tre metodi: **adattivo**, **smoothing** e **Newton**, con quest’ultimo risultando il più preciso.
    
    ![image.png](11%20DataCenter%20Management/image%207.png)
    

---