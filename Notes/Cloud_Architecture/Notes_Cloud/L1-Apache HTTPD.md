# L1-Apache HTTPD

Number: 3
Status: Check phase
Type: lab

<aside>
💡

**Apache HTTPD** è un reverse proxy, cioè un sistema che riceve le richieste dei client e le inoltra ai server che eseguono l’applicazione. Per utilizzarlo si possono seguire due strade principali:

- **Installazione locale**: scaricare i file, decomprimerli e installarli tramite il package manager del mio SO.
- **Container Docker**: creare un container che esegue Apache HTTPD, utile per isolamento e portabilità.
</aside>

# 1. Struttura del FileSystem

Quando installiamo **Apache HTTPD**, scegliamo in che cartella posizionare la sua root. Su Linux, di solito si trova in `/etc/httpd` o `/etc/apache2`. All’interno della root ci sono diverse cartelle:

- **`/bin`**: contiene i binari principali di Apache. Tra gli strumenti più rilevanti:
    - **`apachectl`**: script di controllo per avviare, fermare, riavviare o verificare la configurazione del demone Apache in modo sicuro.
    - **`httpd`**: demone di apache, ovvero il programma principale che resta in esecuzione in background, è in ascolto sulle porte HTTP/HTTPS e serve i contenuti.
    - **`htpasswd`**: crea e aggiorna file con utenti e password criptate per proteggere aree del sito.
    - **`ab` (Apache Benchmark)**: utile per testare prestazioni e throughput del server sottoponendolo a carichi simulati.

![image.png](L1-Apache%20HTTPD/image.png)

- **`/conf`**: contiene i file di configurazione di Apache.
- **`/htdocs`:** contiene i file statici (HTML, CSS, Immagini, PDF) che Apache deve servire quando un client glieli richiede.
- **`/cgi-bin`**: contiene gli script dinamici (es. Python) eseguiti da Apache; l’output viene inviato al client.
- **`/logs`**: raccoglie tutti i file di log generati dal server.

![image.png](L1-Apache%20HTTPD/image%201.png)

---

# 2. Configurazione di Apache

Le impostazioni principali si trovano nel file `/conf/httpd.conf`. Ci sono due diversi tipi di impostazioni:

1. **Global Settings**
    - **ServerRoot** → cartella principale del filesystem del server, solitamente `/etc/httpd` o `/etc/apache2`.
    - **Listen** → porta sulla quale il server ascolta, di solito 80 o 8080 per HTTP e 443 per HTTPS.
    - **User, Group** → utente e gruppo dei processi figli di Apache.
    - **LoadModule** → Apache ha un’architettura modulare. Il core gestisce solo le funzioni base, mentre tutte le funzionalità aggiuntive (autenticazione, SSL/TLS, compressione, scripting come PHP) sono implementate tramite moduli esterni.
    - **[PidFile]** → piccolo file contenente il PID del processo principale, il comando `apachectl` lo utilizza per identificare quale processo httpd è in esecuzione e quale gestire.
    - **[Timeout]** → tempo massimo per una connessione inattiva. Limita l’impatto di attacchi DoS chiudendo rapidamente le connessioni dormienti.
    - **[KeepAlive]** → se impostato a `True` permette di mantenere aperte le connessioni tra più richieste.
    - **[MaxKeepAliveRequests]** → numero massimo di richieste per connessione mantenuta.
    - **[KeepAliveTimeout]** → timeout per connessioni con KeepAlive attivo.
    - **MPM settings (=Multi-process module)** → definiscono il modello di funzionamento e la capacità di gestire il carico dell'intera istanza Apache.
    
    **N.B.** Le `[]` indicano che il parametro è **opzionale**.
    
2. **Default Server Settings:** Apache può ospitare più domini sulla stessa macchina fisica, rappresentando i vari server web come Virtual Hosts della macchina su cui gira Apache.
    - **Default server** → nome del Virtual Host che prende in carica le richieste se non viene specificato il nome di un altro VH oppure viene specificato un VH inesistente.
    - **Ogni VH ha le seguenti impostazioni principali**:
        - ServerName: dominio e porta associati al VH.
        - DocumentRoot: cartella che contiene i file da servire per quel dominio.
        - ErrorLog: file di log degli errori interni del server.
        - CustomLog: file di log delle richieste HTTP gestite dal server. Registra le singole transazioni tra client e server (richiesta e risposta): per ogni richiesta HTTP gestita con successo, Apache scrive una riga di testo nel file specificato.
        
        *Esempio:*
        
        ```python
        <VirtualHost _default_:80>
            ServerName default-server
            DocumentRoot "/var/www/html/default_error_page"
            # L'utente vedrà una pagina che dice: "Sito non trovato su questo server"
        </VirtualHost>
        
        <VirtualHost *:80>
            ServerName www.aziendaX.com
            DocumentRoot "/var/www/siti/aziendaX/public_html"
            # Qui ci sono i file HTML/CSS del sito aziendale
        </VirtualHost>
        
        <VirtualHost *:80>
            ServerName blog.personaleY.net
            DocumentRoot "/var/www/siti/personaleY/blog_content"
            # Qui ci sono i file e gli script del blog
        </VirtualHost>
        ```
        
        Una richiesta a `www.aziendaX.com` sarà servita dal secondo Virtual Host e i file verranno presi da `“/var/www/siti/aziendaX/public_html”`.
        
        <aside>
        📌
        
        **Directory o Location Containers**
        
        In Apache è possibile definire **proprietà e regole che non valgono globalmente**, ma solo per **specifiche parti del filesystem** o per **determinate risorse HTTP**.
        
        - **Directory Containers** → le regole definite valgono per la directory indicata e per tutte le sue sottodirectory.
            
            ```python
            <Directory "/var/www/siti/miosito/download">
                Options +Indexes FollowSymLinks
                Require all granted #Permette a tutti di accedere ai file
            </Directory>
            ```
            
        - **Location Containers** → applicano regole a un **percorso URL**, indipendentemente dalla struttura del filesystem sottostante.
            
            ```python
            <Location "/admin">
                Require ip 192.168.1.0/24 #permette l'accesso solo alla sottorete 192.168.1.x
                Require not host malintenzionato.com
            </Location>
            ```
            
        
        Sia nei Directory che nei Location containers è possibile definire una serie di proprietà:
        
        - **Options:** flag che abilita o disabilita specifici comportamenti del server:
            - Indexes → se un client richiede una directory senza specificare un file, Apache genera automaticamente un elenco navigabile dei file presenti. Questa opzione è spesso disabilitata su directory contenenti file sensibili, perché può esporre la struttura interna del filesystem.
            - FollowSymLinks / SymlinksIfOwnerMatch ****→ gestiscono il comportamento dei collegamenti simbolici:
                - **FollowSymLinks**: Apache segue qualsiasi symlink e serve il contenuto di destinazione.
                - **SymlinksIfOwnerMatch**: Apache segue il symlink solo se il proprietario del link coincide con quello del file di destinazione (misura di sicurezza aggiuntiva).
                - **ExecCGI / Includes:** opzioni avanzate per l’esecuzione di script o file di include (non approfondite).
        - **AllowOverride:** il file `.htaccess` è un file di testo che può essere posizionato in una directory per applicare regole locali valide anche per le sottodirectory. AllowOverride stabilisce se e quali direttive presenti nei file `.htaccess` possono **sovrascrivere** la configurazione principale del server (es. `httpd.conf`).
            
            `AllowOverride None`: ****impostazione più sicura e performante: Apache ignora completamente qualsiasi file `.htaccess`.
            
        - **Allow, Deny e Require**: utilizzati per definire **Access Control List (ACL)**, ovvero per  accettare o rifiutare le richieste da determinati IP.
        </aside>
        

---

# 3. Focus su MPM (Multi-Processing Modules)

I **moduli MPM** definiscono **come Apache utilizza CPU e memoria** per gestire le richieste in ingresso. Poiché stabiliscono **quanti processi e thread possono essere attivi contemporaneamente**, le configurazioni MPM sono fondamentali per la scalabilità, la stabilità e la sicurezza del sever. 

Apache mette a disposizione diversi MPM: la scelta dipende dal tipo di applicazione, dal carico previsto e dalla compatibilità dei moduli utilizzati:

- **MPM Prefork** → è basato **esclusivamente su processi**, senza utilizzo di thread. Nel modello **Prefork** esiste un **processo padre** che funge da *dispatcher* e ha il compito di creare e gestire più **processi child indipendenti**. Ogni processo child può gestire **una sola richiesta alla volta** e, poiché **non vengono utilizzati thread**, ogni richiesta HTTP è completamente isolata all’interno di un processo separato. Questo isolamento garantisce un elevato livello di stabilità e sicurezza, evitando interferenze tra richieste concorrenti.
    
    <aside>
    📌
    
    **Variabili controllabili:**
    
    - StartServers: numero iniziale di processi child avviati all’avvio del server.
    
    - MinSpareServers / MaxSpareServers: definiscono il numero minimo e massimo di processi child inattivi (di scorta):
        - Se i processi disponibili scendono sotto il minimo, Apache ne crea di nuovi.
        - Se superano il massimo, Apache ne termina alcuni.
        
        Questo meccanismo garantisce che ci siano sempre processi pronti a rispondere rapidamente alle richieste.
        
    
    ![image.png](L1-Apache%20HTTPD/image%202.png)
    
    - MaxClients (oggi MaxRequestWorkers): numero massimo di richieste che il server può gestire in parallelo (un child può gestire più richieste ma una alla volta).
    - MaxRequestsPerChild:numero massimo di richieste gestite da un singolo processo child prima che venga terminato e ricreato. Serve a **limitare problemi di memory leak** in processi che restano attivi a lungo.
    </aside>
    
    **PRO:** molto stabile e massima compatibilità con librerie e moduli non thread-safe (tipicamente vecchie estensioni PHP).
    
- **MPM Worker** → si basa sull’uso di **thread** all’interno di processi. Nel modello Worker, un processo padre agisce da dispatcher e crea più processi child. All’interno di ciascun processo child ci sono più thread, ognuno dei quali gestisce una singola richiesta alla volta.
    
    <aside>
    📌
    
    **Variabili Controllabili:**
    
    - ThreadsPerChild: numero di thread creati all’interno di ogni processo child. Più thread per processo significano meno overhead di creazione dei processi.
    - ServerLimit: numero massimo di processi worker che Apache può creare.
    - ThreadLimit: ****numero massimo di thread consentiti per ogni processo.
    - StartServers, MinSpareServers, MaxSpareServers: funzionano come in Prefork, ma si riferiscono **ai processi child multi-threaded**, non ai singoli thread.
    - MaxRequestWorkers (equivalente di MaxClients) → numero massimo di richieste concorrenti gestibili dal server, ovvero il **numero massimo di thread attivi contemporaneamente**.
    </aside>
    
    **PRO:** uso più efficiente della memoria, garantendo prestazioni superiori e una maggiore scalabilità nella gestione di connessioni concorrenti.
    

---

# 4. Clustering con Apache

Fare clustering consiste nel modellare un server come un insieme di server, in modo che una richiesta in ingresso possa essere distribuita tra più server. Questo approccio migliora sia la **scalabilità** sia l’**affidabilità** del sistema.

Per implementare il clustering con Apache si possono utilizzare due meccanismi principali di dispatching:

1. **Regole di riscrittura:** si sfruttano RegEx (Regular Expression) per intercettare gli URL in ingresso, modificarli e indirizzare la richiesta verso l’host corretto.
2. **Modulo Proxy:** mediante regole chiamate direttive, Apache intercetta gli URL e li reindirizza agli host appropriati. Spesso il modulo proxy viene combinato con un modulo di **load balancing**, per distribuire in modo equilibrato il traffico tra i server.

## 4.1. Comunicazione tra Apache e i Client/Worker

La comunicazione tra client e server Apache avviene di solito tramite **HTTP o HTTPS**, mentre tra Apache e i server worker possono essere utilizzati protocolli più efficienti:

- **HTTP** → semplice e standard, ma meno efficiente a livello di overhead di rete. Il protocollo HTTPS non viene generalmente usato tra Apache e i worker, perché le questioni di sicurezza sono gestite direttamente dal server Apache.
- **AJP (Apache JServ Protocol)** → protocollo binario molto più efficiente di HTTP, usato per comunicare con server di applicazioni basati su Java, come Tomcat, riducendo il carico di rete.
- **FTP** → utilizzato principalmente per interagire con server di file.

**N.B.** Questa traduzione dei protocolli tra Apache e i worker è resa possibile dai moduli specifici: `mod_proxy_http`**,** `mod_proxy_ajp`**,** `mod_proxy_ftp`.

## 4.2. Modulo mod_rewrite

Il modulo **mod_rewrite** permette di gestire e riscrivere gli URL nella fase iniziale di traduzione. Quando un URL in ingresso viene ricevuto, il modulo confronta l’URL con una lista di **regular expressions** (pattern). In base al risultato, possono verificarsi tre azioni differenti:

1. sostituire l’URL con un altro URL locale
2. sostituire l’URL con uno che punta a un altro host (**proxy forwarding**)
3. rifiutare la richiesta

### 4.2.1. Configurazione del modulo

Per attivare il modulo, bisogna impostare la variabile `RewriteEngine` a True e definire le **`RewriteRules`**. È possibile attivare il logging con due variabili:

- **`RewriteLog`** → nome del file in cui scrivere i log delle operazioni di riscrittura
- `RewriteLogLevel` → numero da 0 a 9 che indica il livello di dettaglio dei log

### 4.2.2. RewriteRules

Ogni regola è composta da tre parti: `pattern| target | flags`:

- pattern → la regular expression che deve essere matchata.
- t**arget** → il nuovo URL; può utilizzare backreferences (come `$1`, `$2`, ecc..).
- **flags** → opzionali, indicano azioni aggiuntive:
    - `[F]` (forbidden) → interrompe l’elaborazione e restituisce HTTP 403.
    - `[L]` (last) → se la regola viene matchata, impedisce ad Apache di processare altre RewriteRules per quell’URL.
    - `[R]` (redirect) → invia un codice di reindirizzamento 3xx al client, che farà una nuova richiesta verso l’URL riscritto.
    - `[P]` (proxy) → inoltra la richiesta a un server esterno tramite il modulo proxy.

*Esempi:*

```python
RewriteRule /test([0-9]+)\.html /rewrite$1.html
# () nella RegEx catturano la parte dei numeri come una backreference $1.
# \. è un escape char per indicare .

RewriteRule ^/old_(.*)\.html /new_$1.html [R]
# [R] invia un redirect al client
```

<aside>
📌

**RewriteCond:**

Le RewriteRules possono essere **condizionali**, eseguite solo se certe condizioni sono soddisfatte. Le condizioni possono basarsi su backreferences o variabili d’ambiente (ad esempio `TIME_HOUR`).

- Le condizioni (`RewriteCond`) vanno scritte **immediatamente prima** della RewriteRule che devono controllare.
- Se la condizione è soddisfatta, la regola viene eseguita; altrimenti viene saltata.
- La flag `[L]` può essere utilizzata per fermare l’elaborazione delle regole successive.

*Esempio:*

```python
RewriteCond %{TIME_HOUR}%{TIME_MIN} >0700
RewriteCond %{TIME_HOUR}%{TIME_MIN} <1900
RewriteRule ^/page\.html$ page.day.html [L]
RewriteRule ^/page\.html$ page.night.html
```

In questo esempio, durante il giorno viene servita la pagina `page.day.html`, mentre di notte viene servita `page.night.html`.

</aside>

## 4.3. Modulo mod_proxy

Il modulo **mod_proxy** permette ad Apache di funzionare come **reverse proxy**, cioè di inoltrare le richieste ricevute dai client verso uno o più server di backend (worker) invece di servire direttamente i contenuti dal proprio DocumentRoot. I worker possono trovarsi sulla stessa macchina o su macchine remote.

<aside>
🔑

**Moduli da utilizzare insieme a mod_proxy**

Il funzionamento di **mod_proxy** può essere potenziato combinandolo con altri moduli:

- **`mod_cache`** → salva le risposte dei backend in cache locale, così richieste successive possono essere servite senza caricare i worker, riducendo il carico.
- **`mod_proxy_balancer**` → gestisce il bilanciamento del carico tra i worker, selezionando quale server riceverà la richiesta, garantendo scalabilità, tolleranza ai guasti e supporto alle sticky sessions (=meccanismo per assicurare che tutte le richieste provenienti dallo stesso client vengano indirizzate sempre allo stesso worker).
- **`mod_proxy_http`, `mod_proxy_ajp`, `mod_proxy_ftp`** → gestiscono protocolli specifici tra Apache e i worker, consentendo comunicazioni efficienti.
</aside>

### 4.3.1. Configurazione delle Regole di Inoltro

Per configurare le regole di inoltro delle richieste da parte di `mod_proxy` ci sono alcuni parametri fondamentali:

- **ProxyPass** → instrada un URL locale verso un worker: `ProxyPass [URL_Locale_su_Apache] [Indirizzo_del_Worker]`.
    
    <aside>
    
    *Esempio:*
    
    Una richiesta a `http://tuo.server.com/app/pagina.htm`[l](http://tuo.server.com/app/pagina.html) viene inoltrata a `http://backend.example.com:8080/pagina.html` con l’utilizzo della seguente regola:
    
    ```python
    ProxyPass /app/ http://backend.example.com:8080/
    ```
    
    </aside>
    
- **ProxyPassMatch** → simile a ProxyPass, ma usa **ReGex** per il pattern dell’URL, utile per instradamenti dinamici basati su RegEx: `ProxyPassMatch [Espressione_Regolare] [Indirizzo_del_Worker]`.
- **ProxyPassReverse** → il suo scopo è riscrivere i dati contenuti nelle intestazioni della risposta HTTP provenienti dal backend, sostituendo tutti i riferimenti interni con l'URL pubblica corretta. Ovvero effettua l’operazione inversa rispetto al ProxyPass.
    
    *Esempio:*
    
    ```python
    ProxyPass "/mirror/foo/" "http://back.ex.com/"
    ProxyPassReverse "/mirror/foo/" "http://back.ex.com/" 
    ProxyPassReverseCookieDomain "back.ex.com" "public.ex.com" ProxyPassReverseCookiePath "/" "/mirror/foo/"
    ```
    

### 4.3.2. Configurazione dei Workers

Ogni worker a cui Apache inoltra le richieste può essere configurato per ottimizzare prestazioni, stabilità e tolleranza ai guasti:

- **Numero di connessioni** → massimo numero di connessioni attive che Apache può mantenere verso il worker. Limitare questo numero evita di sovraccaricare il backend. Le richieste in eccesso vengono inoltrate ad altri worker disponibili.
- **Timeout** → tempo massimo che Apache attende una risposta dal worker. Impedisce che richieste rimangano bloccate indefinitamente, migliorando l’esperienza utente e consentendo failover verso altri worker o error response.
- **Fattore di carico** → viene utilizzato solo quando è abilitato il **bilanciamento del carico**. Si tratta di un valore numerico che indica la **capacità relativa di un worker** rispetto agli altri nel pool. In pratica permette ad Apache di inviare più o meno richieste a ciascun worker a seconda della sua potenza o della sua disponibilità.
    
    *Esempio:* se Worker A ha fattore 2 e Worker B ha fattore 1, Apache invierà a A il doppio delle richieste rispetto a B, bilanciando il carico secondo le capacità.
    

---