# 8. Storage Virtualizzato

Number: 3
Status: Done
Type: theory

# 1. Introduzione

Nei moderni *data center* cloud, la tendenza è passare da una gestione puramente hardware a un approccio **Software Defined**. Lo **Storage Virtualizzato** applica alla memoria di massa lo stesso principio della virtualizzazione del *computing*: creare un’**astrazione** che nasconda la gestione fisica dei dischi, offrendo una visione logica e semplificata.

Questo modello supera la logica tradizionale di Windows (basata su unità fisiche come `C:` o `D:`) e si avvicina a quella di Unix/Linux, dove l’utente vede un unico albero di *directory* (`/var`, `/home`) indipendente dal disco o server su cui risiedono. Lo storage virtualizzato estende questo concetto **alla scala del Cloud**, dove i dati possono essere distribuiti su dischi, *data center* o persino continenti diversi.

<aside>
✅

**Vantaggi:**

L'astrazione dei dati offre vantaggi fondamentali per la scala cloud:

- **Aumentare la Capacità** → i singoli dischi hanno capacità limitate (es. 20-30 TB), mentre un *data center* gestisce Exabyte. Lo storage virtualizzato consente di **aggregare molti dischi** e trattarli come un’unica risorsa.
- **Aumentare il Throughput** → un disco singolo ha tempi d’accesso troppo elevati per dati di grandi dimensioni. L’astrazione permette di **parallelizzare** letture e scritture su centinaia di dispositivi, aumentando la banda totale.
- **Implementare la Ridondanza** → se un disco si guasta, i dati restano accessibili. È il principio del **RAID** (*Redundant Array of Inexpensive Disks*), ma applicato su scala *cloud*.
- **Gestione Differenziata dei Dati** → i dati non sono tutti uguali; nel *Big Data*, i dati invecchiano e hanno valori diversi. Si possono mettere i dati "caldi" (es. produzione dell'ultimo mese) su dischi veloci (SSD) e i dati "freddi" (es. dati del 2014) su supporti lenti (nastri magnetici), ottimizzando i costi.
</aside>

---

# 2. Classificazione Virtualizzazione Storage

La virtualizzazione dello storage si può classificare in base a due aspetti principali:

1. **Granularità** → la virtualizzazione può avvenire:
    - **Sopra il file-system (file-based),** dove l’astrazione opera a livello di file e directory.
    - **Sotto il file-system (block-based)**, dove vengono gestiti direttamente i blocchi fisici dei dischi.

![image.png](8%20Storage%20Virtualizzato/image.png)

1. **Scope (Prospettiva)** → indica l’estensione della virtualizzazione: su un singolo host o un intero *data center.*

---

# 3. Livelli di Granularità della Virtualizzazione

## 3.1. Virtualizzazione Block-Based (Sotto il File System)

Nella virtualizzazione **block-based**, più dischi fisici vengono aggregati e presentati al sistema operativo come un **unico disco logico**. L’accesso avviene a livello di blocchi tramite protocolli come **SCSI (Small Computer System Interface)**, che oggi possono viaggiare anche su rete e non più solo su bus fisici.

Il sistema operativo non vede i dischi reali, ma una o più unità logiche identificate da un **Logical Unit Number (LUN)**. I dati vengono indirizzati tramite un **Logical Block Address (LBA)**, cioè l’indice del blocco logico da leggere o scrivere. La traduzione tra indirizzi logici (LUN, LBA) e posizioni fisiche sui dischi avviene tramite **metadati**, che mantengono le tabelle di mapping.

<aside>

Esempio:

Immagina che tu abbia 3 dischi fisici da 1 TB ciascuno. l *Logical Volume Manager* li unisce in un unico volume logico da 3 TB. Quando il sistema deve leggere il **blocco logico 1000** (LBA 1000), il LVM controlla nei **metadati** una tabella di mapping, tipo:

| **LUN** | **LBA (blocco logico)** | **Disco fisico** | **Settore fisico** |
| --- | --- | --- | --- |
| 01 | 1000 | Disco 2 | Settore 54829 |
| 01 | 1001 | Disco 3 | Settore 10293 |
| 01 | 1002 | Disco 1 | Settore 49384 |

Quindi, il sistema operativo pensa di leggere “un blocco” del volume logico, ma in realtà il LVM lo **traduce in una posizione fisica reale** su un disco (o anche su più dischi).

</aside>

I metadati sono critici perché contengono la mappatura logico-fisica e gestiscono la **ridondanza**. Il mapping non è 1:1: un blocco logico può corrispondere a più blocchi fisici. Per una ridondanza efficace, le copie devono trovarsi su **unità indipendenti** (dischi, nodi o linee di alimentazione diversi), così da evitare *failure comuni*.

Poiché la perdita dei metadati renderebbe inaccessibili tutti i dati (anche se ridondati), **anch’essi devono essere replicati**. Inoltre, i metadati consentono di implementare il **load balancing**, distribuendo le letture sui dischi meno occupati.

### 3.1.1. Logica Volume Manager (LVM)

Il **Logical Volume Manager (LVM)** è una concreta implementazione di virtualizzazione block-based. LVM introduce un livello di astrazione che non lavora direttamente sui singoli blocchi, ma su unità più grandi chiamate **extent**. I concetti fondamentali sono:

- **Physical Volume (PV)** → il disco fisico reale (es. `/dev/sda`).
- **Physical Extent (PE)** → la unità fisica allocabile all’interno di un PV.
- **Logical Volume (LV)** → il disco virtuale visto dal sistema operativo (es. `/dev/lv1`).
- **Logical Extent (LE)** → l’unità logica che compone un LV e che viene mappata **1:1 su un PE**.

<aside>
📌

**Flusso di accesso ai dati in LVM:**

1. **Richiesta Logica (LUN e LBA)** → il sistema operativo vede il volume come un'unica unità logica identificata da un **LUN** (Logical Unit Number). Quando deve leggere un dato, il SO richiede uno specifico blocco utilizzando il suo indirizzo logico, ovvero l'**LBA** (Logical Block Address).
2. **Individuazione del Logical Extent (LE)** → a questo punto interviene LVM. Invece di mappare ogni singolo blocco (che sarebbe inefficiente), LVM ragiona per unità più grandi chiamate **Logical Extents (LE)**. Il sistema calcola matematicamente in quale LE "cade" l'LBA richiesto.
3. **Mapping tramite Metadati (LE → PE)** → LVM consulta i suoi **metadati**, che fungono da tabella di mapping. Qui cerca la corrispondenza tra l'unità logica (**LE**) e l'unità fisica (**PE - Physical Extent**).
4. **Accesso Fisico al PV** → infine, l'operazione viene indirizzata al **Physical Volume (PV)**, ovvero il disco rigido reale. LVM invia il comando al controller del disco affinché legga il settore fisico corrispondente a quel determinato **PE**.

![image.png](8%20Storage%20Virtualizzato/image%201.png)

</aside>

### 3.1.2. Accesso ai Metadati

I metadati possono essere gestiti in due modi:

1. **Accesso Trasparente** → l'utente accede al *Logical Extent* e il *Software Defined Storage (SDS)* si fa carico di eseguire il *lookup* sui metadati e la mappatura. È semplice per l’utente, ma gravoso per l’SDS.
2. **Accesso API (Non Trasparente)** → l’SDS espone un’API che permette al *file system* di interrogare direttamente i metadati. È più scalabile, poiché parte del lavoro di traduzione viene delegato al *client*.

### 3.1.3. Vantaggi

Nonostante la complessità, la virtualizzazione a blocchi offre vantaggi fondamentali per il *cloud*:

- **Snapshot** → consente di “congelare” l’immagine di un *file system* (es. una VM) in un preciso istante, agendo **solo sui metadati** (la "mappa" LE/PE) senza copiare i dati. Il volume originale e lo *snapshot* condividono gli stessi *Physical Extent (PE)* finché non avviene una modifica. Quando il *file system* tenta di **scrivere** su un blocco condiviso, interviene il meccanismo di **Copy-on-Write (CoW)**: la scrittura viene dirottata su un **nuovo PE**, mentre il blocco originale viene preservato per lo *snapshot*.
    
    Questo processo è veloce (opera solo sui metadati) ed efficiente (copia solo i blocchi modificati), rendendolo fondamentale per gestire più VM che condividono la stessa immagine di base
    
- **Deduplicazione dei Dati** → quando più dati o VM contengono copie identiche, la deduplicazione riduce lo spazio mappando i blocchi uguali sullo stesso *Physical Extent*, memorizzando solo le differenze.

## 3.2. Virtualizzazione File-Based (Sopra il File System)

In questo approccio, la virtualizzazione avviene **sopra il file system**. Il *Software Defined Storage (SDS)* esporta file e directory tramite un **file system di rete**, permettendo l’accesso remoto ai dati come se fossero locali.

### ***3.2.1. NFS (Network File System)***

- Sviluppato da **Sun Microsystems**, si basa su **RPC** (*Remote Procedure Call*).
- **NFS v2:** molto veloce, usa **UDP** ed è **stateless** (non mantiene informazioni sullo stato delle connessioni). Se il server si riavvia, non deve ripristinare alcuna sessione.
- **NFS v3:** introduce **TCP** e il supporto a file di grandi dimensioni (32 bit → 64 bit, quindi file > 4gb)
- **NFS v4:** aggiunge funzionalità **stateful**, come il *file locking* e una gestione più sicura delle connessioni.

### ***3.2.2. SMB/CIFS (Server Message Block)***

Sviluppato da **Microsoft**, è più di un semplice *file system*: gestisce anche **stampanti, porte e servizi di rete**. È **stateful** (richiede autenticazione e mantiene lo stato delle sessioni) e utilizza **TCP**. Dalla versione **3.0**, supporta:

- **Remote DMA (RDMA)** → trasferimento asincrono dei dati direttamente nei buffer di memoria, notificando il sistema operativo solo al completamento.
- **Load balancing** e **crittografia** dei dati.

Ha tuttavia un **overhead maggiore** rispetto a NFS, dovuto alle sue funzionalità aggiuntive.

## 3.3. **Approccio nel Cloud**

Nel *cloud computing*, la scelta del tipo di virtualizzazione dipende dal caso d’uso:

- **Block-Based** → ideale per gestire **immagini di VM** grazie al supporto di *snapshot* e *copy-on-write*.
- **File-Based** → usato per lo **storage condiviso tradizionale** (es. home directory, file aziendali).
- **Object Storage** → oggi molto diffuso, combina concetti di *file system* e *database* (es. **Amazon S3**, **Firebase**). I dati vengono gestiti come **oggetti** in *bucket* o *tuple JSON*, con accesso tramite API REST. Un esempio *open source* completo è **Ceph**, che opera su tutti e tre i livelli: **oggetti**, **blocchi** e **file**.

---

# 4. Scope Virtualization

La virtualizzazione dello *storage* può essere realizzata su diverse infrastrutture:

- **DAS (Direct Attached Storage)** → unità direttamente collegata a un *server* (es. disco USB, Thunderbolt). Non utilizza protocolli di rete e risulta visibile solo all’host locale. Per condividere i dati con altri nodi, l’host deve eseguire un *file system server* di rete (es. SMB).
- **NAS (Network Attached Storage)** → dispositivo *embedded* accessibile tramite rete. I componenti principali sono:
    - **Storage:** spesso ibrido, con dischi rotativi (HDD) per alta capacità e dischi **SSD** usati come *cache* per i dati recenti.
        
        <aside>
        📌
        
        **Usura dei Dischi**
        
        La scelta tra HDD (rotativi) e SSD (stato solido) influisce sull’affidabilità. Le metriche chiave sono:
        
        - **UBER (Unrecoverable Bit Error Ratio):** probabilità che la lettura di un bit termini in un errore non recuperabile.
        - **TBW (Terabyte Written):** quantità di dati scrivibili prima del termine della vita utile (Endurance Rating).
        
        I due tipi di dischi hanno modelli di fallimento diversi:
        
        - HDD → l’usura è graduale (il tasso di errori aumenta linearmente) e le *failure* sono spesso indipendenti.
        - SSD → funzionano perfettamente fino a fine vita, poi si guastano improvvisamente in quanto i meccanismi di bilanciamento delle scritture (*wear leveling*) distribuiscono l’usura in modo uniforme, portando le celle a rompersi simultaneamente: le *failure* risultano quindi **non indipendenti**.
        
        ![image.png](8%20Storage%20Virtualizzato/image%202.png)
        
        </aside>
        
    - **Controller:** gestisce i dischi e implementa una logica di ridondanza (come il **RAID 5**, che protegge dalla rottura di un singolo disco, ma non di due dischi contemporaneamente). Ci sono 2 tipi di raid:
        - RAID Hardware: utilizza *controller* proprietari ad alte prestazioni. Questi sono estremamente veloci, ma presentano un rischio: se il *controller* si rompe, è necessario trovare un ricambio identico, altrimenti i dati sui dischi (anche se integri) potrebbero diventare inaccessibili.
        - RAID Software: Utilizzato nei NAS di fascia più bassa (es. Synology, QNAP). Spesso si tratta di una macchina con processore Intel/ARM che esegue una distribuzione Linux e usa il RAID software del kernel. Il vantaggio è che, se il NAS si rompe, i dischi possono essere estratti, montati su un altro sistema Linux e i dati recuperati.
        
        <aside>
        📌
        
        **Miglioramento delle Prestazioni (RAID)**
        
        L'aggregazione di più dischi, come in una configurazione RAID, migliora drasticamente le prestazioni. Ad esempio, dividendo un blocco I/O su **due dischi in parallelo**, il *service time* viene dimezzato (es. da 40ms a 20ms) e il **throughput raddoppia** .
        
        Tuttavia, il miglioramento nel **tempo di risposta** non è lineare, ma **più che dimezzato** . Come mostra il grafico, un sistema a 2 dischi (curva 20ms) può gestire un *throughput* molto più alto prima che il tempo di risposta cresca esponenzialmente. A parità di utilizzo (U=0.6), il sistema a 2 dischi ha un tempo di risposta significativamente inferiore.
        
        ![image.png](8%20Storage%20Virtualizzato/image%203.png)
        
        </aside>
        
    - **Interfacce di Rete:** spesso multiple, per abilitare bilanciamento del carico e ridondanza.
    
    <aside>
    🚨
    
    **Accesso e Limiti del NAS**
    
    Un NAS offre tipicamente un accesso multi-protocollo, sia a **basso livello** (orientato ai blocchi, tramite **iSCSI**) sia ad alto livello (esportando un **file system di rete** come NFS o SMB).
    
    Il NAS funziona bene, ma incontra un limite quando si deve scalare per scambiare Terabyte di dati (ad es. ospitando centinaia di VM): la rete (anche se con interfacce multiple in *bridge*) diventa un **collo di bottiglia**.
    
    </aside>
    
- **SAN (Storage Area Network)** → utilizzati per scenari su larga scale (fino a **Esabyte).** In questo scenario, è fondamentale che la rete non sia un collo di bottiglia; per questo, si crea una **rete separata e dedicata** (*fabric*) per lo storage.
    
    <aside>
    🔑
    
    **Architettura:**
    
    Una SAN è tipicamente strutturata su tre livelli:
    
    1. **Storage Layer** → è il livello fisico dei dischi. Costituito da *array* di dischi, spesso organizzati in configurazioni RAID (per ridondanza) o **JBOD** (*Just a Bunch of Disks*, simile a RAID 0, solo per aggregare capacità). L'accesso a questo livello è puramente *block-based*.
    2. **Fabric Layer (La Rete)** → è la parte di rete ad alta capacità e bassa latenza che interconnette *Host Layer* e *Storage Layer*.
        - È una **rete separata** dal traffico dati normale per evitare interferenze (mancata *performance isolation*).
        - Utilizza tecnologie ad alte prestazioni come la **Fibra Ottica** (FCP) e *switch* dedicati.
        - Implementa il **Multi-path**, ovvero percorsi multipli tra ogni *host* e ogni *storage*, per massimizzare il parallelismo e la *fault tolerance*.
    3. **Host Layer (Il Frontend)** → sono i *server* che forniscono la connettività e l'astrazione dello storage ai *client*. Espongono le API per l'accesso ai dati. L'astrazione del *file system* viene implementata qui, non a livello di storage.
    </aside>
    
    <aside>
    📌
    
    **Modalità di Accesso**
    
    L'*Host Layer* di una SAN moderna espone i dati in modi diversi, a seconda delle necessità:
    
    - **Interfaccia Block-Based** → espone i dischi logici (LUN) tramite protocolli come iSCSI o FCP.
    - **Interfaccia File-Based** → espone *file system* di rete (NFS, SMB).
    - **Interfaccia Object-Oriented** → espone API per l'accesso a oggetti (simile ad Amazon S3 o Ceph).
    </aside>
    
- **SDS (Software Defined Storage)** → il concetto di SAN su larga scala porta al **Software Defined Storage (SDS)** (termine introdotto nel 2014), che si focalizza sulla creazione di un'infrastruttura di storage virtualizzata, flessibile e scalabile (fino agli Esabyte). Un SDS deve indirizzare diverse sfide:
    
    
    - **Integrazione Legacy** → non si può immaginare di costruire un SDS da zero. Deve essere in grado di **integrare** tecnologie e *storage* preesistenti (NAS, SAN proprietarie).
    - **Interoperabilità (No Vendor Lock-in)** → deve garantire l'**indipendenza dall'hardware**. Non può dipendere da schede RAID proprietarie o *vendor* specifici.
    - **Scalabilità (Scale Out)** → deve poter scalare da decine di Terabyte a Esabyte.
    - **Gestione (Management)** → deve supportare policy di accesso complesse e automatizzare i *task* di **provisioning e deprovisioning** (es. gestione automatica degli account utente, per evitare che un dipendente licenziato mantenga l'accesso).
    - **Interfacce Semplici** → deve fornire interfacce semplici agli utenti (che non devono conoscere i dettagli implementativi) pur gestendo una grande complessità sottostante.
    - **Tiering** → deve gestire **pool di risorse** con prestazioni e costi diversi (es. "Gold", "Silver", "Bronze"), spostando i dati in base al valore o alla frequenza di accesso (come in S3).
    
    ![image.png](8%20Storage%20Virtualizzato/72a8b564-af73-42ad-9765-d0eac5ef74e6.png)
    

---

# 5. Esempi di Piattaforme SDS

## 5.1. Ceph

**Ceph** è un SDS *open source *****progettato per essere **altamente scalabile**, nato da una tesi di dottorato per gestire i dati degli esperimenti del **CERN** (nell’ordine dei *petabyte*).

### 5.1.1. Architettura

Il cuore del sistema è **RADOS (Reliable Autonomic Distributed Object Store)**, il **livello di object storage** interno al sistema SDS che gestisce fisicamente i **data pool** (insiemi di dischi).

<aside>
📌

**Algoritmo CRUSH**

RADOS non utilizza tabelle di metadati (che rappresenterebbero un *bottleneck*), ma l’algoritmo **CRUSH (Controlled, Scalable, Decentralized Placement).** CRUSH calcola dinamicamente, tramite **hashing**, dove memorizzare o leggere ogni oggetto. 

Questo approccio è **altamente scalabile e decentralizzato**, supporta l’assegnazione di **pesi (weights)** per bilanciare carichi su nodi con capacità diverse e garantisce che le repliche dei dati **non risiedano sullo stesso disco o nodo**, evitando guasti comuni (*failure domains*). Inoltre, il meccanismo di hashing gestisce in modo coerente la **distribuzione e le dipendenze dei dati**, anche in caso di aggiunta o rimozione di nodi.

</aside>

Sopra RADOS, Ceph fornisce tre livelli di accesso ai dati:

- **Object Storage (RADOSGW):** il *RADOS Gateway* espone API come nel caso di **S3**.
- **Block Storage (RBD):** il *RADOS Block Device* offre uno **storage a blocchi distribuito**, ideale per dischi virtuali e macchine virtuali.

- **File System (Ceph FS):** *file system* di rete a tutti gli effetti, pienamente **POSIX-compliant**, quindi compatibile con i sistemi operativi tradizionali. Utilizza l’interfaccia **FUSE (Filesystem in Userspace)** del kernel Linux.
    
    <aside>
    🔑
    
    **FUSE**
    
    Il *client* Ceph FS (Ceph FS) utilizza **FUSE** per montare il *file system.*
    
    - **Problema** → i *file system* tradizionali (ext4, NTFS) girano nel **Kernel Space**. Un errore nel *driver* o un blocco di rete (per NFS/SMB) può causare un **Kernel Hang** (=blocco totale del sistema).
    - **Soluzione** → **FUSE** permette a un *file system* di girare come un **processo/demone in User Space**. Se il *file system* (o la rete) si blocca, fallisce solo il processo *demone*, ma il *kernel* del sistema operativo rimane stabile e funzionante.
    </aside>
    
    <aside>
    📌
    
    **Il Ruolo dell'MDS (Meta Data Center)**
    
    RADOS gestisce solo "oggetti" grezzi, non la struttura di directory o file. Per questo, Ceph FS richiede un componente dedicato: l’**MDS**, che gestisce i metadati del *file system* (nomi, permessi, gerarchia). L’MDS utilizza un **journal** per garantire la **consistenza delle scritture** e la **coerenza del file system**.
    
    </aside>
    

![image.png](8%20Storage%20Virtualizzato/image%204.png)

![Screenshot 2025-11-12 at 18.45.15.png](8%20Storage%20Virtualizzato/Screenshot_2025-11-12_at_18.45.15.png)

## 5.2. Amazon S3

**S3** è il servizio di ***object storage*** proprietario di Amazon. È un caposaldo dell'architettura AWS, con *storage* distribuiti su più aree geografiche e un'altissima *availability* (11 nove: 99.999999999%).

**N.B.** **L'availability a 11 nove è complessiva.** L'utente deve comunque selezionare l'opzione di *high availability*(replicazione su almeno 3 regioni) per sopravvivere alla *failure* di un intero *data center.*

### 5.2.1 Classi di Storage (Tiering)

S3 implementa la visione che "i dati non sono tutti uguali", offrendo *tier* (livelli) con costi e *Service Level Agreement* (SLA) diversi:

- **Standard** → per uso generale, accesso frequente.
- **Intelligent-Tiering** → monitora l'accesso ai dati e li sposta automaticamente tra i *tier* per ottimizzare i costi.
- **Standard-IA (Infrequent Access)** → costa molto meno, ma con tempi di accesso più alti.
- **One Zone-IA** → come Standard-IA, ma i dati sono in una sola "zona" (non ridondati geograficamente).
- **Glacier** → per *backup* sicuri a basso costo. L’accesso richiede ore (i dati sono su cassette/nastri recuperati da un braccio meccanico).
- **Glacier Deep Archive** → per archiviazione a lungo termine. L'accesso richiede 12+ ore (le cassette sono in uno *storage* *offline*).

<aside>
📌

**S3 Outposts**

Per aziende con rigidi vincoli legali sulla residenza dei dati (es. dati sanitari) o che non si fidano del *cloud* pubblico, AWS offre **Outposts**: Amazon fornisce il proprio software (l'ecosistema S3 e AWS) che gira **in house** (*on-premises*) nel *data center* del cliente.

</aside>

### 5.2.3. Modello Dati e Integrazione

S3 usa un'unità dati chiamata **Bucket**. Un *bucket* contiene **Data Object** (strutture JSON) o file. Ogni *bucket* può avere policy specifiche di crittografia, controllo accessi (ACL) e sincronizzazione.

I *bucket* S3 sono profondamente integrati nell'ecosistema AWS e possono essere usati, ad esempio, come *input* diretto per *job* **MapReduce** (come PySpark), rendendo l'analisi dei dati su larga scala molto efficiente ma aumentando il **vendor lock-in**.

---