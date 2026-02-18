# 4. Data Management

Number: 3
Status: Done
Type: theory

# 1. Replicazione dei Dati e Scalabilità

## 1.1. Scale-Up vs Scale-Down

Per rendere un sistema scalabile si possono seguire due direzioni:

- **Scale-up**: aumentare la potenza di un singolo dispositivo, ma questo approccio ha limiti fisici.
- **Scale-out**: replicare il servizio su più server. Questo può avvenire a livello locale (più server nello stesso data center) o geografico. La replicazione geografica riduce la latenza e il carico sui punti di peering della rete (come i cavi transoceanici), migliorando le prestazioni per gli utenti.

Un'ulteriore distinzione riguarda l'entità della replicazione:

- **Replicazione totale**: tutte le risorse vengono replicate.
- **Replicazione parziale**: viene replicato solo un sottoinsieme di contenuti. Questo ha senso se si possono identificare dei pattern di accesso ai dati (es. una serie TV è popolare solo in una specifica regione).

![image.png](4%20Data%20Management/image.png)

## 1.2. Consistenza e Sincronizzazione dei Dati

La replicazione su più copie dei dati solleva il problema di mantenerle sincronizzate. La **consistency policy** definisce le regole per gestire questo aspetto:

- **Strong consistency**: tutti i dati sono sempre aggiornati su tutte le copie. Questo sistema non accetta perdite di dati o partizionamenti della rete.
- **Weak consistency**: si accetta che i dati possano essere temporaneamente non allineati, con la garanzia che convergeranno in un secondo momento. Questo riduce l'overhead di sincronizzazione, migliorando le prestazioni e la scalabilità.
- **No consistency**: Le copie dei dati divergono, ma è uno scenario puramente teorico e non funzionale.

**N.B.** Per gestire la consistenza, un sistema richiede un meccanismo di sincronizzazione e un sistema di controllo della concorrenza.

## 1.3. NoSQL e Consistenza Debole

Il concetto di **consistenza debole** è un punto cardine dei database non relazionali, noti come **NoSQL**. In questo contesto, il focus si sposta dal modello **ACID** (forte consistenza):

- **Atomicity** → una transazione è atomica: o viene eseguita completamente oppure non viene eseguita affatto.
- **Consistency** → il database passa sempre da uno stato consistente a un altro stato consistente.
- **Isolation** → ogni transazione è isolata dalle altre e si comporta come se fosse eseguita da sola.
- **Durability** → una volta completata, una transazione rimane permanentemente salvata nel database.

A **BASE**:

- **Basic Availability** → il sistema è sempre disponibile, anche in caso di fallimenti.
- **Soft-state** → i dati possono essere temporaneamente inconsistenti.
- **Eventual consistency** → si garantisce che a un certo punto futuro i dati si allineeranno. Questo approccio è ideale per la gestione di volumi di dati spaventosi, dove la **scalabilità** è più importante di una consistenza stretta e immediata

Oggi i database NoSQL sono la principale tecnologia per i sistemi cloud in cui la consistenza forte non e’ fondamentale ma le prestazioni si.

## 1.4. Tassonomia delle Strategie di Replicazione di Server e Dati

- **Distribuzione Locale** → questo approccio si concentra sulla gestione del carico di lavoro all'interno di una singola area geografica, tipicamente un data center.
    - **Web cluster:** una serie di server che ospitano le copie del servizio.
    - **Web Switch:** utilizzato per distribuire il carico tra i vari server del Web cluster. Esistono due tipi di Web Switch, a seconda del livello della rete su cui operano:
        - Web Switch L4: lavora a livello di trasporto, analizzando solo le connessioni TCP (indirizzo IP e porta). Non può vedere i contenuti delle richieste HTTP, specialmente se cifrate.
        - Web Switch L7: lavora a livello applicativo, permettendo di analizzare il traffico HTTP e prendere decisioni più sofisticate, come la gestione delle sessioni. Per le connessioni HTTPS, tuttavia, è necessario caricare i certificati sul Web Switch per poter ispezionare il traffico.
- **Distribuzione Globale** → questo approccio distribuisce il carico su server replicati in **diverse posizioni geografiche** per essere più vicini agli utenti finali. Si divide in due scenari principali:
    - **Web Cluster replicati in diverse aree (=replicazione dell’infrastruttura):** in questo scenario si replicano interi centri di calcolo. Il routing delle richieste avviene in due livelli:
        - DNS routing (globale) → il DNS indirizza l’utente verso il **cluster geografico più appropriato** (ad esempio quello più vicino).
        - Web switch / load balancer (locale) → all’interno del cluster scelto, un bilanciatore di carico distribuisce la richiesta tra i vari server del cluster.
    - **Sistemi basati su Cache e Proxy (=replicazione dei dati):** in questo scenario i server hanno ruoli differenti e non sono necessariamente cluster completi.Questi sistemi sfruttano la **replicazione tramite cache** per velocizzare la consegna dei contenuti:
        - Proxy server / cache: server posizionati "davanti" al server principale (Master) per ridurre il suo carico servendo contenuti già pronti.
        - Cooperative proxy servers (ISP): gruppi di server cache usati dagli Internet Service Provider per ridurre il traffico in uscita dalla propria rete.
        - CDN (Content Delivery Network): una rete globale di proxy server chiamati **Edge Server**, situati in punti chiamati **PoP (Point of Presence)**. Il sistema instrada l'utente tramite DNS verso l'edge server più vicino, consegnando il contenuto direttamente dalla cache o recuperandolo dal server originale se necessario.

![image.png](4%20Data%20Management/image%201.png)

---

# 2. Meccanismi di Sincronizzazione dei Dati

- **Two-Phase Commit** → utilizzato per garantire la **strong consistency** in database distribuiti. Si basa su due fasi:
    1. **Voting Phase**: un coordinatore invia un messaggio a tutti i nodi. I nodi aggiornano un log interno e rispondono "OK" solo quando sono pronti a eseguire la transazione.
    2. **Decision Phase**: il coordinatore invia il comando di "commit" a tutti i nodi. La transazione è considerata completata quando tutti i nodi confermano l'avvenuta scrittura. 
    
    Questo meccanismo può essere implementato a un livello, con un singolo coordinatore/master, oppure in modo **gerarchico**, dove ogni master ha nodi di backup che contribuiscono alla fase di commit. 
    
    Questo approccio è costoso e il rischio di fallimento aumenta con il numero di repliche. Generalmente è sconsigliato con più di 10 copie. La probabilità di fallimento di una transazione su $N$ copie è data da $(1−P)^N$, dove $P$ è la probabilità di successo di una singola transazione.
    

![image.png](4%20Data%20Management/image%202.png)

- **Gossip Protocols** → sono utilizzati principalmente per la **weak consistency**. In un sistema P2P, i nodi comunicano solo con i loro vicini, "spargendo" le informazioni come nel passaparola. Un esempio è **Apache Cassandra**.
    
    <aside>
    📌
    
    **Cassandra**
    
    Cassandra è un **DBMS distribuito** (originariamente sviluppato in Facebook, poi open-source Apache) che utilizza una **Distributed Hash Table (DHT)** per mappare in modo uniforme le chiavi su tutti i nodi di un cluster.
    
    In una DHT, ogni dato (ad esempio un record di database o un file) è identificato da una **chiave univoca**. Questa chiave viene passata a una **funzione di hash**, che produce un **hash ID**. Lo spazio di tutti i possibili hash ID viene **partizionato** e **distribuito uniformemente tra i nodi della rete**. Ogni nodo è responsabile di un **intervallo contiguo di hash ID**: se l’hash di un dato cade in quell’intervallo, il dato viene memorizzato su quel nodo.
    
    *Esempio:*
    
    Immagina che l'intervallo totale di hash sia da 0 a 1000:
    
    - Il Nodo A può essere responsabile degli hash da 0 a 100
    - Il Nodo B può essere responsabile degli hash da 101 a 200
    
    ![image.png](4%20Data%20Management/2e922ac2-d89c-4aa9-a0b4-994254ae253f.png)
    
    Per aumentare **affidabilità** e **disponibilità**, ogni porzione di dati viene **replicata su N nodi**; una scrittura è considerata riuscita quando un *quorum* di nodi (tipicamente  $\frac{N}{2} + 1$) ha confermato l’aggiornamento.
    
    Ogni nodo mantiene inoltre una **piccola tabella di routing** (chiamata *finger table* in Chord), che contiene solo un insieme limitato di nodi vicini, evitando così overhead di memoria e comunicazione.
    
    Quando un nodo deve raggiungere un nodo con un certo ID:
    
    1. cerca nella propria tabella il vicino con ID più “vicino” all’ID target,
    2. inoltra il messaggio a quel nodo,
    3. il processo continua **hop-by-hop** fino a raggiungere il nodo corretto.
    </aside>
    

- **Primary e Secondary Copies** → un altro approccio è la combinazione di consistenza forte e debole. Un **master** (copia primaria) riceve le richieste di scrittura e garantisce la consistenza su tutte le altre copie primarie. Le scritture avvengono su un'unica copia principale che poi propaga le modifiche. Le copie **secondarie**, invece, agiscono come delle cache e sono sincronizzate solo periodicamente, con una consistenza debole. Questo permette di indirizzare le letture verso i server secondari più vicini, riducendo la latenza. Un esempio noto è il funzionamento del DNS.
    
    Il processo con cui il master replica i dati sulle copie secondarie prende il nome di **disk mirroring**. Sia il master sia le copie secondarie mantengono, oltre ai dati, una struttura chiamata **remote bitmap volume**, che consente di individuare rapidamente i blocchi non sincronizzati. In questo modo è possibile copiare solo i blocchi realmente modificati, riducendo il traffico e il tempo di sincronizzazione.
    

![JPEG image-487F-A888-8A-0.jpeg](4%20Data%20Management/JPEG_image-487F-A888-8A-0.jpeg)

---

# 3. Caching

La replicazione geografica, fondamentale per la scalabilità, sfrutta il **caching** per rendere i dati più accessibili e ridurre la latenza. I principi del caching si basano sulla **località spaziale** (utenti vicini accedono a dati simili) e sulla **località temporale** (un utente riaccede agli stessi dati in un breve periodo).

l popolamento della cache può avvenire in due modi:

- **Pull caching** → le richieste degli utenti determinano quali contenuti vengono salvati nella cache. Un esempio è il **proxy server**, che recupera un contenuto mancante dall'origin server e ne mantiene una copia locale.
- **Push caching** → la replicazione è guidata dal server primario, che spinge i contenuti verso le cache secondarie (es. contenuti dinamici o molto richiesti). Un esempio è il **reverse proxy**, che si posiziona vicino all'origine server ed è popolato con una logica push.

**N.B.** L'efficacia del caching è misurata da **cache hit** (risorsa trovata) e **cache miss** (risorsa non trovata) e dal conseguente $\text{hit rate}=\frac{\text{cache hit}}{\text{numero di richieste}}$

## 3.1. Caching tramite Proxying Server

Un proxy server agisce come intermediario e può essere di due tipi principali, a seconda della sua posizione e del modello di gestione della cache:

- **Forward Proxy** → ****si trova sul lato client e la sua cache viene popolata con un modello **pull**, ovvero sono le richieste degli utenti a determinarne il contenuto. Un limite di questo approccio è che non vi sono metadati espliciti per la scadenza dei contenuti, costringendo il sistema a inferirla da metadati come la data di creazione o ultima modifica.
- **Reverse Proxy** → si trova vicino al server di origine e la sua cache viene popolata con un modello **push**, in cui è il server primario a decidere quali contenuti replicare. Questo approccio elimina i problemi di sincronizzazione, poiché il primario sa sempre quali dati sono stati aggiornati e dove si trovano le copie più vecchie. I reverse proxy sono in grado di gestire in modo efficiente anche contenuti dinamici e personalizzati. Possono essere replicati su più **Autonomous Systems** per posizionare la cache più vicino agli utenti.
    
    Uno dei primi contesti applicativi di questa tecnologia è stato il delivery di **contenuti multimediali**. I video, essendo file di grandi dimensioni ma con un ciclo di vita non troppo aggressivo, sono particolarmente adatti a questo modello. Invece di servire l'intero file da un singolo server, il video viene diviso in una serie di segmenti. Inoltre, si creano più versioni dello stesso video (es. 4K, 1080p, 720p) e ogni blocco viene inviato alla risoluzione più adatta in base allo stato della rete del client. Questo sistema di replicazione è estremamente **agile**, perché permette di dare priorità a segmenti o risoluzioni più richiesti, come ad esempio la parte iniziale di un film, senza dover replicare l'intero contenuto in ogni cache.
    

![image.png](4%20Data%20Management/0d1de184-7dc3-4fae-95ab-c875b9129ec6.png)

## 3.2. Cooperative Proxy Server (ISP)

L'idea del caching cooperativo è che più proxy server possono collaborare per condividere i contenuti, riducendo la latenza e aumentando l'hit rate. Le politiche di cooperazione si dividono in:

- **Gerarchico** → i proxy sono organizzati in livelli. In caso di miss locale, la richiesta viene passata a un nodo del livello superiore, che ha una popolazione di utenti più ampia e quindi un maggior numero di dati in cache. Sebbene aumenti la latenza, questo schema è molto efficace per i cosiddetti *compulsory miss* (prime richieste di un contenuto).

![image.png](4%20Data%20Management/14b3f56b-ac5d-45e9-9f5f-9e6286ba48bd.png)

- **Flat** → i nodi si trovano allo stesso livello e collaborano tra loro. Questo approccio è più veloce in termini di latenza, ma richiede meccanismi più sofisticati per lo scambio di informazioni:
    - **Query-based:** in caso di miss, un proxy interroga i vicini. Il miss globale viene dichiarato solo dopo aver ricevuto una risposta negativa da tutti i vicini.
    - **Informed-based**: i proxy si scambiano periodicamente informazioni sulla propria cache. Per evitare lo scambio di indici di cache che sono molto grandi, si utilizzano rappresentazioni compatte come i **Bloom Filters**.
        
        <aside>
        🔑
        
        **Bloom FIlter**
        
        Un Bloom Filter è una struttura dati compatta lossy che permette di verificare rapidamente e con un'alta probabilità se un elemento è presente in un insieme.
        
        - **Creazione** → si parte con un vettore di bit di lunghezza $W$ composto da soli zeri. Per inserire un elemento (es. una chiave `x`), si applicano più funzioni di hash, ognuna delle quali restituisce un valore. I bit corrispondenti a questi valori nel vettore vengono impostati a `1`.
        - **Verifica** → per controllare se un elemento `w` è presente, si applicano le stesse funzioni di hash. Se tutti i bit corrispondenti sono `1`, l'elemento è probabilmente presente; se anche un solo bit è `0`, l'elemento non è presente.
        
        ![image.png](4%20Data%20Management/40c2790c-8a93-4471-988a-a066c8169d2a.png)
        
        Il Bloom Filter offre una rappresentazione estremamente leggera e compatta dell'indice, riducendo l'overhead di comunicazione. Tuttavia, è un meccanismo probabilistico e può incorrere in **falsi positivi**. Non permette la rimozione di elementi, pertanto deve essere ricostruito periodicamente da zero.
        
        </aside>
        
    - **Partitioning**: questo approccio si basa su una funzione di hash che associa in modo deterministico un contenuto a un server specifico. In questo modo, un proxy sa esattamente a quale nodo chiedere la risorsa, senza dover interrogare tutti i vicini.
- **Hybrid**

![image.png](4%20Data%20Management/2c029be6-8293-429c-83fb-1eeb88ae6542.png)

## 3.3. Modelli di realizzazione di Sistemi Cache

La scelta del modello di implementazione per un sistema di cache varia in base alle dimensioni dell'azienda e alla sua expertise interna. Esistono diversi scenari, che vanno dalla gestione interna completa fino all'affidamento totale a fornitori esterni.

- **Content provider only** → il content provider gestisce l'intero sistema, inclusa tutta l'infrastruttura necessaria. Questo modello è molto costoso.
    
    ![image.png](4%20Data%20Management/3270185a-7d6f-437c-b1ab-bd4aa7cca6cd.png)
    
- **Content provider + cache su secondary server** → l'infrastruttura principale è gestita internamente, ma la cache dei client viene affidata a un'organizzazione esterna specializzata (terza parte).
    
    ![image.png](4%20Data%20Management/42d14357-331a-4136-b30c-a56cfe4d6d60.png)
    
- **Content provider + only master** → in questo scenario, il content provider gestisce solo i server principali, mentre la replicazione sulle copie primarie è gestita da un'organizzazione esterna.
    
    ![image.png](4%20Data%20Management/37b7ebe2-824b-43b3-b158-e90dc7b4ba06.png)
    
- **CP + Third + Fourth party** → questo modello è una variante del precedente, in cui la gestione dei server primari e quella della cache sono affidate a più terze parti esterne.
    
    ![image.png](4%20Data%20Management/4169a928-75e5-4c78-ac52-3b961d8eaee8.png)
    
- **CP + CDN** → questo modello rappresenta la soluzione più moderna in cui il content provider si occupa unicamente della creazione del contenuto, affidando interamente il servizio di delivery a una **Content Delivery Network (CDN)**.
    
    ![image.png](4%20Data%20Management/image%203.png)
    

---

# 4. Content Delivery Networks (CDN)

Una **CDN** è un sistema di caching cooperativo e distribuito geograficamente che replica contenuti selezionati (quelli più rilevanti e richiesti) per renderli disponibili il più vicino possibile all'utente. Il suo scopo è garantire un servizio ad alta performance e affidabilità, soprattutto per contenuti statici di grandi dimensioni come i video. Agiscono su un paradigma di **push caching**, in cui il provider spinge i contenuti verso i server edge per renderli disponibili.

## 4.1. Architettura del CDN

Una CDN è organizzata su due livelli principali:

- **Core Servers** → sono i server primari che gestiscono la replicazione dei contenuti e la logica di management della rete. Non servono direttamente le richieste degli utenti.
- **Edge Servers** → sono i server secondari distribuiti geograficamente, che si trovano il più vicino possibile agli utenti e rispondono direttamente alle loro richieste.

Per instradare il traffico al server più vicino e veloce, la CDN utilizza meccanismi di routing sofisticati che tengono conto di molti parametri. I **Point of Presence (POP)** sono cluster di edge servers distribuiti in vari **Autonomous Systems (AS)** per ottimizzare la vicinanza agli utenti finali.

![image.png](4%20Data%20Management/263ecdee-e77f-41fd-859e-24d38510fef1.png)

## 4.2. Meccanismi di Funzionamento

Le CDN utilizzano principalmente due meccanismi per la gestione dei contenuti e la riduzione della latenza:

- **URL Rewriting** → l'URL originale di un contenuto (es. un'immagine) viene riscritto per puntare a un server della CDN. Questo permette di **alleggerire il carico** di richieste di contenuti statici dall'origin server.
    
    ![image.png](4%20Data%20Management/78cfd76f-d879-474f-a50d-0cf64dff6d78.png)
    
- **DNS Outsourcing** → la CDN gestisce i server DNS di un dominio. Utilizzando algoritmi "intelligenti", il DNS della CDN risolve le query indirizzando il client all'edge server più adatto, evitando completamente l’origin server, in base alla sua posizione geografica e alle condizioni della rete.
    
    ![image.png](4%20Data%20Management/e67c607d-d473-4eb3-813c-5827b7dff47e.png)
    

<aside>
🚨

**URL Rewriting vs DNS Outsourcing**

L'URL Rewriting reindirizza le richieste per i contenuti embedded agendo direttamente sui link all'interno della pagina web, mentre il DNS Outsourcing reindirizza tutto il traffico di un dominio a livello di risoluzione DNS, prima ancora che venga richiesta la pagina principale.

Quindi questi due meccanismi, spesso usati in combinazione (come nell'esempio di Akamai), permettono di **cacheare a due livelli**: a livello di risorsa e a livello di DNS.

</aside>

### 4.2.1. Akamai

Akamai utilizza un mix di **DNS outsourcing** e **URL rewriting** per reindirizzare il traffico. Quando un client clicca su un URL, la richiesta DNS viene gestita dai server DNS della CDN, che sono stati delegati dal provider di contenuti. Usando algoritmi intelligenti, il DNS della CDN risolve la query con l'IP dell'**edge server** più adatto al client, in base a posizione geografica e condizioni di rete.

Questo meccanismo di caching è a **due livelli**:

- **Caching a livello DNS**: la risoluzione dei nomi può essere memorizzata nella cache del client per un certo periodo (TTL breve), velocizzando le richieste successive.
- **Caching a livello di risorsa**: l'edge server mantiene una copia dei contenuti più richiesti, servendo le richieste direttamente dalla sua cache.

<aside>

**Flusso operativo:**

![image.png](4%20Data%20Management/5b851a96-bfe4-47e5-b502-d35ec784214e.png)

1. **Richiesta della pagina**: il client richiede la pagina principale (`index.html`) al server di origine (`cnn.com`). Questa pagina contiene i link degli oggetti embedded (immagini, ecc.) che sono stati riscritti per puntare a un dominio della CDN (es. `image.cache.cnn.com`).
2. **Risoluzione DNS**: il browser del client, vedendo i link riscritti, avvia una complessa risoluzione DNS. La query attraversa una gerarchia di server DNS, che utilizzano CNAME e record alias per reindirizzare la richiesta verso il server DNS regionale più opportuno di Akamai.
3. **Indirizzamento all'Edge Server**: il DNS regionale di Akamai utilizza la sua "intelligenza" e metriche in tempo reale (come ping e traceroute) per restituire l'indirizzo IP del server edge più adatto.
4. **Cache Miss e Popolamento**: se l'edge server non ha il contenuto richiesto nella sua cache (**cache miss**), lo recupera dal server di origine, lo mette in cache e lo consegna al client.
</aside>

## 4.3. Load Balancing e Ottimizzazione

Per distribuire il carico tra gli edge server, si usano logiche di **load balancing**. L'algoritmo *least-loaded* non è una buona idea perché può causare problemi di instabilità e oscillazioni, sovraccaricando rapidamente il server con meno traffico. Un'alternativa migliore è l'algoritmo **k-least-loaded**, che distribuisce le richieste in modo più casuale su un sottoinsieme di server non sovraccarichi.

Le CDN utilizzano anche **overlay network** per trovare percorsi alternativi in caso di problemi di connettività nella rete "reale", gestendo la ridondanza in modo trasparente all'utente.

## 4.4. Performance

L'utilizzo di una CDN offre notevoli benefici in termini di performance. La latenza e la congestione della rete sono ridotte, e il tempo di risposta diminuisce drasticamente. La CDN, inoltre, riduce la **varianza** del tempo di risposta, rendendo le performance più stabili e prevedibili:

![image.png](4%20Data%20Management/image%204.png)

Tuttavia, la risoluzione dei nomi tramite i server DNS della CDN è generalmente un po' più lenta rispetto a quella tradizionale:

![image.png](4%20Data%20Management/image%205.png)

## 4.5. Gestione dei Guasti (Failures)

La resilienza in una CDN è gestita su più livelli:

- **Guasto del singolo server**: se un server ha un problema di storage o crolla, un altro server subentra rapidamente per prenderne il posto. La mappa del sistema a basso livello viene aggiornata velocemente.
- **Guasti maggiori**: in caso di un intero cluster offline o di un percorso di rete fallito verso l'origin server, la CDN può aggiornare la mappa di routing ad alto livello per indirizzare il traffico verso un cluster funzionante o reindirizzare i pacchetti attraverso un nodo intermedio. I protocolli di routing come BGP sono fondamentali per gestire rapidamente questi "switch".

---