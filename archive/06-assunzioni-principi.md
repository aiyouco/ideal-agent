# Assunzioni e Principi Fondamentali di Design

**Stato del Documento:** 🟢 Completo
**Ultimo Aggiornamento:** 2025-11-10
**Autore:** Kilo Code

## Indice

1. [Panoramica](#panoramica)
2. [Assunzioni Fondamentali](#assunzioni-fondamentali)
3. [Principi di Design Core](#principi-di-design-core)
4. [Vincoli Architetturali](#vincoli-architetturali)
5. [Assunzioni Tecnologiche](#assunzioni-tecnologiche)
6. [Assunzioni Operative](#assunzioni-operative)
7. [Assunzioni di Sicurezza](#assunzioni-di-sicurezza)
8. [Assunzioni di Prestazione](#assunzioni-di-prestazione)
9. [Razionale di Design](#razionale-di-design)

---

## Panoramica

Questo documento esplicita tutte le assunzioni e i principi fondamentali che stanno alla base dell'architettura dell'agente ideale. Rendere esplicite le assunzioni è critico per:

1. **Comprendere le decisioni di design**: Perché certe scelte sono state fatte
2. **Identificare i rischi**: Dove le assunzioni potrebbero fallire
3. **Abilitare la validazione**: Testare le assunzioni contro la realtà
4. **Facilitare l'evoluzione**: Sapere cosa può cambiare senza rompere il sistema
5. **Comunicare l'intento**: Garantire che tutti gli stakeholder comprendano le fondamenta

### Categorie di Assunzioni

```
Assunzioni Fondamentali
├── Sui LLM (capacità, limitazioni)
├── Sui Task (complessità, variabilità)
├── Sull'Ambiente (risorse, vincoli)
├── Sugli Utenti (aspettative, comportamenti)
└── Sullo Stato della Tecnologia (cosa esiste, cosa è possibile)

Principi di Design
├── Performance First
├── Tracciabilità e Riproducibilità
├── Testabilità e Benchmarking
├── Modularità e Componibilità
├── Sicurezza e Safety
└── Osservabile di Default
```

---

## Assunzioni Fondamentali

### Assunzioni sui LLM

#### A1: Capacità dei LLM

**Assunzione**: I LLM moderni (GPT-4, Claude 3.5, ecc.) possono:
- Comprendere e generare linguaggio naturale a livello quasi umano
- Eseguire ragionamento multi-step con esplicito chain-of-thought
- Seguire istruzioni e vincoli complessi
- Generalizzare da esempi few-shot
- Gestire input multi-modali (testo, immagini, codice)

**Razionale**: Validato da estesa ricerca empirica e deployment in produzione

**Rischio**: I modelli possono fallire su:
- Domini nuovi senza dati di training
- Input avversariali progettati per sfruttare debolezze
- Task che richiedono ragionamento matematico o logico preciso
- Finestre di contesto molto lunghe (degradazione dell'attenzione)

**Mitigazione**:
- Usare strumenti di verifica per output critici
- Implementare strategie di fallback
- Validare output contro schemi
- Spezzare contesti lunghi in chunk gestibili

#### A2: Limitazioni dei LLM

**Assunzione**: I LLM hanno limitazioni fondamentali:
- Nessuna vera conoscenza del mondo (solo pattern dai dati di training)
- Nessun apprendimento online durante l'inferenza (pesi congelati)
- Output stocastici (stesso input → output diversi)
- Limiti della finestra di contesto (8K-128K token)
- Inferenza lenta (secondi per generazione)
- Inferenza costosa ($0.001-0.10 per 1K token)
- Non possono eseguire codice nativamente (necessitano strumenti)
- Non possono verificare perfettamente i propri output (rischio allucinazione)

**Razionale**: Intrinseco alle attuali architetture LLM

**Impatto sul Design**:
- Sistemi di memoria esterni richiesti (non si può fare affidamento solo sul contesto)
- Strumenti richiesti per capacità esterne
- Layer di verifica e validazione necessari
- Formati di output strutturati per ridurre ambiguità
- Caching e ottimizzazione per gestire costi
- Strategie di sampling multiplo per decisioni critiche

#### A3: LLM come Motore di Ragionamento

**Assunzione**: I LLM sono usati meglio come:
- Motori di ragionamento in linguaggio naturale
- Sistemi di riconoscimento e matching di pattern
- Learner few-shot
- Follower di istruzioni flessibili

**NON come**:
- Database (usare storage esterno)
- Calcolatori (usare strumenti)
- Macchine a stati deterministiche (aggiungere layer di controllo)
- Sistemi real-time (troppo lenti)

**Implicazione Architetturale**: Sistema ibrido dove l'LLM gestisce il ragionamento, mentre sistemi specializzati gestiscono storage, computazione e logica deterministica.

---

### Assunzioni sui Task

#### A4: Complessità dei Task

**Assunzione**: I task vanno da:
- **Semplici**: Singolo step, ben definiti (es., "Quanto fa 2+2?")
- **Medi**: Multi-step, qualche ambiguità (es., "Riassumi questo articolo")
- **Complessi**: Molti step, alta ambiguità, richiedono pianificazione (es., "Costruisci una web app")
- **Open-ended**: Scope indefinito, richiede esplorazione (es., "Ricerca sulla AI safety")

**Implicazione di Design**:
- Supportare sia modalità reattiva (semplice) che di pianificazione (complessa)
- Rilevamento di complessità adattivo
- Allocazione risorse basata sulla complessità
- Strategie diverse per diversi livelli di complessità

#### A5: Decomposizione dei Task

**Assunzione**: I task complessi possono essere decomposti in:
- Subtask con obiettivi chiari
- Dipendenze tra subtask
- Scope gestibile per subtask

**Rischio**: Alcuni task hanno:
- Decomposizione poco chiara (confini ambigui)
- Dipendenze circolari
- Complessità emergente (somma > parti)

**Mitigazione**:
- Raffinamento iterativo della decomposizione
- Rilevamento impasse e ripianificazione
- Human-in-the-loop per casi ambigui

#### A6: Criteri di Successo dei Task

**Assunzione**: I task hanno criteri di successo misurabili:
- Risultati osservabili
- Proprietà testabili
- Stati verificabili

**Realtà**: Molti task hanno:
- Successo soggettivo (soddisfazione utente)
- Successo parziale (alcuni obiettivi raggiunti)
- Feedback ritardato (successo conosciuto dopo)

**Supporto Architetturale**:
- Metriche di successo multiple
- Stima della confidenza
- Tracciamento completamento parziale
- Integrazione feedback utente

---

### Assunzioni sull'Ambiente

#### A7: Disponibilità Risorse

**Assunzione**: L'agente ha accesso a:
- Compute sufficiente (CPU, GPU)
- Memoria sufficiente (RAM, storage)
- Connettività di rete
- Servizi esterni (API, database)

**Realtà**: Le risorse possono essere:
- Limitate (vincoli di costo)
- Variabili (instabilità di rete)
- In competizione (sistemi multi-tenant)

**Risposta di Design**:
- Budget di risorse espliciti
- Degradazione graziosa sotto limiti
- Meccanismi di prioritizzazione
- Caching e ottimizzazione

#### A8: Disponibilità Strumenti

**Assunzione**: L'agente ha accesso a strumenti per:
- Operazioni file system
- Richieste di rete
- Esecuzione codice
- Query database
- Automazione browser
- Chiamate API

**Rischio**:
- Gli strumenti possono fallire o essere non disponibili
- Gli strumenti possono avere rate limit
- Gli strumenti possono richiedere autenticazione
- Gli strumenti possono cambiare interfaccia

**Mitigazione**:
- Registro strumenti con versioning
- Strumenti di fallback
- Gestione errori e logica retry
- Layer di astrazione interfacce

#### A9: Ambiente di Esecuzione

**Assunzione**: L'agente gira in un ambiente che:
- Persiste dati tra le sessioni
- Fornisce esecuzione isolata (sandboxing)
- Permette operazioni long-running
- Supporta esecuzione asincrona

**Requisito Architetturale**:
- Layer di storage persistente
- Meccanismi di isolamento (container, VM)
- Motore di esecuzione async
- Checkpointing dello stato

---

### Assunzioni sugli Utenti

#### A10: Intento Utente

**Assunzione**: Gli utenti forniscono:
- Obiettivi sufficientemente chiari (non perfettamente specificati)
- Criteri di successo ragionevoli
- Contesto rilevante
- Feedback tempestivo

**Realtà**: Gli utenti possono:
- Dare istruzioni ambigue
- Cambiare requisiti a metà task
- Fornire vincoli contraddittori
- Avere aspettative irrealistiche

**Risposta Agente**:
- Domande di chiarimento
- Conferma esplicita della comprensione
- Raffinamento iterativo
- Gestione delle aspettative

#### A11: Modello di Interazione Utente

**Assunzione**: Gli utenti interagiscono via:
- Linguaggio naturale (primario)
- Input strutturati (secondario)
- Feedback e correzioni
- Gate di approvazione per azioni critiche

**Implicazione di Design**:
- Comprensione NL come capacità core
- Fallback strutturati per precisione
- Loop di feedback a punti chiave
- Conferme di sicurezza

#### A12: Fiducia Utente

**Assunzione**: La fiducia deve essere guadagnata attraverso:
- Trasparenza (spiegare decisioni)
- Affidabilità (comportamento consistente)
- Sicurezza (nessuna azione dannosa)
- Competenza (raggiungere obiettivi)

**Requisito Architetturale**:
- Tracce di ragionamento esplicite
- Comunicazione dell'incertezza
- Guardrail di sicurezza
- Metriche di prestazione

---

## Principi di Design Core

### Principio 1: Performance First

**Dichiarazione**: Tutte le decisioni architetturali prioritizzano le prestazioni del sistema: latenza, throughput, efficienza delle risorse.

**Razionale**: L'agente deve essere pratico e utilizzabile. Sistemi lenti o costosi non saranno adottati.

**Applicazione**:
- Cachare tutto il possibile
- Parallelizzare operazioni indipendenti
- Streammare output quando possibile
- Usare modelli veloci quando sufficiente
- Ottimizzare prompt per efficienza token
- Batch operazioni simili
- Lazy evaluation

**Trade-off Accettati**:
- Qualche complessità nell'implementazione
- Più componenti infrastrutturali
- Carico di test aggiuntivo

**Metriche**:
- p50, p95, p99 latenza
- Token per task
- Costo per task
- Throughput task concorrenti

---

### Principio 2: Tracciabilità e Riproducibilità

**Dichiarazione**: Ogni azione, decisione e cambio di stato deve essere tracciabile e riproducibile.

**Razionale**: Richiesto per:
- Debug fallimenti
- Audit decisioni
- Conformità normativa
- Validazione scientifica
- Apprendimento dall'esperienza

**Applicazione**:
- Tracce di esecuzione strutturate
- ID univoci per tutte le operazioni
- Log immutabili
- Capacità di replay deterministico
- Versioning di tutti gli artifact
- Registrazione di tutte le fonti di casualità

**Requisiti Architetturali**:
- Sistema di tracing distribuito
- Storage e query delle tracce
- Infrastruttura di replay
- Version control per tutti i componenti

**Cosa Deve Essere Tracciato**:
- Chiamate LLM (prompt, risposte, parametri)
- Esecuzioni strumenti (input, output, errori)
- Operazioni memoria (letture, scritture)
- Transizioni di stato
- Utilizzo risorse
- Informazioni temporali

---

### Principio 3: Testabilità e Benchmarking

**Dichiarazione**: L'architettura deve abilitare test completi a tutti i livelli e benchmarking sistematico.

**Razionale**: Il comportamento dell'agente deve essere validato contro i requisiti e confrontato con alternative.

**Applicazione**:
- Unit test per componenti
- Test di integrazione per workflow
- Test end-to-end per task
- Benchmark di performance
- Test di regressione
- Generazione test sintetici

**Requisiti di Testabilità**:
- Dependency injection (mock sistemi esterni)
- Modalità deterministiche (seed random fissi)
- Fixture e factory di test
- Librerie di assertion per output agente
- Ambienti di simulazione

**Requisiti di Benchmarking**:
- Suite di task standard
- Metriche di valutazione
- Confronti baseline
- Test significatività statistica
- Profiling delle prestazioni

---

### Principio 4: Modularità e Componibilità

**Dichiarazione**: Sistema composto da moduli loosely-coupled con interfacce ben definite.

**Razionale**: Abilita:
- Sviluppo e test indipendenti
- Miglioramento incrementale
- Riusabilità tra contesti
- Sostituibilità (swap implementazioni)
- Sviluppo parallelo da parte di team

**Confini dei Moduli**:
- Reasoning Engine
- Sistema di Memoria
- Orchestrazione Strumenti
- Osservabilità
- Sicurezza e Safety

**Requisiti Interfacce**:
- Contratti chiaramente specificati
- Compatibilità versioni
- Gestione errori
- Gestione risorse
- Gestione del ciclo di vita

**Anti-pattern da Evitare**:
- Oggetti god (troppa responsabilità)
- Accoppiamento stretto (dipendenze dirette)
- Stato globale (dipendenze implicite)
- Contratti impliciti (assunzioni non documentate)

---

### Principio 5: Sicurezza e Safety

**Dichiarazione**: Sicurezza e safety integrate dalla base, non aggiunte dopo.

**Razionale**: I sistemi AI possono causare danni reali. La prevenzione è meglio della rimediazione.

**Meccanismi di Safety**:
- Validazione input (tutti i confini)
- Filtraggio output (risposte LLM)
- Sandboxing strumenti (esecuzione isolata)
- Limiti risorse (prevenire DoS)
- Gate di approvazione (azioni critiche)
- Capacità di rollback (annullare modifiche)

**Meccanismi di Sicurezza**:
- Autenticazione e autorizzazione
- Crittografia (dati a riposo e in transito)
- Logging di audit (tutte le operazioni sensibili)
- Rate limiting (prevenire abuso)
- Gestione segreti (no credenziali hardcoded)
- Scanning vulnerabilità

**Modello di Minaccia**:
- Utenti malintenzionati (tentativo sfruttare agente)
- Strumenti compromessi (attacchi supply chain)
- Prompt injection (bypass safety)
- Data leakage (informazioni sensibili)
- Esaurimento risorse (DoS)

---

### Principio 6: Osservabile di Default

**Dichiarazione**: Tutti i componenti emettono telemetria di default. L'osservabilità non è opzionale.

**Razionale**: Non si può migliorare ciò che non si misura. I problemi in produzione richiedono visibilità.

**Pilastri Osservabilità**:
- **Log**: Strutturati, ricercabili, correlati
- **Metriche**: Contatori, gauge, istogrammi
- **Tracce**: Distribuite, end-to-end
- **Eventi**: Cambi di stato, operazioni significative

**Cosa Osservare**:
- Ciclo di vita richiesta (inizio, progresso, completamento)
- Utilizzo risorse (CPU, memoria, I/O, costo)
- Tassi e tipi di errore
- Distribuzioni latenza
- Tassi di cache hit
- Performance modello (qualità, consistenza)
- Interazioni utente

**Requisiti Osservabilità**:
- Overhead basso (<5% impatto prestazioni)
- Sampling per eventi alto volume
- Aggregazione e rollup
- Dashboard real-time
- Alerting su anomalie

---

### Principio 7: Fallire con Grazia

**Dichiarazione**: Il sistema degrada con grazia sotto fallimento, fornendo risultati parziali quando possibile.

**Razionale**: Il fallimento completo è raramente necessario. Il progresso parziale è meglio di niente.

**Strategie di Degradazione**:
- Ritornare risultati parziali con punteggi di confidenza
- Usare dati cached/stale quando fresh non disponibile
- Fallback a modelli più semplici
- Ridurre qualità per velocità
- Saltare operazioni non critiche

**Cosa Mai Degrada**:
- Controlli di safety (sempre imposti)
- Autenticazione (sempre richiesta)
- Integrità dati (mai corrotta)
- Logging di audit (sempre registrato)

---

### Principio 8: Ottimizzare per il Caso Comune

**Dichiarazione**: Rendere le operazioni comuni veloci e semplici, anche se le operazioni rare sono più lente.

**Razionale**: Le prestazioni in scenari tipici contano più dei rari casi limite.

**Casi Comuni**:
- Task semplici, singolo step (80% dell'uso)
- Operazioni read-heavy (90% accesso memoria)
- Cache hit (target 80%+ hit rate)
- Esecuzioni strumenti di successo (99%+ tasso successo)

**Ottimizzazioni**:
- Fast path per casi semplici
- Caching per operazioni ripetute
- Esecuzione speculativa
- Precomputazione risultati comuni

**Casi Rari** (possono essere più lenti):
- Pianificazione complessa multi-step
- Cache miss
- Fallimenti strumenti che richiedono retry
- Recupero errori

---

### Principio 9: Esplicito Sopra Implicito

**Dichiarazione**: Rendere tutto esplicito. Nessuno stato nascosto, nessuna magia, nessuna sorpresa.

**Razionale**: I sistemi espliciti sono più facili da comprendere, debuggare e mantenere.

**Elementi Espliciti**:
- Transizioni di stato (FSM chiara)
- Budget risorse (limiti espliciti)
- Dipendenze (dichiarate in anticipo)
- Errori (tipizzati e categorizzati)
- Assunzioni (documentate)
- Trade-off (spiegati)

**Evitare**:
- Stato implicito (variabili globali)
- Magic numbers (costanti non documentate)
- Effetti collaterali (modifiche inaspettate)
- Costi nascosti (utilizzo risorse sorpresa)

---

### Principio 10: Design per il Cambiamento

**Dichiarazione**: L'architettura deve accomodare il cambiamento senza riscritture maggiori.

**Razionale**: I requisiti evolvono, la tecnologia migliora, i modelli avanzano.

**Design Friendly al Cambiamento**:
- Interfacce sopra implementazioni
- Configurazione sopra codice
- Composizione sopra ereditarietà
- Pattern strategy per algoritmi
- Architettura plugin per estensioni

**Cambiamenti Attesi**:
- Nuovi modelli LLM (migliori, più economici, più veloci)
- Nuovi tipi di strumenti (capacità)
- Nuovi backend memoria (storage)
- Nuove strategie prompting (tecniche)
- Nuove metriche di valutazione (standard)

**Stabile vs. Volatile**:
- Stabile: Astrazioni core (Goal, Plan, Memory, Tool)
- Volatile: Implementazioni (quale LLM, quale DB, quale algoritmo)

---

## Vincoli Architetturali

### Vincoli Hard

#### C1: Limite Finestra di Contesto

**Vincolo**: Le finestre di contesto LLM sono finite (8K-128K token).

**Implicazione**: Non si può memorizzare cronologia illimitata nel contesto.

**Risposta Architetturale**:
- Sistemi di memoria esterni
- Riassunto vecchio contesto
- Recupero informazioni rilevanti
- Memoria di lavoro con eviction

#### C2: Latenza di Inferenza

**Vincolo**: L'inferenza LLM richiede secondi per chiamata.

**Implicazione**: Non si possono fare centinaia di chiamate LLM sequenziali.

**Risposta Architetturale**:
- Minimizzare chiamate LLM
- Parallelizzare chiamate indipendenti
- Cachare risultati
- Usare modelli più piccoli quando possibile
- Streammare output

#### C3: Costo

**Vincolo**: L'inferenza LLM costa denaro ($0.001-0.10 per 1K token).

**Implicazione**: L'esplorazione illimitata è proibitivamente costosa.

**Risposta Architetturale**:
- Budget di costo espliciti
- Trade-off costo-qualità
- Prompting efficiente
- Caching risultati
- Routing modello (economico per semplice, costoso per difficile)

#### C4: Determinismo

**Vincolo**: I LLM sono stocastici (temperature > 0).

**Implicazioni**: Lo stesso input può produrre output diversi.

**Risposta Architetturale**:
- Output strutturati riducono variabilità
- Validazione schema impone formato
- Campioni multipli per decisioni critiche
- Temperature=0 per modalità deterministica
- Seed random espliciti per riproducibilità

### Vincoli Soft

#### C5: Affidabilità Strumenti

**Vincolo**: Gli strumenti esterni possono fallire, essere lenti o non disponibili.

**Implicazione**: Non si può assumere che gli strumenti funzionino sempre.

**Risposta Architetturale**:
- Retry con backoff
- Strumenti di fallback
- Circuit breaker
- Limiti timeout
- Classificazione errori

#### C6: Limiti Risorse

**Vincolo**: Compute, memoria e tempo sono limitati in pratica.

**Implicazione**: Non si possono cercare tutte le possibilità o memorizzare tutto.

**Risposta Architetturale**:
- Budget risorse
- Ricerca euristica (non esaustiva)
- Algoritmi approssimativi
- Gestione memoria (dimenticanza)

---

## Assunzioni Tecnologiche

### TA1: Modelli Linguistici

**Stato Corrente** (2024-2025):
- GPT-4, Claude 3.5 Sonnet, Gemini 1.5 Pro
- Finestre di contesto 128K
- Multi-modale (testo, immagini, codice)
- Supporto function calling
- Modalità output strutturato

**Assunzioni**:
- I modelli continueranno a migliorare (migliori, più veloci, più economici)
- Le finestre di contesto cresceranno (1M+ token in arrivo)
- La multi-modalità si espanderà (video, audio)
- Il function calling diventerà più affidabile

**L'Architettura Deve Supportare**:
- Scambio modelli (non hardcodare su un modello)
- Modelli multipli (routing basato su task)
- Versioning modelli (tracciare quale modello usato quando)

### TA2: Database Vettoriali

**Assunti Disponibili**:
- Pinecone, Weaviate, Qdrant, Milvus
- Ricerca vettoriale scala miliardi
- Approximate nearest neighbors (ANN)
- Filtraggio metadata

**Assunzioni**:
- Latenza query sub-100ms
- Recall 90%+ con ANN
- Scalabilità orizzontale

### TA3: Database Grafi

**Assunti Disponibili**:
- Neo4j, Memgraph, DGraph
- Property graph con nodi/archi tipizzati
- Linguaggi query Cypher o Gremlin
- Algoritmi su grafi (shortest path, PageRank, ecc.)

**Assunzioni**:
- Query sub-100ms per attraversamento 3-hop
- Milioni di nodi/archi
- Transazioni ACID

### TA4: Containerizzazione

**Assunti Disponibili**:
- Docker, Kubernetes
- Orchestrazione container
- Limiti risorse (CPU, memoria)
- Esecuzione isolata

**Assunzioni**:
- Startup container <100ms (con caching)
- Condivisione risorse efficiente
- Scaling orizzontale

---

## Assunzioni Operative

### OA1: Modello di Deployment

**Assunzione**: L'agente viene deployato come:
- Servizio cloud (primario)
- On-premise (secondario)
- Dispositivi edge (terziario)

**Implicazioni**:
- Chiamate di rete a API LLM (cloud)
- Inferenza modello locale (on-prem, edge)
- Approcci ibridi (routing)

### OA2: Scala

**Assunzione**: L'agente deve gestire:
- 1-10 utenti concorrenti (development)
- 10-100 utenti concorrenti (small team)
- 100-1000 utenti concorrenti (organization)
- 1000-10000 utenti concorrenti (product)

**Requisiti Architetturali**:
- Scalabilità orizzontale
- Servizi stateless (per scaling)
- Architettura shared-nothing
- Load balancing

### OA3: Disponibilità

**Assunzione**: Disponibilità richiesta:
- 99% (2-3 nines) per development
- 99.9% (3 nines) per produzione
- 99.99% (4 nines) per sistemi critici

**Requisiti Architetturali**:
- Ridondanza (no single point of failure)
- Health check
- Failover automatico
- Degradazione graziosa

### OA4: Ritenzione Dati

**Assunzione**: Dati ritenuti per:
- Tracce: 30-90 giorni (hot), 1 anno (cold)
- Metriche: 30 giorni (risoluzione 1s), 1 anno (risoluzione 1m)
- Log: 30 giorni (ricercabili), 1 anno (archiviati)
- Memoria episodica: Indefinito (o finché esplicitamente cancellato)
- Memoria semantica: Indefinito

**Implicazioni**:
- Storage a livelli (hot, warm, cold)
- Policy ciclo di vita dati
- Compressione e archiviazione
- Cancellazione conforme privacy

---

## Assunzioni di Sicurezza

### SA1: Modello di Minaccia

**Minacce Assunte**:
- Utenti malintenzionati che cercano di sfruttare l'agente
- Attacchi prompt injection
- Tentativi di esfiltrazione dati
- Esaurimento risorse (DoS)
- Attacchi supply chain (strumenti compromessi)

**Non Assunto**:
- Minacce insider con accesso root
- Accesso fisico ai server
- Exploit zero-day in infrastruttura core

### SA2: Confini di Fiducia

**Fidato**:
- Codice agente core (sviluppato internamente)
- Provider LLM verificati (OpenAI, Anthropic)
- Infrastruttura gestita (cloud provider)

**Non Fidato**:
- Input utente (sempre validare)
- API esterne (possono fallire o essere malevole)
- Strumenti terze parti (sandbox esecuzione)
- Codice fornito dall'utente (eseguire in isolamento)

### SA3: Dati Sensibili

**Assunzione**: L'agente può gestire:
- Informazioni personali utenti (PII)
- Dati confidenziali business
- Chiavi API e credenziali
- Codice proprietario

**Requisiti**:
- Crittografia a riposo e in transito
- Controllo accessi (RBAC)
- Logging di audit
- Minimizzazione dati (raccogliere solo il necessario)
- Policy di ritenzione (cancellare quando non più necessario)

---

## Assunzioni di Prestazione

### PA1: Requisiti Latenza

**Latenze Target**:
- Query semplice: <2 secondi (end-to-end)
- Task medio: <30 secondi
- Task complesso: <5 minuti
- Risposta interattiva: <500ms (primo token)

**Assunzioni**:
- 80% dei task sono semplici
- 15% sono medi
- 5% sono complessi

### PA2: Requisiti Throughput

**Throughput Target**:
- 100 task/secondo (picco)
- 50 task/secondo (sostenuto)

**Budget Risorse** (per task):
- Token: 10K-100K
- Tempo: 2s-5min
- Costo: $0.01-$1.00
- Memoria: 100MB-1GB

### PA3: Assunzioni Caching

**Tassi di Cache Hit Attesi**:
- Risposte LLM: 20-40% (per operazioni deterministiche)
- Recuperi memoria: 60-80% (riuso memoria di lavoro)
- Risultati strumenti: 40-60% (per strumenti idempotenti)

**Storage Cache**:
- Redis o simile (accesso sub-millisecondo)
- Scadenza basata su TTL
- Policy eviction LRU

---

## Razionale di Design

### Perché Architettura Ibrida (Cognitiva + LLM)?

**Decisione**: Combinare pattern di architetture cognitive (SOAR, ACT-R) con ragionamento LLM.

**Razionale**:
- Le architetture cognitive forniscono struttura e prevedibilità
- I LLM forniscono flessibilità e generalizzazione
- L'ibrido ottiene il meglio di entrambi i mondi

**Alternative Considerate**:
- LLM puro (troppo imprevedibile, difficile da debuggare)
- Architettura cognitiva pura (troppo fragile, richiede ingegneria manuale)

### Perché Memoria Esterna?

**Decisione**: Implementare sistemi di memoria separati (episodica, semantica, procedurale) piuttosto che fare affidamento sul contesto LLM.

**Razionale**:
- Finestra di contesto troppo piccola per memoria a lungo termine
- Il contesto è costoso (ogni token costa)
- La memoria esterna permette query strutturate
- Gestione memoria esplicita (consolidamento, dimenticanza)

**Alternative Considerate**:
- Solo contesto (capacità insufficiente)
- Modelli fine-tuned (troppo lenti, costosi da aggiornare)

### Perché Astrazione Strumenti?

**Decisione**: Interfaccia strumento unificata piuttosto che integrazioni hardcoded.

**Razionale**:
- Nuovi strumenti possono essere aggiunti senza cambiare codice
- Gli strumenti possono essere scambiati (es., diversi motori di ricerca)
- Gli strumenti possono essere testati indipendentemente
- Gli strumenti possono essere condivisi tra agenti

**Alternative Considerate**:
- Integrazioni hardcoded (inflessibili)
- LLM genera codice per chiamare API (non sicuro, inaffidabile)

### Perché Budget Risorse Espliciti?

**Decisione**: Ogni operazione ha budget espliciti token/tempo/costo.

**Razionale**:
- Previene costi fuori controllo
- Abilita trade-off costo/qualità
- Fornisce comportamento prevedibile
- Abilita pianificazione resource-aware

**Alternative Considerate**:
- Esecuzione illimitata (troppo rischioso)
- Solo limiti globali (inflessibile)

### Perché Tracce Strutturate?

**Decisione**: Tracce strutturate e tipizzate piuttosto che log free-form.

**Razionale**:
- Abilita analisi programmatica
- Supporta replay e debugging
- Più facile da interrogare e visualizzare
- Richiesto per conformità

**Alternative Considerate**:
- Solo log testuali (struttura insufficiente)
- Nessun tracing (impossibile debuggare)

---

## Conclusione

Queste assunzioni e principi formano le fondamenta dell'architettura dell'agente ideale. Guidano tutte le decisioni di design e forniscono un framework per la valutazione.

**Punti Chiave**:

1. **Assunzioni esplicite** abilitano validazione e gestione dei rischi
2. **Performance first** garantisce usabilità pratica
3. **Tracciabilità** abilita debugging e conformità
4. **Modularità** abilita sviluppo e test indipendenti
5. **Sicurezza** è integrata, non aggiunta
6. **Osservabilità** è non opzionale
7. **Architettura ibrida** combina struttura cognitiva con flessibilità LLM

**Documento Vivente**: Queste assunzioni dovrebbero essere rivisitate quando:
- La tecnologia evolve (nuovi modelli, nuovi strumenti)
- I requisiti cambiano (nuovi use case, nuovi vincoli)
- L'evidenza empirica si accumula (cosa funziona, cosa no)

---

## Riferimenti

1. Laird, J. E. (2012). The Soar Cognitive Architecture. MIT Press - Design principles chapter.
2. Anderson, J. R. (2007). How Can the Human Mind Occur in the Physical Universe? - Rational analysis framework.
3. Bass, L., Clements, P., & Kazman, R. (2012). Software Architecture in Practice (3rd ed.). - Architecture principles.
4. Newman, S. (2015). Building Microservices. O'Reilly - Modularity principles.
5. Kleppmann, M. (2017). Designing Data-Intensive Applications. O'Reilly - Data systems design.
6. OpenAI. (2024). GPT-4 System Card. - Model capabilities and limitations.
7. Anthropic. (2024). Claude 3.5 Sonnet Model Card. - Model characteristics.
