# 3. Modelli Economici

Number: 1
Status: Done
Type: theory

# 1. Modelli di Business e Fonti di Reddito

Le aziende basate su internet possono generare entrate da diverse fonti:

- **Pubblicità**
- **Investitori**, **sponsor** o **donazioni**
- **Freemium** (offre una versione gratuita e una premium)
- **Vendita di prodotti** o **servizi**

Quando si vende un servizio, è cruciale identificare il cliente (B2B, B2C, C2C) e capire se la sua scelta è motivata dal **prezzo** o dalla **fiducia**. Ad esempio, per servizi standardizzati il prezzo è il fattore principale, mentre per settori come quello sanitario o militare la fiducia è fondamentale.

---

# 2. Vendita di Prodotto vs Servizio

- **Servizio (Consulenza) →** si basa sulla vendita di competenze. I costi di setup sono bassi, ma è un modello *human-intensive* che scala poco, poiché raddoppiare i clienti richiede di raddoppiare il personale.
    
    <aside>
    🚨
    
    **Problema: Flusso di Cassa**
    
    Nelle aziende di servizi, i costi operativi (stipendi e bollette) sono ricorrenti e devono essere pagati regolarmente. Tuttavia, i pagamenti dei clienti sono spesso dilazionati nel tempo, creando un disallineamento tra il momento in cui l'azienda paga e quello in cui viene pagata. Questa mancanza di liquidità può portare a una crisi o al fallimento, anche in presenza di contratti redditizi.
    
    </aside>
    
    I clienti scelgono un servizio di consulenza basandosi principalmente su due fattori:
    
    - **Prezzo**: È l'elemento più importante quando il servizio offerto è standard e facilmente confrontabile, come nel caso di componenti elettroniche.
    - **Fiducia**: La fiducia prevale sul prezzo in ambiti critici in cui la competenza e l'affidabilità sono essenziali. Esempi sono i settori **sanitario**, **FinTech** o **militare**, dove le certificazioni e la reputazione del fornitore sono fondamentali.
- **Prodotto** → è un modello altamente scalabile, perché il prodotto viene creato una volta e venduto molteplici volte. Richiede un grande investimento iniziale e comporta un **rischio d'impresa** molto più alto, con nessuna garanzia di ritorno. Spesso, il primo prototipo o un software creato per un singolo cliente non è ancora un prodotto vendibile.

---

# 3. Analisi dei Costi e Metriche di Valutazione

## 3.1. Classificazione dei Costi

- **Costi diretti**: costi che si possono imputare direttamente a un progetto (es. ore di lavoro di un dipendente).
- **Costi indiretti**: costi non direttamente legati a un singolo progetto (es. spese di gestione, affitto ufficio).

Un altro modo per definire i costi sono:

- **Costi operativi (OPEX)**: costi ricorrenti a breve termine, come stipendi, bollette e materiali di consumo.
- **Costi in conto capitale (CAPEX)**: spese non ricorrenti su beni con una vita utile superiore a un anno (es. licenze decennali, costruzione di un data center). Questi costi vengono ammortizzati nel tempo.

## 3.2. Metriche di Valutazione

- **Total Cost of Ownership (TCO)**: misura il costo totale di un'opzione, sommando tutti i costi diretti e indiretti su un orizzonte temporale (es. 3, 5 o 7 anni). È uno strumento molto utilizzato per confrontare diverse alternative, come ad esempio la scelta tra un data center on-premise e una soluzione cloud.
    
    Per calcolare la TCO in modo accurato, è fondamentale includere anche i **costi nascosti**. Questi costi non sono immediatamente evidenti, ma hanno un impatto significativo, come il periodo di formazione per il personale o i cali di produttività durante l'adozione di un nuovo software.
    
    Tuttavia, la TCO presenta dei limiti importanti:
    
    - È una metrica che si concentra unicamente sulle spese e non tiene conto dei potenziali ricavi o benefici di un progetto.
    - Non considera la **tempistica dei costi**. Due opzioni con la stessa TCO totale possono avere un flusso di cassa molto diverso, ad esempio una con un costo iniziale elevato e l'altra con costi ricorrenti più alti.
    
    *Example:*
    
    ![image.png](3%20Modelli%20Economici/image.png)
    
- **Return on Investment (ROI)**: è un indice che misura il ritorno atteso da un investimento, mettendo in relazione i guadagni con le spese. È una metrica utile per confrontare progetti diversi e per prevedere il **flusso di cassa** di un'iniziativa. Per calcolare il ROI è necessario considerare tutti i costi e i ritorni attesi. Un errore comune è sottovalutare gli investimenti, non tenendo conto di elementi come:
    - Il **costo del personale**, ovvero le ore di lavoro dei dipendenti impiegate nel progetto
    - Il **costo del capitale** o gli interessi da pagare sui debiti contratti per l'investimento
    - L'**inflazione**, che introduce un costo "nascosto" legato al deprezzamento del valore del denaro nel tempo
    
    Il ROI viene calcolato con la seguente formula:
    
    $$
    ROI = \frac{\text{Revenue}}{\text{Investment}}\cdot 100
    $$
    
    Il ROI consente di creare un grafico che illustra il flusso di cassa di un progetto nel tempo, aiutando a identificare due punti chiave:
    
    - Il **Time to Market** (TTM), ovvero il momento in cui si raggiunge il picco delle spese e la tendenza si inverte, iniziando a generare un flusso di cassa positivo.
    - Il **Break-even point**, ovvero il momento in cui il progetto recupera tutte le spese sostenute e inizia a generare profitto.
    
    ![image.png](3%20Modelli%20Economici/5abafe8b-d8c8-46fe-ac5a-c5a66c3f2c71.png)
    
    Per monitorare l'andamento di un'iniziativa, si creano solitamente diversi scenari (ottimista, pessimista e intermedio). Si definiscono poi delle **milestone** per misurare i progressi e valutare se il progetto è in linea con le stime iniziali o se si sta discostando in termini di costi e tempi.
    
    ![image.png](3%20Modelli%20Economici/image%201.png)
    
- **Cost Benefits Analysis (CBA):** è uno strumento utilizzato per valutare la **convenienza economica e sociale** di un progetto, investimento o decisione. Consiste nel **confrontare i costi sostenuti** con i **benefici attesi** (sia monetari sia non monetari), mostrando a ogni costo quale beneficio corrisponde.

---

# 4. Considerazioni sul Cloud

Il cloud si distingue dagli scenari tradizionali per la sua **estrema velocità di cambiamento** e l'importanza fondamentale dell'**elasticità**. La rigidità, al contrario, rappresenta un costo significativo.

**N.B.** Quando andiamo a valutare i costi da ammortizzare dobbiamo distribuirli in un periodo di tempo minore.

## 4.1. Convenienza dell’adozione del Cloud

L'adozione del cloud può ridurre i costi iniziali, un beneficio notevole per le startup che possono concentrarsi sul loro *core business*. Sfruttare soluzioni **PaaS (Platform-as-a-Service)** permette di utilizzare API preesistenti per accelerare i tempi di sviluppo e ridurre il **time to market**. Le aziende senza economie di scala interne trovano nel cloud un modo per accedere a risorse a prezzi competitivi. Inoltre, i fondi risparmiati sulla gestione dell'infrastruttura possono essere reinvestiti in ricerca e sviluppo, o nella crescita professionale del personale.

<aside>
🚨

**Rischi:**

Un pericolo critico è il **vendor lock-in**, ovvero una forte dipendenza da un singolo fornitore cloud. Per questo, molte aziende iniziano con un approccio di cloud pubblico per poi, una volta raggiunte dimensioni sufficienti, iniziare a pensare alla creazione di un proprio cloud.

Inoltre, è importante ricordare che il **Total Cost of Ownership (TCO)**, pur essendo uno strumento utile, può essere un'arma a doppio taglio. I casi della **Ford Pinto** e del **Boeing 737 MAX** dimostrano che la ricerca del costo più basso può portare a scelte rischiose. È essenziale che gli strumenti economici siano usati per supportare le decisioni, ma senza mai sostituire il giudizio e la valutazione etica.

</aside>

---