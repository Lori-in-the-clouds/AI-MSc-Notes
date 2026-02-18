# 1-Introduzione

Status: Done
Type: theory

# Big Data

<aside>
💡

Il termine **Big Data**, coniato tra il **2010 e il 2012**, si riferisce a dataset così grandi e complessi da non poter essere gestiti con strumenti tradizionali come i database relazionali. Anche se i dati possono essere scaricati su un PC comune, le query standard non riescono a processarli efficacemente.

</aside>

## Proprietà principali (3V)

- **Volume:** un elevato volume di dati permette di descrivere meglio un evento, ma rende i metodi tradizionali di gestione inefficaci.
- **Velocità:** l’analisi deve essere rapida per evitare che i dati diventino obsoleti o inutilizzabili.
- **Varietà:**  i dati possono provenire da fonti diverse e presentare valori nulli o differenze nelle unità di misura. Per questo è necessario applicare tecniche di normalizzazione e data cleaning.

## Pipeline della gestione dei dati

1. **Acquisizione dei dati**
2. **Pulizia dei dati ed estrazione di essi**
3. **Aggregazione e rappresentazione →** una volta ottenuti i dati puliti, si rimuovono le caratteristiche irrilevanti per l’analisi, riducendo il rumore.
4. **Modellazione →** si sceglie il modello più adatto per analizzare i dati e si imposta l’algoritmo.
5. **Interpretazione →**  il risultato prodotto dall’algoritmo deve essere compreso e contestualizzato.

<aside>
🚨

**Aspetti aggiuntivi nella gestione dei dataset:**

- **Eterogeneità dei dati:** le informazioni provengono da fonti diverse. È utile utilizzare la *provenance* per annotare le origini e valutarne l’affidabilità.
- **Privacy:** l’unione di dati da più sorgenti può rivelare informazioni personali, richiedendo attenzione alla tutela dei dati.
- **Chiarezza nella visualizzazione:** presentare i dati in modo chiaro e intuitivo è essenziale per un’analisi efficace (ad esempio, come nel caso del video di Walmart).
</aside>

---

# Data Scienze

La **data science** consiste nell’analisi dei dati con l’obiettivo di generare profitto. A differenza di altri usi dei dati (es. organizzare i treni senza finalità di guadagno), qui lo scopo principale è trarre vantaggio economico dalle informazioni raccolte.

## Compiti dei data scientists

I data scientists si occupano di:

- Analizzare il comportamento dei clienti per aumentare l’engagement (= livello di coinvolgimento e interazione di un utente con un prodotto, servizio o contenuto).
- Verificare la qualità del codice, ottimizzandolo e correggendo eventuali bug.
- Risolvere problemi specifici di dominio, creando modelli predittivi (es. rilevamento di malware, previsione del consumo elettrico).
- Preparare e, se necessario, sostituire i dataset non coerenti con il problema da affrontare.

---