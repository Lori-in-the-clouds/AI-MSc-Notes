# L2-Container Apache

Number: 3
Status: Check phase
Type: lab

# 1. Steps

- **Utilizzare un’immagine esistente** → per creare e avviare un container basato su un’immagine già disponibile, si può usare il comando:
    
    ```bash
    docker run --rm -it --name my-apache-app -p 8080:80 httpd:2.4
    ```
    
    Spiegazione dei parametri:
    
    - `httpd:2.4` → immagine di Apache esistente e versione da scaricare ed eseguire.
    - `-name my-apache-app` → assegna un nome al container (altrimenti Docker ne genera uno di default).
    - `-p 8080:80` → mappa la porta 8080 dell’host alla porta 80 del container, permettendo di accedere al servizio dall’host.
    - `-rm` → rimuove il container automaticamente quando il processo termina.
    - `-it` → combina `-i` (mantiene stdin aperto) e `-t` (attacca un terminale al container).
- **Creare una propria immagine con un Dockerfile**
    - Docker file:
        
        ```docker
        FROM httpd:2.4 
        COPY index.html /usr/local/apache2/htdocs/
        ```
        
        **N.B.** Quando viene eseguito il comando `COPY` viene creato un nuovo layer. Invece `index.html` è la pagina principale che Apache mostra quando qualcuno visita il sito senza specificare un file.
        
    - Costruzione dell’immagine:
        
        ```bash
        docker build . -t my-apache2
        ```
        
        Dove:
        
        - `.` → percorso della cartella contenente Dockerfile e file necessari per costruire l’immagine, in questo caso la cartella che contiene il Dockerfile e il file `index.html`.
        - `-t` → associa un nome all’immagine
    - Avvio del container dalla propria immagine:
        
        ```bash
        docker run --rm -it --name my-apache-app -p 8080:80 my-apache2
        ```
        

---

# 2. **Esperienza di lab: clustering con Apache e Docker**

L’obiettivo del laboratorio è realizzare un’architettura composta da **due worker** che espongono un servizio HTTP e da **uno switch** che agisce come **reverse proxy e load balancer**.

![image.png](L2-Container%20Apache/image.png)

L’intera architettura viene implementata tramite **tre container Docker**:

![image.png](L2-Container%20Apache/image%201.png)

<aside>
📌

**Conoscenze Preliminari: comando `sed`**

`sed` è un comando Linux utilizzato per modificare file di testo sostituendo le occorrenze di un valore con un nuovo valore. Esempio:

```bash
sed -i s/NAME/$workername/ /usr/local/apache2/htdocs/cluster/index.html
```

Dove:

- `s` indica l’operazione di *substitute*
- `NAME` è il pattern da cercare (il testo da sostituire)
- `$workername` è il valore che lo sostituisce
- Il file indicato è modificato **in-place** (`-i`)

Quindi il comando cerca la prima occorrenza della stringa fissa `NAME` in ogni riga del file `/usr/local/apache2/htdocs/cluster/index.html` e la sostituisce con il valore contenuto nella variabile di shell `$workername`. Le modifiche vengono salvate direttamente nel file originale.

</aside>

## 2.1. Passaggi per creare l’architettura

1. **Creazione delle immagini e dei container dei worker** → per farlo posso utilizzare due meccanismi:
    - **Creo Dockerfile per creare le immagini scrivendo i comandi di setup a mano:**
        1. Creo la cartella `httpworker/`
        2. Dentro tale cartella creo il docker file:
            
            
            ```docker
            FROM httpd:2.4
            ARG workername='RLworker'
            EXPOSE 80
            
            RUN mkdir /usr/local/apache2/htdocs/cluster
            COPY index.html /usr/local/apache2/htdocs/cluster
            RUN sed -i s/NAME/$workername/ /usr/local/apache2/htdocs/cluster/index.html
            ```
            
            ```html
            <html>
            	 <body>
            		 <h1>It works on NAME</h1>
            	 </body>
            </html>
            ```
            
            Questo Dockerfile parte dall’immagine ufficiale `httpd`, accetta un argomento chiamato `workername`, copia una pagina HTML all’interno del container e personalizza il contenuto della pagina sostituendo la stringa `NAME` con il nome del worker.
            
        3. Creo le immagini:
            
            ```docker
            docker build . -t rlworker-test
            workername=WORKER1 docker build . -t worker1
            workername=WORKER2 docker build . -t worker2
            docker image ls
            ```
            
    - **Creazione docker compose per non scrivere comandi di setup a mano** → docker compose viene utilizzato per automatizzare il build e l’avvio dei container, evitando l’uso ripetitivo dei comandi da terminale.
        1. Creo docker compose:
            
            
            ```yaml
            services:
              httpworker1:
                build:
                  context: httpworker
                  args:
                    workername: worker1
                image: httpworker:worker1
                container_name: httpworker1
            
              httpworker2:
                build:
                  context: httpworker
                  args:
                    workername: worker2
                image: httpworker:worker2
                container_name: httpworker2
            ```
            
            Dove:
            
            - **Image** → specifica l’immagine da eseguire. Docker la cerca prima in locale e, se non la trova, la scarica dal Docker Hub.
            - **Build** → indica i parametri necessari per costruire una propria immagine a partire da un Dockerfile. Docker cerca il Dockerfile nella cartella indicata da context e utilizza gli argomenti definiti in args. Questa fase viene eseguita solo se l’immagine specificata non esiste già localmente.
        2. Creo le immagini:
            
            ```bash
            docker compose build
            ```
            
        3. Avvio i containers:
            
            ```bash
            docker compose up
            docker ps -a
            ```
            
            **N.B.** `docker ps -a` permette di verificare lo stato di tutti i container presenti sul sistema.
            
        
        <aside>
        🔑
        
        **Altri comandi di Docker Compose:**
        
        - Arresto e rimozione dei container:
            
            ```bash
            docker compose down
            ```
            
            Questo comando **ferma ed elimina** tutti i container definiti nel file `docker-compose.yml`. Una volta eseguito, i container non esistono più e **non possono essere riutilizzati**: per avviarli nuovamente è necessario crearne di nuovi.
            
        - Arresto dei container senza rimozione:
            
            ```bash
            docker compose stop
            docker compose restart
            ```
            
            Con docker compose stop i container vengono **fermanti ma non eliminati**. In questo modo, eseguendo docker compose restart, vengono **riavviati gli stessi container**, senza crearne di nuovi.
            
        </aside>
        
2. **Creazione immagine e container dello switch** → per farlo ho due meccanismi:
    - **Creo Dockerfile dello switch:**
        
        ```docker
        FROM httpd:2.4
        EXPOSE 80
        COPY mod_proxy.conf /usr/local/apache2/conf/extra
        RUN echo "Include conf/extra/mod_proxy.conf" >> /usr/local/apache2/conf/httpd.conf
        ```
        
        Questo Dockerfile include nel file `httpd.conf` la configurazione dedicata al proxy.
        
        <aside>
        📌
        
        **Configurazione `mod_proxy.conf`:**
        
        ```python
        LoadModule proxy_module modules/mod_proxy.so
        LoadModule proxy_http_module modules/mod_proxy_http.so
        LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
        LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
        LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
        
        <Proxy balancer://mycluster>
            BalancerMember http://httpworker1
            BalancerMember http://httpworker2
        </Proxy>
        
        ProxyPass /cluster balancer://mycluster/cluster
        ProxyPassReverse /cluster balancer://mycluster/cluster
        ```
        
        - All’interno di tale configurazione carichiamo i seguenti moduli:
            - `mod_proxy` → per fare reverse proxy.
            - `mod_proxy_http` → per tradurre il linguaggio in entrata (https) in http.
            - `mod_proxy_balancer` → per distribuire il carico tra più worker.
            - `mod_lbmethod_byrequests` → algoritmo di bilanciamento.
            
            La sintassi per caricare un modulo è `LoadModule <nome modulo> <filename.so>`.
            
        - In seguito definisco i workers su cui il LoadBalancer farà bilanciamento del carico. La sintassi per definire i worker di un LoadBalancer è `BalanceMember http://<nome_del_container_del_worker>`.
        - Definisco ProxyPass e rispettivo reverse per indirizzare le richieste al balancer, in particolare se l’URI in entrata matcha `/cluster`, chiama il balancer definito sopra.
        
        </aside>
        
    - **Aggiorno lo scorso docker-compose:**
        
        ```yaml
        services: 
        	httpworker1: 
        		build: 
        			context: httpworker 
        			args: workername: worker1 
        		image: 
        			httpworker:worker1 
        		container_name: httpworker1 
        
        	httpworker2: 
        		build: 
        			context: httpworker
        			args: workername: worker2
        		image: 
        			httpworker:worker2
        		container_name: httpworker2
        	switch: 
        		build: 
        			context: switch 
        			image: httpswitch 
        		depends_on: 
        			- httpworker1 
        			- httpworker2 
        		ports: 
        			- "80:80" 
        		container_name: switch
        
        ```
        
        **N.B.** `depends_on` assicura che lo switch venga avviato solo dopo i worker.
        
3. **Aggiungo Dashboard del Load Balancer** → per monitorare il bilanciamento del carico, viene abilitata la dashboard nativa di Apache (balancer-manager), che è una **funzionalità interna** e non un worker esterno. 
    
    Per attivarla, si modifica il file `mod_proxy.conf` aggiungendo un **handler** che instrada tutte le richieste verso l’URI `/balancer-manager` alla funzionalità interna di `mod_load_balancer`. Siccome `balancer-manager` è una funzione nativa di `load_balancer` e quindi un processo interno di Apache, inoltrare una richiesta a questa funzione non è come inoltrare una richiesta ad un worker esterno. In particolare:
    
    - **Richieste verso funzioni interne** → si utilizzano gli **handlers**. Ad esempio, per abilitare la dashboard del load balancer (balancer-manager), si configura:
        
        ```python
        <Location /balancer-manager>
        	SetHandler balancer-manager 
        	Order Deny,Allow 
        	Allow from all 
        </Location
        ```
        
    - **Richieste verso worker esterni** → quando Apache è in modalità proxy grazie a mod_proxy, si utilizza **proxy pass** per inoltrare le richieste ai container dei worker e gestire il bilanciamento del carico:
        
        ```python
        <Proxy balancer://mycluster>
        	BalancerMember http://httpworker1 
        	BalancerMember http://httpworker2 
        </Proxy>
        
        ProxyPass /cluster balancer://mycluster/cluster
        ProxyPassReverse /cluster balancer://mycluster/cluster
        ```
        
    
    **N.B.** In questo modo, le richieste all’URI `/cluster` vengono distribuite ai worker definiti nel cluster, mentre /balancer-manager viene gestito internamente dal load balancer di Apache.
    
    <aside>
    📌
    
    **Nuovo `proxy.conf`:**
    
    ```python
    LoadModule proxy_module modules/mod_proxy.so 
    LoadModule proxy_http_module modules/mod_proxy_http.so
    LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
    LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
    LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
    
    <Proxy balancer://mycluster>
    	BalancerMember http://httpworker1 
    	BalancerMember http://httpworker2 
    </Proxy>
    
    ProxyPass /cluster balancer://mycluster/cluster
    ProxyPassReverse /cluster balancer://mycluster/cluster
    
    <Location /balancer-manager>
    	SetHandler balancer-manager 
    	Order Deny,Allow 
    	Allow from all 
    </Location>
    ```
    
    </aside>
    

## 2.2. Load Balancer e Sticky Sessions

Per far sì che il load balancer gestisca correttamente le sessioni degli utenti, si possono usare due approcci principali:

- **Binding table** → Apache mantiene in memoria una tabella interna che associa ogni sessione al worker corretto, inoltrando tutte le richieste relative a quella sessione allo stesso worker. Questo approccio funziona, ma è complesso da gestire perché richiede aggiornamenti continui della tabella in memoria.
- **Cookie di sessione lato client** → in questo metodo, il client conserva le informazioni di routing nel cookie di sessione.
    - Un cookie standard ha la forma: `<CookieName>=<SessionID>`
    - Per gestire lo sticky routing tra worker, il cookie include anche la **RouteID** del worker: `<CookieName>=<SessionID>.<RouteID>`

Per abilitare lo sticky routing usando il cookie, dobbiamo aggiungere info a `mod_proxy.conf`:

```python
LoadModule proxy_module modules/mod_proxy.so 
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
LoadModule slotmem_shm_module modules/mod_slotmem_shm.so

<Proxy balancer://mycluster>
	BalancerMember http://httpworker1 route=worker1
	BalancerMember http://httpworker2 route=worker2
</Proxy>

ProxyPass /cluster balancer://mycluster/cluster stickysession=MYSESSID
ProxyPassReverse /cluster balancer://mycluster/cluster

<Location /balancer-manager>
	SetHandler balancer-manager 
	Order Deny,Allow 
	Allow from all 
</Location>
```

- Ogni **BalancerMember** ora ha un nome di **route** (`route=worker1`, `route=worker2`) che identifica il worker.
- La direttiva ProxyPass include la flag `stickysession=MYSESSID`, indicando il nome del cookie che memorizza la route del worker.

---