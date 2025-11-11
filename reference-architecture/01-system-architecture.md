# Architettura di Sistema - Specifiche Complete

## Diagramma Generale

```
┌────────────────────────────────────────────────────────────────────────┐
│                    REFLECTIVE ADAPTIVE AGENT                            │
│                                                                         │
│  INPUT: Specifica del Task                                             │
│    │                                                                    │
│    ↓                                                                    │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │ COGNITIVE LAYER - Ragionamento & Decisioni                      │  │
│  │                                                                  │  │
│  │   ┌──────────────┐    ┌───────────────┐    ┌──────────────┐   │  │
│  │   │ Analisi      │    │ Planning      │    │ Execution    │   │  │
│  │   │ Goal         │───→│ Engine        │───→│ Engine       │   │  │
│  │   │              │    │               │    │              │   │  │
│  │   │ • Parse task │    │ • Decomposiz. │    │ • Esecuzione │   │  │
│  │   │ • Estrazione │    │ • Schedule    │    │ • Monitoring │   │  │
│  │   │   goal       │    │ • Gestione    │    │ • Adattam.   │   │  │
│  │   │ • Identif.   │    │   dipendenze  │    │ • Verifica   │   │  │
│  │   │   vincoli    │    │               │    │              │   │  │
│  │   └──────┬───────┘    └───────┬───────┘    └──────┬───────┘   │  │
│  │          │                    │                    │           │  │
│  │          └────────────┬───────┴────────────────────┘           │  │
│  │                       ↓                                        │  │
│  │                 ┌────────────────┐                             │  │
│  │                 │  Modulo        │                             │  │
│  │                 │  Reflection    │                             │  │
│  │                 │                │                             │  │
│  │                 │ • Analisi      │                             │  │
│  │                 │   performance  │                             │  │
│  │                 │ • Estrazione   │                             │  │
│  │                 │   pattern      │                             │  │
│  │                 │ • Aggiornamento│                             │  │
│  │                 │   strategie    │                             │  │
│  │                 └────────┬───────┘                             │  │
│  │                          │                                     │  │
│  └──────────────────────────┼─────────────────────────────────────┘  │
│                             ↓                                        │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │ MEMORY LAYER - Gestione Stato & Conoscenza                     │  │
│  │                                                                  │  │
│  │   ┌──────────────┐    ┌───────────────┐    ┌──────────────┐   │  │
│  │   │ Working      │    │ Episodic      │    │ Pattern      │   │  │
│  │   │ Memory       │    │ Memory        │    │ Cache        │   │  │
│  │   │              │    │               │    │              │   │  │
│  │   │ • Contesto   │    │ • Episodi     │    │ • Strategie  │   │  │
│  │   │   corrente   │    │   passati     │    │   apprese    │   │  │
│  │   │ • Variabili  │    │ • Risultati   │    │ • Pattern    │   │  │
│  │   │   attive     │    │ • Embeddings  │    │   successo   │   │  │
│  │   │ • Stato temp │    │ • Retrieval   │    │ • Euristiche │   │  │
│  │   │              │    │   similitudine│    │              │   │  │
│  │   └──────┬───────┘    └───────┬───────┘    └──────┬───────┘   │  │
│  │          │                    │                    │           │  │
│  │          └────────────────────┴────────────────────┘           │  │
│  │                               │                                │  │
│  └───────────────────────────────┼────────────────────────────────┘  │
│                                  ↓                                   │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │ CAPABILITY LAYER - Azioni Esterne & Intelligenza               │  │
│  │                                                                  │  │
│  │   ┌──────────────┐    ┌───────────────┐    ┌──────────────┐   │  │
│  │   │ Tool         │    │ Model         │    │ Safety       │   │  │
│  │   │ Registry     │    │ Router        │    │ Verifier     │   │  │
│  │   │              │    │               │    │              │   │  │
│  │   │ • Discovery  │    │ • Routing al  │    │ • Validazione│   │  │
│  │   │ • Binding    │    │   modello     │    │   input      │   │  │
│  │   │ • Esecuzione │    │   appropriato │    │ • Controllo  │   │  │
│  │   │ • Validazione│    │ • Ottimizz.   │    │   bounds     │   │  │
│  │   │              │    │   costi       │    │ • Enforce    │   │  │
│  │   │              │    │               │    │   safety     │   │  │
│  │   └──────┬───────┘    └───────┬───────┘    └──────┬───────┘   │  │
│  │          │                    │                    │           │  │
│  │          └────────────────────┴────────────────────┘           │  │
│  │                               │                                │  │
│  └───────────────────────────────┼────────────────────────────────┘  │
│                                  ↓                                   │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │ INFRASTRUCTURE LAYER - Observability & Gestione Risorse         │  │
│  │                                                                  │  │
│  │   ┌──────────────┐    ┌───────────────┐    ┌──────────────┐   │  │
│  │   │Observability │    │ Resource      │    │ Error        │   │  │
│  │   │ System       │    │ Manager       │    │ Handler      │   │  │
│  │   │              │    │               │    │              │   │  │
│  │   │ • Tracing    │    │ • Tracking    │    │ • Rilevazione│   │  │
│  │   │ • Metriche   │    │   budget      │    │ • Classific. │   │  │
│  │   │ • Logging    │    │ • Rate        │    │ • Recovery   │   │  │
│  │   │ • Monitoring │    │   limiting    │    │ • Escalation │   │  │
│  │   │              │    │ • Throttling  │    │              │   │  │
│  │   └──────────────┘    └───────────────┘    └──────────────┘   │  │
│  └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  OUTPUT: Risultato Task + Conoscenza Aggiornata                        │
└────────────────────────────────────────────────────────────────────────┘
```

## Flusso di Esecuzione - Loop Principale

```
INIZIO
  │
  ├─→ [1. RICEZIONE TASK]
  │      │
  │      ├─ Parse input
  │      ├─ Validazione formato
  │      └─ Estrazione parametri
  │      │
  ├─→ [2. ANALISI GOAL]
  │      │
  │      ├─ Identificazione obiettivi
  │      ├─ Estrazione vincoli
  │      ├─ Definizione criteri successo
  │      └─ Classificazione complessità
  │      │
  ├─→ [3. RECUPERO CONTESTO]
  │      │
  │      ├─ Query episodic memory (task simili passati)
  │      ├─ Controllo pattern cache (strategie apprese)
  │      └─ Caricamento working memory (sessione corrente)
  │      │
  ├─→ [4. PIANIFICAZIONE]
  │      │
  │      ├─ Generazione piano alto livello
  │      ├─ Decomposizione in subtask
  │      ├─ Identificazione dipendenze
  │      ├─ Stima risorse (token, tempo, costo)
  │      └─ Selezione strategia esecuzione
  │      │
  ├─→ [5. LOOP ESECUZIONE]
  │      │
  │      ├─ PER ogni step nel piano:
  │      │   │
  │      │   ├─→ [5a. Pre-esecuzione]
  │      │   │      ├─ Verifica sicurezza
  │      │   │      ├─ Controllo risorse
  │      │   │      └─ Decisione routing modello
  │      │   │
  │      │   ├─→ [5b. Esecuzione]
  │      │   │      ├─ Reasoning LLM (se necessario)
  │      │   │      ├─ Esecuzione tool (se necessario)
  │      │   │      └─ Cattura output
  │      │   │
  │      │   ├─→ [5c. Post-esecuzione]
  │      │   │      ├─ Verifica risultato
  │      │   │      ├─ Controllo criteri successo
  │      │   │      └─ Aggiornamento working memory
  │      │   │
  │      │   └─→ [5d. Adattamento]
  │      │          ├─ Se successo: Continua
  │      │          ├─ Se fallimento: Strategia recovery
  │      │          └─ Se bloccato: Ripianificazione
  │      │
  ├─→ [6. VERIFICA]
  │      │
  │      ├─ Validazione output finale
  │      ├─ Controllo criteri successo
  │      └─ Verifica sicurezza finale
  │      │
  ├─→ [7. REFLECTION]
  │      │
  │      ├─ Analisi performance episodio
  │      ├─ Estrazione pattern di successo
  │      ├─ Identificazione miglioramenti
  │      ├─ Aggiornamento pattern cache
  │      └─ Memorizzazione in episodic memory
  │      │
  └─→ [8. RESTITUZIONE RISULTATO]
```

## Interazione tra Componenti - Diagramma di Sequenza

```
User    Analisi   Planning   Memory    Model     Tool      Safety
 │      Goal      Engine     System    Router    Registry  Verifier
 │        │         │          │         │          │         │
 ├──task─→│         │          │         │          │         │
 │        │         │          │         │          │         │
 │        ├─goals──→│          │         │          │         │
 │        │         │          │         │          │         │
 │        │         ├─recupero→│         │          │         │
 │        │         │←─context─┤         │          │         │
 │        │         │          │         │          │         │
 │        │         ├─piano────┴─────────┤          │         │
 │        │         │                    │          │         │
 │        │         │        PER ogni step:         │         │
 │        │         │                    │          │         │
 │        │         ├────────────────────┼──────────┼──verif→│
 │        │         │                    │          │←─ok────┤
 │        │         │                    │          │         │
 │        │         ├────────chiamata modello──────→          │
 │        │         │←─────risposta──────┤          │         │
 │        │         │                    │          │         │
 │        │         ├─────────────────────chiamata tool──────→│
 │        │         │←────────────────────risultato─┤         │
 │        │         │                    │          │         │
 │        │         ├──salva risultato──→│          │         │
 │        │         │                    │          │         │
 │        │         │        FINE LOOP   │          │         │
 │        │         │                    │          │         │
 │        │         ├──reflection───────→│          │         │
 │        │         │←──pattern aggiornat─┤         │         │
 │        │         │                    │          │         │
 │←result─┴─────────┴────────────────────┴──────────┴─────────┘
```

## Architettura Flussi Dati

```
┌────────────────────────────────────────────────────────────────┐
│                         FLUSSI DATI                            │
│                                                                │
│  ┌──────────┐                                                 │
│  │  INPUT   │                                                 │
│  │   TASK   │                                                 │
│  └────┬─────┘                                                 │
│       │                                                       │
│       ↓                                                       │
│  ┌─────────────────────┐                                     │
│  │  LAYER VALIDAZIONE  │                                     │
│  │  ├─ Check schema    │                                     │
│  │  ├─ Sanitizzazione  │                                     │
│  │  └─ Detect injection│                                     │
│  └────┬────────────────┘                                     │
│       │                                                       │
│       ↓                                                       │
│  ┌─────────────────────┐       ┌──────────────┐             │
│  │  ESTRAZIONE GOAL    │──────→│   Working    │             │
│  │                     │       │   Memory     │             │
│  └────┬────────────────┘       └──────────────┘             │
│       │                                                       │
│       ↓                                                       │
│  ┌─────────────────────┐       ┌──────────────┐             │
│  │ RECUPERO CONTESTO   │←─────→│  Episodic    │             │
│  │                     │       │   Memory     │             │
│  └────┬────────────────┘       └──────────────┘             │
│       │                        ┌──────────────┐             │
│       │                   ┌───→│   Pattern    │             │
│       │                   │    │    Cache     │             │
│       ↓                   │    └──────────────┘             │
│  ┌─────────────────────┐ │                                  │
│  │   GENERAZIONE PIANO │─┘                                  │
│  │                     │                                    │
│  └────┬────────────────┘                                    │
│       │                                                      │
│       ↓                                                      │
│  ┌─────────────────────┐       ┌──────────────┐            │
│  │  EXECUTION ENGINE   │──────→│    Model     │            │
│  │                     │       │    Router    │            │
│  │  PER ogni step:     │       └──────────────┘            │
│  │    ├─ Route model   │                                   │
│  │    ├─ Chiama tools  │       ┌──────────────┐            │
│  │    ├─ Verif. output │──────→│    Tool      │            │
│  │    └─ Aggiorna stato│       │   Registry   │            │
│  └────┬────────────────┘       └──────────────┘            │
│       │                                                      │
│       │                        ┌──────────────┐             │
│       ├───────────────────────→│    Safety    │             │
│       │                        │   Verifier   │             │
│       │                        └──────────────┘             │
│       ↓                                                      │
│  ┌─────────────────────┐                                    │
│  │     REFLECTION      │                                    │
│  │                     │                                    │
│  │  ├─ Analizza        │────→ Aggiorna Pattern Cache       │
│  │  ├─ Estrae          │────→ Memorizza Episodio           │
│  │  └─ Apprende        │                                   │
│  └────┬────────────────┘                                    │
│       │                                                      │
│       ↓                                                      │
│  ┌─────────────────────┐                                    │
│  │  VALIDAZIONE FINALE │                                    │
│  └────┬────────────────┘                                    │
│       │                                                      │
│       ↓                                                      │
│  ┌──────────┐                                               │
│  │  OUTPUT  │                                               │
│  │ RISULTATO│                                               │
│  └──────────┘                                               │
└────────────────────────────────────────────────────────────────┘
```

## Macchina a Stati - Stati di Esecuzione

```
┌─────────────────────────────────────────────────────────────────┐
│                    MACCHINA A STATI AGENTE                      │
│                                                                 │
│                     ┌──────────┐                               │
│                     │   IDLE   │                               │
│                     └────┬─────┘                               │
│                          │ task ricevuto                        │
│                          ↓                                      │
│                   ┌─────────────┐                              │
│          ┌────────│  ANALYZING  │────────┐                     │
│          │        └─────────────┘        │                     │
│          │                               │                     │
│    validazione fail                 validazione ok             │
│          │                               │                     │
│          ↓                               ↓                     │
│    ┌───────────┐                  ┌────────────┐              │
│    │  FAILED   │                  │  PLANNING  │              │
│    └───────────┘                  └─────┬──────┘              │
│          │                              │                     │
│          │                         piano pronto               │
│          │                              │                     │
│          │                              ↓                     │
│          │                       ┌─────────────┐              │
│          │                       │  EXECUTING  │←─────┐       │
│          │                       └──────┬──────┘      │       │
│          │                              │             │       │
│          │                    ┌─────────┴────────┐    │       │
│          │                    │                  │    │       │
│          │              step successo      step fallito│      │
│          │                    │                  │    │       │
│          │              step successivo    recuperabile│      │
│          │                    │                  │    │       │
│          │                    └─────────┬────────┘    │       │
│          │                              │             │       │
│          │                    tutti step completi   retry     │
│          │                              │             │       │
│          │                              ↓             │       │
│          │                       ┌─────────────┐     │       │
│          │                       │ VERIFYING   │     │       │
│          │                       └──────┬──────┘     │       │
│          │                              │            │       │
│          │                    ┌─────────┴─────────┐  │       │
│          │                    │                   │  │       │
│          │             verifica ok       verifica fail│       │
│          │                    │                   │  │       │
│          │                    │             serve replan     │
│          │                    │                   │  │       │
│          │                    │                   └──┘       │
│          │                    ↓                              │
│          │             ┌─────────────┐                       │
│          │             │ REFLECTING  │                       │
│          │             └──────┬──────┘                       │
│          │                    │                              │
│          │                    ↓                              │
│          │             ┌─────────────┐                       │
│          └────────────→│  COMPLETE   │                       │
│                        └──────┬──────┘                       │
│                               │                              │
│                               ↓                              │
│                        ┌──────────┐                          │
│                        │   IDLE   │                          │
│                        └──────────┘                          │
└─────────────────────────────────────────────────────────────────┘
```

## Dipendenze tra Componenti

```
┌──────────────────────────────────────────────────────────────────┐
│                    GRAFO DIPENDENZE                              │
│                                                                  │
│                    ┌─────────────────┐                          │
│                    │  Analisi Goal   │                          │
│                    └────────┬────────┘                          │
│                             │                                   │
│                             ↓                                   │
│                    ┌─────────────────┐                          │
│          ┌─────────│ Planning Engine │─────────┐               │
│          │         └─────────────────┘         │               │
│          │                                      │               │
│          ↓                                      ↓               │
│  ┌───────────────┐                    ┌─────────────────┐      │
│  │    Memory     │                    │  Model Router   │      │
│  │    System     │                    └────────┬────────┘      │
│  └───────┬───────┘                             │               │
│          │                                      │               │
│          ↓                                      ↓               │
│  ┌───────────────┐                    ┌─────────────────┐      │
│  │   Execution   │───────────────────→│ Tool Registry   │      │
│  │    Engine     │                    └────────┬────────┘      │
│  └───────┬───────┘                             │               │
│          │                                      │               │
│          └──────────────┬───────────────────────┘               │
│                         │                                       │
│                         ↓                                       │
│                ┌──────────────────┐                             │
│                │ Safety Verifier  │                             │
│                └────────┬─────────┘                             │
│                         │                                       │
│                         ↓                                       │
│                ┌──────────────────┐                             │
│                │   Reflection     │                             │
│                └────────┬─────────┘                             │
│                         │                                       │
│                         ↓                                       │
│          ┌──────────────────────────────┐                      │
│          │  (ritorna alla Memory)       │                      │
│          └──────────────────────────────┘                      │
│                                                                  │
│  Cross-cutting:                                                 │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ Observability System (monitora tutti i componenti)  │       │
│  └─────────────────────────────────────────────────────┘       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ Resource Manager (controlla tutti i componenti)     │       │
│  └─────────────────────────────────────────────────────┘       │
│  ┌─────────────────────────────────────────────────────┐       │
│  │ Error Handler (avvolge tutti i componenti)          │       │
│  └─────────────────────────────────────────────────────┘       │
└──────────────────────────────────────────────────────────────────┘
```

## Architettura di Deployment

```
┌────────────────────────────────────────────────────────────────────┐
│                      VISTA DEPLOYMENT                              │
│                                                                    │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  TIER APPLICAZIONE                                           │ │
│  │                                                              │ │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐            │ │
│  │  │  Agent     │  │  Agent     │  │  Agent     │            │ │
│  │  │ Instance 1 │  │ Instance 2 │  │ Instance N │            │ │
│  │  └────────────┘  └────────────┘  └────────────┘            │ │
│  │         │               │               │                   │ │
│  └─────────┼───────────────┼───────────────┼───────────────────┘ │
│            │               │               │                     │
│            └───────────────┴───────────────┘                     │
│                            │                                     │
│  ┌─────────────────────────┼──────────────────────────────────┐ │
│  │  TIER CACHE              │                                  │ │
│  │                          ↓                                  │ │
│  │  ┌─────────────────────────────────────────┐               │ │
│  │  │         Redis / Memcached                │               │ │
│  │  │  (Pattern Cache, Stato Sessione)         │               │ │
│  │  └───────────────────┬─────────────────────┘               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                            │                                     │
│  ┌─────────────────────────┼──────────────────────────────────┐ │
│  │  TIER DATI               │                                  │ │
│  │                          ↓                                  │ │
│  │  ┌─────────────┐   ┌──────────────┐   ┌─────────────┐    │ │
│  │  │  PostgreSQL │   │   Pinecone   │   │  S3/Blob    │    │ │
│  │  │  (Metadata) │   │  (Episodic   │   │  (Log,      │    │ │
│  │  │             │   │   Memory)    │   │  Artifact)  │    │ │
│  │  └─────────────┘   └──────────────┘   └─────────────┘    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  SERVIZI ESTERNI                                             │ │
│  │                                                              │ │
│  │  ┌───────────┐   ┌────────────┐   ┌─────────────┐          │ │
│  │  │  Anthropic│   │  OpenAI    │   │   Tool      │          │ │
│  │  │  Claude   │   │  GPT-4     │   │   Custom    │          │ │
│  │  └───────────┘   └────────────┘   └─────────────┘          │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────────┐ │
│  │  OBSERVABILITY                                               │ │
│  │                                                              │ │
│  │  ┌───────────┐   ┌────────────┐   ┌─────────────┐          │ │
│  │  │Prometheus │   │  Grafana   │   │    ELK      │          │ │
│  │  │ (Metriche)│   │ (Dashboard)│   │   (Log)     │          │ │
│  │  └───────────┘   └────────────┘   └─────────────┘          │ │
│  └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

## Caratteristiche di Performance

### Breakdown Budget Latenza

```
Target Totale: 5-30s per task complessità media

┌────────────────────────────────────────┐
│ Componente          Tempo    % Totale  │
├────────────────────────────────────────┤
│ Analisi Goal        0.5s      2%       │
│ Recupero Contesto   1.0s      5%       │
│ Pianificazione      2.0s      10%      │
│ Esecuzione:                            │
│   - Chiamate model  15.0s     70%      │
│   - Esecuzione tool 3.0s      13%      │
│ Verifica            0.5s      2%       │
│ Reflection          1.0s      5%       │
├────────────────────────────────────────┤
│ TOTALE              ~23s      ~100%    │
└────────────────────────────────────────┘

Strategie di ottimizzazione:
- Routing modelli (usa modelli veloci quando sufficiente)
- Esecuzione parallela tool
- Riuso pattern cached
- Streaming per latenza percepita
```

### Caratteristiche di Scalabilità

```
┌──────────────────────────────────────────────────────────┐
│ Dimensione        Corrente  Target    Strategia Scaling  │
├──────────────────────────────────────────────────────────┤
│ Utenti Concurrent 10-50     100-500   Orizzontale        │
│ Task/Ora          100-500   5K-10K    Orizzontale        │
│ Episodi Memory    10K       1M+       Scala Vector DB    │
│ Pattern Cache     1K        10K+      Cache distribuita  │
│ Latenza Media     5-30s     <30s      Ottimizz. modelli │
│ Success Rate      85-95%    >90%      Apprendimento cont│
└──────────────────────────────────────────────────────────┘
```

---

**Prossimo**: [02-cognitive-layer.md](02-cognitive-layer.md) → Specifiche dettagliate dei componenti di ragionamento
