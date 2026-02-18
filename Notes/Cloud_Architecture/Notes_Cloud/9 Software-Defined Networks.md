# 9. Software-Defined Networks

Number: 1
Status: Done
Type: theory

# 1. Le Sfide del Networking Tradizionale nel Cloud

L'approccio SDN è nato per risolvere le sfide che il *networking* tradizionale non riesce a gestire in scenari *cloud* dinamici:

- **Resource Sharing:** il traffico nei *data center* è "bursty" (altamente variabile). L'infrastruttura tradizionale, dimensionata sui picchi di domanda, porta a un enorme spreco di risorse sottoutilizzate.
- **Performance Isolation:** nei *data center multi-tenant* (che ospitano clienti diversi) è essenziale impedire che i flussi di un cliente interferiscano con quelli di un altro (es. il traffico YouTube non deve rallentare le ricerche Google). Le soluzioni tradizionali (VLAN, *traffic shaping*) sono complesse e non si integrano bene.
- **Redundancy vs Load Balancing:** per la ridondanza si usa lo **Spanning Tree Protocol (STP)**, che crea un albero. Questo però **impedisce il *load balancing*** su più *link* in parallelo (che creerebbero anelli, mandando in crisi gli algoritmi di *routing*). Si creano obiettivi in conflitto.
- **Problema della Gestione:** la rete moderna richiede molteplici "middle-boxes" (apparati specializzati) come Firewall, NAT, *Intrusion Detection Systems* (IDS) e *load balancer. Questi introducono grandi problemi di gestione:*
    - **API Eterogenee** → ognuno di questi apparati è spesso un *appliance* separato, magari di *vendor* diversi, ognuno con la propria API.
    - **Migrazione delle VM** → quando una VM migra da un *host* fisico a un altro, è necessario **riconfigurare simultaneamente *tutti*** questi sistemi: le regole di *routing*, le regole del *firewall*, il NAT, il *traffic shaping* e avvertire l'IDS che il nuovo traffico non è un attacco.
    - **Vendor Lock-in** → questo costringe le aziende a scegliere tra gestire manualmente una decina di API diverse (rischiando il caos) o acquistare soluzioni chiuse "chiavi in mano" da un unico *vendor*, che sono molto costose e creano un forte *vendor lock-in*.
- **Instabilità:** i protocolli tradizionali come il **BGP**, usati per il routing tra data center, sono **intrinsecamente instabili**. Essendo basati su un *distance-vector*, la convergenza finale della rete può dipendere dall'**ordine** in cui arrivano i messaggi di update. Dopo un crash, la rete può convergere a uno stato **diverso e imprevedibile** ("management nightmare").

---

# 2. Soluzioni di Virtualizzazione della Rete

## 2.1. Il Paradigma “Software Defined”

Seguendo un approccio *bottom-up* (virtualizzazione, storage), l'ultimo *building block* del *data center* cloud è la gestione delle reti. Anche in questo caso, il paradigma si sposta da un sistema basato su dispositivi fisici a uno **"Software Defined"**, che predilige flessibilità e programmabilità.

Per la virtualizzazione della rete esistono due approcci concettualmente diversi, anche se spesso usano le stesse tecnologie: 

- **Network Function Virtualization (NFV)**
- **Software Defined Networking (SDN)**

La differenza risiede negli obiettivi e nei requisiti che si vogliono soddisfare.

---

# 2. Network Function Virtualization (NFV)

L'approccio NFV nasce principalmente negli ambienti degli **ISP (Internet Service Provider)** e degli *Hyperscaler* (Google, AWS, Meta), che gestiscono *autonomous system* enormi. L'obiettivo primario di NFV è la **riduzione dei costi**, sia **CAPEX** che **OPEX**:

- **CAPEX** (=costi in conto capitale) → l'investimento in hardware di rete (router, switch) è massiccio.
- **OPEX** (=costi operativi) → la manutenzione è complessa. I *Network Operation Center (NOC)* devono essere gestiti 24/7 da personale altamente qualificato per fronteggiare scenari dinamici (DDoS, *burst* di richieste).

<aside>
📌

**Problema: Hardware Verticale**

L'hardware tradizionale è **"super verticale"** (proprietario, es. Cisco), costoso, poco flessibile e introduce un forte **vendor lock-in**. Inoltre, costringe all'acquisto di apparati diversi per funzioni simili (switch L2, router L3, firewall), che in realtà sono tutti casi particolari di gestione dei pacchetti. 

</aside>

<aside>
✅

**Soluzione (Separazione HW/SW)**

L'idea di NFV è **separare le funzioni di rete dall'hardware specializzato**. Si mantiene l'hardware dedicato solo per le funzioni critiche (come lo *switching* ad alta velocità), spostando la logica di controllo (es. BGP) su **hardware generico COTS (Commodity Off-The-Shelf)**, che è flessibile ed economico. Questo abilita la **generalizzazione delle funzioni.**

</aside>

---

# 3. Software Defined Networking (SDN)

L'SDN, a differenza di NFV, sposta il suo **focus** sulla necessità di garantire un livello elevato di **configurabilità** e **gestibilità** all'interno del *data center* cloud.

## 3.1. Separazione dei Piani nell’SDN

<aside>
🚨

**Problema della Dinamicità**

Nello scenario *cloud*, le policy di rete (NAT, *firewall*, *routing*) devono essere gestite in modo **dinamico**. Ad esempio, la migrazione di una VM richiede che tutte le sue regole di rete vengano istantaneamente riconfigurate sul nuovo *host*. L'SDN deve integrarsi con l'**Infrastructure as Code (IaC).**

</aside>

<aside>
✅

**Soluzione (Separazione Logica/Azione)**

L'SDN non si limita a virtualizzare gli apparati (come NFV), ma **separa i concetti di configurazione e funzione**. Questo disaccoppiamento è realizzato attraverso un'**interfaccia comune** ("lingua franca" → come OpenFlow) tra i due piani:

- **Control Plane** → decide **quali regole** applicare (la logica di configurazione). È costituito da **Controller**, *layer* software che virtualizza i dettagli sottostanti e offre un unico punto di gestione. Può gestire sia *switch* hardware sia *switch* software (es. **Open vSwitch (OVS)** o **Mininet**). I controller utilizzano due **interfacce** principali:
    - **Northbound Interface (verso l'alto):** è l'interfaccia usata dal *Controller* per comunicare con le applicazioni, gli *hypervisor* (es. OpenStack) e i sistemi IaC. È spesso implementata tramite **API RESTful**. La sua natura bidirezionale le permette sia di **inviare *policy*** e configurazioni, sia di **leggere lo stato** della rete per il *monitoring* e il *billing* .
    - **Southbound Interface (verso il basso):** è l'interfaccia che collega il *Controller* ai dispositivi fisici del *Data Plane* (switch/router). Utilizza protocolli standardizzati come **OpenFlow** (ma anche SNMP/BGP per apparati *legacy*). È bidirezionale: il *Controller* la usa per **inviare comandi** (configurare i dispositivi), e i dispositivi la usano per **segnalare anomalie** al *Controller* (es. inviando un pacchetto sconosciuto che il *Data Plane* non sa come gestire).
- **Data Plane** → È composto dai dispositivi di rete (switch, router) che **eseguono** le azioni in base alle regole (IP Forwarding).

![image.png](9%20Software-Defined%20Networks/image.png)

L’analogia classica è quella del **routing**: l’**IP forwarding** appartiene al **Data Plane** ed è un’operazione meccanica e veloce, che consiste nell’inoltro dei pacchetti. Il calcolo del percorso migliore, invece, eseguito da protocolli come **OSPF** o **BGP**, appartiene al **Control Plane** ed è la parte più complessa e lenta del processo.

</aside>

<aside>
📌

**Esempio di Controller: OpenDayLight**

Un esempio di *controller* è **OpenDaylight**. La sua architettura (come quella di altri *controller* come ONOS o Riu) conferma questa struttura, presentando:

- Una **Northbound Interface** basata su **API RESTful** per integrarsi con applicazioni di alto livello (es. OpenStack, Management GUI).
- Una **Southbound Interface** basata su *plugin* per pilotare molteplici protocolli (come OpenFlow, BGP, OVSDB, SNMP).
- Una **Logica Centrale** (Service Abstraction Layer e Base Network Service Functions) che tiene traccia dello stato della rete (topologia, statistiche, *host*) e implementa le *policy*.

![image.png](9%20Software-Defined%20Networks/image%201.png)

</aside>

## 3.2. OpenFlow

**OpenFlow** è il protocollo *southbound* più diffuso e generalizza il concetto di tabella di *routing*. Invece di fare *match* solo sull’IP di destinazione, ogni *switch* OpenFlow usa una **flow table** molto più flessibile. Ogni **flow entry** contiene:

1. **Match →** i campi su cui filtrare il pacchetto. La potenza di OpenFlow è che il predicato di *match* è **cross-layer** e può ispezionare (a seconda della versione, da 12 a 41 campi): 
    - Livello 2 (H2N): porta di ingresso, ID VLAN, MAC src/dst
    - Livello 3 (Network): IP src/dst, IP Protocol, ToS (per QoS)
    - Livello 4 (Transport): porta TCP/UDP src/dst
    
    Questa flessibilità permette a un singolo apparato hardware di agire come **Switch** (match su L2), **Router** (match su L3) o **Firewall** (match su L4 e azione "Drop").
    
2. **Azione** → definiscono cosa fare dopo un *match:*
    - Modify (Mangling): modifica i campi del pacchetto. È usato per implementare **NAT** (riscrittura IP/porta), **VLAN tagging** o *Load Balancing* .
    - Send to Controller: se un pacchetto non corrisponde a nessuna regola (es. il primo pacchetto di una nuova connessione TCP), viene inviato al *Controller*. Il *Controller* lo analizza (sul *slow path*) e **inietta una nuova regola** nello *switch* (sul *fast path*) per gestire tutti i pacchetti successivi di quel flusso.
    - Table Management: inoltra il pacchetto a una tabella secondaria, spesso usata per regole meno frequenti o implementate via software.
3. **Contatori** → statistiche su pacchetti/byte processati.
4. **Priorità** → l'ordine di valutazione delle regole.
5. **Timeout** → per rimuovere regole inattive (evitando di saturare la tabella).

![image.png](9%20Software-Defined%20Networks/image%202.png)

<aside>
🚨

**Criticità di OpenFlow**

L'hardware degli *switch* (TCAM) ha limiti fisici:

- **Dimensione delle Tabelle:** le tabelle hanno un numero di regole limitato.
- **Emulazione Software:** se si supera la dimensione della tabella, o se si usano predicati di *match* troppo complessi (es. funzioni di OpenFlow 1.3 su hardware 1.0), lo *switch* entra in modalità di **emulazione software**. Uno *switch* progettato per 1 Gbps in hardware può crollare a **20 Mbps** in emulazione software.
</aside>

## 3.3. Criticità SDN

L'SDN, sebbene potente, presenta ancora delle criticità:

- **Latenza (Switch/Controller):** l’interazione *Switch–Controller* è lenta, ogni nuovo flusso che non ha ancora una regola di *match* deve essere inviato al controller (*slow path*), introducendo latenza significativa. Questo diventa problematico per carichi con molti flussi brevi (*short flows*). Un esempio è il *3-way handshake* TCP: il primo pacchetto **SYN** viene intercettato dallo switch e inviato al controller, che deve analizzarlo e installare una regola prima che la connessione possa proseguire con **SYN-ACK** e **ACK**.
- **Scalabilità (Controller):** il *controller* centralizzato è un *bottleneck* computazionale. La soluzione è partizionare il grafo della rete e assegnare diverse porzioni a *controller* federati.
- **Affidabilità (Controller):** il *controller* è un *Single Point of Failure*. La soluzione è usare *controller* ridondanti (un ***hot spare*** o *slave*) pronti a subentrare (fare *takeover*) in microsecondi, mantenendo i dati sincronizzati, in caso di *failure* del *master*.

---