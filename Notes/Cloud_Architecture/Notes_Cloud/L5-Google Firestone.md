# L5-Google Firestone

Number: 2
Status: Check phase
Type: lab

# 1. Opzioni di Storage di GCP

Google Cloud Platform offre diverse soluzioni di storage, classificabili in tre macro-categorie:

1. **SQL**: sono database tradizionali, adatti per dati strutturati.
    - CloudSQL → è un servizio di **database relazionale** che consente all’utente di scegliere il motore tra **MySQL, PostgreSQL e SQL Server**. Una volta selezionato il database, Google si occupa della sua gestione completa: hosting, esposizione in rete, aggiornamenti e patch di sicurezza. È la scelta standard per **applicazioni web, e-commerce e sistemi CRM**, che richiedono una struttura dei dati rigida e il supporto alle **transazioni ACID**.
    - Cloud Spanner → database relazionale pensato per scalabilità orizzontale globale. Garantisce un'affidabilità altissima (99.999%, meno di 5 minuti di downtime all'anno). È molto costoso e giustificato solo per applicazioni su scala globale che richiedono consistenza stretta.
    - AlloyDB → versione ottimizzata da Google di PostgreSQL per prestazioni superiori.
    - Bare Metal Solution for Oracle → hardware dedicato per ospitare database Oracle, utile per questioni di licenze e performance specifiche (l'utente gestisce il DB, Google l'infrastruttura).
2. **NoSQL**
    - Firestore → database **NoSQL document-based**. Memorizza i dati in **documenti strutturati in stile JSON** ed è **completamente serverless**, con scalabilità automatica da zero fino a milioni di utenti senza necessità di gestione dell’infrastruttura. È particolarmente indicato per **applicazioni mobile e web**, soprattutto quando servono elevata scalabilità e accessi in tempo reale.
    - Firebase →  è un database **NoSQL** molto simile a **Firestore**, ma oggi è considerato **in gran parte deprecato** a favore di quest’ultimo. Utilizza una struttura dati **JSON** e offre sincronizzazione in tempo reale, ma presenta **minore flessibilità, scalabilità e capacità di query** rispetto a Firestore, che è ormai la scelta consigliata per i nuovi progetti.
    - Google Cloud Partner Services (es. MongoDB Atlas) → è un servizio NoSQL gestito da terze parti ma ospitato sull’infrastruttura Google. Dal punto di vista ingegneristico è vantaggioso: un’app sviluppata su MongoDB Atlas può essere facilmente migrata su un’istanza MongoDB self-hosted senza modifiche sostanziali al codice.
    - Cloud Bigtable ****→ database NoSQL basato su key-value, progettato per gestire carichi massivi con throughput elevatissimo (milioni di letture/scritture al secondo) e latenze inferiori ai 10 ms. È la tecnologia alla base di servizi Google come Search e Maps.
    - Memorystore → database NoSQL **in-memory**, progettato per applicazioni dove la velocità è critica e i dati devono risiedere in RAM per ridurre i tempi di accesso. È tipico per **caching**, gestione delle sessioni utente o classifiche in tempo reale.
    - BigQuery → database **SQL analitico**, progettato per workload **OLAP**. È pensato per l’**analisi di Big Data**, permettendo di eseguire query complesse su grandi volumi di dati in modo scalabile e ad alte prestazioni.

---

# 2. Introduzione a Firestone

Database con le seguenti caratteristiche:

- **Fully managed**: ****Google si occupa di hosting, esposizione del servizio e aggiornamenti di sicurezza.
- **Serverless**: non è necessario gestire server; il sistema scala automaticamente in base al carico.
- **NoSQL, document-based**: i dati sono organizzati in **documenti** (stile JSON) raggruppati in collezioni. In pratica è un modello **chiave-valore**, dove il valore è un documento. All’interno di un documento è possibile salvare qualunque informazione, senza una struttura rigida predefinita.
- **Meccanismo di sincronizzazione e supporto online**
- **Security features integrate**: controllo degli accessi e regole di sicurezza definite a livello di database.
- **Real-time**:
    - nei database tradizionali non real-time il modello è **request/response (pull)**: l’applicazione richiede i dati, il server risponde e la connessione si chiude. Se i dati cambiano, il client non se ne accorge finché non effettua una nuova richiesta.
    - in un database **real-time** il modello è **event-based (push)**: l’app apre una connessione bidirezionale e si **sottoscrive** a un documento o a una collezione. Quando un dato cambia, il server **notifica immediatamente** tutti i client connessi, aggiornando l’interfaccia in tempo reale senza ricaricare la pagina.

<aside>
📌

**Modalità di Funzionamento:**

Firestone presenta due modalità di funzionamento:

- **Native mode (quella che utilizziamo noi)**: modalità che presenta un limite di scritture al secondo ma che permette funzionalità real-time. I dati sono strutturati in documenti.
- **Datastore mode**: modalità che non presenta un limite di scritture al secondo ma che non permette funzionalità real-time. I dati sono strutturati in **entità** (modello più vicino a un key-value classico).
</aside>

---

# 3. Struttura dei Dati: Collections

In Firestore i dati sono organizzati in **collection** e **documenti**. I documenti sono contenuti all’interno di una collection oppure di una sub-collection, e ciascun documento può a sua volta avere una o più sub-collection annidate.

Una collection non esiste finché non viene creato il primo documento al suo interno e viene eliminata automaticamente quando l’ultimo documento viene cancellato. All’interno di una stessa collection, i documenti sono identificati da un ID univoco; questo ID può essere scelto manualmente oppure generato automaticamente da Firestore in modo casuale.

Per quanto riguarda la struttura dei dati, Firestore non impone uno schema rigido: i documenti appartenenti alla stessa collection o sub-collection tendono ad avere gli stessi campi, ma non è un vincolo obbligatorio, permettendo così una grande flessibilità nella modellazione dei dati.

***Esempi:***

```python
# Riferimento a un documento "dvader" nella collection "sith"
d_vader_ref = db.collection(u'sith').document(u'dvader')

# Riferimento equivalente usando il path completo
d_vader_ref = db.collection(u'sith/dvader')

# Riferimento alla collection "sith"
siths_ref = db.collection(u'sith')
```

![image.png](L5-Google%20Firestone/image.png)

---

# 4. Pricing

Il **free tier** di Firestore include:

- **1 GB di storage**
- **50.000 reads**, **20.000 write** e **20.000 delete al giorno**

Oltre queste soglie, le operazioni vengono fatturate in base al numero di **read**, **write** e **delete** effettuate:

![image.png](L5-Google%20Firestone/image%201.png)

**N.B.** È molto facile superare il limite di **write**. Un caso tipico è durante il **deploy**: se ci si dimentica di inserire correttamente l’ambiente `.venv` nel file `.gcloudignore`, si può generare anche 50.000 write.

---

# 5. Configurazione e Utilizzo con Flask

1. **Setup Iniziale:**
    1. Aggiungere il pacchetto di firestone `google-cloud-firestore` nei `requirements.txt`.
    2. Abilitare il servizio su gcloud: `gcloud services enable firestore.googleapis.com` (oppure da console su gcloud).
    3. Creare credenziali per autenticazione: per utilizzare Firestore è necessario interfacciarsi con **IAM (Identity and Access Management)**, il servizio di sicurezza di Google Cloud che definisce chi (identità) può fare cosa (ruoli) su quali risorse. Prima di tutto bisogna configurare le credenziali su IAM e successivamente utilizzare queste credenziali nell’applicazione per autenticarsi e ottenere i permessi necessari per accedere al database.
        
        ```python
        export NAME=webuser 
        export PROJECT_ID=<...> 
        
        #Come prima cosa si crea un’identita’, ovvero un service-account su IAM
        #Un service-account e’ un account utilizzato dalle applicazioni per effettuare chiamate alle API di Google.
        gcloud iam service-accounts create ${NAME} 
        
        #Si assegna al service account appena creato (--member) il ruolo di owner (--role)
        gcloud projects add-iam-policy-binding ${PROJECT_ID} --member "serviceAccount:${NAME}@${PROJECT_ID}.iam.gserviceaccount.com" --role "roles/owner" 
        
        #Si genera una chiave crittografica privata (in formato JSON) che permette a chiunque la possieda di "impersonare" il service account.
        touch credentials.json 
        gcloud iam service-accounts keys create credentials.json --iam-account ${NAME}@${PROJECT_ID}.iam.gserviceaccount.com 
        
        #Si crea una variabile d’ambiente che indica la posizione di credentials.json
        export GOOGLE_APPLICATION_CREDENTIALS="$(pwd)/credentials.json"
        ```
        
        **N.B.** Il file `credentials.json` deve essere inserito sia nel `.gitignore` che nel `.gcloudignore`.
        
2. **Connessione al database Firestone:**
    
    ```python
    from google.cloud import firestore
    client = firestore.Client(database=’...’) #L’oggetto Client è il punto di ingresso per tutte le operazioni su Firestore
    ```
    
    <aside>
    📌
    
    **Cosa succede quando viene creato il Client?**
    
    Alla creazione del client, Firestore esegue automaticamente tre fasi fondamentali:
    
    1. **Identificazione del progetto** → il client cerca di determinare a quale progetto Google Cloud deve connettersi:
        - Su **App Engine**: il progetto viene rilevato automaticamente
        - In **locale**: legge la variabile `GOOGLE_CLOUD_PROJECT`, se presente, oppure ricava il project ID dal file `credentials.json`
    2. **Autenticazione (Handshake)** → il client cerca le credenziali per dimostrare di avere i permessi necessari, in locale utilizza il file JSON indicato da `GOOGLE_APPLICATION_CREDENTIALS` invece in cloud usa automaticamente il service account associato all’app.
    3. **Apertura del Pool di Connessioni** → prepara il canale di comunicazione per inviare query e ricevere documenti.
    </aside>
    

## 5.2. Operazioni CRUD (Create, Read, Update, Delete)

- **Scrittura dei dati:** esistono due metodi principali per creare o sovrascrivere documenti:
    1. **`set()`:** si usa quando si vuole definire manualmente l'ID del documento. Se il documento esiste, viene sovrascritto; se non esiste, viene creato.
        
        ```python
        # Crea un documento con ID 'varenne' nella collezione 'horses'
        doc_ref = db.collection('horses').document('varenne')
        doc_ref.set({'name': 'Varenne', 'breed': 'Trotter'})
        # Prima creo il riferimento, poi effettivamente scrivo qualcosa nel database quando chiamo set()
        ```
        
    2. **`add()`:** si usa quando non serve un ID specifico. Firestore genera automaticamente un ID univoco casuale.
        
        ```python
        db.collection('horses').add({'name': 'Furia', 'breed': 'Mustang'})
        ```
        
- **Lettura dei dati:**
    - Leggere un singolo documento
        
        ```python
        doc = db.collection('horses').document('varenne').get()
        if doc.exists:
            print(doc.to_dict())
        ```
        
    - Leggere una collezione (`stream`):
        
        ```python
        def print_stream(docs): 
        	for doc in docs: 
        		print(f'{doc.id} => {doc.to_dict()}') 
        ```
        
- **Aggiornare un documento:**
    
    ```python
    db.collection('horses').document('varenne').update({'wins': 50})
    ```
    
- **Cancellazione:**
    
    ```python
    db.collection('horses').document('varenne').delete() #Rimuovere un elemento specificato
    ```
    

## 5.3. Query Avanzate

- **Per filtrare documenti con attributi specifici:**
    
    ```python
    docs = db.collection('horses').where('breed', '==', 'trotter').stream()
    ```
    
- **Ordinare i risultati:** `order_by(field)` ****(è possibile fare anche doppio ordinamento `order_by(field_1).order_by(field_2)`).
- **Limitare il numero di documenti restituiti:** `limit(n)`.
- **Per controllare o scaricare un documento specifico:** `.get()`, `.exists`.
    
    ```python
    user_ref = db.collection("horses").document("varenne") 
    doc_snapshot = user_ref.get() 
    if doc_snapshot.exists:
    ```
    

---

# 6. Funzioni tipiche da esame

```python
import json
from google.cloud import firestore

class Horses:
    def __init__(self):
        # Crea il client Firestore
        self.db = firestore.Client()
        # Popola il database da file JSON
        self.populate_db('horses.json')

    def populate_db(self, filename):
        """Popola la collection 'horses' con i dati del file JSON"""
        horses_ref = self.db.collection('horses')
        with open(filename, 'r') as f:
            horses = json.load(f)

        for h in horses:
            # Ogni documento è identificato dal nome del cavallo
            horses_ref.document(h['name']).set(h)

    def get_horse(self, name):
        """Ritorna il documento di un cavallo dato il suo nome"""
        name = name.upper()
        if not name:
            return None

        doc = self.db.collection('horses').document(name).get()
        return doc.to_dict() if doc.exists else None
```

---