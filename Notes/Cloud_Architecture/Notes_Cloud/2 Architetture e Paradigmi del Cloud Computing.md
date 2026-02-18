# 2. Architetture e Paradigmi del Cloud Computing

Number: 4
Status: Done
Type: theory

# 1. Panoramica dei Servizi Cloud di Oggi

Un servizio è composto da un **provider** che espone un **endpoint** e un **consumer** che lo utilizza. La loro interazione è regolata da un **contratto** e una **politica**. La comunicazione avviene tramite lo scambio di **messaggi.**

![image.png](2%20Architetture%20e%20Paradigmi%20del%20Cloud%20Computing/image.png)

Il software per servizi cloud deve possedere le seguenti caratteristiche principali:

- **Disponibilità continua** → tolleranza zero ai downtime.
- **Sicurezza** → protezione dei dati e delle operazioni.
- **Scalabilità ed elasticità** → capacità di adattarsi dinamicamente ai carichi di lavoro.

---

# 2. **Le rivoluzioni che hanno portato alle applicazioni cloud-based**

Due svolte principali hanno reso possibile il passaggio dalle applicazioni eseguite localmente a quelle basate su cloud:

1. **Astrazione del software dall’hardware** → l’utente finale non sa (e non deve sapere) quanta potenza di calcolo viene utilizzata per eseguire il programma. Questo è possibile grazie al **cloud computing**, che rende trasparente l’infrastruttura sottostante.
2. **Astrazione del business service dal software** → l’utente finale non conosce né gli strumenti software né i dettagli tecnici utilizzati per erogare il servizio: vuole solo che sia economico e che fornisca risposte rapide. Questo principio è valido anche per le applicazioni tradizionali, ma nel cloud diventa ancora più importante, richiedendo uno sviluppo del software progettato per supportare efficienza, scalabilità e affidabilità.

<aside>
📌

**Errori Critici da Evitare**

Per ottenere questi risultati, bisogna evitare alcune criticità tipiche:

- **Single Point of Failure (SPoF)** → causa problemi di disponibilità costante.
- **Bottlenecks nel codice / non replicabilità** → limita la scalabilità del sistema.
- **Comportamento sincrono** → se un processo attende la risposta di un altro, si sprecano tempo e risorse, riducendo la scalabilità.
</aside>

---

# 3. Paradigmi di Sviluppo per il Software Cloud

Possiamo distinguere i paradigmi software da **evitare** e quelli da **preferire** nella progettazione di applicazioni cloud-based:

- **A livello Process-wise:**
    - **Da evitare:**
        1. *Large scale synchronisation* → introduzione di barriere che obbligano i processi a terminare prima di procedere agli step successivi. Questo approccio non è praticabile nel cloud, dove le latenze sono elevate e le attese generano **bottleneck**.
    - **Da preferire:**
        1. *Large scale parallelism* → algoritmi che parallelizzano il lavoro su più processi contemporaneamente, evitando barriere e riducendo i colli di bottiglia.
- **A livello Data-wise:**
    - **Da evitare:**
        1. *Strong consistency* → difficile da garantire su dati replicati in più regioni e costoso da mantenere.
        2. *Batch processing* → memorizzare tutti i dati prima di processarli non è adatto a flussi continui di dati (es. social network, sensori IoT).
    - **Da preferire:**
        1. *Lazy consistency* → i dati si aggiornano in modo eventuale. Esempi: database distribuiti come DynamoDB. Riduce traffico e costi.
        2. *Data distribution* → replicazione geografica dei dati per accesso più veloce e maggiore affidabilità.
        3. *Caching* → memorizzare i dati più usati vicino all’utente (edge caching), diminuendo il traffico verso i server centrali.
- **A livello Communication-wise:**
    - **Da evitare:**
        1. *Comportamento sincrono* → il processo attende la risposta prima di continuare, impraticabile su milioni di nodi con latenze variabili.
        2. *All-to-all communication* → ogni nodo comunica con tutti gli altri, causando esplosione di messaggi e bottleneck.
    - **Da preferire:**
        1. *Comportamento asincrono* → un processo non aspetta la risposta prima di procedere, evitando dipendenze e rendendo il software robusto.
        2. *Uso limitato di IPC* → minimizzare la comunicazione tra processi riduce dipendenze e rende il sistema più resiliente.

**N.B.** È evidente che non si può semplicemente portare un’applicazione pensata per PC sul cloud: occorre **ridefinire il design del software** secondo questi principi.

---

# 4. Software Structures

## 4.1. S**oftware Structure: categorie basate sui componenti**

Le principali architetture per lo sviluppo di servizi sono:

- **Monolitica:** un'unica grande applicazione software. Costoso da mantenere e difficile da replicare.
- **Pre-SOA:** un’architettura ibrida tra quella monolitica e quella SOA. Il servizio è sviluppato come insieme di software monolitici distribuiti su più macchine. Apre alla scalabilità permettendo di replicare il codice su più nodi, ma la comunicazione tra i moduli non è ancora standardizzata.
    
    *Esempio*: una banca con tre filiali potrebbe avere tre software monolitici separati, ciascuno su un server diverso, che comunicano tra loro con protocolli non standard.
    
- **SOA:** ogni funzionalità del software è resa con un servizio indipendente che comunica con gli altri tramite protocolli dedicati. Uno di questi modelli si chiama SOAP(Simple Object Access Protocol) e usa messaggi XML-based, questo formato e’ ben strutturato e leggibile da diversi linguaggi.
    
    Questa architettura offre universalità e buona interoperabilità grazie al linguaggio standard, ma la condivisione dei messaggi XML comporta overhead in termini di dimensioni e di elaborazione.
    
- **Microservizi**: l’evoluzione di SOA porta a microservizi più piccoli e con compiti specifici, progettati per esporre poche informazioni e ridurre il traffico tra servizi; utilizzata da aziende come Netflix, Amazon e Spotify, può avere microservizi classici con architettura REST/API (request-response) o microservizi event-driven che si attivano solo al ricevimento di eventi da un bus di messaggi (es. Kafka), offrendo il vantaggio di tanti piccoli servizi facilmente replicabili.
- **Serverless**: un'evoluzione dei microservizi in cui il client non deve occuparsi della gestione del server: invoca direttamente le funzioni esposte da un provider, come **AWS Lambda** o **Google Cloud Functions**, ed è il provider stesso a gestire l’intera infrastruttura server. In questo modello, i servizi non rimangono sempre attivi. Le funzioni sono **spente** e vengono attivate solo quando vengono invocate. Poiché le funzioni non sono continuamente in esecuzione ma si attivano in risposta a un evento o a una chiamata, questo tipo di architettura viene definito **event-driven**.

![image.png](2%20Architetture%20e%20Paradigmi%20del%20Cloud%20Computing/image%201.png)

## 4.2. S**oftware Structure:** **categorie basate sui metodi di elaborazione dei task**

A seconda delle esigenze del business, si può optare per due approcci principali:

- **Batch processing**: i dati vengono raccolti, raggruppati in lotti ed elaborati in un secondo momento, solitamente in modalità offline. Questo metodo è ideale per compiti che non richiedono una risposta immediata.
- **Real-time processing**: i dati vengono gestiti man mano che arrivano in un flusso continuo (*stream*). Questo approccio è necessario per servizi che richiedono una risposta istantanea o una logica in tempo reale.

## 4.3. **Software Structure: categorie in base alla modalità di comunicazione dei componenti**

- **Web services basati su SOA** → utilizzano protocolli complessi come SOAP o RPC e operano generalmente in modalità **sincrona**.
- **RESTful APIs** → sfruttano il protocollo HTTP, semplice e standardizzato, e sono generalmente **sincrone**.
- **Event-driven architectures (EDA)** → si basano su **messaggistica asincrona**, dove i componenti comunicano tramite eventi invece di chiamate dirette.

## 4.4. Software structure: categorie in base a modalità di coordinamento dei componenti

Esistono due modelli per coordinare l'interazione tra i servizi:

- **Orchestrazione**: c'è un **orchestratore centralizzato** che agisce come un "direttore d'orchestra", decidendo quali servizi invocare e in quale ordine. Questo approccio è più rigido e le interazioni sono spesso sincrone.
- **Coreografia**: non c'è un controllore centrale. Il coordinamento avviene in modo spontaneo, con i servizi che reagiscono a uno **stream di eventi** condiviso.

**N.B.** Generalmente viene utilizzato il linguaggio BPEL.

![image.png](2%20Architetture%20e%20Paradigmi%20del%20Cloud%20Computing/image%202.png)

---

# 5. Resilienza nei Sistemi Distribuiti

In un ambiente cloud, i guasti non sono rari ma la **norma**. La **resilienza** è la capacità di un sistema di continuare a funzionare nonostante i malfunzionamenti. Un problema critico è la gestione degli **errori a cascata**, in cui il fallimento di un servizio si propaga ad altri.

Per aumentare la resilienza, si adottano diverse strategia:

- **Servizi stateless** → un problema critico riguarda la **consistenza dello stato**. Se un servizio che gestisce, ad esempio, il carrello di un utente si arresta, tutto lo stato associato (i prodotti scelti) andrebbe perso. Le architetture **RESTful** in questo senso sono ideali perché i servizi sono senza stato. Se un servizio si arresta, un'altra istanza può prenderne il posto senza che si perda lo stato del cliente, migliorando scalabilità e recupero.
- **Graceful degradation** → è una politica che permette a un servizio di fornire una versione meno performante ma funzionale in caso di sovraccarico, anziché bloccarsi del tutto. Ad esempio, un sito e-commerce potrebbe mostrare consigli generici invece di quelli personalizzati.
- **Circuit Breaker →** è un pattern che gestisce i guasti. Quando un servizio client deve invocare un servizio remoto, lo fa attraverso un **proxy**. Questo proxy agisce come un "interruttore" a stati finiti che protegge il sistema dal servizio non funzionante. Un circuit breaker ha tre stati principali: `Closed` (tutto funziona), `Open` (il servizio è segnato come guasto) e `Half-Open` (si prova a ristabilire periodicamente la connessione).
    
    ![Screenshot 2025-09-23 at 12.05.18.png](2%20Architetture%20e%20Paradigmi%20del%20Cloud%20Computing/Screenshot_2025-09-23_at_12.05.18.png)
    

---

# 6. Modelli Pre-SOA e SOA

## 6.1. Modelli Pre-SOA

Prima dell'architettura SOA, i sistemi distribuiti si basavano su approcci come:

- **Remote Procedure Calling (RPC)**: è un meccanismo che permette a un client di invocare una procedura che risiede su un altro host **come se fosse una normale funzione locale**.
    
    <aside>
    📌
    
    **Registrazioni Procedure su un server binder**
    
    Per rendere possibile un sistema distribuito, è necessario che le varie macchine sappiano **dove si trovano i servizi** e come raggiungerli. In RPC questo problema viene risolto tramite un **server binder** (detto anche *name server* o *directory service*).
    
    Quando un server che offre un servizio viene avviato, esso **registra le proprie procedure presso il binder**. Il binder mantiene una mappatura tra:
    
    - il **nome del servizio / procedura**
    - il relativo **indirizzo IP e porta**
    
    Una volta ricevuto l’**ACK** dal binder, il server risulta correttamente registrato ed è pronto a ricevere richieste RPC. In questo modo, gli altri componenti del sistema distribuito possono interrogare il binder per sapere **dove si trova un determinato servizio**.
    
    ![image.png](2%20Architetture%20e%20Paradigmi%20del%20Cloud%20Computing/image%203.png)
    
    </aside>
    
    Quando il client effettua una chiamata RPC:
    
    1. La chiamata viene intercettata da uno **stub client**, un componente software che fa da intermediario.
    2. Lo stub esegue il **marshalling**, cioè impacchetta i parametri della chiamata in un formato trasmissibile (byte, JSON, binario, ecc.).
    3. I dati vengono inviati sulla rete verso il server.
    4. Sul server, lo **stub server** esegue l’operazione inversa (**unmarshalling**), ricostruendo i parametri originali.
    5. La procedura remota viene eseguita e il risultato viene rimandato al client con lo stesso meccanismo
    
    ![Screenshot 2025-09-20 at 14.12.36.png](2%20Architetture%20e%20Paradigmi%20del%20Cloud%20Computing/Screenshot_2025-09-20_at_14.12.36.png)
    
    L’**IDL (Interface Definition Language)** viene utilizzato per definire l’interfaccia delle procedure remote (nomi dei metodi, tipi dei parametri e valori di ritorno) e per generare automaticamente gli stub client e server.
    
    Le chiamate possono avere una **semantica**:
    
    - **At-least-once →** la chiamata viene ripetuta finché non ha successo
    - **At-most-once**  → la chiamata viene eseguita al massimo una volta
    
    <aside>
    🚨
    
    **Performance:**
    
    Dal punto di vista semantico, l'RPC (Remote Procedure Call) è paragonabile a una chiamata a funzione locale. Grazie all'uso del codice **stub**, la complessità della chiamata remota viene nascosta al programmatore. È sufficiente invocare la procedura con i parametri desiderati per inviare la richiesta all'host remoto.
    
    Nonostante questa apparente semplicità, a livello di performance la differenza è notevole. L'overhead legato al **marshalling** e **unmarshalling** dei parametri, insieme ai ritardi della rete, influisce significativamente sul tempo di esecuzione.
    
    </aside>
    
- **Object-oriented RPC (RMI, gRPC):** un'evoluzione orientata agli oggetti. Invece di chiamare una procedura remota (una semplice funzione), si invoca un metodo su un oggetto remoto. **Java RMI** è un esempio comune nel mondo Java. **gRPC** di Google è una variante più veloce, usata in scenari ad alte prestazioni (es. Netflix, Spotify).
- **Message-oriented Middleware (MOM):** mentre **RPC** e **OO-RPC** si basano su un approccio **client-server sincrono**, il **MoM (Message-oriented Middleware)** si basa su un meccanismo **asincrono**. In particolare, utilizza **code di messaggi (message queue)** che vengono popolate dai **producer**, i quali inviano i messaggi, e successivamente vengono smaltite dai **consumer**, che li ricevono ed eseguono le operazioni associate. Questo modello permette al producer e al consumer di non dover essere attivi contemporaneamente e riduce le dipendenze temporali tra i componenti del sistema, migliorando scalabilità e robustezza.

## 6.2. SOA

### 6.2.1. SOA Architecture

Questa architettura definisce vari blocchi di codice, ognuno legato. ad una funzionalità del business. Mirano a:

- Riuso di servizi per diverse applicazioni della stessa compagnia.
- Offrire qual servizio a compagnie esterne.

![image.png](2%20Architetture%20e%20Paradigmi%20del%20Cloud%20Computing/image%204.png)

**N.B.**  L'architettura SOA può essere applicata a scenari **intra-aziendali** e **inter-aziendali** (B2B).

### 6.2.2. SOA come Servizio

SOA non è solo una nuova tecnologia per codice modulare e distribuito, ma include anche un modello economico. Gli elementi chiave sono:

- **Service**: componente software che fornisce un concetto di business.
- **Contract**: specifica lo scopo, la funzionalità e i vincoli del servizio.
- **Interface**: descrive come utilizzare il servizio. Sostituisce quello che in RPC era il file `.idl`.
- **Implementation:** è l’effettiva implementazione del servizio tramite codice e database.

- **Technology Framework**: contiene la logica di business e i dati necessari per l'esecuzione del servizio.
    - **Front-end** → elementi che erogano il servizio all'utente finale (ad esempio, un API web).
    - **Repository** → **r**egistro dove i servizi vengono pubblicati per facilitarne la **scoperta (discovery)** tramite i loro attributi (es. IP, nome servizio ecc…).
    - **Enterprise Service Bus (ESB)** → ****componente **middleware** che facilita e coordina la comunicazione tra i vari servizi di un sistema distribuito. Il **bus** rappresenta l’unico punto in cui i servizi vengono esposti e interconnessi. Tutte le comunicazioni passano attraverso di esso.  L’ESB si occupa quindi del routing dei messaggi verso le loro destinazioni, della gestione tra differenti modelli di comunicazione (queuing asincrono, event-driven messaging, reliable messaging), della connessione come ponte tra protocolli differenti, del supporto a policy e QoS (Quality of Service), e in generale del monitoraggio, logging e sicurezza. Questo approccio si contrappone a quello *all-to-all communication*, che può risultare complesso da mantenere.

![image.png](2%20Architetture%20e%20Paradigmi%20del%20Cloud%20Computing/image%205.png)

---

# 7. Architettura a Microservizi

I **microservizi** sono uno stile architetturale che consiste nello sviluppare una singola applicazione come un insieme di servizi molto piccoli. Ogni microservizio gira in un proprio processo, comunica tramite meccanismi leggeri (es. HTTP API) e può essere scritto in un linguaggio diverso.

Sia SOA che microservizi permettono di modellare applicazioni complesse in pezzi più piccoli e gestibili. Tuttavia, i microservizi sono solitamente più **granulari** dei servizi SOA. Sono progettati per fare una cosa sola e farla bene, il che li rende più facili da testare.

Un grande vantaggio è che sono **indipendenti**. Si può aggiornare un singolo microservizio o spegnerlo senza che l'intera applicazione crolli, migliorando l'affidabilità e la gestione dei guasti. Questo approccio facilita il **continuous delivery and deployment.**

<aside>
🚨

**Problematiche principali:**

- **Comunicazione** → il grande numero di servizi e le loro continue interazioni possono creare un sovraccarico e rallentare le prestazioni complessive.
- **Coordinazione** → necessità di far lavorare insieme più servizi in modo che si comportino all'unisono per eseguire un'unica operazione. Questo può causare un sovraccarico, soprattutto se si utilizza un **orchestratore** centralizzato.
- **Service discovery** → riguarda la capacità dei microservizi di trovare e comunicare con altri servizi in un ambiente dinamico.
</aside>

---

# 8. Architetture Basate su Modalità di Comunicazione tra Componenti

## 8.1. **Web Services basati su SOA**

I **Web Services** sono un insieme di **standard e protocolli** utilizzati per implementare l’architettura **SOA (Service-Oriented Architecture)**. Essi permettono a sistemi software diversi di comunicare tra loro in modo interoperabile attraverso la rete.

<aside>
📌

**Web Service**

Un **Web Service** è un sistema software identificato da un **URI**, le cui **interfacce pubbliche e i meccanismi di collegamento (binding)** sono definiti e descritti tramite **XML**. La sua definizione può essere **scoperta da altri sistemi software**. Questi sistemi possono quindi **interagire con il Web Service** secondo le modalità prescritte dalla sua definizione, utilizzando **messaggi basati su XML** trasmessi tramite **protocolli Internet**.

</aside>

Gli elementi principali di un Web Service sono:

- **SOAP** (Simple Object Access Protocol): protocollo di comunicazione client–server. I messaggi SOAP vengono **incapsulati nel body dei messaggi HTTP/HTTPS**. Un messaggio SOAP è strutturato in tre parti principali:
    - **Envelope** → definisce il messaggio e ne delimita il contenuto
    - **Header** → contiene informazioni di controllo (sicurezza, autenticazione, routing, ecc.)
    - **Body** → contiene i dati effettivi della richiesta o della risposta
    
    **N.B.** Queste sezioni sono definite tramite un file in formato **XML**.
    
- **WSDL** (Web Services Definition Language): linguaggio di descrizione dei servizi basato su XML, che specifica le operazioni disponibili, i parametri e i formati dei messaggi.
- **UDDI** (Universal Description, Discovery and Integration): registro utilizzato per la **pubblicazione, la ricerca e la scoperta dei servizi**.

**N.B.** Un limite di SOAP e WSDL è la loro **lentezza**, dovuta al costo del *parsing* dell’XML, e il fatto che **non sfruttano pienamente i codici di risposta HTTP**.

## 8.2. REST Architecture

**REST** (REpresentational State Transfer) è un'architettura che ha spostato il paradigma di comunicazione dai servizi basati su **azioni** (come RPC) a quelli basati su **risorse** (*things*). Le API **RESTful** sono servizi web che aderiscono a una serie di vincoli architetturali, rendendoli particolarmente adatti per le applicazioni cloud.

<aside>

**Vincoli Architetturali di REST**

1. **Client-Server**: i ruoli di client e server sono separati. Il client si occupa dell'interfaccia utente, mentre il server gestisce dati e logica di business, permettendo loro di evolvere in modo indipendente.
2. **Stateless**: ogni richiesta dal client al server deve contenere tutte le informazioni necessarie. Il server non memorizza alcun contesto di sessione lato client tra una richiesta e l'altra. Questo è cruciale per la scalabilità e la resilienza del sistema.
3. **Cacheable**: le risposte devono essere etichettate come "memorizzabili in cache" per consentire ai client di riutilizzare i dati delle risposte passate, riducendo il carico sul server e migliorando le performance.
4. **Interfaccia Uniforme**: il vincolo più importante, che garantisce un modo standardizzato di interagire con le risorse. Si basa su identificazione delle risorse tramite **URI**, messaggi auto-descrittivi e, opzionalmente, **HATEOAS** (Hypermedia as the Engine of Application State).
5. **Sistema a Livelli**: l'architettura può essere composta da più livelli intermedi (es. proxy o load balancer) tra client e server, senza che il client ne sia consapevole.
6. **Code on Demand**: questo è un vincolo opzionale che permette al server di estendere le funzionalità del client inviando codice eseguibile.
</aside>

### 8.2.1. Paradigma CRUD e Caratteristiche Aggiuntive

Le azioni sulle risorse sono mappate sui metodi **HTTP** seguendo il paradigma **CRUD**.

<aside>
📌

**Paradigma CRUD**

**CRUD** è un acronimo che descrive le quattro operazioni fondamentali per la gestione delle risorse:

- **Create (Creazione)**: Viene mappata sul metodo `POST` per creare nuove risorse.
- **Retrieve (Lettura)**: Viene mappata sul metodo `GET` per recuperare una risorsa.
- **Update (Aggiornamento)**: Viene mappata sui metodi `PUT` o `PATCH` per modificare una risorsa.
- **Delete (Cancellazione)**: Viene mappata sul metodo `DELETE` per rimuovere una risorsa.

**Codice di Risposta HTTP**

A seguito di richieste HTTP, si riceve un codice che identifica se la richiesta e’ andata a buon fine o meno. In particolare ci sono 5 famiglie:

- `1xx` → informativo
- `2xx` → successo
- `3xx` → ridirezione
- `4xx` → errore client (es bad request se passo parametri sbagliati)
- `5xx` → errore server
</aside>

Per migliorare le performance, la comunicazione RESTful utilizza spesso il formato **JSON**, che è più leggero ed efficiente rispetto a XML. L'uso dei metodi HTTP e dei codici di risposta standard (es. `200 OK`, `404 Not Found`) è una parte fondamentale dell'interfaccia uniforme.

### 8.2.2. **Focus sulla Cache in REST**

REST (come HTTP) supporta nativamente il **caching**: una risorsa può essere salvata **lato client** o **lato proxy**, riducendo traffico e tempi di risposta.

- **Caching con TTL (Time To Live):**
    1. Si assegna un **TTL** alla risorsa in cache
    2. Finché il TTL non scade, la risorsa viene restituita direttamente dalla cache
    3. Alla scadenza del TTL, la risorsa deve essere ricaricata o verificata con il server
- **Caching con ETag:** l’ETag è un identificatore univoco della versione di una risorsa:
    1. Il server invia l’ETag nella risposta
    2. Il client salva risorsa ed ETag in cache
    3. Alla nuova richiesta, il client invia: `If-None-Match: <ETag>`
    4. Se la risorsa non è cambiata, il server risponde **304 Not Modified**; altrimenti invia la nuova versione

### 8.2.3. Focus su Interfaccia di Servizio

Nel modello **SOAP**, l’interfaccia del servizio è definita tramite **WSDL (Web Services Description Language)**. Nel modello **REST**, invece, non esiste uno standard nativo equivalente al WSDL per descrivere automaticamente l’interfaccia di un servizio. In assenza di una descrizione formale, lo sviluppatore dovrebbe fare riferimento direttamente al codice o a una documentazione scritta manualmente. Per colmare questa mancanza, si è affermato uno standard *de facto* chiamato **OpenAPI** (ex **Swagger**), che genera in automatico un’interfaccia utente chiara.

### 8.2.4. **Livelli di maturità di un’applicazione basata su REST**

Un’applicazione REST può essere valutata in base al suo livello di maturità, che cresce progressivamente con l’adozione dei principi REST.

1. **Definizione delle risorse** (nomi e gli attributi corrispondenti)
2. **Implementazione dei verbi HTTP:** si implementano correttamente i principali verbi HTTP. Se un certo metodo non è consentito per una specifica risorsa, il server deve gestire il caso restituendo un messaggio di errore appropriato (ad esempio: *“Questo metodo non è consentito per questa risorsa”* oppure un codice HTTP come **405 Method Not Allowed**).
3. **Hypermedia Controls:** definiscono il modo in cui il client può interagire con lo stato dell’applicazione. Il principio è che, dopo aver ricevuto la rappresentazione di una risorsa, il client dovrebbe avere a disposizione **tutte le azioni possibili e i “passaggi successivi” direttamente nella risposta**. In altre parole, la risposta del server non contiene solo i dati della risorsa, ma anche dei **link (controlli hypermedia)** che indicano:
    - quali operazioni possono essere eseguite
    - quali URI devono essere utilizzati
    - quale metodo HTTP deve essere chiamato
    
    ![image.png](2%20Architetture%20e%20Paradigmi%20del%20Cloud%20Computing/image%206.png)
    
    **N.B.** Gli Hypermedia Controls sono l’implementazione pratica del vincolo **HATEOAS (Hypermedia As The Engine Of Application State)**: il client non ha bisogno di conoscere in anticipo la struttura delle API, ma può scoprire dinamicamente come interagire con il sistema.
    

Inoltre, una buona pratica nello sviluppo delle API è **introdurre i cambiamenti in modo graduale**. Se un’API deve essere modificata, non va cambiata improvvisamente: è necessario lasciare un periodo di transizione e **avvisare i client**. Con HATEOAS è possibile includere un **messaggio di warning** nella risposta, ad esempio: “Dalla versione v2 questa chiamata ha cambiato nome”.

### 8.2.5. **REST APIs come standard per le web applications**

L’architettura REST è oggi lo **standard de facto** per le applicazioni web. I motivi principali sono:

- **Affidabilità** → basata su gestione degli errori e controlli continui.
- **Design centrato sulle risorse** → si parte dalle risorse da esporre e si definiscono solo i metodi necessari per gestirle, senza complicazioni superflue.
- **Semplicità e stateless** → le API non mantengono stato tra le richieste, facilitando la scalabilità e la distribuzione.
- **Firewall-friendly** → funziona bene su HTTP standard, rendendola adatta a un’ampia diffusione e integrazione.

## 8.3. Event-Driven Architecture (EDA)

L'**Event-Driven Architecture (EDA)** è un approccio che si basa su interazioni asincrone. L'idea è che un componente genera delle notifiche (**eventi**) a cui altri componenti rispondono. Questo approccio è ideale quando il tempo di elaborazione non è noto e il tempo di risposta non è necessariamente limitato.

Gli eventi possono essere qualsiasi cosa che modifichi lo stato del sistema, come un nuovo ordine, un "mi piace" a un prodotto o una richiesta di amicizia. La natura asincrona degli eventi permette un forte **disaccoppiamento** (*loose coupling*) tra i servizi, poiché chi produce un evento non ha bisogno di sapere chi lo consumerà.

Spesso l'EDA viene implementata tramite **code di messaggi** e un modello **Publisher/Subscriber (Pub/Sub)**. Se il carico di lavoro aumenta, si possono semplicemente aggiungere nuovi consumatori per processare gli eventi, rendendo l'architettura estremamente scalabile. Un altro concetto fondamentale è la **consistenza finale** (*eventual consistency*), per cui i dati si allineano nel tempo anziché in modo istantaneo, un modello adatto a molti contesti come per esempio i social network.

<aside>
✅

**Vantaggio delle architetture asincrone**

In un'architettura asincrona, come quella basata su eventi, l'arrivo di un **transitorio di messaggi** può essere gestito in modo efficiente. La presenza di una coda di messaggi funge da "polmone", permettendo al sistema di accodare i messaggi e processarli gradualmente. Questo impedisce al sistema di rallentare o bloccarsi a causa di un carico improvviso.

Al contrario, i sistemi sincroni non hanno questo meccanismo di buffering. Se un gran numero di richieste arriva contemporaneamente, l'intero sistema può rallentare notevolmente, in quanto ogni richiesta blocca il thread in attesa di una risposta.

</aside>

### 8.3.1. Esempi di EDAs

- **MQTT** → un protocollo leggero e veloce, nato in ambito automotive e ora molto usato nell'IoT. Funziona tramite un **broker** che fa da tramite tra i *publishers* e i *subscribers*. I messaggi sono definiti da una coppia *topic* e *valore*. I topic possono essere gerarchici (es. `riscaldamento/sensori/aula`) e le sottoscrizioni possono usare *wildcard* per abbonarsi a interi rami di topic. Il suo focus principale è la **bassa latenza**, mentre la **scalabilità** non è la sua priorità. È progettato per un modello con un server relativamente piccolo. Un esempio di broker che usa il protocollo MQTT è Moskito (esempio visto in classe).

![image.png](2%20Architetture%20e%20Paradigmi%20del%20Cloud%20Computing/image%207.png)

- **Apache Kafka** → è un sistema di *event streaming* che sacrifica un po' di latenza per gestire enormi volumi di dati. La sua architettura lo fa assomigliare più a un database che a un sistema di messaggistica tradizionale. La sua base è una struttura dati chiamata **log**, che consiste in una sequenza di record con timestamp. Kafka offre un set di API per lavorare su questi dati, che possono essere salvati, replicati o filtrati.
    
    Un cluster Kafka è composto da più **broker**, che consentono **scalabilità** (=è sufficiente aggiungere nuovi broker), **alta disponibilità** e **tolleranza ai guasti**. I dati sono organizzati in **topic** suddivisi in **partizioni**, permettendo elaborazione parallela e una distribuzione efficiente del carico (ad esempio per area geografica). Le partizioni possono essere **replicate** su più broker, garantendo continuità del servizio in caso di guasti. Questo lo rende estremamente popolare e utilizzato da molte grandi aziende come Netflix, Oracle e Spotify per la gestione di dati massicci.
    

![image.png](2%20Architetture%20e%20Paradigmi%20del%20Cloud%20Computing/image%208.png)

## 8.4. **Event-driven architecture + SOA / Microservizi**

Esistono architetture ibride in cui:

- Il **business process** è modellato come una **catena di eventi** (Event-Driven Architecture, EDA).
- Le singole **business functions** sono implementate come **servizi SOA o microservizi**.

Il concetto chiave è che le business functions sono **indipendenti tra loro** e comunicano esclusivamente tramite **pubblicazione di eventi**, senza chiamate dirette tra di loro.

![image.png](2%20Architetture%20e%20Paradigmi%20del%20Cloud%20Computing/image%209.png)

---