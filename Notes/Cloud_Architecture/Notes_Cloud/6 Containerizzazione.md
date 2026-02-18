# 6. Containerizzazione

Number: 4
Status: Done
Type: theory

# 1. Introduzione

<aside>
💡

Un **container** è un'**unità software** che contiene tutte le **dipendenze necessarie** per essere eseguita in autonomia ed è completamente **isolata** dal resto del sistema operativo. A differenza delle VM, il container lavora a livello di **Sistema Operativo**, condividendo lo stesso kernel dell'Host, il che lo rende estremamente **leggero** e **veloce da avviare**.

</aside>

## 1.1. Motivazione: Superare i Limiti delle VM

L'esigenza di ospitare più applicazioni sulla stessa macchina ha trovato una soluzione più efficiente rispetto alle VM, le quali presentano:

- **Elevato Overhead:** le VM replicano l'hardware e richiedono l'avvio di un intero Sistema Operativo Guest, risultando in un elevato consumo di memoria e risorse.
- **Lento Startup:** il tempo di avvio è alto perché richiede il boot completo del sistema operativo ospitato.
- **Isolamento su Livello Hardware:** l'astrazione avviene a livello Hardware, rendendo le VM pesanti.

## 1.2. Contesto del Problema: Software Units e Dipendenze

Il software è composto da:

1. **Parte Statica** → il segmento di testo (programma, file eseguibile) e dati statici (file, configurazioni).
2. **Parte Dinamica** → la parte che cambia nel tempo, inclusi registri, immagine di memoria e risorse.
3. **Dipendenze** → librerie di funzioni (come `libc` per `printf`, librerie grafiche come X11, ecc.).

Quasi tutti i programmi dipendono da **librerie esterne** per funzionalità comuni (es. `libc`, librerie grafiche).

### 1.2.1.Modello di Distribuzione Classico

Nel modello classico la distribuzione avviene tramite **pacchetti** (`.deb`, `.rpm`), che contengono:

- i binari del programma,
- i **metadati delle dipendenze**, cioè le versioni delle librerie richieste sull’host.

<aside>
🚨

**Problema: “Dependency Hell”**

Il problema nasce quando più programmi richiedono **versioni incompatibili** della stessa libreria, rendendo difficile o impossibile la coesistenza sullo stesso sistema.

</aside>

<aside>
✅

**Soluzione: Container**

Il container risolve questo problema essendo **autocontenuto**. Invece di richiedere che le dipendenze siano installate sull'Host, **le include tutte** al suo interno, eliminando il problema di compatibilità con l'Host.

</aside>

---

# 2. Tecniche di Gestione delle Dipendenze

Per integrare le librerie esterne nel codice, esistono due approcci:

- **Static Linking** → durante la compilazione, il **Linker** copia fisicamente il codice delle librerie dentro l'eseguibile. Il processo coinvolge due tipi di file principali:
    - file `.o`, che contiene il codice macchina del programma e una tabella dei simboli con le dipendenze non ancora risolte.
    - file `.a` (archive), raggruppa più file oggetto (`.o`) in un'unica **libreria statica**.
    
    Il linker copia il codice dai file `.a` nei punti corretti del `.o`, risolvendo tutti i simboli, e produce l’eseguibile finale (`.exe` su Windows, `.out` su Linux). Si attiva con l'opzione `-static`.
    
    Svantaggi:
    
    - L'eseguibile risulta **enorme**, spesso decine o centinaia di MB anche per programmi semplici con GUI
    - Se esce una nuova versione di una libreria, è necessario **ricompilare** e ridistribuire l'intero programma
- **Dynamic Linking** → il codice delle librerie non viene copiato. Il programma contiene solo "rimandi" a file esterni chiamati **Shared Objects (`.so`)**.
    
    <aside>
    📌
    
    **Meccanismo del Dynamic Linking**
    
    Poiché le librerie `.so` vengono caricate a **indirizzi variabili**, il programma non conosce a priori l’indirizzo delle funzioni. Per gestire questa incertezza, il linker introduce due elementi:
    
    - **Stub (Il Segnaposto):** piccoli frammenti di codice inseriti nel programma che fungono da intermediari. Lo stub intercetta la chiamata a una funzione esterna e, se necessario, delega la risoluzione al linker dinamico (”non so dove sia la funzione, vado a chiedere”).
    - **GOT (Global Offset Table):** tabella in memoria che contiene gli indirizzi reali delle funzioni esterne.
    
    Fase di Binding (Lazy Binding):
    
    1. **Prima chiamata:**
        1. Il programma chiama lo **Stub**
        2. Lo Stub invoca il **Linker Dinamico (`ld.so`)**
        3. `ld.so` cerca la libreria nel sistema, trova l'indirizzo della funzione nella libreria `.so` e lo scrive nella **GOT**
    2. **Chiamate Successive:** il programma legge l'indirizzo direttamente dalla GOT e salta alla funzione senza coinvolgere nuovamente il linker dinamico.
    </aside>
    
    Vantaggi: 
    
    - I file `.so` sono mappati in memoria e possono essere **condivisi** tra più processi diversi, riducendo lo spazio occupato sia su disco che in memoria.
    - Le librerie possono essere **aggiornate** semplicemente sull'Host senza dover toccare e ricompilare l'eseguibile.
    
    Svantaggi: il programma ha bisogno delle corrette librerie esterne (`.so`) per funzionare, introducendo una **dipendenza** dal sistema Host. Questo è il fattore che crea il "Dependency Hell".
    

---

# 3. Principi Storici e Isolamento Linux

L'evoluzione dei container ha seguito un percorso progressivo, simile a quello della virtualizzazione, partendo da isolamenti parziali fino ad arrivare a soluzioni complete.

## 3.1. Evoluzione Storica del Concetto di Container

1. **chroot (anni ‘70)** → è il **primo esempio di isolamento**. La *system call* `chroot` permette a un processo di operare con una **diversa root del file system**, creando un ambiente isolato **solo dal punto di vista del *filesystem***.
    
    ![image.png](6%20Containerizzazione/image.png)
    
    Limiti: è possibile evadere il `chroot`. L'isolamento resta valido solo sul *filesystem*; la lista dei processi e il consumo di risorse rimangono globali (rischio **DoS**).
    
    ![image.png](6%20Containerizzazione/image%201.png)
    
2. **jail (Anni 2000, FreeBSD):** evoluzione di `chroot`. Permette di partizionare il sistema in più *jails* completamente isolati, ognuno con il proprio indirizzo IP, configurazione di rete, **lista processi separata**, *hostname* e *root user* isolato. L'isolamento di sicurezza è cruciale per contenere eventuali attacchi.
3. **VServer (Linux, 2001):** meccanismo avanzato simile ai *jail*, che introduce un **partizionamento delle risorse** (memoria, CPU, *filesystem*), garantendo **isolamento delle prestazioni** e protezione dagli attacchi **DoS**.
4. **Solaris Containers (Solaris, 2004):** Solaris conia per la prima volta il termine "**Containers**" (evoluzione delle Solaris Zones), consentendo server isolati all'interno di una singola istanza del S.O.
5. **LXC - Linux Containers (Linux, 2008):** rappresenta la prima soluzione **completa** su Linux, basata sui due meccanismi fondamentali di isolamento:
    - **Namespaces** → meccanismo che incapsula una specifica **risorsa di sistema** in un livello di **astrazione**. In pratica, i *namespaces* permettono di creare delle **istanze isolate** di queste risorse. Un processo che viene eseguito all'interno di un *namespace* vede solo le risorse assegnate a quello spazio, ignorando completamente le altre istanze e il resto del sistema.
        
        Ogni tipo di *namespace* ha lo scopo di astrarre e isolare una risorsa specifica:
        
        | **NameSpace** | **Risorsa Isolata** | **Esempio Pratico nel Container** |
        | --- | --- | --- |
        | **Mount (`mnt`)** | File System | Il container vede solo la sua *root* e i suoi file (più sicuro del vecchio `chroot`). |
        | **Network (`net`)** | Stack di Rete | Il container ha un suo indirizzo IP e una sua interfaccia di rete virtuale. |
        | **User ID (`user`)** | Identificatori Utente/Gruppo (UID/GID) | L'utente `root` all'interno del container è un utente con privilegi ridotti ("non-root") sul sistema Host. |
        | **PID (`pid`)** | Lista dei Processi | Il primo processo avviato nel container ha sempre **PID 1**, isolato dai processi dell'Host. |
        | **UTS (`uts`)** | Hostname e Domainname | Il container ha un nome di rete interno che non interferisce con quello del server Host. |
        | **IPC (`ipc`)** | Comunicazione Interprocesso | Isola meccanismi di comunicazione come memoria condivisa e semafori tra container diversi. |
    - **cgroups** → rappresentano il meccanismo che permette di **controllare e limitare** le risorse fisiche (CPU, Memoria, I/O, Banda) che i processi all'interno di un *namespace* possono consumare. Sono cruciali per l'**isolamento delle prestazioni** e per evitare attacchi di tipo **Denial of Service (DoS)**, garantendo un controllo completo sull'accesso alle risorse.
6. **Docker (2013):** rappresenta una svolta, semplificando notevolmente il *deployment*. Inizialmente basato su LXC, è poi passato a *containerd* e *libcontainer*, consolidando l'ecosistema per la **creazione e gestione delle immagini** e l'**orchestrazione**.

---

# 4. DevOps

Tradizionalmente, il tempo tra una *release* e la successiva era lungo a causa di un ciclo di sviluppo rigido (*Waterfall*). La risoluzione dei *bug* richiedeva di attendere la *release* successiva, rallentando la risposta ai *feedback* dei clienti.

Nel 2009, **Patrick Debois** introduce la metodologia **DevOps**, un nuovo modello di sviluppo basato su **cicli più brevi e iterativi** (*sprint* di 2-4 settimane). L'approccio si fonda sull'introduzione di poche funzionalità per volta e sul **testing fortemente automatizzato** (es. test eseguiti automaticamente a ogni *commit*).

## 4.1. Vantaggi del DevOps

L'approccio DevOps è fondamentale per l'adozione dei container, in quanto promuove:

- **Velocità** → più rapida introduzione di nuove *feature* e migliore risposta ai requisiti.
- **Rapid Delivery** → rilasci frequenti tramite tecniche automatizzate.
- **Affidabilità (Reliability)** → *testing* automatizzato e *feedback* veloce.
- **Scalabilità** → facilità di *deployment* e modularità di progettazione.
- **Sicurezza** → design modulare e controllo granulare delle operazioni.

## 4.2. Pratiche Abilitanti di DevOps

- **Continuous Integration and Delivery (CI/CD):**
    - Continuous Integration (CI) → si adotta il principio del **“fail early, fail often”**, che permette di individuare rapidamente gli errori già nelle fasi iniziali dello sviluppo grazie a una forte serie di test automatici.
    - Continuous Delivery (CD) **→** richiede una solida *codebase*, un sistema di gestione delle versioni, il supporto di **test automatizzati**, l’approvazione del *product owner* e un processo strutturato di messa in produzione.
- **Microservizi** → un'applicazione basata su microservizi è costruita come un insieme di **componenti indipendenti** che comunicano tramite API (spesso basate su HTTP), permettendo un elevato riuso del codice.
    
    **N.B.** I **container** sono la tecnologia abilitante per il *deployment* dei microservizi, facilitando la distribuzione coerente.
    
- **Infrastructure as Code (IaC)** → questo paradigma prevede la definizione dell’infrastruttura (numero di repliche, connessioni di rete, regole di *routing*, ecc.) tramite **file di configurazione** gestiti con sistemi di *version control*.
    
    Vantaggi: la configurazione diventa **ripetibile**, il *deployment* automatizzato risulta semplice e si risolvono i problemi del tipo “funziona solo sul mio sistema”.
    
    <aside>
    📌
    
    **Policy as Code**
    
    Estensione dell'IaC, dove le **regole di sicurezza** e conformità sono espresse nei file di configurazione, consentendo l’identificazione automatica delle non-conformità e una visione *security-by-design*.
    
    </aside>
    
- **Monitoring and Logging** → centrali per la metodologia DevOps, in quanto permettono la raccolta continua di informazioni e il monitoraggio attivo **24/7.**
    - Consentono la *post-mortem analysis* per identificare le cause di un incidente.
    - La *runtime analysis* rileva le anomalie **prima** che si trasformino in malfunzionamenti critici, riducendo i rischi di indisponibilità.
- **Communication and Collaboration** → questo elemento definisce l'aspetto organizzativo e culturale fondamentale di DevOps, rafforzando la collaborazione tra i team.
    - **Policy Esplicite** → richiede la definizione di **policy di collaborazione esplicite** che chiariscono processi, responsabilità e protocolli di comunicazione all'interno dei team.
    - **Fattore Culturale** → assume grande importanza il **fattore culturale**, che deve promuovere una collaborazione efficace e condivisa.
    - **Matrix Management** → la gestione delle responsabilità può essere strutturata tramite *matrix management*, che prevede:
        - Una **separazione verticale**, tipica per prodotto o progetto.
        - Una **separazione orizzontale**, basata sui ruoli all’interno dell’organizzazione.

---

# 5. Architettura dei Sistemi Containerizzati

## 5.1. Architettura Client-Server

I **container** si basano su un’architettura **client–server**:

- **Client (parte di controllo)** → consente all’utente di creare, avviare, fermare e gestire i container.
- **Server (parte di servizio)** → si occupa effettivamente del management e della creazione dei container. Una delle principali implementazioni è **Containerd**.

<aside>
📌

**Nota importante sulla Compatibilità**

I container possono eseguire immagini solo dello **stesso sistema operativo dell’host**, perché le **system call** invocate dai container devono essere supportate dal kernel dell’host. Questo comporta che:

- Su un host **Linux** → è possibile eseguire immagini Linux, ma non è possibile eseguire immagini Windows.
- Su un host **Windows** → è possibile eseguire immagini Windows, inoltre è possibile eseguire immagini Linux tramite **Docker Desktop**, che installa e gestisce una **VM Linux leggera** per fornire l’ambiente di esecuzione.
</aside>

## 5.2. **Containerd**

**Containerd** nasce inizialmente all’interno del progetto Docker, ma successivamente diventa un progetto indipendente. È un **backend** che espone delle **API** utilizzabili tramite **gRPC (Google Remote Procedure Call)** e può essere utilizzato da diversi frontend, come **Docker, Docker Swarm e Kubernetes**. Containerd è un **processo demone**, eseguito in background, che si occupa delle operazioni fondamentali per il controllo del ciclo di vita dei container. Le sue principali responsabilità sono:

- **Gestione del ciclo di vita dei container** → crea un container a partire da specifiche definite, lo esegue, permette di metterlo in pausa e di riavviarlo, e controlla che il processo del container sia in esecuzione correttamente.
- **Esecuzione tramite runc** → le operazioni di basso livello sono implementate da **runc**, un modulo che segue lo standard **OCI (Open Container Initiative)** ed esegue direttamente le chiamate al kernel per creare e gestire i container.
- **Gestione delle immagini** → Containerd scarica le immagini da registri remoti (pull, ad esempio da Docker Hub), carica immagini verso registri remoti (push) e gestisce lo storage locale in modo efficiente, sfruttando meccanismi di **copy-on-write**.

Containerd **non si occupa** di funzionalità di livello più alto, come il building delle immagini, il montaggio dei volumi, il networking tra più container o l’orchestrazione. Queste responsabilità sono demandate a livelli superiori, ad esempio al **container engine** nel caso di Docker. Dal punto di vista della compatibilità, Containerd nasce per il sistema operativo **Linux**, ma successivamente viene reso disponibile anche per altri sistemi operativi, come **Solaris e Windows**.

![image.png](6%20Containerizzazione/image%202.png)

---

# 6. Docker

**Docker** è un **container engine** che si colloca a un livello superiore rispetto a **Containerd** e fornisce all’utente gli strumenti per creare, gestire ed eseguire container. Docker gestisce principalmente tre tipi di oggetti: 

- **Immagini** → è un modello statico a partire dal quale vengono creati i container ed è l’unità con cui si distribuisce un’applicazione. È un oggetto **read-only**: ogni modifica genera una nuova immagine.
- **Container** → sono oggetti formati da **immagine + stato**. Lo stato può essere modificato tramite comandi Docker. Alcuni comandi rilevanti sono:
    - `docker build` → costruisce un’immagine a partire da un **Dockerfile**, che contiene le specifiche dell’immagine.
    - `docker image` → permette di gestire e modificare le immagini.
    - `docker pull` / `docker push` → scarica o carica immagini da/verso repository remoti.
    - `docker create` → crea un container a partire da un’immagine.
    - `docker container run`, `stop`, ecc. → avvia, ferma e modifica lo stato del container.
    - `docker attach` → collega una shell a un container.
    - `docker rm` → rimuove un container.
    - `docker ps` → mostra i container disponibili.
    - `docker network` → gestisce le impostazioni di rete (è facile sbagliare configurazione).
    - `docker volume` → gestisce i volumi.
- **Volumi** → un **volume** è una zona di memoria usata per salvare dati in modo persistente, evitando che vadano persi quando un container viene eliminato. Un container nasce da un’immagine e, durante l’esecuzione, crea e modifica file nel proprio **layer read-write**. Quando il container viene eliminato con docker rm, questo layer viene cancellato e tutti i dati vengono persi, rimanendo solo l’immagine iniziale. Questo comportamento è coerente con la natura **usa e getta** dei container, che privilegia immutabilità e facile sostituzione, non la conservazione dei dati.
    
    I volumi permettono di collegare una parte del filesystem dell’host al filesystem del container. In pratica, una directory dell’host viene montata all’interno del container in una posizione scelta dall’utente. Quando il container salva dati in quella directory, i dati rimangono persistenti sull’host e possono essere riutilizzati da altri container.
    

## 6.1. **Docker Filesystem**

Il filesystem di Docker è basato su un’architettura a **layer** e utilizza il meccanismo di **copy-on-write**. Il primo filesystem layered utilizzato da Docker era **AuFS**, mentre oggi viene utilizzato principalmente **OverlayFS**.

### 6.1.1. Meccanismo di funzionamento

1. Il primo layer corrisponde all’**immagine** ed è **read-only.** 
2. Quando viene creato un container, viene aggiunto un secondo layer **read-write**, nel quale vengono salvate tutte le modifiche rispetto all’immagine iniziale. 
3. Se viene aggiunto un ulteriore layer, quello precedente diventa read-only e solo l’ultimo rimane in modalità read-write.

**N.B.** Questo approccio è molto efficiente dal punto di vista delle prestazioni e dello spazio su disco: più container che si basano sulla stessa immagine possono condividere gli stessi layer, evitando di duplicare il filesystem in memoria o su disco.

![Screenshot 2025-10-06 at 15.06.53.png](6%20Containerizzazione/e5e6116c-e2b4-4a55-98bd-5e785bd717c0.png)

### 6.1.2. Struttura dei layer sul filesystem

Ogni **layer** corrisponde a una directory sul sistema host, il cui nome è una lunga stringa esadecimale (hash). Esistono differenze nella struttura a seconda del livello del layer.

- **Livello 1 (prima immagine):**
    - Directory `diff/`, che contiene il filesystem vero e proprio del layer.
    - File `link`, che fa riferimento alla directory stessa.
    - Altri file interni di gestione (non approfonditi).
- **Livelli superiori:**
    - Directory `diff/`, che contiene solo i file che differiscono rispetto al layer sottostante.
    - File `link`, che identifica la directory del layer.
    - File `lower`, che è un link simbolico al file link del layer padre (inferiore).
    - Directory `merged/`, che rappresenta la vista unificata del filesystem fino a quel punto: contiene i contenuti del livello corrente (`diff/`) combinati con quelli di tutti i livelli precedenti, senza duplicazione dei file grazie ai link.
        
        ![image.png](6%20Containerizzazione/image%203.png)
        
    - Directory `work/`, utilizzata internamente dal driver di storage per operazioni temporanee.

Quando si esegue un comando docker pull, vengono scaricati più layer separati che vengono poi composti per costruire l’immagine del container. 

![image.png](6%20Containerizzazione/image%204.png)

### 6.1.3. Dockerfile

Il **Dockerfile** è il file di testo che contiene tutte le istruzioni per *costruire* automaticamente un'immagine Docker. Esempio con i principali comandi:

- **`FROM`** → definisce il nome e la versione dell’immagine di base da cui partire.
- **`EXPOSE`** → indica la porta su cui il container deve essere esposto.
- **`ARG`** → definisce una variabile d’ambiente disponibile anche **a build time**.
- **`ENV`** → definisce una variabile d’ambiente disponibile solo **a run time**.
- **`RUN`** → permette di eseguire un comando all’interno del container durante la fase di costruzione dell’immagine. Ogni istruzione `RUN` genera un nuovo layer; per motivi di efficienza è quindi conveniente raggruppare più comandi in un’unica istruzione, in questo modo: `RUN cmd1 && cmd2`.
- **`CMD`** → specifica il comando da eseguire quando il container viene avviato. Se non viene definito nel Dockerfile, il container eredita il comando dalla propria immagine di base.
- **`ENTRYPOINT`** → indica un comando che viene sempre eseguito prima del CMD, effettuando di fatto un’append automatica dei parametri. Ad esempio, se si vogliono eseguire comandi Git, l’entrypoint può essere git.
- **`COPY`** → copia un file o una directory dall’host all’interno del filesystem del container.
- **`WORKDIR`** → imposta la directory di lavoro all’interno del container ed è equivalente al comando cd.

![image.png](6%20Containerizzazione/image%205.png)

### 6.1.4. Come Pushare un Immagine nel DockerHub

1. È necessario accedere al proprio profilo Docker: `docker login`. Oppure dal web, effettuando il login su Docker Hub tramite browser.
2. Dal proprio profilo web si crea una **repository pubblica** che conterrà l’immagine.
3. Si assegna all’immagine un tag che corrisponde al nome della repository appena creata: `docker tag tomcatworker:latest ${REPO}/tomcatworker`.
4. Si carica l’immagine sulla repository remota: `docker push tomcatworker`.

---

# 7. Orchestrazione: Docker Compose, Swarm e Kubernetes

<aside>
💡

L'**Orchestrazione** è un meccanismo che coordina l'esecuzione di più servizi, specialmente in ambienti distribuiti, per garantire che il sistema raggiunga e mantenga uno stato coerente e desiderato. Quando bisogna occuparsi di più containers ci sono due soluzioni:

- **Approccio Imperativo:** l'utente invia una **sequenza di comandi** (*come fare* l'azione).
- **Approccio Dichiarativo:** l'utente **dichiara lo stato finale** desiderato (*cosa si vuole ottenere*). Gli strumenti di orchestrazione utilizzano questo approccio per gestire autonomamente la transizione allo stato atteso.
</aside>

## 7.1. Docker Compose (Orchestrazione su singolo Host)

**Docker Compose** nasce per semplificare la gestione di scenari **multi-servizio** adottando un approccio **dichiarativo**. Funziona come un *wrapper* automatizzato attorno alle operazioni `docker run` di base. Lo strumento si articola di due componenti principali:

- **Compose File (YAML):** agisce come un "secondo livello di Dockerfile". Dichiara una lista di **servizi** (*top level entities*). Le specifiche includono: immagine, opzione `build` (per costruire l'immagine se necessario), rete, e altre proprietà.
    
    *Esempio:*
    
    ![image.png](6%20Containerizzazione/image%206.png)
    
- **Command Line Interface (CLI):** fornisce invece i comandi necessari per avviare, gestire e arrestare i servizi definiti nel file. La **CLI di Docker Compose** utilizza il comando principale `docker compose <sub_command>`. Gli argomenti successivi specificano le operazioni da eseguire, in modo simile ai comandi Docker tradizionali, ma estesi a scenari **multi-container**. Ad esempio, `docker compose build` consente di creare più immagini a partire dalle specifiche del file YAML, mentre `docker compose up` e `docker compose down` avviano o arrestano contemporaneamente più container e i relativi servizi associati.

<aside>
🚨

**Limiti:**
Docker Compose è particolarmente utile per il deployment di più servizi su un singolo host ma non può essere usato per il *deployment* su multi-host. 

</aside>

## 7.2. Docker Swarm (Orchestrazione Multi-Host Leggera)

**Docker Swarm** è la soluzione di orchestrazione **nativa** integrata in Docker che introduce funzionalità di **cluster management**. È una soluzione efficace per scalare su **multi-host**, mantenendo la semplicità della Docker CLI e **non richiedendo software aggiuntivo**. 

Swarm adotta un **modello dichiarativo** in cui l'utente definisce lo **stato desiderato** del servizio tramite il livello di replica atteso.

### 7.2.1. Unità di Lavoro

Uno ***Swarm*** è un insieme di **più *host* Docker** in *swarm mode*. Ogni *host* è un **Nodo** (un'istanza del *Docker Engine*), e tipicamente c'è un nodo per macchina (salvo ambienti di test).

- **Ruoli dei Nodi:**
    - **Manager Node** → ****riceve la definizione dei **Service** e distribuisce le unità di lavoro (**Task**). Per *default*, agisce anche come Worker.
    - **Worker Node** → riceve ed esegue i **Task**.
    
    <aside>
    📌
    
    **Architettura Decentralizzata**
    
    Non esiste un nodo Manager fisso (*single center*). I ruoli di Manager e Worker sono **assegnati a *runtime*** e sono dinamici. Generalmente all’arrivo di una nuova richiesta il primo nodo che risponde diventa il suo manager e si occuperà della sua gestione.
    
    - Sul nodo che deve diventare **manager** si esegue il comando: `docker swarm init --advertise-addr <IP-MANAGER>`, al termine dell’esecuzione viene restituito uno **swarm join token**, un token di sicurezza che i nodi worker dovranno usare per unirsi in modo sicuro al cluster.
    - Su ciascun nodo che deve diventare **worker** si esegue il comando: `docker swarm join --token <TOKEN> <IP-MANAGER>:2377`. In questo comando si specificano:
        - il **token di sicurezza**, necessario per autenticarsi,
        - l’**IP del nodo manager** e la porta 2377, usata per la comunicazione con il manager.
    - Sul nodo manager è possibile verificare lo stato del cluster con: `docker node ls`
    - Un nodo worker può essere promosso a manager, e un nodo manager può essere declassato a worker tramite i comandi `docker node promote` e `docker node demote`.
    </aside>
    
    <aside>
    📌
    
    **Networking Multi-Host**
    
    Utilizza una **rete interna *overlay*** configurata automaticamente che permette ai pacchetti di circolare uniformemente, garantendo l'interconnessione diretta tra tutti i *container* distribuiti. Integra un sistema di **Service Discovery** basato su un DNS interno.
    
    </aside>
    
- **Service:** è la **definizione di un t*ask* da eseguire** ed è la principale struttura dati. I *service* possono essere **replicati** per l'esecuzione parallela sui nodi Worker.
- **Task:** è un docker container che implementa un servizio. Viene assegnato a un nodo specifico e **non può migrare**.

<aside>
📌

**Flusso di Deployment**

Il *deployment* delle applicazioni avviene inviando la **definizione del *Service*** al Manager. Il Manager, basandosi sul numero di repliche desiderate, **genera i rispettivi Task** e ne coordina l'esecuzione sui nodi Worker, assicurando che il *cluster* mantenga lo stato desiderato.

</aside>

### 7.2.2. Altre caratteristiche

- **Desired State Reconciliation:** il nodo che in quel momento è Manager **monitora lo stato del cluster** e interviene in caso di *crash* o divergenze per riportarlo allo stato dichiarato, assicurando la **resilienza**.
- **Load Balancing:** integra un bilanciatore interno che fornisce un'astrazione di *routing e* supporta anche l'esposizione a bilanciatori esterni.
- **Sicurezza:** è *secure by default*. Tutte le comunicazioni tra i nodi sono **sempre cifrate** e verificate tramite **TLS mutual authentication**.
- **Aggiornamenti (*Rolling Updates*):** consente l'aggiornamento dei servizi in modo **incrementale** per garantire lo **zero *downtime*** e limitare l'impatto di eventuali *crash* della nuova versione:
    1. Viene avviato un **nuovo *container*** (versione aggiornata)
    2. Solo se il nuovo *container* risulta **sano** (*health check* superato), il sistema **elimina** una replica della vecchia versione
    3. Questo assicura che il numero totale di repliche attive **non scenda mai sotto un livello accettabile**

## 7.3. Kubernetes

**Kubernetes** (K8s), progetto *open-source* sviluppato originariamente da Google, è lo **standard *de facto*** per l'orchestrazione di applicazioni *containerizzate* in ambienti distribuiti. Condivide l'obiettivo con Docker Swarm, ma è significativamente più **complesso** e spesso considerato ***overkill*** per *deployment* semplici. Si basa anch'esso su un **modello dichiarativo** che utilizza la descrizione dello **stato desiderato** per gestire e controllare le transizioni del *cluster.*

### 7.3.1. Unità di Lavoro

- **Pod →** è l'**unità di lavoro di base** in Kubernetes (l'equivalente K8s del *Task* in Swarm). Può contenere un singolo *container* o **più *container*** che sono **co-located** (situati sullo stesso nodo) e ***tight coupled*** (fortemente accoppiati, non avrebbe senso che uno funzionasse senza l'altro). Ogni Pod riceve un **IP unico**. I *container* all'interno dello stesso Pod comunicano tramite *loopback interface*. Se un *pod template* viene modificato, i Pod esistenti vengono **ricreati** dal nuovo *template*. Se un Pod muore o viene rischedulato su un altro Nodo (cosa che Kubernetes fa spesso per il *self-healing* o lo *scaling*), il suo **indirizzo IP unico cambia**. Un pod si definisce tramite un template in un file `.yaml`.
- **Service** → crea un'**astrazione di rete** che permette ad altre applicazioni (o utenti esterni) di comunicare con un'applicazione, **indipendentemente da dove o quanti Pod la stiano eseguendo.** Al Service viene assegnato un **IP virtuale** (*ClusterIP*), il servizio DNS interno di kubernetes si occupa poi di mappare il nome del servizio al suo indirizzo IP. Chiunque voglia comunicare con l'applicazione X punta sempre a quell'IP/nome.

### 7.3.2. Storage Orchestration

Anche nei sistemi orchestrati i container hanno bisogno di un meccanismo per rendere **persistenti** i propri dati. È quindi necessario un sistema di **orchestrazione dello storage** che permetta, in caso di fallimento di un pod e sua ricreazione su un altro nodo, di accedere agli stessi dati.

Questo meccanismo si basa sul concetto di **volume**, in modo simile a quanto avviene in Docker, ma con una maggiore flessibilità. Un volume può essere:

- Una **risorsa fisica**, locale sul nodo host oppure, più spesso, una risorsa virtuale nel cloud, come ad esempio **AWS EBS**.
- Un **object storage**, cioè un sistema che gestisce i file come oggetti accessibili tramite un’API. Nell’object storage non esiste una reale struttura a cartelle basata su path: i dati sono memorizzati all’interno di **bucket** e ogni file è rappresentato come un oggetto identificato da una **chiave univoca** e accompagnato da **metadati**. Le operazioni di lettura e scrittura avvengono esclusivamente tramite chiamate all’API.

### 7.3.3. Architettura

L'architettura di Kubernetes è divisa tra:

- **Control Plane** (lato Manager) → coordina i cluster e i nodi. Al suo interno sono presenti:
    - **API Server** → punto **centrale** per tutte le interazioni di *management* con il *cluster*. Tutta la comunicazione tra i nodi e il *Control Plane* **transita attraverso l'API Server**.
    - **Key-Value Store** → **database distribuito** utilizzato da tutti i **master node** per memorizzare in modo **persistente** l’intero stato del cluster. Contiene informazioni fondamentali, come i pod in esecuzione, i servizi attivi e le configurazioni dei vari servizi. Il database che implementa questo meccanismo è **`etcd`**, che garantisce consistenza e affidabilità nella gestione dello stato del cluster.
    - **Scheduler** → **tiene traccia dei Pod** e assegna i nuovi Pod ai nodi più adatti, implementando **bin-packing** (= si tende a riempire i nodi esistenti il più possibile prima di utilizzare un nuovo Nodo), bilanciamento e ridondanza.
    - **Controller Manager** → gestisce diversi *controller* che monitorano e mantengono lo stato dei componenti.
- **Nodi** (lato Worker) → eseguono i carichi di lavoro:
    - **Kubelet** → agente presente su ogni nodo che funge da ***wrapper*** attorno al *Container Engine* (es. Containerd, CRI-O). Garantisce che i *container* all'interno dei Pod siano in esecuzione secondo le specifiche (*PodSpecs*), inclusi i controlli di salute.
    - **Kube-proxy** → **proxy di rete** che viene eseguito su **ogni Nodo** (Worker) del *cluster* e ha il compito di rendere i **Service** raggiungibili e funzionanti.
    - **Container Runtime →** lo strato sottostante per la gestione dei *container* stessi.
    - **DNS server** → gestisce la mappatura tra nomi dei services e IP cluster.
    - **Dashboard**, **cluster logging** e **container resource monitoring** → strumenti che permettono di raccogliere dati sull’utilizzo di risorse del nodo e sui processi in esecuzione sul nodo.

<aside>
📌

**Flusso di Deployment**

1. L'utente definisce l’applicazione e l’accesso simultaneamente attraverso due file di configurazione distinti, entrambi in formato **YAML**:
    - **Oggetto deployment:**
        - Immagine del Container: `my-webapp:v1.0`
        - Repliche Desiderate: **3**
        - Requisiti Risorse Pod: CPU = 0.5 core, Memoria = 512 MB
    - **Oggetto Service:** (definisce come accedere all'applicazione)
        - Selector: Utilizza la stessa **Etichetta** (`app: my-webapp`) per selezionare i Pods.
        - IP Stabile (`10.96.0.100`)
2. L’API Server riceve la richiesta (il file YAML) e la convalida. In questa fase **3** nuovi **Pod** vengono creati come record logici in `etcd`, tutti con stato **"Non Assegnato"**.
3. Lo scheduler individua i **3 Pod non assegnati** in `etcd`, controlla le risorse di ciascun Nodo e usa l'algoritmo di *bin-packing* per distribuire uniformemente il carico. Infine comunica le decisioni all'API Server:
    - **Pod 1** viene assegnato al **Nodo A**.
    - **Pod 2** viene assegnato al **Nodo B**.
    - **Pod 3** viene assegnato al **Nodo C**.
4. Il **Kubelet** sul Nodo A vede che il Pod 1 gli è stato assegnato. Ordina al *Container Runtime* di avviare il *container* `my-webapp:v1.0`. Lo stesso accade sui Nodi B e C. Entro pochi secondi, abbiamo **3 Pod in stato "In Esecuzione"** (1 su A, 1 su B, 1 su C), ognuno con il proprio **IP unico e instabile** (es. `10.42.0.5`, `10.42.1.8`, `10.42.2.10`).

**Scenario di Self-Healing (Controller Manager):**

1. Dopo un’ora, il **Nodo B si guasta**. Il Pod 2 smette di rispondere.
2. Il *Node Controller* rileva il fallimento del Nodo B. Il *Replication Controller* nota che lo **Stato Attuale (2 repliche)** non corrisponde allo **Stato Desiderato (3 repliche)**.
3. Il *Controller Manager* **crea immediatamente un nuovo Pod 4** (non assegnato) e lo invia all'API Server.
4. Lo **Scheduler** individua il Pod 4, lo assegna al **Nodo A** (se ha spazio) o al **Nodo C**, e il **Kubelet** avvia la quarta istanza, riportando lo stato a **3 repliche attive** (self-healing completato).

**Accessibilità:**

1. Su ogni Nodo, il *Kube-proxy* configura le regole di rete per intercettare il traffico destinato a `10.96.0.100` e lo **distribuisce in modo bilanciato** tra i Pod sani attivi (inizialmente Pod 1, 2 e 3; dopo il crash, Pod 1, 3 e 4).

![image.png](6%20Containerizzazione/image%207.png)

</aside>

### 7.3.4. Networking

K8s gestisce automaticamente il *routing* multi-nodo e **non richiede una rete *overlay* privata** (o sottorete dedicata).

---