# E-commerce Catalog Harmonizer (AI-Driven ETL Pipeline)
Una pipeline industriale di backend puro per il Batch Processing, la Data Transformation e l'integrazione dati sviluppata a budget zero in ambiente locale. Il sistema è progettato per ingerire cataloghi e-commerce grezzi forniti da partner esteri, eseguirne la normalizzazione semantica e la traduzione tramite modelli LLM, e memorizzare i dati strutturati in un database relazionale, esportando infine un report finale pulito.

## 🛠️ Stack Tecnologico & Architettura
*   **Orchestratore/Backend:** n8n (Esecuzione self-hosted in locale)
*   **Database:** PostgreSQL 15 (Data persistence per il catalogo armonizzato)
*   **Engine IA:** Groq Cloud API (Modello `groq/compound-mini` compatibile con standard OpenAI)
*   **Ambiente d'Infrastruttura:** Docker Desktop (Isolamento dei servizi e Docker Network dedicata)
*   **Database Client:** Beekeeper Studio (Data inspection e amministrazione schemi)

## 🔄 Flusso Logico dei Dati
Il workflow gestisce un'architettura ETL asincrona sequenziale forzata per garantire l'integrità dei dati:

1.  **Pipeline di Ingestione e Trasformazione (Lotti):** Manual/Schedule Trigger ➔ Read/Write Files from Disk (Ingestione del file `fornitore_grezzo.csv` dal Desktop tramite Docker Volume condiviso `/home/node/.n8n-files`) ➔ Extract from File (Parsing in oggetti JSON) ➔ Loop Over Items (Batch Size = 1 per isolamento computazionale e gestione Rate Limiting) ➔ Groq API (Normalizzazione semantica: traduzione descrizioni, standardizzazione taglie in formato internazionale e rimappatura categorie fornitore su macro-aree e-commerce con `Temperature = 0`) ➔ Code Node (Parsing JS avanzato ed escaping degli apostrofi) ➔ PostgreSQL (Inserimento dei record via `INSERT`).
2.  **Pipeline di Esportazione (Finale):** Attivata automaticamente alla fine delle transazioni del database ➔ PostgreSQL (Query SQL `SELECT` parametrizzata con ordinamento incrementale) ➔ Convert to File (Generazione dinamica del foglio di calcolo XLSX) ➔ Read/Write Files from Disk (Rilascio immediato del file `catalogo_armonizzato_finale.xlsx` sul Desktop di Windows).

## 🔒 Considerazioni sulla Sicurezza & Cybersecurity
Il perimetro infrastrutturale è isolato localmente all'interno della rete Docker. A livello di sicurezza del codice, il nodo JavaScript esegue un controllo e una pulizia preventiva delle stringhe di testo generate dall'IA. Questo processo applica l'escaping automatico dei caratteri speciali (raddoppio degli apostrofi come in *Resistente all''acqua*), neutralizzando sul nascere potenziali vulnerabilità di SQL Injection accidentale e prevenendo anomalie sintattiche che provocherebbero il roll-back o il crash delle transazioni sul database.

## 📈 Scalabilità & Analisi dei Costi (Local vs Cloud TCO)
Progetto originariamente ingegnerizzato in locale per ottimizzazione delle risorse hardware e gestione di file di test a budget zero.

*   **Analisi dei costi Cloud (VPS):** Per una messa in produzione aziendale su larga scala, l'intera suite Docker (n8n + Postgres) prevede la migrazione su una VPS Linux (es. Aruba Cloud / Hetzner) a un costo stimato di ~5.00€/mese, mantenendo i costi fissi abbattuti grazie al piano Free Tier delle API di Groq.
*   **Scalabilità Verticale:** Il design pattern basato su `Loop Over Items` con elaborazione a lotti permette alla pipeline di scalare da 3 a oltre 10.000 righe di catalogo senza richiedere un aumento della RAM del server, processando i record in modo sequenziale ed efficiente.

## 👨‍💻 Note di Sviluppo
Ho sviluppato questo secondo progetto per dimostrare una forte skill in ambito aziendale B2B: l'integrazione dell'Intelligenza Artificiale Generativa all'interno di pipeline dati industriali. Durante lo sviluppo locale su Docker ho dovuto combattere con i severi blocchi di sicurezza di n8n v1+, che vietavano l'accesso a cartelle esterne all'host, problema risolto riconfigurando i volumi di Docker Compose direttamente sulla directory blindata `/home/node/.n8n-files`. Inoltre, ho affrontato e risolto un bug di duplicazione dati (9 righe al posto di 3 nell'Excel) causato dall'accumulo asincrono della memoria di n8n nei cicli di loop, risolto bypassando il ramo nativo `done` (soggetto a Race Condition) e forzando la sequenzialità hardware tramite l'opzione `Execute Once` sul nodo di SELECT superiore.
