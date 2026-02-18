# 7. Serverless Computing

Number: 2
Status: Done
Type: theory

# 1. Introduzione

Il modello *Serverless* è un'evoluzione del **PaaS** (*Platform as a Service*) e si basa sull'idea di astrarre completamente la gestione e la manutenzione dell'infrastruttura, delegandola al *provider*. L'obiettivo finale del *serverless* è il passaggio dal modello **DevOps** al **NoOps**, eliminando la necessità di un team di Operations (= il livello di automazione è talmente avanzato che la gestione dell'infrastruttura non richiede più l'intervento umano).

## **1.2. Vantaggi e Modello di Costo**

Il *provider* gestisce automaticamente funzionalità cruciali come la **scalabilità**, la **disponibilità**, il *monitoring* e la **sicurezza** , riducendo preoccupazioni e costi per il team di sviluppo. Il modello di pagamento è **per-uso** (*per-use cost model*). Non si paga per il tempo di inattività (*idle server time*), ma i costi si basano su metriche precise:

- Numero di invocazioni
- Memoria media utilizzata e tempo di esecuzione (spesso misurato in **Giga-secondi**)

Questo approccio *fine-grained* permette un notevole taglio dei costi , ma deve essere gestito con attenzione: un *loop* infinito o un *design* errato possono generare fatture molto elevate.

## 1.3. Categorie di Servizi Serverless

Il modello *Serverless* si articola in due categorie principali:

- ***Backend as a Service (BaaS)*** → offre servizi *managed* per il *backend* di un'applicazione (ad esempio, database o servizi di autenticazione). Un esempio è **Google Firestore** o **Amazon S3**.
- ***Function as a Service (FaaS)*** → fornisce un ambiente per l'esecuzione di singole funzioni o blocchi di codice in risposta a specifici eventi. Un esempio è **Google Functions**.

---

# 2. Function as a Service (FaaS)

Il **FaaS** è un approccio ideale per carichi di lavoro elastici, poiché permette di scalare l'esecuzione di singole funzioni in modo molto fine.  La definizione del software si basa su singole **funzioni.** Le tecnologie alla base del FaaS sono le stesse che abilitano la containerizzazione. 

## 2.1. Caratteristiche Fondamentali

Le funzioni sono:

- ***Stateless*** → non mantengono stato tra le diverse esecuzioni e non possono avere risorse locali condivise, il che le rende **estremamente parallele**.
- ***Ephemeral*** → non si ha controllo sulla loro esecuzione o localizzazione e possono essere eliminate (*teared down*) quando non sono in uso.
- ***Event-Driven*** → l'esecuzione è attivata da ***trigger*** specifici (es. richieste HTTP, caricamento di file, messaggi in una coda).
- ***Fully Managed*** → l'intera piattaforma e il *runtime* sono gestiti dal *provider*, permettendo agli sviluppatori di concentrarsi sulla sola logica di business.

<aside>
🚨

**Svantaggi:**

- ***Difficoltà di Debugging e Monitoring*** →Il *tracing* e il *debugging* delle funzioni sono compless. Non si ha accesso a un *debugger* o alla macchina su cui il codice è in esecuzione. Ci si deve affidare unicamente ai **log** forniti dal *provider*, e analizzare una catena di invocazioni fallite (magari per dati corrotti ricevuti da una funzione a monte).
- ***Cold Start vs. Warm Start*** → una funzione può subire un ritardo alla prima esecuzione (*cold start*) se il suo *container* non è attivo, a differenza del riavvio quasi istantaneo (*warm start*) di una funzione già pronta.
- ***Limitazioni*** → l'approccio FaaS non è adatto per applicazioni con latenza critica o per calcolo ad alte prestazioni (*HPC*), poiché sono previsti limiti rigidi sulla durata e sulle risorse dei *task*.
- ***Vendor Lock-in*** → il codice è strettamente legato all'ambiente del *provider*, rendendo difficile la migrazione tra piattaforme. Il *time-to-market* è ridotto, ma uscire dall'ecosistema diventa "estremamente oneroso e costoso". La strategia migliore è usare i sistemi *managed* per la versione 1.0 e iniziare subito a lavorare alla 2.0 per disinnescare il *lock-in*.
</aside>

## 2.2. Considerazioni sulla Sicurezza

La sicurezza nel modello Function as a Service (FaaS) può essere vista come un'arma a doppio taglio, poiché sposta il focus della responsabilità:

- **Lato Positivo:** se la funzione è sicura, il resto lo è, poiché le dipendenze e l'ambiente sono gestiti dal *provider*.
- **Lato Negativo:**
    - **Spostamento della Vulnerabilità** → la responsabilità per le vulnerabilità si trasferisce dal tuo codice a quello del provider.
    - **Problema della Monocultura** → poiché la stessa infrastruttura FaaS è comune a una moltitudine di progetti, anche molto diversi tra loro, una singola vulnerabilità in un componente del provider potrebbe compromettere simultaneamente un vasto numero di deployment cloud.

## 2.3. Analisi Sperimentale (FaaS vs Docker)

Da un esperimento che confronta un'applicazione FaaS con un'applicazione basata su *container* Docker, eseguite con le stesse funzioni di *Face Recognition* e *Image Conversion***,** emergono tre punti chiave:

- **Durata di esecuzione**
    - **FaaS** presenta una **grande differenza tra *cold start* e *warm start***. Un *cold start* è molto lento, ma un *warm start* è più veloce di Docker perché la funzione è già preparata.
    - **Docker non presenta alcuna differenza** tra *cold start* e *warm start*. Il *container* è sempre in esecuzione in attesa di richieste.

![image.png](7%20Serverless%20Computing/image.png)

- **Utilizzo delle risorse**
    - **Utilizzo della CPU:** durante un *warm start*, il FaaS si dimostra **più efficiente di Docker** nel consumo di CPU. Docker ha un *overhead* aggiuntivo per il *parsing* e la decodifica delle richieste HTTP.
    - **Utilizzo della Memoria:** Il FaaS ha un ***footprint*** **di memoria maggiore** rispetto all'approccio *container*. Ciò può essere attribuito alla presenza di più *software layer* e a *data cache* per gestire le invocazioni. Questo giustifica la nascita di FaaS ottimizzati per sistemi piccoli (es. OpenFaaS con `FastD` per Raspberry Pi).

![image.png](7%20Serverless%20Computing/image%201.png)

---

# 3. Piattaforme FaaS

L'idea di base di FaaS è uguale dappertutto, ma **ci sono *vendor* che si producono *framework* diversi.**

## 3.1. Classificazione Piattaforme FaaS

Esiste uno spettro di piattaforme FaaS:

- **Container-based (es. OpenFaaS)** → richiedono all'utente di fornire un'immagine *container* e di avere un *container engine* installato.
- **Code-based (es. OpenWhisk)** → l'utente fornisce solo il codice sorgente (es. Python) e il *Dockerfile* viene generato automaticamente (*hidden*).

![image.png](7%20Serverless%20Computing/image%202.png)

## 3.2. OpenWhisk

Sviluppato da IBM Research e rilasciato con licenza Apache per spingere sulle tecnologie aperte e contrastare il *vendor lock-in*.

### 3.2.1. Modello di Programmazione e Azioni

OpenWhisk segue il classico modello *event-driven* (guidato dagli eventi) del FaaS. L’unità fondamentale è l’**Azione**, ovvero il frammento di codice o la funzione da eseguire.

Un’**Azione** viene attivata da un **Trigger** (una classe di eventi, ad esempio una modifica al file system o un aggiornamento di un topic MQTT) quando una **Regola** (una condizione *if-this-then-that*) rileva un evento proveniente da un **Feed** (una sorgente di dati).

![image.png](7%20Serverless%20Computing/image%203.png)

Le Azioni sono identificate in modo gerarchico (per raggruppamento e per evitare conflitti) da un **Namespace**, un **Package name** e l'**Action name** (es. `/whisk.system/samples/greeting`).

### 3.2.2. Runtime, Requisiti e Semantica

Le azioni vengono eseguite all'interno di un **Runtime** fornito (container già pronti per linguaggi come Go, Java o Python).

- **Requisiti** → le funzioni devono essere **Stateless** (poiché non è garantito che lo stesso *runtime* gestisca due invocazioni successive), **non atomiche** (per cui è necessario prestare attenzione ai *deadlock*) e non devono dipendere dall'ordine di esecuzione. Inoltre devono essere **Idempotenti**.
    
    <aside>
    🔑
    
    **Idempotenza:**
    
    Proprietà fondamentale che garantisce che un'operazione, anche se eseguita più volte, produca lo stesso risultato finale che si otterrebbe eseguendola una sola volta.
    
    **N.B.** È la capacità di un sistema di dire *"Ho già fatto questo lavoro, non lo rifarò, ma ti confermo che è andato a buon fine"*.
    
    </aside>
    
- **Input e Output** → l'input e l'output per le azioni sono **dizionari JSON** (sequenze chiave-valore, dove la chiave è una stringa e il valore è un JSON valido).
- **Semantica di Invocazione** → ****OpenWhisk supporta due modalità:
    1. Request/Response (Sincrona): l'invocazione è **bloccante** e restituisce un output (utile per *Web Actions* HTTP).
    2. Fire-and-Forget (Asincrona): l'azione viene avviata senza che l'utente aspetti un *feedback* sincrono (utile per *data ingestion*). Per monitorare lo stato o recuperare il risultato in un secondo momento, è necessario richiedere un "activation record" (=un identificativo univoco dell'esecuzione).
- **Log e Architettura** → per le richieste **Fire-and-Forget**, il *feedback* asincrono deve essere tracciato analizzando i **Log**. Questi *log* sono conservati come **Activation Records**.
    
    L'architettura tipica di OpenWhisk utilizza:
    
    - Un *frontend* (NGINX) per gestire le richieste in ingresso.
    - **Kafka** come *message bus* per la spina dorsale del sistema di messaggistica.
    - **Istanze Docker** come *Invokers* per far girare le azioni.
    - **CouchDB** per lo *storage* dei metadati e dei log.
    
    ![image.png](7%20Serverless%20Computing/8e5a3e0e-3f8d-4431-ad3d-f23f056fa4a4.png)
    

## 3.3. OpenFaaS e FaaSD

OpenFaaS è un'altra piattaforma FaaS popolare, che però richiede un approccio *container-based* (l'utente deve fornire l'immagine).

<aside>
📌

**FaaSD**

Per ovviare al *footprint* di memoria dei sistemi FaaS, è stata creata una versione estremamente ridotta di OpenFaaS chiamata **FaaSd**. FaaSd è progettata per **sistemi piccoli** (*embedded*), si basa su C*ontainerD*. Esistono pacchetti binari che ne semplificano l'installazione su schede come **Raspberry Pi**, rendendolo un riferimento per l'**Edge Computing** .

</aside>

## 3.4. AWS Lambda

È il prodotto commerciale FaaS di Amazon:

- **Ecosistema e Pricing** → è profondamente integrato con l'ecosistema AWS (es. S3, DynamoDB). Permette coreografie complesse tramite **Step Functions** (script che definiscono flussi di funzioni). Il *pricing* ha due componenti:
    1. **costo per invocazione** (con un *free tier* da 1 milione) 
    2. **costo per durata/risorse** (Giga-secondo)
    
    Include un **"free tier"**, una sorta di franchigia che si rinnova **ogni mese**: ad esempio, il primo milione di richieste e i primi 400 Giga-secondo di utilizzo sono gratuiti, e si paga solo ciò che sfora 
    
- **Deployment e Eventi *→*** il *deployment* avviene tramite un **"Deployment Package"** (un archivio `.zip` con codice e librerie, simile a un file `.war` Java) o, in alternativa, fornendo un'**immagine *container***. Gli eventi sono documenti JSON che possono essere *custom* o **Eventi di Sistema AWS** (es. "file disponibili in S3"). Usare gli eventi AWS è comodo ma aumenta il **vendor lock-in.**
- **Qualifier** → i *Qualifier* gestiscono il **versioning** delle funzioni, permettendo a più versioni della stessa funzione di coesistere. Questo sistema è utile per la **gestione dei sistemi legacy**, cioè applicazioni o componenti obsoleti ma ancora fondamentali per l’azienda. Quando un software evolve, si può creare una nuova versione della funzione: i **nuovi servizi** usano la versione aggiornata, mentre le **applicazioni legacy** continuano a invocare quella stabile, consentendo una **migrazione graduale e sicura**.

## 3.5. Google Functions

Sostanzialmente simile ad AWS Lambda:

- **Integrazione** → altamente integrato con l'ecosistema Google (Cloud Storage, Pub/Sub, Firestore).
- **Semantica "At Least Once"** → a differenza di altri sistemi, i *trigger* hanno una semantica **"*At Least Once*"**. Questo significa che un singolo evento potrebbe scatenare **più invocazioni** della stessa funzione.
    
    <aside>
    🚨
    
    **Criticità**
    
    Questo rende complesse operazioni banali come un **contatore**, a causa del rischio di *race condition* e duplicati . La soluzione è usare *append* su un database con ID univoci e compattare periodicamente i log. 
    
    </aside>
    
- **Eventi HTTP**: per le funzioni HTTP, Google Function utilizza oggetti *request* **simili a Flask**, rendendo lo sviluppo in Python molto comodo.
- **Pricing**: simile ad AWS, con costi per invocazione, utilizzo risorse (Giga-secondo) e traffico di rete *outbound.*

---