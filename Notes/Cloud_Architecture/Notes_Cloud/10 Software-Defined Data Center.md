# 10. Software-Defined Data Center

Number: 3
Status: Done
Type: theory

# 1. Introduzione

<aside>
💡

L'SDDC è un *data center* in cui tutta l'infrastruttura è virtualizzata e fornita come servizio, permettendo di soddisfare le esigenze del *cloud*.

</aside>

L’obiettivo è superare l’hardware rigido e statico, introducendo uno strato software che permette:

- **Elasticità**
- **Infrastructure as Code (IaC)**
- **Riprogettazione dinamica dell’architettura**, senza interventi o ricablaggi fisici

Come per i modelli IaaS, PaaS e SaaS (dove IaaS è la base per gli altri), l'SDDC si basa sul partizionamento di risorse fisiche condivise per creare risorse logiche isolate.

## 1.1. Requisiti e Sfide Ingegneristiche

### 1.1.1. Requisiti Chiave

- **Economicità (Affordable)** → sfruttare le **economie di scala**. Nei *cloud provider* il costo del personale (OPEX) viene spalmato su decine di rack; in una piccola azienda lo stesso personale gestirebbe un solo rack, con costi superiori all’hardware (CAPEX).
- **Scalabilità (Scalable)** → deve essere **modulare**. L’infrastruttura va ampliata rapidamente per seguire una crescita tipica del 10–15% annuo.
- **Affidabilità (Reliable)** → deve essere **Fault-Tolerant by Design**. Guasti a server, dischi o link non sono eccezioni, ma eventi normali.
- **Sicurezza (Secure)** → deve essere garantita su più livelli:
    - Fisica**:** controllo accessi, niente finestre, personale di sicurezza.
    - Software: policy, antivirus, *firewall.*
    - Management: procedure corrette (es. *deprovisioning* rapido degli account degli ex dipendenti).

### 1.1.2. Costi Energetici e di Raffreddamenti

I costi di un *data center* non sono legati solo al *computing*, i **consumi energetici** sono un fattore primario, una parte enorme di questi costi è il **raffreddamento**. La regola generale è che per 1 kW di server, sono necessari 600 W di condizionamento.

### 1.1.3. Monitoraggio e Service Level Agreement (SLA)

L'infrastruttura richiede un monitoraggio costante per due ragioni:

- **Efficienza →** analizzare e ottimizzare l’utilizzo dei server.
- **Accounting** → sia nel *public cloud* (fatturazione) che nel *private cloud* (es. una banca che deve ripartire i costi IT sui vari dipartimenti), è fondamentale tracciare le risorse utilizzate.

Il monitoraggio è legato al rispetto dei **Service Level Agreement (SLA)** (es. 99.999% *availability*). Gli SLA devono essere definiti con precisione: un "tempo di risposta < 100ms" non ha valore se non si specifica come viene misurato (medio? mediano? massimo?) e su quale intervallo (24 ore?). Un *provider* può facilmente rispettare un tempo medio di 100ms su 24 ore, pur avendo prestazioni pessime nelle ore di punta.

---

# 2. Architettura Fisica del Data Center

## 2.1. Componenti Hardware Generici

Dal punto di vista hardware, un *data center* ha una struttura gerarchica:

- **Blade Systems** → sono le unità di calcolo base (*computing block*). Non sono semplici schede madri, ma moduli *plug-and-play* con **CPU, RAM** e talvolta **storage locale** (per *hypervisor* o cache).
- **Chassis** → è l’involucro che ospita i blade. Fornisce alimentazione ridondata, raffreddamento (ventole), sensori e software di controllo. Il software può gestire identità hardware (ID CPU, MAC address), permettendo di sostituire un blade guasto con uno nuovo che eredita automaticamente la stessa configurazione.
- **Rack** → contengono più chassis. I rack si interconnettono tra loro nel *data center* e sono organizzati in **POD**.

![image.png](10%20Software-Defined%20Data%20Center/image.png)

## 2.2. Organizzazione Fisica

- **Architettura Tradizionale**: i *rack* sono organizzati in file (*rows*). Ogni *rack* contiene elementi di calcolo e storage, e uno **Switch Top-of-Rack (ToR)** nella parte alta, che collega i server al suo interno. Le file di *rack* sono collegate da *switch* **End-of-Row (EoR)** o **Middle-of-Row (MoR)**, che aggregano il traffico dei ToR.
    
    <aside>
    📌
    
    **Gestione del Calore**
    
    Il layout fisico è fortemente influenzato dal bisogno di dissipare calore, soprattutto nei *data center* ad alta densità. Lo schema più diffuso è **Corridoio Caldo / Corridoio Freddo**:
    
    - i rack sono disposti *fronte-fronte* e *retro-retro*
    - **Corridoio Freddo** → l’aria fredda viene pompata dal pavimento (forato) davanti ai rack.
    - **Corridoio Caldo** → il retro dei rack espelle aria calda, che sale ed è aspirata dai sistemi di condizionamento, per essere poi reimmessa nel pavimento freddo.
    
    ![image.png](10%20Software-Defined%20Data%20Center/image%201.png)
    
    </aside>
    

- **Architettura Modulare (POD):** un approccio alternativo per la scalabilità è l'architettura a **POD**. Un POD è un **container** di dimensioni standard, pre-cablato con *rack*, *storage*, condizionamento e connessioni. Per essere operativo basta collegare **alimentazione**, **rete** e **raffreddamento a liquido.** Questo approccio rende l’espansione **rapida, modulare e lineare**, ideale per i grandi cloud provider.

![Screenshot 2025-11-16 at 11.21.28.png](10%20Software-Defined%20Data%20Center/Screenshot_2025-11-16_at_11.21.28.png)

---

# 3. Architettura di Rete del Data Center

## 3.1. Assunzioni Semplificative

A differenza di Internet, un *data center* è un ambiente controllato. Si possono fare delle assunzioni semplificative:

- **Eterogeneità Limitata:** pochi tipi diversi di server e *switch* (4-5 generazioni).
- **Management Unificato:** il gestore del *data center* decide le regole.
- **Sicurezza:** le fonti di traffico sono controllate; si assume una presenza limitata di traffico malizioso (grazie alla sicurezza fisica).
- **Topologia Regolare:** la crescita della rete è pianificata, non incontrollata. La topologia non è un grafo casuale.

## 3.2. Topologia Gerarchica (Fat-Tree)

La topologia tradizionale dei *data center* è gerarchica, ma un **albero classico** presenta due problemi principali:

- i livelli superiori diventano **single point of failure**
- il traffico converge verso l’alto, creando **bottleneck**

Per superare questi limiti si utilizza la **Fat-Tree**, un’evoluzione dell’albero in cui i livelli alti hanno **più link paralleli**, aumentando sia la banda complessiva sia la tolleranza ai guasti.

Questa struttura si organizza in tre livelli logici:

- **Edge Layer (Access Layer):** generalmente gli *switch* **Top-of-Rack (ToR)**. Fornisce connettività L2 ai *server* (1-10 Gbps) e implementa policy semplici (ACL, QoS, *fault management*).
- **Aggregation Layer:** interconnette diverse zone (spesso gli *switch* **End-of-Row (EoR)** o *pod*). Gestisce policy più complesse (QoS, *traffic shaping*). Questo livello è **altamente ridondato** con link multipli.
- **Core Layer (Backbone):** è l'elemento portante che interconnette l'intero *data center* e gestisce i **link in uscita** (outbound). È il punto più critico, richiede prestazioni *at wire speed* (hardware) e altissima ridondanza.

![image.png](10%20Software-Defined%20Data%20Center/image%202.png)

![image.png](10%20Software-Defined%20Data%20Center/image%203.png)

## 3.3. Clos Networks

La topologia *Fat-Tree* è spesso implementata usando **Clos Networks**. Una *Clos Network* è un modo per costruire *switch* molto grandi (es. 256 porte) aggregando *switch* più piccoli (es. 32 porte), riducendo la complessità dei collegamenti da $O(n^2)$ a $O(n \log n)$ e garantendo cammini multipli.

<aside>

**Esempio: Google**

Un esempio concreto è l’evoluzione delle reti interne di **Google** (Firehose, Saturn, Jupiter). Per sostenere un traffico in crescita esponenziale (raddoppia ogni 12–15 mesi), Google ha adottato: **switch COTS** (*Commodity Off-The-Shelf*)*, t*opologie **Clos/Fat-Tree** altamente parallele, controllo centralizzato tramite **SDN**.

</aside>

![image.png](10%20Software-Defined%20Data%20Center/image%204.png)

---

# 4. Architettura Logica e Gestione (Workload)

Il *data center* non gestisce singole VM isolate, ma **cluster di VM** che cooperano per fornire servizi scalabili e affidabili. L’hardware fisico deve quindi supportare un’organizzazione logica flessibile e facilmente espandibile.

## 4.1. Web Cluster Architecture

Per aumentare la capacità di un’applicazione (ad esempio un *web service*), si può procedere in due modi:

- **Scale-Up:** migliorare le prestazioni del singolo server ottimizzando l'OS (es. *scatter-gather DMA*), il *web server* (da *multiprocessing* a *multithreading*) o usando *kernel-level server*.

- **Scale-Out:** si aumentano le prestazioni aggiungendo nuovi server al cluster. Esistono due forme di Scale-Out:
    - **Replicazione Verticale** → specializzare i nodi (es. separare il *database* dall'*application server*).
    - **Replicazione Orizzontale** → mettere in batteria tanti nodi identici che si dividono il traffico.
    
    **N.B.** Nella pratica, quando si scala verso l’esterno (*scale-out*), si combinano spesso **entrambi** gli approcci.
    

![image.png](10%20Software-Defined%20Data%20Center/image%205.png)

## 4.2. Web Switching e Virtual IP

Quando un’applicazione viene scalata *orizzontalmente*, il cluster deve apparire all’esterno come **un unico server**, con:

- un solo **Hostname**
- un solo **Virtual IP (VIP)**

Questo è possibile grazie al **Web Switch** (load balancer o reverse proxy), che riceve le richieste sul VIP pubblico e le distribuisce ai server di backend, ciascuno con un IP privato. Il Web Switch si trova quindi esattamente al confine tra:

- rete esterna (client → VIP)
- rete interna (VIP →  backend)

Lo *switching* può avvenire a due livelli:

- **Switching L4** → lavora analizzando solo la **quadrupla TCP** (IP sorgente/destinazione + porta sorgente/destinazione).
    
    Vantaggi: È velocissimo (implementabile in hardware/SDN). Vede il primo pacchetto **SYN**, sceglie il *server* e il resto dell'handshake (TCP e HTTPS) avviene direttamente tra *client* e *server* .
    
    Svantaggi: Non può supportare le **sessioni utente**. Un utente potrebbe essere reindirizzato a *server* diversi a ogni richiesta, perdendo lo stato (es. carrello, login).
    

![image.png](10%20Software-Defined%20Data%20Center/image%206.png)

- **Switching L7** → analizza la richiesta **HTTP/HTTPS**. Può leggere elementi del livello applicativo come: URL, header HTTP, cookie.
    
    Vantaggi: supporta le sessioni. Può leggere un *cookie* di sessione (es. `JSESSIONID=...jvm1`) e usare il *root ID* (es. `jvm1`) per reindirizzare l'utente sempre allo stesso *backend server*, rendendo lo *switch* **stateless** (non ha bisogno di una *binding table*).
    
    Svantaggi: è più lento. Lo *switch* L7 deve **completare l'handshake TCP** (e quello HTTPS) *prima* di poter ricevere e analizzare la richiesta HTTP.
    

![image.png](10%20Software-Defined%20Data%20Center/image%207.png)

## 4.3. Analisi del Traffico e Pattern (Mice and Elephants)

L'analisi del traffico nei *data center* (anche senza virtualizzazione) rivela *pattern* non casuali, cruciali per il *management*. 

1. **Mappe di Calore** → le mappe di calore mostrano chi comunica con chi:
    
    
    - **Punti sulla Diagonale** → molti server comunicano intensamente tra di loro in cluster locali [= macchine *affini* che svolgono lo stesso compito (es. job MapReduce)].
    - **Linee Verticali/Orizzontali** → server centrali (es. *scheduler*, *storage* o *monitor*) con cui tutti comunicano. Devono essere posizionate in modo **baricentrico** nella rete.
    
    ![image.png](10%20Software-Defined%20Data%20Center/image%208.png)
    
2. **Località del Traffico** → questi grafici mostrano **come il traffico si distribuisce dentro un rack** (within rack) e **tra rack diversi** (across racks):
    
    
    - **Volume di traffico** → l’asse X rappresenta il **logaritmo del numero di byte scambiati** tra due nodi:
        - Within rack: i valori si concentrano tra **10⁶ e 10¹⁴ byte**, con picchi elevati. Indica **scambi molto intensi** tra macchine vicine.
        - Across rack: i valori sono mediamente **più bassi**. La densità è più alta su traffico **limitato**, e solo pochi scambi raggiungono volumi molto grandi.
    
    ![image.png](10%20Software-Defined%20Data%20Center/4716aeec-cede-4df3-aa1b-ff9611dbb244.png)
    
    - **Numero di Corrispondenti** → questi grafici mostrano la **frazione di nodi con cui una macchina comunica**.
        - Within rack: la maggior parte delle VM parla con una **piccolissima percentuale (0–10%)** dei nodi locali. Poche comunicazioni, ma ripetute e consistenti.
        - Across racks: la frazione è quasi sempre **prossima allo zero**, pochissimi scambi e di entità ridotta.
    
    ![image.png](10%20Software-Defined%20Data%20Center/image%209.png)
    
    <aside>
    🔑
    
    **Interpretazione Complessiva (Mice & Elephants)**
    
    Questi dati mostrano una forte **località del traffico**, che riflette i due principali tipi di flussi presenti nei data center:
    
    - **Mice flows** → numerosissimi flussi piccoli, di breve durata e **sensibili alla latenza**. Avvengono tipicamente all’interno dello stesso rack (*within rack*) e sono usati per query, RPC, heartbeat e traffico di controllo, dove anche micro-ritardi incidono sulle prestazioni delle applicazioni.
    - **Elephant flows** → pochi flussi **molto grandi**. Si verificano spesso tra rack diversi (*across racks*), quando macchine distanti devono scambiarsi grandi volumi di dati, come nella migrazione di VM, nei backup o nei job di Big Data e MapReduce. Sono **sensibili alla banda**, perché richiedono un throughput elevato e stabile.
    </aside>
    
    <aside>
    📌
    
    **Congestione**
    
    Paradossalmente, il momento di massima congestione della rete non è durante il giorno, ma di notte, quando partono i *job* di *housekeeping* (es. chiusura casse) e i **backup** (*"il momento più caldo per la rete"*).
    
    ![image.png](10%20Software-Defined%20Data%20Center/image%2010.png)
    
    </aside>
    

---

# 5. Ottimizzazione e Gestione dell’SSDC

Un aspetto fondamentale dei data center moderni è che il traffico **non è casuale**, ma forma **bolle di macchine affini**: gruppi di VM che comunicano molto più tra loro che con il resto del cluster. Questo produce una forte **località del traffico**, soprattutto all’interno dello stesso rack. Grazie alla virtualizzazione (VM + SDN/NFV), un SDDC può sfruttare questa informazione per ottimizzare il *placement* e la *migrazione* dei workload.

## 5.1. Placement Communication-Aware

Il *placement* non deve basarsi solo su CPU e RAM, ma deve essere anche **consapevole della comunicazione**.

<aside>
🔑

**Principi chiave**

- **L’unità non è la singola VM, ma il cluster** → le VM che collaborano tra loro (affini) vanno trattate come un gruppo.
- **Località** → le VM con alta affinità (che comunicano molto) devono essere posizionate vicine: sullo **stesso *host***(comunicazione locale) o sullo **stesso *rack*** (distanza 1-hop), per evitare di sovraccaricare i *link* dell'Aggregation e Core Layer.
- **SDDC (Computation & Network Merge)** → in un data center tradizionale si decide prima il placement e poi si riconfigura la rete. Nell’SDDC, grazie a SDN e NFV, questi due passaggi vengono **eseguiti insieme**: posizioni le VM e contemporaneamente vengono configurati router, tunnel e virtual switch.
</aside>

## 5.2. Ottimizzazione delle Risorse

Placement e migrazione vengono usati per ottimizzare **quattro obiettivi principali**:

1. **Load Balancing** → evitare host sovraccarichi (che causano violazioni SLA) e host quasi vuoti.
2. **Power Consumption** → i server consumano molta energia anche da *idle*. È più efficiente **consolidare** le VM su meno macchine (es. due server al 70%) e **spegnere** quelle sottoutilizzate, risparmiando la potenza base (`P_min`). Il consumo di un server $P$ in funzione del suo utilizzo $U$ è modellato come:
    
    
    $$
    P=P_{\text{min}}+U\times(P_{\text{max}}-P_{\text{min}})\quad \text{con}\quad U\in[0,1]
    $$
    
     **N.B.** Modelli più complessi richiederebbero molti dati e renderebbero il **workload model** difficile da gestire. Con modelli troppo complicati, i parametri diventano quasi impossibili da stimare. Con un modello lineare, invece, pur essendo meno accurato, otteniamo **parametri affidabili** e sufficienti per prendere decisioni che riducono davvero il consumo energetico.
    
    ![image.png](10%20Software-Defined%20Data%20Center/image%2011.png)
    
3. **Ottimizzazione della Rete** → mantenere la **località del traffico** riduce la congestione e migliora prestazioni:
    - traffico locale → veloce, latenza minima
    - traffico cross-rack → più lento e più caro in termini di banda
    
    Il **placement communication-aware** serve proprio a minimizzare il traffico cross-rack.
    

1. **Heat Management (Gestione Calore)** → la disposizione dei workload influenza anche la **temperatura** del data center. Si usa la migrazione delle VM per ridistribuire il carico termico o per spegnere intere aree e i relativi condizionatori, risparmiando energia.

![image.png](10%20Software-Defined%20Data%20Center/image%2012.png)

## 5.3. Gestione su Scala Globale (Multi-DC)

L’orchestrazione cloud si estende oltre il singolo data center, gestendo il **carico tra diverse sedi geografiche**. I principali punti di congestione sono:

- **First Mile** → uscita del DataCenter
- **Last Mile** → rete del client che istanzia la connessione
- **Peering Point (IXP)**→ punti scambio fra AS

Gli AS interni generalmente non hanno problemi di banda, essendo costituiti da milioni di dispositivi molto vicini tra loro. I costi di località sono più alti per i cavi nei peering point, specialmente transoceanici. I cloud provider possono ottimizzare **first mile** e **peering**, mentre non possono intervenire sulla **last mile** (fibra nelle abitazioni degli utenti).

![image.png](10%20Software-Defined%20Data%20Center/image%2013.png)

<aside>
📌

**Soluzione di Google**

Gli hyperscaler come Google risolvono il problema diventando **ISP globali**, possedendo anche cavi sottomarini. La loro infrastruttura è organizzata su **tre livelli**:

1. **Data Center** → backend di calcolo e storage, centinaia di sedi.
2. **Edge Point of Presence (POP)** → migliaia di punti di peering con altre reti, gestiscono il routing interno alla rete di Google.
3. **Edge Nodes (Google Global Cache)** → moltissimi nodi di cache per contenuti statici (YouTube, Play Store), posizionati vicino agli utenti finali, spesso **all’interno dell’AS dell’utente**, riducendo drasticamente il numero di hop.

![image.png](10%20Software-Defined%20Data%20Center/image%2014.png)

Il traffico viene instradato il più possibile sulla **rete proprietaria**, utilizzando reti terze solo per l’ultimo tratto verso l’utente.

---

**Policy “Cold Potato”**

A differenza del *routing* standard "hot potato" (passare il pacchetto al vicino il prima possibile), Google adotta una politica **"Cold Potato"**. Mantiene il traffico sulla **propria rete privata globale** il più a lungo possibile, inoltrandolo all'ISP dell'utente solo all'*Edge POP* più vicino. Questo garantisce il pieno controllo sulle prestazioni e sull'SLA *end-to-end.*

</aside>

---

# 6. Dal DataCenter Classico al Software-Defined DataCenter

Il passaggio al modello **Software-Defined** trasforma radicalmente l'unità di gestione del *data center*.

- **Visione Classica** → la gestione era centrata sulle **singole VM** e su apparati di rete **fisici** (switch, router).
- **Visione Software-Defined (SDDC)** →  il focus si sposta su **cluster di VM affini**, organizzati in **Virtual Networks**. Gli apparati di rete vengono implementati come **software** (es. *virtual router*) eseguiti direttamente dalle macchine.

Nel SDDC il livello virtuale controlla sia il **compute** sia il **network**, decidendo come mappare VM e reti virtuali sull’infrastruttura fisica. Si ottiene così una vera **fusione tra calcolo e rete**: la configurazione delle VM e delle reti avviene in modo coordinato, permettendo di creare gruppi di macchine affini e interconnesse dinamicamente.

![image.png](10%20Software-Defined%20Data%20Center/image%2015.png)

## 6.1. Limiti della Rete Statica (Traditional DC)

- **Frammentazione delle Risorse (VLAN)** → le VLAN sono utili per la *performance isolation*, ma frammentano la rete. Il *traffic shaping* su VLAN statiche è complesso, e riconfigurarle rapidamente in seguito a migrazioni delle VM è difficile e rischioso.
- **Il Collo di Bottiglia del Core** → se due VM che comunicano molto non si trovano nello stesso rack o pod, il traffico attraversa la zona **Core**, dove i link sono costosi e molto scarsi (switch ad alte prestazioni).
- **Oversubscription Rate** → indica il rapporto tra la banda totale che i server potrebbero richiedere e la banda reale disponibile verso l’alto. Mentre ai livelli bassi (ToR) è comune un rapporto accettabile di **10:1**, ai livelli alti della gerarchia questo fattore può arrivare a **80:1** (= la banda reale nel cuore della rete è 80 volte inferiore a quella che i server potrebbero richiedere). Un oversubscription elevato significa che il core ha **molto meno throughput** rispetto al traffico potenziale proveniente dai rack: i link del core sono quindi una risorsa preziosa da utilizzare solo quando necessario.

## 6.2. Tecnologie Abilitanti

- **HyperConvergence** → l’idea di base è di utilizzare **building block** che hanno già **integrato** al loro interno le funzioni del **datacenter software defined** (calcolo, storage, networking). L’obiettivo principale è ridurre **OPEX** e tempi di **deployment**: invece di gestire componenti eterogenei da vendor diversi, si acquistano sistemi “chiavi in mano”, progettati per scalare semplicemente aggiungendo nuovi blocchi.

![image.png](10%20Software-Defined%20Data%20Center/image%2016.png)

- **Fabric Computing** → pensato per superare i limiti della gerarchia tradizionale e dell’oversubscription. Mira a **appiattire la topologia** riducendo i livelli di switching (da 3 → 2 o 1), creando una rete a maglia (*fabric*) altamente interconnessa, basata su dispositivi con **alto fan-out**. Appiattendo la rete si riduce la latenza, si ottiene una comunicazione locale efficiente e tutti i nodi risultano logicamente più vicini semplificando il placement delle VM affini.

![image.png](10%20Software-Defined%20Data%20Center/image%2017.png)

---

# 7. Il Problema dei Flussi di Elefanti → Soluzione: Optical Switching

Mentre i flussi *Mice* sono gestiti bene dallo *packet switching* tradizionale, i flussi **Elephants** saturano i link condivisi e beneficiano di circuiti dedicati ad altissima velocità.

Una soluzione innovativa per gli *Elephants* è l'uso di **link in fibra ottica riconfigurabili**. Non potendo collegare tutte le coppie di macchine con fibra dedicata (troppo costoso), si utilizza un sistema dinamico basato su **switch ottici**.

<aside>
📌

**MEMS a Specchi**

Il sistema utilizza specchi motorizzati ultra-precisi (**MEMS**). Quando il *controller* SDN rileva la necessità di un trasferimento massiccio tra due macchine (es. Macchina A e Macchina B), comanda ai motorini di orientare gli specchi per creare un **canale ottico diretto** (un circuito fisico di luce) tra A e B.

![image.png](10%20Software-Defined%20Data%20Center/2c81159a-68b5-4f0d-928c-9150d6b98a9f.png)

</aside>

Questo approccio ha un *overhead* iniziale (il tempo fisico per muovere lo specchio), quindi ha senso **solo per i flussi Elephant**, dove il tempo di configurazione è trascurabile rispetto al tempo di trasferimento dei dati. I flussi *Mice*, invece, continuano a viaggiare sulla rete a pacchetto tradizionale.

---

# 8. Data Center Design Principle

La progettazione di un Data Center non riguarda solo l’IT: coinvolge decisioni **geografiche, strutturali, energetiche e gestionali** fondamentali.

## 8.1. Criteri di Scelta della Location

La scelta del luogo in cui costruire un DC dipende da vari fattori:

- **Costi e Tasse** → si privilegiano aree con tassazione agevolata o terreni a basso costo (es. la differenza tra costruire a Rimini o a San Marino).
- **Risorse Umane** → i DC tendono a essere costruiti in **cluster** già esistenti: in queste zone è più facile reperire personale qualificato.
- **Connettività e Logistica** → è importante la vicinanza alle dorsali Internet e la facilità di trasporto e manutenzione delle infrastrutture.
- **Aspetti Energetici** → la disponibilità di energia economica, stabile e ridondata è il fattore più critico.

<aside>
📌

**Fenomeno “Rust Belt”**

Tendenza a riconvertire ex aree industriali pesanti in data center. Queste zone dispongono già di **grandi capannoni** e soprattutto di **collegamenti elettrici ad alta potenza**, costruiti per alimentare le vecchie industrie. Sono quindi luoghi ideali per ospitare **server ad alta densità** senza dover creare da zero l’infrastruttura energetica.

</aside>

## 8.2. Sicurezza Fisica

La sicurezza informatica non è sufficiente: la **sicurezza fisica** è essenziale perché l’errore umano o l’accesso non autorizzato rappresentano un rischio elevato. Un DC moderno presenta:

- **Sicurezza a strati concentrici** → recinzioni perimetrali, tornelli, guardie armate, badge, controllo biometrico.
- **Enclosure** → se macchine sono spesso in gabbie (*cages*) con serratura, per evitare che un tecnico che lavora su un server possa toccare quello accanto.

## 8.3. Gestione Energetica e Raffreddamento

L’efficienza energetica è la metrica di costo più importante: in un DC tradizionale circa **il 43% dell’energia totale** è utilizzato per il **raffreddamento**. Per ridurre questi costi si adottano diverse tecniche:

- **Free Cooling** → sfrutta l’aria esterna nei climi freddi, riducendo l’uso dei condizionatori industriali.
- **Ottimizzazione del PUE (Power Usage Effectiveness)** → il PUE misura il rapporto tra energia utilizzata dai server e quella totale consumata. Migliorare il PUE (avvicinarsi a 1.0) significa ridurre l’energia spesa in raffreddamento e servizi ausiliari, aumentando l’efficienza complessiva.

Per garantire invece continuità operativa:

- **Ridondanza Energetica** → obbligo di avere almeno **due fornitori elettrici indipendenti** e sistemi di backup (UPS e generatori). Questo garantisce continuità operativa anche in caso di guasti o blackout estesi.

---