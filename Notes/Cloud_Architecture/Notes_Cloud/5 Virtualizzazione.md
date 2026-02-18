# 5. Virtualizzazione

Number: 3
Status: Done
Type: theory

# 1. Introduzione

La virtualizzazione è una delle tecnologie abilitanti fondamentali per il cloud computing. L'idea di base è quella di passare da tecnologie strettamente legate all'hardware (**hardware-centric**) a un modello definito dal software (**software-defined**), che risulta molto più agile da gestire e configurare. In pratica, si crea una versione virtuale, basata su software, di una risorsa fisica. 

Questo approccio si è affermato per tre ragioni principali:

- **Server Consolidation** → permette di eseguire più macchine virtuali (VM) su un unico server fisico, ottimizzando l'uso di hardware spesso sottoutilizzato e riducendo i costi.
- **Business Continuity** → aumenta l'affidabilità. In caso di guasto hardware, una VM può essere rapidamente riavviata su un altro host. Molti **hypervisor** (= software di virtualizzazione) includono meccanismi di **alta affidabilità** che automatizzano questo processo. Anche gli aggiornamenti diventano più sicuri: è possibile creare uno **snapshot** della VM, eseguire l'aggiornamento e, in caso di problemi, ripristinare facilmente lo stato precedente.
- **Software Development and Testing** → consente di creare e isolare facilmente ambienti di sviluppo e test, replicando fedelmente le configurazioni di produzione

Le risorse che possono essere virtualizzate includono:

- **Risorse hardware** (CPU, GPU, memoria)
- **Risorse di storage**
- **Risorse di rete**

---

# 2. Livelli di Astrazione della Virtualizzazione

A seconda di dove si posiziona il layer di astrazione, distinguiamo tre categorie principali:

- **Emulazione** → il software (emulatore) ricrea completamente un' **architettura hardware** diversa da quella dell'host. Offre massima flessibilità ma a costo di un **overhead prestazionale enorme**.
    
    *Esempi:* **BOCHS** (emulatore x86) e **VICE** (emulatore del Commodore 64).
    
- **Virtualizzazione** → un **hypervisor** si interpone tra l'hardware dell'host e il sistema operativo guest. Le VM condividono la stessa architettura della CPU dell'host ma possono eseguire sistemi operativi diversi.
    
    *Esempi di hypervisor:* **VMware**, **VirtualBox**, **Xen**, e **KVM**.
    
- **Containerizzazione** → l'astrazione avviene sopra il **kernel del sistema operativo** dell'host. I container condividono lo stesso kernel dell'host, ma operano in spazi di processo isolati. Questo li rende estremamente leggeri e veloci da avviare. Condividendo lo stesso tipo di kernel, si limita la varietà di sistemi operativi eseguibili.
    
    *Esempi:* **Docker** e **Kubernetes**.
    

![image.png](5%20Virtualizzazione/image.png)

---

# 3. Storia della Virtualizzazione

## 3.1. Anni ‘70: I Mainframe IBM

Contrariamente a quanto si possa pensare, la virtualizzazione non è una tecnologia recente. Le sue radici affondano negli **anni '70**, con i mainframe di IBM. L'esigenza nacque dalla necessità di separare gli ambienti di sviluppo e produzione su macchine costose come l'IBM System/360, che era single-task e single-user. La risposta fu l’**IBM System/370**, che introdusse per la prima volta il supporto alla virtualizzazione, permettendo di eseguire più istanze del sistema operativo su un'unica macchina fisica.

## 3.2. Anni ‘80: Il Declino e l’Era del Personal Computer

Negli **anni '80**, con l'avvento dei personal computer, l'hardware divenne molto più economico. Di conseguenza, l'interesse per la virtualizzazione diminuì drasticamente, poiché era più conveniente acquistare macchine fisiche separate piuttosto che virtualizzare un mainframe. Questa perdita di interesse ebbe una conseguenza critica: le architetture progettate in quel periodo, come l'**Intel x86**, non furono pensate per la virtualizzazione e vennero create senza il necessario supporto hardware.

## 3.3. Dagli Anni 2000 a Oggi: La Rinascita con il Cloud Computing

L'interesse per la virtualizzazione è tornato prepotentemente a partire dagli anni 2000. Le ragioni principali furono due: la crescente necessità di testare software su molteplici sistemi operativi e, soprattutto, l'affermazione del **cloud computing**, per il quale la virtualizzazione è una tecnologia abilitante fondamentale. Il problema storico dell'architettura x86 ha spinto allo sviluppo di nuove tecniche software e all'introduzione di estensioni hardware specifiche (come Intel VT-x e AMD-V) per superare le limitazioni iniziali.

---

# 4. Principi Fondamentali della Virtualizzazione

## 4.1. I Tre principi di Popek e Goldberg

Un hypervisor (o Virtual Machine Manager) deve soddisfare tre caratteristiche essenziali:

1. **Equivalenza/Fedeltà** → un programma eseguito su una VM deve comportarsi in modo identico a come si comporterebbe sulla macchina fisica corrispondente. Le uniche differenze ammesse sono quelle legate al timing e alla disponibilità di risorse.
2. **Controllo delle Risorse/Sicurezza** → l'hypervisor deve avere il controllo completo e assoluto di tutte le risorse del sistema. Nessuna VM può accedere a risorse non allocate esplicitamente, e l'hypervisor deve sempre poterle revocare (preemption).
3. **Efficienza/Performance** → la stragrande maggioranza delle istruzioni della VM deve essere eseguita direttamente dalla CPU fisica, senza l'intervento dell'hypervisor. Un overhead eccessivo, tipico degli emulatori, renderebbe la virtualizzazione inutilizzabile.

## 4.2. Classificazione delle Istruzioni

Per soddisfare questi principi, Popek e Goldberg classificano le istruzioni della CPU in tre categorie rilevanti:

- **Istruzioni Safe** → sono la stragrande maggioranza delle istruzioni. Non accedono allo stato critico del sistema e non richiedono privilegi speciali. Per rispettare il principio di **efficienza**, l'hypervisor le lascia eseguire **direttamente e senza alcun intervento** dalla CPU.
- **Istruzioni Privilegiate** → istruzioni che possono essere eseguite solo in modalità "supervisore" (es. `ring 0` in x86). Se eseguite in modalità utente, generano un'eccezione hardware, un **trap**.
- **Istruzioni Sensibili (Sensitive)** → istruzioni che tentano di modificare lo stato del sistema (es. accesso ai registri di controllo, gestione degli interrupt) o il cui comportamento dipende da tale stato. Queste istruzioni devono essere intercettate e gestite dall'hypervisor.

![image.png](5%20Virtualizzazione/image%201.png)

<aside>
🔑

**Teorema fondamentale della virtualizzazione di Popek e Goldberg**

Tale teorema afferma che un'architettura supporta efficientemente la virtualizzazione se e solo se **l'insieme delle istruzioni sensibili è un sottoinsieme delle istruzioni privilegiate**. In questo modo, ogni volta che una VM tenta di eseguire un'operazione "pericolosa", si genera un trap hardware che trasferisce il controllo all'hypervisor. Questo meccanismo è il cuore del modello **Trap-and-Execute.**

</aside>

---

# 5. Approcci alla Virtualizzazione

La virtualizzazione può essere classificata secondo diversi criteri, che definiscono l'approccio utilizzato per creare e gestire gli ambienti virtuali. Le principali categorie sono:

- **Gestione del Principio di Fedeltà:** se il sistema operativo guest viene modificato (`Paravirtualizzazione`) o meno (`Virtualizzazione Completa`).
- **Supporto Hardware:** se la tecnica si basa interamente sul software (`Software-based`) o sfrutta estensioni della CPU (`Hardware-assisted`).
- **Gestione delle Istruzioni Sensibile:** come l'hypervisor gestisce le operazioni critiche (`Trap-and-Execute`, `Binary Translation`, `Binary Patching`).

![image.png](5%20Virtualizzazione/image%202.png)

## 5.1. Tecniche di Virtualizzazione Completa

Questi approcci mirano a eseguire un sistema operativo guest **non modificato**, facendolo apparire come se girasse su hardware reale:

- **Trap-and-Execute** → questo è l'approccio più classico e datato. Il meccanismo è di tipo reattivo e funziona nel seguente modo:
    
    
    1. Il guest, eseguito con un privilegio inferiore, tenta di eseguire un' **istruzione sensibile**.
    2. Poiché l'architettura è virtualizzabile, questa istruzione sensibile è classificata come **privilegiata**. Di conseguenza, il tentativo di eseguirla in modalità "user" (privilegio inferiore) causa un **trap** immediato verso l'hypervisor.
    3. Il trap trasferisce istantaneamente il controllo all'**hypervisor**.
    4. L'hypervisor **emula** in modo sicuro l'istruzione intercettata, per poi restituire il controllo al guest.
    
    ![image.png](5%20Virtualizzazione/image%203.png)
    
    **N.B.** Ogni trap comporta un context switch, un'operazione costosa che può rallentare l'esecuzione, specialmente su CPU moderne con pipeline complesse.
    
- **Binary Translation** → questa tecnica (usata da **VMware**) adotta un approccio **proattivo**. Invece di attendere un trap (approccio reattivo), l'hypervisor analizza il codice binario del kernel guest prima dell'esecuzione:
    1. Il codice viene suddiviso in **basic block** (sequenze di istruzioni che terminano con un salto).
    2. L'hypervisor ispeziona questi blocchi: le istruzioni "safe" vengono lasciate invariate, mentre le istruzioni sensibili e non privilegiate vengono **tradotte** al volo (on-the-fly) in una sequenza di istruzioni sicure che ottengono lo stesso risultato, spesso invocando l'hypervisor.
    3. I blocchi di codice tradotti vengono salvati in una **cache**, in modo che le esecuzioni successive dello stesso codice siano molto più veloci.
    
    **N.B.** Questo approccio permette di ottenere ottime prestazioni, superando le limitazioni dell'hardware, e abilita anche la **nested virtualization** (eseguire un hypervisor dentro una VM).
    
- **Binary Patching** → è una variante della binary translation (usata da **VirtualBox)**. La differenza principale è che, invece di mantenere separato il codice tradotto, il binary patching **sovrascrive il codice originale** con la versione "patchata" e sicura.

## 5.2. Paravirtualizzazione

Questo approccio, il cui esempio principale è **Xen**, abbandona il principio di fedeltà assoluta per ottenere prestazioni superiori. Il concetto fondamentale è che il kernel del sistema operativo guest viene **modificato** per essere "consapevole" di girare in un ambiente virtualizzato. Invece di tentare di eseguire istruzioni sensibili (che causerebbero un trap o richiederebbero una traduzione), il kernel paravirtualizzato effettua chiamate esplicite e ottimizzate all'hypervisor, dette **hypercalls**.

Nei sistemi basati su paravirtualizzazione come Xen, l'architettura si basa su domini (le VM). Esiste una macchina virtuale "madre", chiamata **`dom0`** (Domain 0), che si occuperà di gestire le richieste di tutti i guest (**domN**):

![image.png](5%20Virtualizzazione/image%204.png)

Vantaggi: prestazioni molto elevate, in quanto si elimina la necessità di trap o traduzione binaria.

Svantaggi: richiede la modifica del sistema operativo guest, cosa non sempre possibile (es. con sistemi operativi closed-source come Windows, per cui servono driver specifici forniti dal vendor).

<aside>
📌

**Applicazione della Paravirtualizzazione:**

Il concetto di paravirtualizzazione non si limita alla gestione delle istruzioni della CPU, ma si estende a ottimizzazioni cruciali per i dispositivi e la memoria:

- **Split Driver**: tramite la paravirtualizzazione è possibile caricare un device driver che è conscio di essere all’interno di un ambiente virtualizzato, evitando l’emulazione hardware. Il device driver è composto da:
    - Frontend driver → driver astratto che non gestisce l'hardware reale ma scrive le richieste di I/O in un'area di memoria condivisa. Una volta pronti i dati, il Frontend invia un'**Hypercall** all'Hypervisor per notificare la presenza di nuovo lavoro.
    - Backend driver → riceve la notifica dall’hypervisor, legge i dati direttamente dal buffer condiviso e utilizza i driver reali dell'host per eseguire l'operazione sull'hardware fisico.
- **Memory Ballooning**: tecnica di gestione dinamica della memoria che permette all'Hypervisor di recuperare RAM da una macchina virtuale (Guest) senza che questa debba essere spenta o riconfigurata. Presenta il seguente meccanismo:
    1. L'hypervisor comanda al driver di "gonfiarsi" occupando RAM interna al Guest.
    2. Il Guest OS, credendo che la RAM sia usata da un processo interno, la rende indisponibile alle proprie app.
    3. L'hypervisor “ruba” quella RAM fisica per darla ad altri Guest.
</aside>

## 5.3. Virtualizzazione assistita dell’Hardware

Per risolvere definitivamente il problema dell'x86, sia Intel (**VT-x**) che AMD (**AMD-V**) hanno introdotto estensioni hardware specifiche per la virtualizzazione. Queste estensioni:

1. In precedenza i livelli di privilegio erano quattro (da `ring 0`, riservato al kernel, a `ring 3` riservato alle applicazioni). Con la virtualizzazione hardware viene introdotto un **flag di modalità** che sdoppia questi livelli, portando il totale a **8 anelli** (4 per ogni modalità):
    - **VMX Root**: modalità per l'**Hypervisor** (il `ring 0` viene soprannominato `ring -1`).
    - **VMX Non-Root**: modalità per il **Guest**, dove il sistema operativo continua a girare in un Ring 0 '*depotenziato*'.
2. Garantiscono che **tutte le istruzioni sensibili generino un trap** quando eseguite dal guest, trasferendo il controllo all'hypervisor in modo efficiente.
3. Forniscono supporto hardware per la **virtualizzazione della memoria (IOMMU)**, con tecnologie come Intel EPT e AMD RVI, che gestiscono il doppio livello di indirizzamento (virtuale del processo → "fisico" del guest → fisico dell'host) direttamente in hardware, superando la necessità di tecniche software come le shadow pages.

---

# 6. Gestione delle Risorse Virtualizzate

## 6.1. Gestione della CPU e CPU Capping

In ambienti cloud, è comune fare **overcommitment** delle CPU, ovvero assegnare più vCPU (CPU virtuali) di quante CPU fisiche siano disponibili, contando sul fatto che l'utilizzo medio è basso.

Tuttavia, se una VM utilizza il 100% della sua vCPU per un tempo prolungato (un "CPU hog"), può degradare le prestazioni di altre VM sull'host. Per evitare ciò, gli hypervisor applicano il **CPU Capping**: limitano arbitrariamente le risorse di calcolo assegnate a quella VM. 

Questo fenomeno è visibile attraverso la metrica **CPU Steal**, che indica la percentuale di tempo in cui la vCPU era pronta a eseguire codice ma è stata "derubata" del suo tempo di esecuzione dall'hypervisor.

## 6.2. Gestione della Memoria (RAM)

La virtualizzazione aggiunge un livello di complessità alla gestione della memoria, richiedendo una traduzione a due stadi:

$$
\text{Guest Virtual Address (GVA)} \Rightarrow \text{Guest Physical Address (GPA)} \Rightarrow \text{Host Physical Address (HPA)} 
$$

Sono state sviluppate diverse tecnologie per gestirla in modo efficiente:

- **Shadow Page Tables**: un approccio software in cui l'hypervisor mantiene una copia "ombra" delle tabelle delle pagine del guest. Questa tabella ombra non mappa `GVA -> GPA`, ma mappa direttamente `GVA -> HPA`. La **Shadow Page Table** si trova nella **RAM fisica dell'Host**, in uno spazio di memoria riservato esclusivamente all'**Hypervisor**. Nella memoria del Guest invece, esiste la page table "classica" creata dal sistema operativo della VM. Il Guest crede che questa sia la tabella definitiva usata dalla CPU, ma l'Hypervisor l'ha resa di sola lettura (read-only). Ecco cosa succede quando il Guest prova a cambiare la sua tabella:
    1. Il **Guest** tenta di scrivere nella sua tabella delle pagine (che lui crede sia in RAM fisica).
    2. Poiché l'**Hypervisor** ha protetto quella zona di memoria, avviene un **Trap** (un'eccezione hardware).
    3. L'**Hypervisor** prende il controllo, vede cosa voleva scrivere il Guest e:
        1. Aggiorna la tabella del Guest (per fargli credere che l'operazione sia riuscita).
        2. Aggiorna la **Shadow Page Table** traducendo l'indirizzo in un HPA reale.
- **Nested Page Tables (Intel EPT / AMD RVI)**: soluzione moderna, resa possibile dalle estensioni di virtualizzazione hardware (**Intel EPT**, **AMD RVI**). L’idea fondamentale è che la  CPU ora gestisce autonomamente la **doppia mappatura degli indirizzi**, supportando due set di tabelle di pagina ed eseguendo la traduzione in due fasi interamente in hardware, senza interventi continui dell’hypervisor.
    
    **Elimina quasi completamente l'overhead** delle Shadow Page Tables, migliorando drasticamente le prestazioni della memoria virtualizzata.
    

<aside>
📌

**Page Sharing**

Indipendentemente dalla tecnica usata, l'hypervisor può implementare ottimizzazioni come il **Page Sharing**. Se più macchine virtuali hanno in memoria pagine identiche (ad esempio, le stesse librerie di un sistema operativo), l'hypervisor può mappare tutte queste pagine virtuali a un'unica pagina fisica sulla RAM dell'host, risparmiando una notevole quantità di memoria.

</aside>

## 6.2. Gestione dell’I/O

L'accesso alle periferiche (rete, dischi, GPU) è una delle sfide più grandi.

- **Emulazione**: l'hypervisor emula una periferica hardware generica. È la soluzione più compatibile ma anche la più lenta.
- **Accesso Diretto (Passthrough)**: una periferica fisica viene assegnata in modo esclusivo a una singola VM, che può usarla con prestazioni quasi native. È la soluzione più performante ma non è condivisibile.
- **Split Driver**: l'approccio ibrido basato sulla paravirtualizzazione (descritto in precedenza), che offre un accesso condiviso e performante.

## 6.3. Gestione dello Storage

La virtualizzazione dello storage affronta la sfida di mappare la complessa struttura di un disco fisico (tradizionalmente un blocco di dati identificato da coordinate come cilindro, testina e settore) in un semplice **file** gestito dal sistema operativo host.

### 6.3.1. Dischi Virtuali per VM

I formati di file più comuni per i dischi virtuali sono **VMDK** (usato da VMware) e **QCOW2** (usato da QEMU/KVM, **che aggiunge nativamente supporto a compressione e cifratura**). Questi file non sono semplici contenitori di dati; la loro parte iniziale è costituita da un **header (metadati)** che contiene informazioni vitali come il **CID (identificatore univoco)**, la geometria emulata del disco e i puntatori per gestire le catene di dipendenza tra i file. Questi formati offrono **funzionalità avanzate**:

- **Thin Provisioning (file sparsi)** → Il file del disco occupa solo lo spazio effettivamente utilizzato (la dimensione fisica è minore della dimensione logica dichiarata). Quando il file viene creato, il sistema non alloca tutti i blocchi (zero-filling), ma li assegna dinamicamente solo al momento della scrittura.
    
    ![image.png](5%20Virtualizzazione/image%205.png)
    
    **N.B.** Questo approccio può portare alla frammentazione dei dati sul disco fisico, aumentando i tempi di ricerca (seek time). Tuttavia, con l'uso moderno degli **SSD**, questo problema è diventato trascurabile grazie alla loro velocità di accesso.
    
- **Snapshot** → questa operazione permette di "congelare" lo stato di un disco in un preciso istante, creando un punto di ripristino. Questa potente funzionalità è resa possibile dal principio del **Copy-on-Write (COW)**, che funziona nel seguente modo:
    
    
    1. **Congelamento del Disco Base**: quando si crea uno snapshot, il file del disco originale ("base disk") viene reso di sola lettura.
    
    1. **Creazione del Nuovo Layer (File Delta)**: a questo punto, viene creato un **nuovo layer** (strato) su cui la VM andrà effettivamente a scrivere. Tutte le modifiche future vengono salvate in questo nuovo strato, realizzato tramite un file separato chiamato **"delta link"** o file delta, che conterrà solo le differenze rispetto allo snapshot del disco base.
    
    ![Untitled](5%20Virtualizzazione/Untitled.png)
    
    1. **Gestione delle Letture**: quando la VM deve leggere un dato, il sistema controlla prima nel file delta. Se il dato non è lì (perché non è stato modificato dopo lo snapshot), viene letto dal file base originale.
    
    ![image.png](5%20Virtualizzazione/image%206.png)
    
    Questo approccio permette di creare punti di ripristino quasi istantanei e di tornare facilmente a uno stato precedente semplicemente scartando il file delta, il tutto occupando solo lo spazio necessario per memorizzare le differenze.
    

### 6.3.2. Storage a Layer per Container (SquashFS)

I container utilizzano un approccio diverso, basato su layer immutabili (spesso gestiti tramite Union File System). Lo **SquashFS** è una tecnologia spesso usata in questo contesto: non è un disco virtuale scrivibile, ma un file system compresso e di sola lettura. Il suo scopo è archiviare un'intera struttura di file e cartelle in un unico file immutabile e di dimensioni molto ridotte.

Le immagini dei container sono costruite da una serie di layer sovrapposti, e ogni layer è spesso un file SquashFS. Similmente al meccanismo Copy-on-Write, ogni layer contiene solo le differenze rispetto a quello sottostante. Quando il container viene avviato:

1. Questi layer di sola lettura vengono sovrapposti (merged).
2. Viene aggiunto un ultimo **layer scrivibile** in cima per gestire le modifiche effettuate durante l'esecuzione (lasciando intatti i layer sottostanti).

## 6.4. Migrazione di Macchine Virtuali

Per bilanciare il carico o per manutenzione, è spesso necessario spostare una VM da un host a un altro:

- **Stop-and-Restart**: la VM viene spenta, i suoi file vengono copiati sul nuovo host e poi viene riavviata. Semplice ma causa un downtime significativo.
- **Live Migration**: permette di spostare una VM in esecuzione con un downtime minimo (nell'ordine dei millisecondi). La tecnica più comune è la **pre-copy**:
    1. Preparation → viene creato un "guscio" vuoto della VM sull'host di destinazione e il disco viene trasferito.
    2. Memory Copy → l'hypervisor inizia a copiare le pagine di memoria dall'origine alla destinazione mentre la VM è ancora in esecuzione. Le pagine che vengono modificate durante la copia ("dirty pages") vengono ricopiate in più passaggi.
    3. Migration → quando rimangono da copiare solo le pagine usate più di frequente ("hot set") e lo stato dei registri, la VM viene brevemente messa in pausa (stop), queste ultime informazioni vengono trasferite, e la VM viene riavviata (resume) sull'host di destinazione.
    
    **N.B.** Durante la **live migration**, il tempo di arresto della macchina virtuale è molto ridotto (tipicamente intorno ai **10 ms**), ma la fase di **copia della memoria** può causare un degrado delle prestazioni che può durare anche diversi minuti, con una riduzione compresa tra il **5% e il 50%**. Nonostante questo calo, la VM continua a rimanere operativa durante la migrazione; per questo motivo, la **gestione delle performance** rappresenta un aspetto critico nei sistemi di virtualizzazione.
    

## 6.5. Sfide di Isolamento: Interferenza e Side Channels

La condivisione di risorse fisiche (CPU, cache, storage, rete) tra più VM può portare a **interferenza**: le azioni di una VM possono influenzare negativamente le prestazioni di un'altra ("noisy neighbor problem"). Questo fenomeno può anche essere sfruttato per creare **side channels**, canali di comunicazione nascosti per far trapelare dati tra VM collocate sullo stesso host, modulando il carico su una risorsa condivisa. Questi comportamenti violano il principio di isolamento delle macchine virtuali. 

---

# 7. Recap Virtual Memory

La **memoria virtuale** permette a ogni processo di vedere uno spazio di indirizzamento logico molto grande, indipendente dalla quantità reale di RAM disponibile.

- Lo spazio di indirizzi virtuali è suddiviso in **pagine virtuali**, ciascuna composta da $K$ **byte consecutivi**.
- Un indirizzo virtuale è formato da: `[ Virtual Page Number (VPN) | Offset ]`

Il numero massimo di pagine virtuali dipende dall’architettura della CPU (numero di bit dell’indirizzo) e dalla **memoria fisica effettivamente disponibile**. Quando un programma viene compilato, ogni riferimento in memoria è espresso come **offset all’interno di una pagina virtuale**.

## 7.1. Page Table

La **Page Table** è la struttura dati che permette di tradurre un indirizzo virtuale in un indirizzo fisico. È importante notare che la Page Table è organizzata come un array: il **VPN (Virtual Page Number)** non è memorizzato all'interno della tabella, ma funge da **indice** per accedere alla riga corrispondente. Ogni riga (**Page Table Entry**) contiene:

- **PPN** (=Physical Page Number)
- **Bit di controllo** [ad esempio: dirty, valid, bit si sicurezza (se posso accedere a quella pagina)]

![image.png](5%20Virtualizzazione/image%207.png)

## 7.2. Esecuzione di un Programma e Gestione della Memoria Virtuale

Per poter eseguire un programma, le sue pagine devono essere caricate in **RAM:**

1. **Creazione del processo:** viene avviato il nuovo processo e il SO crea la **Page Table** del processo, il puntatore alla Page Table viene caricato nei registri della CPU.
2. **Caricamento iniziale delle pagine:** una o più **pagine virtuali** del programma (blocchi di K byte) vengono copiate dalla memoria di massa in **frame liberi della RAM**. La Page Table viene aggiornata inserendo le associazioni: `VPN → PPN`.
3. **Condivisione delle pagine:**
    - Se due processi utilizzano la **stessa pagina in sola lettura (read-only)**, possono condividere lo stesso frame in RAM.
    - Se invece una pagina deve essere modificata (**write**), viene applicata una copia in un nuovo frame (**copy-on-write**), e le Page Table dei processi punteranno a frame diversi.
4. **Accesso alla memoria durante l’esecuzione:**
    1. La CPU genera un indirizzo virtuale `[VPN | Offset]` e consulta la Page Table del processo.
    2. Se la pagina è presente in RAM: ottiene la PPN, costruisce l’indirizzo fisico `[ PPN | Offset ]` e accede direttamente alla memoria fisica.
    3. Se la pagina non è presente in RAM,  avviene un **page fault**:
        1.  Il programma viene temporaneamente sospeso
        2. Il controllo passa al **Sistema Operativo** che copia la pagina dalla memoria di massa (disco) in un frame libero della RAM (swap-in)
        3. La Page Table viene aggiornata con la nuova associazione VPN → PPN
        4. L’istruzione viene rieseguita
5. **Ripetizione continua:** il meccanismo si ripete per ogni accesso alla memoria durante l’esecuzione del programma.

---