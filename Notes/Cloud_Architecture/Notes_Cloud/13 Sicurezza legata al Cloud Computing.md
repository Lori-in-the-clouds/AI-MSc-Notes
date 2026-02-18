# 13. Sicurezza legata al Cloud Computing

Number: 1
Status: Done
Type: theory

# 1. Definizione di Sicurezza Informatica

Un sistema è definito **sicuro** se garantisce tre proprietà fondamentali dei dati (**CIA**):

- **Confidentiality** → le informazioni sono accessibili solo a soggetti autorizzati. L'uso del cloud implica il trasferimento di dati a un provider, ma se fatto intenzionalmente e contrattualmente regolato, non costituisce una violazione della confidenzialità.
- **Integrity** → i dati non subiscono modifiche non autorizzate e rimangono corretti. È possibile avere una violazione della confidenzialità senza perdere l'integrità (es. qualcuno legge i dati senza toccarli).
- **Availability** → dati e servizi sono disponibili quando richiesti dagli utenti legittimi. A differenza delle altre proprietà, una violazione della disponibilità è spesso temporanea e reversibile senza danni al dato stesso.

Esistono garanzie addizionali fondamentali per le **comunicazioni**:

- **Autenticazione** → certezza dell'identità della controparte nella comunicazione.
- **Non ripudio** → impossibilità per un soggetto di negare di aver compiuto un'azione, poiché esiste una prova (evidenza) verificabile da terze parti.

---

# 2. Tipologie di Attaccanti

La sicurezza è necessaria a causa dell'esistenza degli attaccanti. Essi si dividono in categorie basate su competenze, risorse e obiettivi:

- **Ricreativi:** spesso studenti o principianti che usano strumenti banali (script) per vandalismo o curiosità. Facilmente bloccabili con misure base.
- **Criminali:** la maggior parte degli attacchi. Sono attori **razionali** motivati economicamente. Valutano il ROI (Return on Investment): attaccano se il costo dell'attacco è inferiore al guadagno.
    
    *Obbiettivo della difesa:*  alzare l'asticella dei costi per l'attaccante in modo da rendere l'attacco non conveniente.
    
- **APT (Advanced Persistent Threats):** gruppi spesso finanziati da stati, dotati di risorse, tempo e competenze quasi illimitate. Mirano a obiettivi specifici di alto valore (spionaggio, sabotaggio). Estremamente difficili da fermare.

<aside>
📌

**Obbiettivo degli Attaccanti:**

- Denaro diretto (es. ransomware, furto credenziali bancarie).
- Dati monetizzabili (proprietà intellettuale, dati sanitari).
- Potenza di calcolo (per mining di criptovalute o botnet).
- Danni fisici o interruzione servizi (attacchi a sistemi cyber-fisici o industriali)
</aside>

![image.png](13%20Sicurezza%20legata%20al%20Cloud%20Computing/image.png)

---

# 3. Vulnerabilità dei Sistemi

Un sistema informativo è composto da Hardware, Software e Persone. Tutte le componenti sono vulnerabili:

1. **Hardware** → attacchi fisici (es. accesso ai bus), rari perché richiedono presenza in loco.
2. **Software** → principale vettore di attacco remoto. Include bug, mancato patching (software obsoleto) o configurazioni errate (es. password di default, porte esposte).
3. **Persone** → spesso l'anello debole. Suscettibili a ingegneria sociale (phishing).
4. **Management** → una cattiva gestione (mancanza di policy, assenza di inventario degli asset) impedisce ai tecnici di proteggere efficacemente l'infrastruttura.

---

# 4. Cloud e Shared Responsibility

Nel cloud, la sicurezza non è delegata interamente al provider. Si applica il principio della responsabilità condivisa. Si distingue tra:

- **Security of the Cloud:** responsabilità del Provider (infrastruttura, hardware, data center, rete fisica)
- **Security in the Cloud:** responsabilità del Cliente (dati, configurazioni, accessi)

Le responsabilità variano in base al modello di servizio:

- **IaaS (Infrastructure as a Service)** → il provider è responsabile della sicurezza dell’infrastruttura di base, che include la protezione fisica dei data center, l’alimentazione, il raffreddamento e la gestione dell’hypervisor.
    
    La **maggiore responsabilità** ricade però sul cliente, che deve occuparsi della gestione e dell’aggiornamento del sistema operativo guest (patching), della configurazione della rete virtuale e dei firewall (security group), dell’installazione e della sicurezza delle applicazioni, della cifratura dei dati sia a riposo sia in transito e della gestione degli accessi e delle identità (IAM).
    
    ![image.png](13%20Sicurezza%20legata%20al%20Cloud%20Computing/image%201.png)
    
- **PaaS (Platform as a Service)** → il provider si occupa della gestione dell’hardware, del sistema operativo e del runtime applicativo, inclusi gli aggiornamenti e la manutenzione del software di base. Il cliente, invece, è responsabile dello sviluppo e del corretto funzionamento dell’applicazione, della gestione dei dati e della definizione delle policy di accesso.
    
    ![image.png](13%20Sicurezza%20legata%20al%20Cloud%20Computing/image%202.png)
    
- **SaaS (Software as a Service)** → il provider gestisce l’intero stack tecnologico, comprese le applicazioni, il sistema operativo e la rete. Il cliente resta comunque responsabile della gestione dei dati (ad esempio evitando di inviare informazioni sensibili ai destinatari sbagliati), del controllo delle identità e degli accessi (password forti, MFA, revoca account) e della formazione del personale per la sicurezza informatica, come la prevenzione del phishing.
    
    ![image.png](13%20Sicurezza%20legata%20al%20Cloud%20Computing/image%203.png)
    
    **N.B.** Delegare al cloud provider spesso migliora la sicurezza (es. disaster recovery, sicurezza fisica) rispetto a soluzioni on-premise "fatte in casa", specialmente per piccole realtà.
    

---

# 5. Security Controls e Cloud Control Matrix

Per implementare la sicurezza, si utilizzano dei *Security Controls*. Questi possono essere:

1. **Inherited (Ereditati)** → controlli gestiti interamente dal provider (es. guardie armate al data center).
2. **Customer Specific** → controlli che il cliente deve configurare in base alle proprie esigenze (es. scegliere una Region europea per compliance GDPR).
3. **Shared (Condivisi)** → controlli che richiedono azioni da entrambe le parti (es. *Patch Management*: provider patcha l'infrastruttura, cliente patcha l'OS guest).

<aside>
📌

**Cloud Control Matrix**

È uno strumento fornito dalla *Cloud Security Alliance (CSA)* che aiuta a identificare gli aspetti di sicurezza da considerare nello sviluppo di applicazioni web nel cloud. La matrice mette in relazione gli standard di sicurezza (ISO 27001, NIST) con i modelli cloud e per ciascun controllo indica se la responsabilità ricade sul provider del servizio cloud (CSP), sul cliente (CSC) o è condivisa. In questo modo consente di capire chiaramente quali responsabilità restano all’azienda quando migra al cloud.

</aside>

---

# 6. Strumenti di Sicurezza e CASB

Nel cloud, gli strumenti di sicurezza cambiano: non si installano appliance fisiche (firewall fisici), ma si configurano servizi software equivalenti (es. Security Groups, WAF virtuali).

<aside>
✅

**Vantaggi del Cloud: DDoS Protection**

I grandi provider offrono una capacità di banda e ridondanza ineguagliabile per mitigare attacchi volumetrici di Denial of Service.

</aside>

## 6.1. CASB (Cloud Access Security Broker)

È una tecnologia specifica per la sicurezza cloud. Funge da intermediario tra gli utenti (ovunque si trovino) e i servizi cloud, garantendo visibilità e controllo. Funzionalità principali:

- Monitoraggio accessi e rilevamento anomalie (es. login sospetti)
- Data Loss Prevention (DLP)
- Compliance

<aside>
📌

**Modalità di Funzionamento**

- **Proxy (Forward o Reverse):** il traffico passa attraverso il CASB che lo filtra in tempo reale. Meno usato oggi per problemi di compatibilità.
    - Il **forward proxy →** il CASB agisce "per conto dell'utente". Si interpone tra l'utente e *tutto* internet.
    - Il **reverse proxy** → il ****CASB agisce "a protezione dell'applicazione". Si interpone proprio davanti all'ingresso di quel servizio specifico. Serve a proteggere **l'ingresso** all'applicazione autorizzata.
- **API Mode (Approccio moderno):** il client parla direttamente col cloud. Il CASB si collega alle API del cloud provider (es. Google, AWS) per scaricare log, monitorare attività e agire in modo asincrono (es. disabilitare un utente compromesso).

![image.png](13%20Sicurezza%20legata%20al%20Cloud%20Computing/image%204.png)

</aside>

---