# L4-Google Cloud Platform e Google App Engine

Number: 4
Status: Check phase
Type: lab

# 1. Google Cloud Platform (GCP)

Google Cloud Platform offre diversi livelli di astrazione per la gestione delle risorse:

- **IaaS (Infrastructure as a Service)** → l'utente gestisce quasi tutto (OS, middleware, runtime), tranne l'hardware fisico. Esempi:
    - Google Compute Engine: macchine virtuali (VM) pure.
    - Google Kubernetes Engine (GKE): orchestrazione di container (pod) basata su Kubernetes.
- **PaaS (Platform as a Service)** → l'utente gestisce solo l'applicazione e i dati. Il cloud provider gestisce OS, networking, aggiornamenti e sicurezza. Esempio:
    - Google App Engine (GAE): piattaforma per hostare applicazioni web scalabili senza gestire l'infrastruttura sottostante.
- **SaaS (Software as a Service)** → l'utente utilizza solo il software finale (es. Gmail, Google Drive, Firebase per l'autenticazione).

## 1.1. Pricing (Free-Tier)

Sono disponibili diversi *free tier*; quelli di nostro interesse riguardano **App Engine** e **Firestore**:

- **App Engine** → il livello gratuito include fino:
    - **28 ore al giorno di istanze F**
    - **9 ore al giorno di istanze B**
    
    Il tempo viene calcolato come *tempo totale di esecuzione delle richieste*: ad esempio, se una macchina gestisce 3 richieste da 10 minuti ciascuna, il consumo complessivo è di 30 minuti. Inoltre, sono inclusi **1 GB al giorno di traffico in uscita (egress)**, cioè dati inviati verso l’esterno.
    
- **Firestore** → il free tier prevede **1 GB di storage per progetto** e un limite giornaliero di **50.000 letture**, **20.000 scritture** e **20.000 cancellazioni**, sempre per progetto.

## 1.2. Networking e Infrastruttura Ibrida

Quando ci si sposta sul cloud, la gestione della rete diventa critica (spesso causa di disservizi globali legati a DNS o load balancing). GCP offre strumenti per scenari ibridi (connessione tra on-premise e cloud):

- **Virtual Private Cloud (VPC):** per definire reti virtuali private, regole firewall e indirizzi IP.
- **Cloud Interconnect:** collegamento fisico diretto per prestazioni elevate tra l'infrastruttura locale e Google

## 1.3. Tool per interagire con GCP

Per interagire con **GCP** sono disponibili due strumenti principali:

- **Cloud Console** →  l’applicazione web di Google che permette di gestire risorse, servizi e configurazioni tramite interfaccia grafica.
- **Command Line Interface (CLI)** → uno strumento da installare in locale che consente di gestire GCP da terminale in modo più rapido e automatizzabile.
    
    La sintassi generale dei comandi della CLI è: `gcloud GROUP COMMAND [OPTIONS]` .
    

---

# 2. Google App Engine (GAE)

Google App Engine è un ambiente PaaS che permette di concentrarsi sullo sviluppo del codice (es. Python, Java), delegando a Google la gestione di server, scaling e bilanciamento del carico.

## 2.1. Struttura GAE

L'architettura di un'applicazione su GAE è strutturata gerarchicamente:

1. **Application** → il contenitore logico principale (un progetto GCP può avere una sola App Engine application).
2. **Services (Microservizi)** → l'applicazione viene suddivisa in componenti logici indipendenti (es. frontend, backend, autenticazione).
3. **Versions** → ogni servizio può avere più versioni deployate contemporaneamente per gestire traffico, rollback o A/B testing.
4. **Instances** → le copie effettive del codice in esecuzione che gestiscono il traffico

![image.png](L4-Google%20Cloud%20Platform%20e%20Google%20App%20Engine/image.png)

## 2.2. Free Tier

Il free tier di Google App engine include:

- 28 frontend instance hours per day
- 9 backend instance hours per day
- 5 GB Cloud Storage
- 1 GB of egress per day
- Shared memcache
- 1000 search operations per day
- 10 MB of search indexing
- 100 email messages per day

---

# 3. Sviluppo Backend con Flask

Per il laboratorio viene utilizzato **Flask**, un framework Python "lightweight" (leggero), scelto per la sua flessibilità e idoneità ai microservizi su GAE. Di default fornisce solo due funzionalità fondamentali:

- **Request routing** → la capacità di associare un URL a una funzione Python.
- **Templating** → la possibilità di inserire dati dinamici all’interno di template HTML. Il templating è supportato dal motore **Jinja2**, che consente, ad esempio, di scrivere:
    
    ```html
    <h1>Ciao {{ nome_utente }}!</h1>
    ```
    

Tutte le funzionalità aggiuntive devono essere **incluse manualmente o implementate dall’utente**. A differenza di framework più completi come **Django**, Flask:

- non ha un modello di database integrato
- non fornisce un sistema di mapping automatico tra modelli Python e tabelle SQL
- non include una dashboard di amministrazione
- non offre un sistema di autenticazione predefinito

Con Flask, per accedere al database è necessario scrivere direttamente codice SQL oppure utilizzare librerie esterne (come ORM) che replicano funzionalità simili. Il principale vantaggio di Flask è la **massima leggerezza e flessibilità**: lo sviluppatore ha il controllo totale su quali componenti utilizzare, senza vincoli imposti dal framework.

## 3.1. Struttura del Progetto

Una corretta organizzazione delle cartelle è fondamentale per il funzionamento e la sicurezza dell'app:

- **/templates:** contiene i file HTML dinamici gestiti da Jinja2.
    
    <aside>
    🔑
    
    **Jinja2:** template engine usato da Flask per generare **HTML dinamico:**
    
    - Per inserire un placeholder da sostituire (dal backend): `{{variable}}`
    - Per generare contenuti ripetuti: `{% for x in %} ... {% endfor %}`
    - Per mostrare contenuti in modo condizionale: `{% if %} ... {% else %} ... {% endif %}`
    </aside>
    
- **/static**: raccoglie i file statici (immagini, CSS, JavaScript). È l’unica cartella accessibile direttamente dall’utente; le altre restano protette per motivi di sicurezza. I template non sono inclusi perché vengono renderizzati dinamicamente.
- **/venv**: ambiente virtuale per isolare le dipendenze Python del progetto. Al momento della creazione, un venv contiene solo 3 componenti principali:
    - **copia (o un link simbolico) dell’interprete Python**, in modo che il comando python utilizzi quella versione isolata e non quella globale.
    - **pip**, per installare i pacchetti.
    - **setuptools**, librerie di supporto utilizzate da pip per installare e compilare correttamente i pacchetti.
    
    <aside>
    📌
    
    **Comandi Principali**
    
    - Creazione dell’ambiente virtuale**:** `python3 -m venv <directory>`  (la directory si chiama solitamente `.venv` o `venv`)
    - Attivazione dell’ambiente**:** `source <directory>/bin/activate`
    - Installazione dei pacchetti: ****`pip install <pacchetto>`
    - Salvataggio delle dipendenze: ****`pip freeze > requirements.txt`
    </aside>
    
- **main.py:** file principale dell’applicazione. Per GAE è fondamentale che questo file si chiami esattamente `main.py`, altrimenti il sistema non riuscirebbe a individuare automaticamente l'applicazione Flask da avviare.
- **requirements.txt:** elenca tutte le dipendenze necessarie al progetto ed è usato da GAE durante il deploy.
    
    **N.B.** È buona pratica "fissare" le versioni dei pacchetti (es. `Flask==2.0.1` invece di `Flask`). Questo evita che un aggiornamento futuro di una libreria rompa la compatibilità con il tuo codice.
    

## 3.2. Scrittura dell’app

- **Inizializzazione** → per inizializzare un'applicazione Flask si utilizza l'oggetto `Flask`:
    
    ```python
    from flask import Flask
    app = Flask(__name__)
    
    if __name__ == '__main__':
        app.run(host='127.0.0.1', port=8080, debug=True)
    ```
    
    - **`app = Flask(__name__)`**: crea l'oggetto che configura il server web, gestendo route, database e configurazioni.
    - `__name__` è una variabile speciale di Python che rappresenta il nome del modulo corrente; serve a Flask per capire dove cercare i file del progetto (come template e statici).
    - **`app.run(...)`**: avvia il server di sviluppo locale. I parametri specificano l'host (localhost), la porta (8080) e abilitano la modalità debug.
    
    Durante l'inizializzazione è possibile specificare esplicitamente i **percorsi per i file statici**:
    
    ```python
    app = Flask(__name__, static_url_path='/static', static_folder='static')
    ```
    
    - **`static_url_path='/static'`**: è il prefisso URL che il client/browser deve usare per richiedere la risorsa (es. `http://sito/static/style.css`).
    - **`static_folder='static'`**: è il nome fisico della cartella nel progetto dove risiedono i file.
    
    Esempio di collegamento in HTML:
    
    ```html
    <html>
     <link rel="stylesheet" href="/static/hello.css">
     <body>...</body>
    </html>
    ```
    
- **Associare richiesta-funzione (routing)** → per gestire le richieste web, si usano i **decoratori** `@app.route`. Un decoratore "avvolge" una funzione Python, attivandola solo quando viene chiamato uno specifico URL.
    - Route base:
        
        ```python
        @app.route('/', methods=['GET'])
        def home():
            return "Homepage"
        ```
        
    - Parametri dinamici: è possibile catturare parti dell'URL (es. `<name>`) e passarle alla funzione:
        
        ```python
        @app.route('/greet/<name>')
        def greet(name):
            return f"Ciao {name}"
        #Funzione che si adatta automaticamente a qualsiasi nome venga inserito nell'URL.
        ```
        
- **Templating** → esistono due modi per renderizzare template HTML in Flask:
    - Usare un template definito in una stringa **(**`render_template_string`**):** utile per test rapidi, definisce l'HTML direttamente nel codice Python.
        
        ```python
        from flask import render_template_string
        template = '''<html><body><h1>Hello {{recipient}}!</h1></body></html>'''
        
        @app.route('/', methods=['GET'])
        def hello():
            return render_template_string(template, recipient='World')
        ```
        
    - Usare un template definito in un file .html nella cartella /templates ****(`render_template`): è il metodo standard. Flask cerca un file `.html` all'interno della cartella `templates`.
        
        ```python
        from flask import render_template
        
        @app.route('/', methods=['GET'])
        def hello():
            return render_template('hello.html', recipient='World')
        ```
        
- **Gestione della Richiesta (Request Payload)** → per accedere ai dati della richiesta HTTP in arrivo (es. parametri, JSON, path), si utilizza l'oggetto globale `request`. Esempio di utilizzo:
    
    ```python
    from flask import request
    
    # Recuperare il payload JSON (utile per metodi POST/PUT)
    data = request.get_json()
    
    # Leggere il percorso richiesto
    request_path = request.path
    ```
    
    **N.B.** Nelle richieste `GET`, il payload è solitamente vuoto, ma l'oggetto `request` è utile per leggere parametri di query o header.
    
- **Gestione degli Errori** → è possibile definire pagine di errore personalizzate intercettando specifici codici HTTP (come il 404 Not Found) tramite il decoratore `@app.errorhandler` .
    
    ```python
    @app.errorhandler(404)
    def page_not_found(e):
        # Restituisce il template personalizzato e il codice di stato 404
        return render_template('404.html', path=request.path), 404
    ```
    

---

# 4. Integrazione: Flask su Google App Engine

Capire come il framework (Flask) si aggancia alla piattaforma (GAE) è fondamentale per il deploy. I due livelli lavorano in sinergia attraverso specifici file di configurazione e comandi della CLI. Il collegamento tra il codice Python e l'infrastruttura Google avviene in quattro punti chiave:

- **Entry Point (`main.py`):**
    - **Flask:** definisce l'oggetto `app`.
    - **GAE:** cerca specificamente questo oggetto nel file `main.py` per avviare l'istanza. Se il file ha un altro nome o l'oggetto non si chiama `app` (e non è specificato diversamente), il deploy fallisce.
- **Routing:**
    - **GAE:** riceve tutto il traffico in ingresso. Tramite le regole in `app.yaml` (in particolare `script: auto`), decide quali richieste inoltrare all'applicazione Python.
    - **Flask:** riceve la richiesta inoltrata da GAE e usa i suoi decoratori `@app.route` per eseguire la logica specifica.
- **Gestione File Statici:** sebbene Flask possa servire file statici, su **Google App Engine** è buona pratica configurarli in app.yaml tramite static_dir affinché vengano serviti direttamente dall’infrastruttura Google. In questo modo le richieste ai contenuti statici **bypassano l’istanza Python**, riducendo l’uso di risorse di calcolo e risparmiando **ore di istanza (F1)**, che restano dedicate esclusivamente alle richieste dinamiche.
- **Dipendenze (`requirements.txt`):** google legge questo file durante il build per installare l'ambiente Python sui suoi server, replicando l'ambiente creato in locale con `venv`.

## 4.1. Configurazione file `.yaml`

Il file `app.yaml` è il cuore della configurazione del deploy. Definisce le risorse, il routing e il comportamento dell'ambiente. 

```yaml
runtime: python313
#(Mia aggiunta di sicurezza)
instance_class: F1
automatic_scaling:	
  max_instances: 1

handlers:			#definisce una serie di regole di routing
	- url: /static
		static_dir: static

	- url: /.*
		secure: always 	
		script: auto
#La richiesta per qualsiasi tipo di url (tutti quelli non definiti sopra), viene mappata qui
```

Ecco le chiavi principali:

- **`runtime`** → specifica la versione di Python della VM (es. `python313`).
- **`instance_class`** → definisce la dimensione della VM. `F1` è la più piccola ed economica.
- **`automatic_scaling`** → controlla i limiti di scalabilità. Impostare `max_instances: 1` è utile per evitare costi imprevisti durante i test.
- **`entrypoint`**: (Opzionale) Indica la porta su cui l'applicazione Flask è in ascolto (es. `entrypoint:8080`).
- **`env_variables`**: Definisce variabili d'ambiente accessibili nel codice (tramite `os.environ.get`). Utile per distinguere configurazioni di sviluppo da quelle di produzione:
    
    ```yaml
    env_variables: 
    	MODE: "production"
    ```
    
    Nel codice dell’app:
    
    ```python
    import os
    mode = os.environ.get('MODE','development')
    ```
    
- **`handlers`** → definisce alcune regole di routing (=come Google App Engine deve gestire le richieste HTTP in base all’URL):
    - `http_headers` →  sono coppie chiave–valore aggiunte alla risposta inviata al browser. Il più importante è Content-Type, che indica il tipo di contenuto della risposta HTTP. Altri header riguardano caching, sicurezza e gestione delle sessioni.
        
        Con `http_headers` è possibile impostare questi header **a livello di infrastruttura GCP**, senza passare da Flask. Questo approccio ha pro e contro che qui non analizziamo. Esempio:
        
        ```yaml
        - url: /.*
          http_headers:
            X-Foo-Header: foo
            X-Bar-Header: bar value
            vary: Accept-Encoding
        ```
        
    - `static_dir` → serve per indicare una directory contenente file statici. In questo modo, tutte le richieste al path /static vengono mappate direttamente sulla cartella static e servite **senza coinvolgere l’istanza Python**, risparmiando risorse di calcolo. Esempio:
        
        ```yaml
        - url: /static
          static_dir: static
        ```
        
    - `static_files` → usato per mappare **un singolo file statico** a un URL specifico. Esempio:
        
        ```yaml
        - url: /favicon.ico
          static_files: static/images/favicon.ico
          upload: static/images/favicon.ico
        ```
        
    - `mime_type` → permette di specificare il tipo di file. Esempio:
        
        ```yaml
        - url: /config
          static_files: data.config
          mime_type: application/json
        ```
        
    - `script` → indica quale codice deve gestire la richiesta. L’opzione `auto` dice a GAE di cercare automaticamente l’oggetto app nel file [`main.py`](http://main.py/). Oggi è una scelta obbligata, in passato bisognava scrivere il nome del file dentro il quale gcloud doveva cercare l'elemento app.
- **`error_handlers`** → vengono utilizzati quando l’applicazione crasha pesantemente e gli handler definiti in Flask non sono eseguibili. I file `.html` serviti devono essere **statici**, così da poter essere restituiti anche se il codice Python non può essere eseguito.
    - Handler di default**:** viene servita una pagina generica per tutti gli errori non gestiti:
        
        ```yaml
        error_handlers:
        - file: default_error.html
        ```
        
    - Handler specifico per tipo di errore**:** permette di definire pagine diverse per errori specifici, ad esempio:
        
        ```yaml
        error_handlers:
        - error_code: over_quota
          file: over_quota.html
        ```
        
        **N.B.** L’error code `over_quota` viene attivato quando si supera la **quota assegnata dalle istanze GAE** (CPU, memoria, richieste giornaliere, ecc.).
        

---

# 5. Simulazione Locale e Gcloud CLI

## 5.1. Simulazione in Locale

Prima del deploy è fondamentale testare l’app in un ambiente che simula GAE, così da risparmiare tempo e risorse. Per lo sviluppo locale servono:

- `main.py`
- `app.yaml`
- `requirements.txt`

Gli step principali sono:

1. Installare il componente `app-engine-python` della gcloud CLI.
2. Avviare il server locale con il comando: `dev_appserver.py app.yaml`.
3. Accedere all’app tramite **localhost** (es. `http://localhost:8000`). GAE fornisce anche una **console locale** che simula l’ambiente cloud, mostrando informazioni sul progetto e permettendo di testare eventuali chiamate ad API (come Firebase) senza effettuare deploy reali.

## 5.2. Accesso e Gestione dei Progetti su Gcloud CLI

- **Autenticazione per accedere a gcloud** → `gcloud auth init`
- **Creazione di un progetto** → `gcloud projects create ${PROJECT_ID} --set-as-default`
    - `--set-as-default` imposta il progetto come predefinito della CLI
    - Senza `--set-as-default`, bisogna specificare sempre `--project=${PROJECT_ID}` nei comandi
- **Gestione del progetto di Default**:
    - Per controllare quale progetto è di default: `gcloud config get-value project`
    - Per impostare un progetto come default: `gcloud config set project ${PROJECT_ID}`
- **Per avere informazioni sul progetto**: `gcloud projects describe ${PROJECT_ID}`
    
    ```yaml
    #Output
    createTime: '2025-12-30T14:58:35.207871Z'
    lifecycleState: ACTIVE
    name: primo-test-30-12-2025
    projectId: primo-test-30-12-2025
    projectNumber: '1027882270498'
    ```
    
    Dove **lifecycleState** indica lo stato di disponibilità del progetto:
    
    - **`ACTIVE`**: il progetto è operativo e le risorse funzionano correttamente. Un progetto può sostenere costi solo se è **ACTIVE** e il **billing è abilitato**.
    - **`DELETE_REQUESTED`**: il progetto è in fase di eliminazione. Rimane in questo stato per **30 giorni**, durante i quali può ancora essere recuperato, prima della cancellazione definitiva.
    - **`LIFECYCLE_STATE_UNSPECIFIED`**: stato raro o transitorio, generalmente legato a condizioni di errore temporanee.
- **Informazioni sulla fatturazione**:
    - Per verificare lo stato di una fatturazione: `gcloud billing projects describe ${PROJECT_ID}`
        
        ```yaml
        #Output
        billingAccountName: ''
        billingEnabled: false 
        name: projects/primo-test-30-12-2025/billingInfo
        projectId: primo-test-30-12-2025
        ```
        
        **N.B.** In questo esempio il progetto non è associato a nessun **billing account**: anche se è **attivo**, non può fatturare e quindi **il deploy non è consentito**.
        
    - Per associare o rimuovere un account di fatturazione: `gcloud billing projects link ${PROJECT_ID} --billing-account <billing_account_id>`, `gcloud billing projects unlink ${PROJECT_ID}`
    - Per visualizzare gli account di fatturazione disponibili: `gcloud billing accounts list`
        
        ```yaml
        #Output
        ACCOUNT_ID           NAME                        		OPEN	    MASTER_ACCOUNT_ID
        0106DF-8D5258-557723  Account di fatturazione per studenti  	True
        ```
        
        **N.B.** `OPEN` indica se l’account è attivo e pronto per pagare.
        
- **Eliminazione e recupero di un progetto:**
    - Eliminazione: `gcloud projects delete ${PROJECT_ID}`
    - Recupero del progetto (entro 30 giorni): `gcloud projects undelete ${PROJECT_ID}`
- **Per controllare i servizi attivi di un progetto**: `gcloud services list --enabled`

## 5.3. Deploy in Cloud

<aside>
🔑

**Componenti Necessari**

Per effettuare il deploy di un’applicazione su App Engine sono necessari i seguenti elementi:

- **`app.yaml`** →file che descrive la configurazione di deploy
- **`.gcloudignore`** → specifica i file e le cartelle da escludere dall’upload (es. `.git`, `.venv`, `__pycache__`) per ridurre tempi e dimensioni del deploy
- **`requirements.txt`** → elenco delle dipendenze Python da installare nell’ambiente GAE
- **Codice dell’applicazione**: `main.py`, cartelle `static/` e `templates/`

![image.png](L4-Google%20Cloud%20Platform%20e%20Google%20App%20Engine/b9ca506d-4437-407d-8c09-4fb4f6361e80.png)

</aside>

### 5.3.1. Fasi del Deploy

1. **Attivazione delle API** → il deploy richiede l’attivazione di tre API:
    - `appengine.googleapis.com`: necessaria per creare e gestire le istanze GAE nel progetto.
    - `cloudbuild.googleapis.com`: utilizzata compilare e preparare il codice.
    - `storage.googleapis.com`: utilizzata per salvare gli artefatti prodotti dalla build nell’infrastruttura gcloud.
    
    Per attivare questi servizi si utilizza il seguente comando:
    
    ```bash
    gcloud services enable [appengine.googleapis.com](http://appengine.googleapis.com/) [cloudbuild.googleapis.com](http://cloudbuild.googleapis.com/) [storage.googleapis.com](http://storage.googleapis.com/)
    ```
    
2. **Creazione App** → `gcloud app create --project=${PROJECT_ID}` il progetto deve avere un billing account collegato. La creazione dell’app non genera costi: riserva solo l’URL e configura i servizi interni. I costi iniziano con il deploy. 
    
    Per visualizzare le informazioni: `gcloud app describe --project=${PROJECT_ID}` 
    
    ```python
    #Output
    authDomain: gmail.com
    codeBucket: staging.primo-test-30-12-2025.appspot.com
    databaseType: CLOUD_DATASTORE_COMPATIBILITY	→ app compatibile con Firestone in 
    modalita’ datastore
    defaultBucket: primo-test-30-12-2025.appspot.com
    defaultHostname: primo-test-30-12-2025.ey.r.appspot.com → URL pubblico
    featureSettings:
      splitHealthChecks: true
      useContainerOptimizedOs: true
    gcrDomain: eu.gcr.io
    id: primo-test-30-12-2025
    locationId: europe-west3
    name: apps/primo-test-30-12-2025
    serviceAccount: primo-test-30-12-2025@appspot.gserviceaccount.com
    servingStatus: SERVING	→ app pronta a ricevere traffico
    sslPolicy: DEFAULT		→ app gestisce certificati HTTPS
    ```
    
3. **Deploy** → `gcloud app deploy` (per aprire l’app nel browser: `gcloud app browse`). Se controllo nella dashboard della console, durante la fase di build sono state chiamate le seguenti APIs:
    
    ![image.png](L4-Google%20Cloud%20Platform%20e%20Google%20App%20Engine/image%201.png)
    
    Durante il deploy:
    
    - **Cloud Build** crea un’immagine container con runtime, dipendenze e codice.
    - L’immagine viene salvata tramite i servizi di storage/registry di Google (**Artifact Registry** )
    - **IAM** verifica i permessi per deploy e logging.
    - **Cloud Logging** raccoglie stdout e stderr dell’app.
    - **Pub/Sub** può essere usato internamente per notificare il completamento del deploy.
4. **Eliminazione Progetto** → al termine degli esperimenti, è consigliato eliminare il progetto per evitare costi: `gcloud projects delete ${PROJECT_ID}`

---

# 6. Per Capire

**Sviluppo backend con Flask:**

- `/templates`
- `/static`
- `/venv`
- `main.py`
- `requirements.txt`

**Simulazione in locale:**

- `main.py`
- `app.yaml`
- `requirements.txt`
- `/templates`
- `/static`

**Deploy in applicazione:**

- **`app.yaml`**
- **`.gcloudignore`**
- **`requirements.txt`**
- `main.py`
- `static/`
- `templates/`

---

![da inserire in struttura progetto in flask](L4-Google%20Cloud%20Platform%20e%20Google%20App%20Engine/image%202.png)

da inserire in struttura progetto in flask