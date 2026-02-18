# 14. Omnet++

Number: 1
Status: Done
Type: theory

# 1. Introduzione

Omnet++ è un framework di simulazione a eventi discreti, modulare ed estensibile, scritto in C++. È utilizzato principalmente per la simulazione di reti di comunicazione, ma grazie alla sua flessibilità permette di modellare qualsiasi sistema basato su componenti che scambiano messaggi. Esistono due versioni principali:

- **Omnet++** → versione open source, gratuita per uso accademico e non commerciale
- **OMNEST++** → versione commerciale, offre supporto tecnico e funzionalità avanzate per l'esecuzione distribuita

---

# 2. Struttura delle Directory Principali

- **`bin`**: eseguibili del simulatore
- **`include`** e **`lib`**: header file e librerie per lo sviluppo
- **`doc`**: documentazione (molto curata rispetto ad altri simulatori)
- **`src`**: codice sorgente del kernel di simulazione
- **`samples`**: esempi già pronti. Di particolare importanza è la directory `qnet` (queueing networks), utilizzata nel corso per studiare le reti di code e validare le simulazioni con la teoria.
- **`parsim`**: strumento per lanciare simulazioni in **parallelo**, utile quando la simulazione è **computazionalmente pesante**. È particolarmente vantaggioso quando si devono eseguire **molte repliche** (ad esempio per stimare intervalli di confidenza) o confrontare **più configurazioni/versioni** dello stesso modello.

![image.png](14%20Omnet++/image.png)

---

# 3. Architettura della Simulazione

**OMNeT++** gestisce la simulazione tramite lo scambio di **messaggi**, che rappresentano gli eventi. L’architettura si basa sui seguenti elementi fondamentali:

- **Moduli** → entità attive del sistema:
    - Semplici: componenti atomici (“scatole nere”) scritti in C++, che definiscono il comportamento attivo del sistema.
    - Composti: insiemi di moduli (semplici o composti) interconnessi. Non contengono codice C++, ma solo la descrizione strutturale in linguaggio **NED** (=Network Description),
    
    Ogni modulo presenta **porte** (gates) per il passaggio dei messaggi; per ciascuna porta si specificano il **nome**, il **numero**, la **cardinalità** e la **direzione** (in, out, inout).
    
- **Gate** → punti di ingresso e uscita dei moduli attraverso cui transitano i messaggi. Possono essere `input`, `output` o `inout`.
- **Canali**→ collegamenti tra i gate.
    
    
    1. Caratteristiche: possono essere definiti con diversi livelli di realismo:
        - *Ideali*: Trasmissione istantanea.
        - *Con Ritardo*: Introducono una latenza (delay).
        - *Con Ritardo e Banda*: Aggiungono un limite al datarate (bandwidth).
        - *Con Tasso di Errore*: Specificano una probabilità di errore (error rate) per simulare guasti o interferenze.
    2. Direzionalità: possono gestire flussi direzionati (unidirezionali) o essere bidirezionali.
    3. Ereditarietà: i canali supportano l'ereditarietà; è possibile definire un canale base con proprietà comuni ed estenderlo per creare canali specifici, favorendo il riuso del codice.
    
    ![image.png](14%20Omnet++/image%201.png)
    
    Per connettere due gate con un canale:
    
    ![image.png](14%20Omnet++/image%202.png)
    
- **Network** → modulo di livello più alto (top-level) che rappresenta l’intero sistema simulato. Non ha gate verso l’esterno ed è l’“universo” della simulazione.
    
    ![image.png](14%20Omnet++/image%203.png)
    

![image.png](14%20Omnet++/image%204.png)

---

# 4. Focus sul Modulo Semplice

Ogni modulo semplice è definito da due file complementari:

1. **File C++:** definisce il comportamento logico ereditando dalla classe `cSimpleModule`. I metodi principali sono:
    - `initialize()` → eseguito all'inizio per leggere i parametri e schedulare i primi eventi.
    - `handleMessage(cMessage *msg)` → cuore del modulo. Viene invocato ogni volta che il modulo riceve un messaggio (evento). Qui si gestisce la logica di processamento.
    - `finish()`→ eseguito al termine della simulazione, utile per salvare statistiche finali.
    
    Funzioni chiave in C++:
    
    - `send(msg, "gateName")` → invia un messaggio attraverso una porta.
    - `scheduleAt(time, msg)` →schedula un evento (messaggio) per se stesso in un futuro specifico (auto-messaggio).
    - `par("paramName")` → legge il valore di un parametro dal file NED/INI.

1. **File NED:** descrive l'interfaccia del modulo verso l'esterno (nome, parametri, gate). Un parametro dichiarato volatile viene **rivalutato a ogni lettura**: è fondamentale per ottenere valori casuali diversi a ogni occorrenza (ad es. tempi di inter-arrivo basati su distribuzioni statistiche). Altri elementi importanti:
    - **@signal** → definisce il nome di un segnale emesso dal modulo.
    - **@statistic** → specifica come raccogliere e aggregare i segnali (media, istogrammi, ecc.).
    - **@display** → informazioni grafiche (icona, colore, layout).
    - **@class** → associa il modulo NED a una classe C++ e consente di sfruttare l’**ereditarietà**.

![image.png](14%20Omnet++/image%205.png)

---

# 5. Focus su Messaggi

I **messaggi** in OMNeT++ sono definiti in file con estensione `.msg`. Questi file descrivono la struttura del messaggio specificandone i **campi**; il compilatore `opp_msgc` li traduce automaticamente in **classi C++**, che vengono poi compilate e collegate all’eseguibile finale della simulazione.

Durante la simulazione, un messaggio **esce da un gate di tipo out**, attraversa un **canale** e **entra in un gate di tipo in** del modulo destinatario, rappresentando così il verificarsi di un evento.

Un **pacchetto** è una particolare tipologia di messaggio: oltre ai campi logici, possiede anche una **dimensione fissa**, utilizzata per modellare aspetti fisici della comunicazione come ritardi, banda e congestione.

---

# 6. Raccolta Dati

Omnet++ offre meccanismi avanzati per la raccolta dati tramite **Segnali** (`@signal`) e **Statistiche** (`@statistic`) dichiarati nei file NED. Sono presenti diversi formati di salvataggio:

- **Vettori (.vec)** → serie temporali di valori (es. utilizzo della CPU nel tempo). Possono occupare molto spazio su disco.
- **Istogrammi** → distribuzioni di probabilità. Omnet++ calcola automaticamente i "bin" (intervalli), anche se la scelta ottimale dei bin è complessa.
- **Quantile estimation** → tecnica utilizzata per stimare la distribuzione dei dati senza dover fissare i bin a priori. È utile per calcolare percentili in modo efficiente.

**N.B.** Nel codice **NED** si specifica **quali eventi o segnali devono essere raccolti** (tramite @signal e @statistic), cioè *cosa* osservare durante la simulazione. Successivamente, **nell’IDE di OMNeT++** si definisce **come questi dati devono essere valutati e analizzati**, ad esempio scegliendo il tipo di elaborazione statistica, i filtri, i grafici e il calcolo delle metriche finali.

---

# 7. Workflow di Sviluppo e Simulazione

Il processo tipico per realizzare ed eseguire una simulazione in Omnet++ segue questi passaggi fondamentali:

1. **Definizione dei Moduli:**
    - Creazione dei file `.ned` per descrivere la struttura e le interfacce.
    - Stesura dei sorgenti `.cc` e `.h` per implementare il comportamento (C++).
2. **Creazione Classi Messaggio:** definizione della struttura dei messaggi tramite file `.msg`.
3. **Compilazione e Linking:** generazione dell'eseguibile del simulatore che include i nuovi moduli e messaggi.
4. **Definizione Scenari e Parametri:**
    - Uso dei file `.ned` per definire la topologia della rete (network definition).
    - Uso dei file `.ini` per assegnare valori ai parametri e configurare configurazioni multiple (scenari).
5. **Esecuzione Simulazione**: lancio della simulazione (via interfaccia grafica o riga di comando).
6. **Analisi dei Dati**

![image.png](14%20Omnet++/image%206.png)

<aside>
📌

**Risultati delle Simulazioni**

I **risultati delle simulazioni** vengono salvati nel file `.sca`, che registra tutti i dati generati durante l’esecuzione degli eventi. L’opzione `-u Cmdenv` seleziona l’interfaccia testuale, utile perché scrive poco e non rallenta la simulazione.

Per l’analisi, i dati del file `.sca` vengono importati in un database SQLite. A questo scopo, si utilizza un file `config.json` per mappare l’identificatore di ciascun file `.sca` alla chiave corrispondente nel database, consentendo di organizzare e interrogare facilmente i risultati.

</aside>

---