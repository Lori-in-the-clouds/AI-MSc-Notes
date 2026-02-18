# 1. Servizi Internet-Based

Number: 2
Status: Done
Type: theory

# 1. Introduzione ai Servizi Cloud

<aside>
💡

**Servizio**

Un **servizio** è l'abilità di un sistema, gestito da un **service provider**, di fornire risposte continue a specifiche richieste di un **cliente**. La continuità può essere definita da un contratto formale, come un **Service Level Agreement (SLA)** che prevede pagamenti e penali, oppure basata sulla **reputazione**. In quest'ultimo caso, la mancata erogazione del servizio non comporta penali contrattuali, ma la perdita di fiducia e clienti.

</aside>

## 1.1. Dalle applicazioni locali ai servizi cloud

- **1960** → nasce il computer, ma Internet non esiste ancora. I prodotti principali sono software distribuiti con licenze a pagamento (ad esempio Windows).
- **Tra il 1970 e la fine degli anni 1980** → nasce e si diffonde Internet. I primi servizi permettono principalmente lo scambio di informazioni (es. mail, FTP etc…).
- **Tra il 1995 e il 2005** → si diffonde il **WWW (World Wide Web)**, uno dei principali servizi di Internet. Il WWW permette di navigare e usufruire di un insieme molto vasto di contenuti e di accedere a numerosi servizi disponibili per gli utenti di Internet.
    
    I servizi che nascono sopra il WWW hanno come obiettivo uno scambio di informazioni più semplice e una comunicazione più articolata rispetto a semplici email o trasferimenti di dati (es. servizi di e-commerce).
    
- **2005** → nasce il concetto vero e proprio di **cloud computing**, con AWS che lancia EC2. I nuovi contenuti che nascono sopra il WWW hanno come obiettivo la creazione di un ecosistema di servizi che offra un’esperienza simile a quella di un’applicazione tradizionale.
    
    Si parla quindi di **ecosistemi di servizi** fruibili da dispositivi di diverso tipo (PC, tablet, smartphone, ecc.), cioè applicazioni **mashup / multichannel**, che vengono ospitate in modo scalabile grazie al **cloud computing**.
    

**N.B.** Quindi oggi il **software non è più il prodotto principale**, ma è diventato uno strumento per erogare servizi. In passato le aziende basavano i propri guadagni principalmente sulla distribuzione di software tramite licenze, ad esempio Windows per Microsoft. Oggi, invece, l’obiettivo è mettere a disposizione degli utenti **servizi online**, spesso tramite modelli di **abbonamento**.

## 1.2. Caratteristiche dei Servizi Cloud

I sistemi devono essere:

- **Robusti e sicuri** → devono funzionare sempre, poiché un'interruzione del servizio o una violazione della sicurezza può causare perdita di clienti e di fiducia.
- **Scalabili** → devono funzionare bene indipendentemente dal numero di utenti, gestendo picchi di traffico e attacchi DDoS (Distributed Denial of Service). Questa scalabilità è cruciale per la sicurezza e la stabilità del servizio.
    
    **N.B.** In ambito cloud, un attacco DDoS non mira più a bloccare il servizio (**denial of service**), che rimane funzionante grazie alla scalabilità, ma a causare un danno economico al fornitore (**denial of wallet**) a causa dei costi elevati del traffico generato.
    
- **Usabili** → devono essere facilmente utilizzabili da chiunque.
- **Apparentemente gratuiti** → per alcuni servizi, il modello economico si basa su aspetti diversi dai pagamenti diretti.
- **Infrastruttura distribuita** → i server si trovano in numerosi data center in tutto il mondo. La posizione fisica dei server è irrilevante per l'utente.

Dal punto di vista ingegneristico, l'obiettivo è progettare e implementare servizi che garantiscano **performance elevate** e **affidabilità**. Le metriche chiave per misurare le performance sono il **throughput** (tasso di risposta) e il **tempo di risposta**.

## 1.3. Availability Costante → Errore Critico da Evitare

L’**errore critico da evitare** per garantire un’**availability costante** è avere **single points of failure (SPoF):**

1. Individuarli non è semplice: bisogna analizzare l’intero sistema.
2. Una volta individuati, i SPoF vanno **replicati** per rendere il sistema più robusto.

Tuttavia, replicare i SPoF può essere **costoso e complesso** e può introdurre problemi di **coerenza dei dati**. Per questo motivo, prima di replicare, è fondamentale fare un’**analisi costi-benefici** e valutare se la replica è realmente necessaria.

Inoltre, è essenziale gestire gli errori in modo efficace (**graceful error handling**). Anche in assenza di SPoF, gli errori sono inevitabili e vanno gestiti correttamente. Ad esempio, se si verifica un problema, è preferibile far terminare solo le sessioni degli utenti interessati, senza compromettere il servizio per tutti gli altri.

## 1.4. Buone Performance → Errore Critico da Evitare

L’**errore critico da evitare** per garantire buone performance, anche con carichi di lavoro variabili, è avere **bottleneck**. Una volta individuati, queste parti possono essere **replicate e parallelizzate**, rendendo il sistema più **elastico**. Tuttavia, la **replica** può essere costosa: è quindi necessario fare un’**analisi costi-benefici**, valutando il miglioramento delle metriche rispetto al costo per ottenerlo.

Nel **design di un’applicazione cloud** è importante distinguere le parti di codice **parallelizzabile** da quelle **sequenziali**. In particolare, le porzioni sequenziali vanno mantenute al minimo, perché generano **bottleneck non replicabili** e quindi **non risolvibili**.

<aside>
📌

**Legge di Amdahl**

La **legge di Amdahl** è un modello che stima il potenziale aumento di velocità (**speedup**) che si può ottenere in un sistema parallelo. Si basa sull'assunzione che qualsiasi software ha una parte che può essere parallelizzata e una parte che, per sua natura, deve rimanere sequenziale.

Consideriamo:

- $T_p$ → tempo di esecuzione per la parte parallelizzabile
- $T_s$ → tempo di esecuzione della parte sequenziale
- $n$ → numero di processori utilizzati

Lo **speedup** $S$ definito come il rapporto tra il tempo di esecuzione su un singolo processore e il tempo di esecuzione su $n$ processori:

$$
S=\frac{T_s+ T_p}{T_s+\frac{T_p}{n}}
$$

Definiamo $p$ come la frazione del tempo totale che può essere parallelizzata:

$$
p=\frac{T_p}{T_s+T_p}
$$

Sapendo che:

- $T_p​=p⋅(T_s​+T_p​)$
- $T_s​=(1−p)⋅(T_s​+T_p​)$

La formula dello speedup diventa:

$$
S=\frac{T_p+T_s}{(1-p)\cdot(T_s+T_p) + p\cdot\frac{(T_s+T_p)}{n}}=\frac{\frac{T_p+T_s}{\textcolor{red}{T_p+T_s}}}{\frac{(1-p)\cdot(T_p+T_s)}{\textcolor{red}{(T_p+T_s)}} + p\cdot\frac{(T_p+T_s)}{n\cdot\textcolor{red}{(T_p+T_s)}}}=\frac{1}{(1-p)+\frac{p}{n}}
$$

Lo speedup massimo invece si ottiene quando $n \rightarrow \infty$:

$$
S = \frac{1}{1-p}
$$

Un aspetto importante di questa legge è che se la parte sequenziale del codice è grande rispetto a quella parallelizzabile, il guadagno di performance sarà limitato, anche con un numero infinito di processori. Di conseguenza, il codice che non può essere replicato o parallelizzato è probabilmente un **collo di bottiglia** e deve essere considerato un errore di progettazione.

---

Esempio:

Un esempio pratico è lo studio dei costi e dei tempi di esecuzione tra **Spark** e **Hive**, dove il tempo di setup fisso di Hive, non parallelizzabile, incide in modo significativo sul costo totale, rendendo meno vantaggiosa l'aggiunta di molte macchine.

![image.png](1%20Servizi%20Internet-Based/image.png)

</aside>

---

# 2. Cloud Computing

<aside>
💡

**Cloud computing**

Il cloud computing è un modello che permette un accesso on-demand, conveniente e diffuso a un pool condiviso di risorse informatiche configurabili. Queste risorse possono essere rapidamente allocate e rilasciate con uno sforzo minimo di gestione, promuovendo disponibilità, elasticità e sicurezza

**Obiettivo del Cloud:** trasformare le risorse inutilizzate in profitto, identificandole e rendendole vendibili come un servizio.

</aside>

## 2.1. L’evoluzione dei Paradigmi di Sviluppo

L'evoluzione della tecnologia ha portato dall'approccio **monolitico**, in cui una singola modifica può compromettere l'intera applicazione, a quello **orientato ai servizi**, tipico del cloud. In questo modello, le applicazioni sono costruite con blocchi di servizio che comunicano tra loro, il che supporta la multi-canalità (funzionamento su diversi dispositivi) e la facilità di modifica. 

<aside>
🚨

**Problema della Scala**

Con la crescita esponenziale dei dati, le soluzioni tradizionali e le loro assunzioni falliscono, poiché la gestione di quantità enormi di informazioni, come la creazione di indici in memoria o l'esecuzione di transazioni atomiche, diventa un problema di scala complesso e non più gestibile con gli strumenti convenzionali.

</aside>

## 2.2. Primo approccio: Grid Computing

Il percorso che ha portato al cloud computing è iniziato con la visione di trattare l'informatica come un servizio di pubblica utilità. I primi tentativi includevano il **Grid Computing**, il cui obiettivo era connettere le risorse di più organizzazioni per formare un "super-computer virtuale". Lo strumento principale era il Globus Toolkit, orientato a compiti di calcolo parallelo e batch, che però era complesso e con limitata interattività.

**Svantaggi:**

- **Complessità delle Policy:** la mancanza di un modello economico chiaro rende estremamente difficile definire regole (policy) per la gestione e lo scambio delle risorse tra i vari nodi.
- **Incompatibilità con il Real-time:** il Grid Computing è strutturato per l'esecuzione **batch** (in coda), risultando inadeguato per le necessità del Web moderno che richiede l'elaborazione immediata di piccoli task in tempo reale.

<aside>
🚨

**Problema: Workload variabile**

Uno dei limiti del Grid Computing era l'incapacità di gestire la variabilità dei carichi di lavoro. I carichi di lavoro reali non sono stabili ma mostrano pattern variabili. Questa variabilità si manifesta in tre modi:

- **Daily patterns:** andamenti giornalieri
- **Weekly patterns:** andamenti settimanali
- **Seasonal/event-related patterns:** andamenti stagionali o legati a eventi specifici, come il Black Friday

Per un'azienda, dimensionare la propria infrastruttura IT per soddisfare il carico minimo, medio o di picco comporta sempre delle inefficienze:

- **Capacità minima:** si perdono clienti durante i picchi di domanda
- **Capacità media:** non si riesce a soddisfare i picchi e c'è ancora potenza inutilizzata
- **Capacità di picco:** si hanno costi elevati e molta potenza sprecata quando la domanda è bassa

![image.png](1%20Servizi%20Internet-Based/image%201.png)

**N.B.** Le aziende hanno bisogno di qualcosa di **elastico** che aggiunga o rimuova macchine al bisogno, in modo da far corrispondere i costi dell'infrastruttura alla domanda reale.

</aside>

## 2.3. L’evoluzione dai Silos al Cloud

1. **Modello a Silos** → ****ogni applicazione era legata a un server fisico dedicato (1 applicazione → 1 Server).
    
    Contro:
    
    - I server sono sotto utilizzati
    - Il **management è complesso**, perché se un server si guasta è necessario riconfigurare l’applicazione su un altro server; questo richiede tempo e un intervento manuale per effettuare correttamente la migrazione.
2. **Server Consolidation** → l'introduzione delle macchine virtuali (VM) ha permesso di eseguire più applicazioni su un singolo server fisico (1 applicazione → 1 VM).
    
    Pro:
    
    - I server possono essere **utilizzati al massimo**, perché su ciascun server è possibile ospitare **più VM**
    - Permette il **vertical scaling**: è possibile modificare la potenza di calcolo delle VM o addirittura spostarle da un server all’altro. Ad esempio, se un servizio richiede maggiore potenza di calcolo, è possibile fornirgliela modificando le impostazioni della VM oppure spostandola su un server con maggiore disponibilità di risorse.
3. **Transizione al Cloud** → per passare da una semplice virtualizzazione al cloud computing, sono state aggiunte funzionalità chiave:
    - **Gestione e monitoraggio:** per avere una visione completa dell'infrastruttura.
    - **Provisioning automatico:** per allocare rapidamente le risorse.
    - **Metering:** per misurare e fatturare l'uso delle risorse.

![image.png](1%20Servizi%20Internet-Based/image%202.png)

---

# 3. Paradigmi del Cloud Computing

Il cloud computing si basa su due paradigmi principali:

1. **Paradigmi di Servizio (=cosa viene offerto)**
    
    
    - **IaaS (Infrastructure-as-a-Service) →** offre risorse di base come macchine virtuali e storage. L'utente ha il massimo controllo sull'infrastruttura.
    - **PaaS (Platform-as-a-Service) →** vengono offerte delle API che viene utilizzata per fornire dei servizi. L'utente gestisce solo l'utilizzo delle API, mentre il provider si occupa di tutto il resto.
    - **SaaS (Software-as-a-Service) →** fornisce un software pronto all'uso. L'utente ha il minor controllo.
    
    ![image.png](1%20Servizi%20Internet-Based/image%203.png)
    
    <aside>
    📌
    
    I **confini tra i modelli cloud non sono rigidi:**
    
    In realtà **non esiste una separazione netta** tra SaaS, PaaS, IaaS e FaaS; spesso i servizi reali sono **combinazioni ibride**:
    
    - **SaaS + PaaS** → il software viene fornito come servizio, ma può essere **esteso e personalizzato** tramite plugin, estensioni o funzioni **FaaS** (es. logica custom, automazioni, integrazioni).
    - **IaaS+ (IaaS potenziato)** → infrastruttura basata su **VM aventi alcuni setting automatici.** Rimane comunque la **piena libertà di intervento** sulla VM (sistema operativo, networking, configurazioni low-level).
    </aside>
    
2. **Paradigmi di Deployment (=dove si trova l’infrastruttura)**
    
    
    - **Cloud Pubblico** → l'infrastruttura è venduta a chiunque da un provider esterno (es. Amazon, Google).
    - **Cloud Privato** → l'infrastruttura è utilizzata e gestita esclusivamente da una singola organizzazione (es. grandi aziende o enti governativi).
    - **Cloud di Comunità** → l'infrastruttura è condivisa da più organizzazioni con interessi comuni (es. consorzi di ospedali).
    - **Cloud Ibrido** → combina due o più tipi di cloud, permettendo, ad esempio, di gestire il carico di base on-premise e i picchi su un cloud pubblico.
    
    **N.B.** In alcuni casi, aziende/consorzi sono obbligati per ragioni di privacy e sicurezza, a mantenere dati sensibili all’interno della propria organizzazione.
    
    ![image.png](1%20Servizi%20Internet-Based/027475bb-58cd-4c70-b6e6-01677d6dcb27.png)
    

---