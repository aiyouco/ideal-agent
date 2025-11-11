# Fondamenti dell'Architettura Cognitiva

**Stato del Documento:** 🟢 Completo
**Ultimo Aggiornamento:** 2025-11-10
**Autore:** Kilo Code

## Indice dei Contenuti

1. [Panoramica](#panoramica)
2. [Cosa sono le Architetture Cognitive?](#cosa-sono-le-architetture-cognitive)
3. [Architettura SOAR](#architettura-soar)
4. [Architettura ACT-R](#architettura-act-r)
5. [Architettura CLARION](#architettura-clarion)
6. [Altre Architetture Influenti](#altre-architetture-influenti)
7. [Applicabilità agli Agenti LLM](#applicabilità-agli-agenti-llm)
8. [Architettura Ibrida Cognitiva-LLM](#architettura-ibrida-cognitiva-llm)
9. [Principi Chiave per l'Agente Ideale](#principi-chiave-per-lagente-ideale)

---

## Panoramica

Le architetture cognitive provenienti dalle scienze cognitive forniscono decenni di ricerca su come costruire sistemi intelligenti capaci di ragionare, apprendere e agire. Sebbene originariamente progettate per modellare la cognizione umana, offrono pattern architetturali preziosi per gli agenti basati su LLM.

Questo documento analizza tre principali architetture cognitive (SOAR, ACT-R, CLARION) ed estrae principi applicabili per progettare un agente universale ideale.

### Perché Studiare le Architetture Cognitive?

1. **Framework comprovati**: Testati per decenni in diversi domini
2. **Design basato su principi**: Fondato sulla ricerca nelle scienze cognitive
3. **Teorie unificate**: Tentano di gestire tutti gli aspetti della cognizione
4. **Caratteristiche di performance**: Proprietà computazionali ben comprese
5. **Pattern di integrazione**: Mostrano come i componenti interagiscono

### Differenze Chiave: Architetture Cognitive vs. Agenti LLM

| Aspetto | Arch. Cognitive Tradizionali | Agenti LLM |
|--------|---------------------------|------------|
| Conoscenza | Regole simboliche esplicite | Implicita nei pesi |
| Apprendimento | Rinforzo incrementale | Pre-training + prompting |
| Memoria | Database strutturati | Finestra di contesto + esterna |
| Ragionamento | Regole di produzione | Generazione di token |
| Velocità | Veloce (microsecondi) | Lento (secondi) |
| Interpretabilità | Alta (regole visibili) | Bassa (black box) |
| Generalizzazione | Limitata al dominio | Ampia ma inaffidabile |
| Uso di Risorse | Basso | Alto (intensivo computazionalmente) |

---

## Cosa sono le Architetture Cognitive?

### Definizione

Un'**architettura cognitiva** è un framework computazionale che:
1. Integra molteplici capacità cognitive (memoria, ragionamento, apprendimento, percezione, azione)
2. Opera secondo principi strutturali fissi
3. Produce comportamento attraverso l'interazione dei componenti
4. Può essere applicata a molti task e domini

### Componenti Principali (Comuni alla Maggior parte delle Architetture)

```
┌────────────────────────────────────────┐
│   Componenti Architettura Cognitiva    │
│                                        │
│  ┌──────────────┐  ┌──────────────┐  │
│  │  Percezione  │  │    Azione    │  │
│  └──────┬───────┘  └──────▲───────┘  │
│         │                 │           │
│  ┌──────▼─────────────────┴────────┐ │
│  │     Memoria di Lavoro           │ │
│  │  (Stato corrente, obiettivi)    │ │
│  └──────┬──────────────────▲───────┘ │
│         │                  │          │
│  ┌──────▼──────────┐  ┌───┴────────┐ │
│  │  Memoria        │  │  Meccanismo │ │
│  │  a Lungo        │  │  di         │ │
│  │  Termine        │  │  Decisione  │ │
│  │  (Conoscenza)   │  │  (Regole)   │ │
│  └─────────────────┘  └────────────┘ │
└────────────────────────────────────────┘
```

### Principi Fondamentali

1. **Memoria di Lavoro**: Capacità limitata, focus attuale dell'attenzione
2. **Memoria a Lungo Termine**: Capacità illimitata, recuperata nella memoria di lavoro
3. **Sistema di Produzione**: Regole che si attivano in base allo stato corrente
4. **Meccanismi di Apprendimento**: Migliorano le prestazioni nel tempo
5. **Gestione degli Obiettivi**: Mantengono e perseguono gli obiettivi
6. **Loop Percezione-Azione**: Interazione continua con l'ambiente

---

## Architettura SOAR

**SOAR** = State, Operator, And Result (Stato, Operatore e Risultato)

**Paper:** "The SOAR Cognitive Architecture" (Laird, 2012)
**Origine:** 1983, University of Michigan
**Paradigma:** Teoria unificata della cognizione

### Principi Fondamentali

1. **Tutte le decisioni sono problemi di selezione**: Scegliere operatori da applicare
2. **Gli impasse guidano l'apprendimento**: Quando non si può prendere una decisione, creare un sottoobiettivo
3. **Chunking**: Compilare l'esperienza in regole
4. **Memoria simbolica persistente**: Memorie semantiche ed episodiche
5. **Architettura unificata singola**: Stessi meccanismi per tutti i task

### Panoramica dell'Architettura

```
┌─────────────────────────────────────────────┐
│  Architettura SOAR                          │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │      Memoria di Lavoro              │   │
│  │  (Stato corrente, operatori, obiett)│   │
│  └──────────┬─────▲───────────────────┘   │
│             │     │                        │
│  ┌──────────▼─────┴──────────────┐        │
│  │   Procedura di Decisione       │        │
│  │   (Seleziona operatore)        │        │
│  └──────────┬─────▲───────────────┘        │
│             │     │                        │
│             │  Impasse?                    │
│             │     │                        │
│  ┌──────────▼─────┴──────────────┐        │
│  │   Creazione Sottoobiettivi     │        │
│  │   (Ricerca spazio problemi)    │        │
│  └──────────┬─────────────────────┘        │
│             │                              │
│  ┌──────────▼─────────────────────┐       │
│  │   Meccanismo di Chunking       │       │
│  │   (Impara dai risultati)       │       │
│  └────────────────────────────────┘       │
│                                            │
│  ┌────────────────────────────────────┐   │
│  │  Memorie a Lungo Termine           │   │
│  │  • Procedurale (regole/operatori)  │   │
│  │  • Semantica (fatti/concetti)      │   │
│  │  • Episodica (esperienze)          │   │
│  └────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

### Meccanismi Chiave

#### 1. Ciclo di Decisione

SOAR opera in cicli di decisione discreti:

```
Fase 1: Input
  - Percepire l'ambiente
  - Aggiornare la memoria di lavoro

Fase 2: Proposta
  - Confrontare regole della memoria a lungo termine
  - Generare operatori candidati

Fase 3: Decisione
  - Selezionare singolo operatore (basato su preferenze)
  - Se nessuna selezione possibile → Impasse

Fase 4: Applicazione
  - Eseguire l'operatore selezionato
  - Aggiornare la memoria di lavoro

Fase 5: Output
  - Eseguire azioni nell'ambiente
```

**Prestazioni**: ~50 cicli/secondo

#### 2. Risoluzione degli Impasse

Quando SOAR non può prendere una decisione (parità, nessun operatore, ecc.), crea un **sottoobiettivo**:

```
Obiettivo Principale: Risolvere task X
  ↓ (Non può decidere quale operatore)
  ↓ Impasse rilevato
  ↓
Sottoobiettivo: Determinare quale operatore è migliore
  ↓ (Cercare, valutare, confrontare)
  ↓ Sottoobiettivo risolto
  ↓
Obiettivo Principale: Continuare con operatore selezionato
  ↓
Nuova Regola Creata via Chunking
```

Questa è una forma di **subgoaling universale** - qualsiasi problema può essere risolto per decomposizione.

#### 3. Chunking (Apprendimento)

Quando un sottoobiettivo si completa, SOAR crea un nuovo **chunk** (regola):

```
Condizioni: Stati che esistevano quando il sottoobiettivo è stato creato
Azioni: Risultati prodotti dal sottoobiettivo
Effetto: La prossima volta che si verificano le stesse condizioni, saltare il sottoobiettivo
```

**Impatto sulle Prestazioni**: Accelera future decisioni simili (O(n) → O(1))

#### 4. Sistemi di Memoria

**Memoria Procedurale**
- Regole di produzione (if-then)
- Operatori che possono essere applicati
- Appresi via chunking

**Memoria Semantica**
- Fatti e concetti a lungo termine
- Struttura a grafo (nodi + archi)
- Recuperata per attivazione diffusa

**Memoria Episodica**
- Sequenza temporale di stati della memoria di lavoro
- Esperienze "autobiografiche"
- Recuperata per matching basato su indizi

### Punti di Forza per il Design di Agenti

1. ✅ **Decomposizione naturale dei task**: Gli impasse creano sottoobiettivi automaticamente
2. ✅ **Apprendimento dall'esperienza**: Il chunking compila i percorsi di successo
3. ✅ **Sistemi di memoria espliciti**: Separazione chiara dei tipi di memoria
4. ✅ **Decisioni tracciabili**: Ogni ciclo è esplicito e ispezionabile
5. ✅ **Conoscenza persistente**: Le memorie sopravvivono attraverso le sessioni
6. ✅ **Framework unificato**: Stesso meccanismo per tutto il ragionamento

### Limitazioni per gli Agenti LLM

1. ❌ **Fragilità simbolica**: Le regole non gestiscono bene l'ambiguità
2. ❌ **Ingegneria manuale della conoscenza**: Le regole devono essere codificate
3. ❌ **Generalizzazione limitata**: Le regole sono specifiche ai domini addestrati
4. ❌ **Nessun ragionamento in linguaggio naturale**: Simbolico, non linguistico
5. ❌ **Apprendimento lento**: Richiede molte prove per imparare

### Principi Applicabili

1. **Subgoaling universale**: Quando bloccato, decomporre in sottoproblemi
2. **Memoria di lavoro come stato**: Capacità finita, attenzione focalizzata
3. **Tipi multipli di memoria**: Separare episodica, semantica, procedurale
4. **Rilevamento degli impasse**: Riconoscere quando il ragionamento è bloccato
5. **Concetto di chunking**: Memorizzare pattern di ragionamento di successo
6. **Cicli di decisione**: Passi di ragionamento discreti con fasi chiare

---

## Architettura ACT-R

**ACT-R** = Adaptive Control of Thought - Rational (Controllo Adattivo del Pensiero - Razionale)

**Paper:** "How Can the Human Mind Occur in the Physical Universe?" (Anderson, 2007)
**Origine:** 1993, Carnegie Mellon University
**Paradigma:** Analisi razionale della cognizione

### Principi Fondamentali

1. **Moduli con buffer**: Processori separati per diverse funzioni
2. **Elaborazione parallela**: I moduli operano in parallelo, coordinati attraverso i buffer
3. **Diffusione dell'attivazione**: Recupero della memoria basato sui livelli di attivazione
4. **Utilità subsimboliche**: Probabilità e costi guidano le decisioni
5. **Aritmetica cognitiva**: Previsioni di timing precise
6. **Apprendimento tramite meccanismi di livello base e associativi**

### Panoramica dell'Architettura

```
┌──────────────────────────────────────────────────┐
│  Architettura ACT-R                              │
│                                                  │
│         Sistema di Produzione Centrale           │
│               (Pattern Matcher)                  │
│                     │                            │
│    ┌────────────────┼────────────────┐          │
│    │                │                │          │
│    ▼                ▼                ▼          │
│  ┌────┐          ┌────┐          ┌────┐        │
│  │Goal│          │Ret │          │Vis │        │
│  │Buf │          │Buf │          │Buf │        │
│  └─┬──┘          └─┬──┘          └─┬──┘        │
│    │               │                │          │
│    ▼               ▼                ▼          │
│  ┌──────┐      ┌──────┐        ┌──────┐       │
│  │ Goal │      │Mem.  │        │Visual│       │
│  │Module│      │Dich. │        │Module│       │
│  └──────┘      └──────┘        └──────┘       │
│                                                 │
│  Altri moduli: Manual, Vocal, Imaginal...      │
└──────────────────────────────────────────────────┘
```

### Meccanismi Chiave

#### 1. Architettura Modulo-Buffer

Ogni modulo ha:
- **Buffer**: Singolo chunk di informazioni (capacità limitata)
- **Processore**: Opera autonomamente
- **Comunicazione**: Via sistema di produzione centrale

Moduli:
- **Goal**: Intenzioni e sottoobiettivi correnti
- **Memoria Dichiarativa**: Fatti e conoscenze
- **Visual**: Percezione visiva
- **Manual**: Azioni motorie
- **Vocal**: Produzione del linguaggio
- **Imaginal**: Immaginazione mentale e spazio di lavoro per problem-solving

#### 2. Sistema di Produzione

Le produzioni (regole) hanno la forma:
```
IF il_contenuto_del_buffer_corrisponde_al_pattern
THEN modifica_buffer_e_richiedi_azioni_modulo
```

Esempio:
```
IF l'obiettivo è sommare X + Y
   E X recuperato dalla memoria
THEN richiedi recupero di Y
     imposta obiettivo a sommare una volta recuperato Y
```

**Risoluzione dei Conflitti**: Quando più produzioni corrispondono, selezionare per utilità (valore appreso + rumore)

#### 3. Attivazione e Recupero

I chunk della memoria dichiarativa hanno valori di **attivazione**:

```
Attivazione = LivelloBase + AttivazionePerDiffusione + Rumore

LivelloBase: Funzione di recency e frequenza d'uso
AttivazionePerDiffusione: Correlata ai contenuti attuali del buffer
Rumore: Variazione casuale (modella la variabilità RT)
```

**Tempo di recupero**: `t = F × e^(-Attivazione)`

Alta attivazione → Recupero veloce
Bassa attivazione → Recupero lento/fallito

Questo modella i fenomeni della memoria umana (effetti di recency, frequenza, interferenza).

#### 4. Meccanismi di Apprendimento

**Apprendimento a Livello Base**
- L'attivazione aumenta con l'uso, decade nel tempo
- Modella gli effetti della pratica

**Apprendimento Associativo**
- Le associazioni tra chunk vengono rafforzate dalla co-occorrenza
- Modella gli effetti di priming e contesto

**Compilazione delle Produzioni**
- Combina più produzioni in una
- Accelera le sequenze praticate

**Apprendimento dell'Utilità**
- Le produzioni che portano a ricompense ottengono utilità più alta
- Modella la selezione della strategia

#### 5. Utilità Subsimboliche

A differenza del sistema puramente simbolico di SOAR, ACT-R usa:
- **Probabilità**: Probabilità di successo della produzione
- **Costi**: Tempo e sforzo delle produzioni
- **Utilità attesa**: Prob(successo) × Valore - Costo

Questo abilita l'**analisi razionale**: Scegliere azioni che massimizzano l'utilità attesa dati i vincoli.

### Punti di Forza per il Design di Agenti

1. ✅ **Elaborazione parallela**: I moduli operano concorrentemente
2. ✅ **Timing realistico**: Previsioni di latenza precise
3. ✅ **Memoria basata su attivazione**: Oblio graduale, interferenza
4. ✅ **Selezione basata su utilità**: Scelte probabilistiche e razionali
5. ✅ **Ben validata**: Corrisponde alle prestazioni umane su molti task
6. ✅ **Composizionale**: Combinare moduli per nuove capacità

### Limitazioni per gli Agenti LLM

1. ❌ **Set di moduli fisso**: Difficile aggiungere nuovi moduli
2. ❌ **Codifica manuale delle produzioni**: Le regole sono ancora create a mano
3. ❌ **Buffer single-chunk**: Memoria di lavoro molto limitata
4. ❌ **Specifico per dominio**: Modella task ristretti, non intelligenza generale
5. ❌ **Nessun linguaggio naturale**: Rappresentazioni simboliche

### Principi Applicabili

1. **Elaborazione parallela modulare**: Moduli indipendenti coordinati attraverso stato condiviso
2. **Recupero basato su attivazione**: Memorie rilevanti attivate dal contesto
3. **Selezione guidata dall'utilità**: Scegliere azioni per valore atteso
4. **Livello subsimbolico**: Probabilità e costi, non solo simboli
5. **Buffer multipli**: Spazi di memoria di lavoro separati
6. **Vincoli di timing**: Modellare prestazioni in tempo reale

---

## Architettura CLARION

**CLARION** = Connectionist Learning with Adaptive Rule Induction ON-line (Apprendimento Connessionista con Induzione di Regole Adattive On-line)

**Paper:** "The CLARION Cognitive Architecture" (Sun, 2006)
**Origine:** Anni '90, Rensselaer Polytechnic Institute
**Paradigma:** Architettura ibrida simbolica-connessionista

### Principi Fondamentali

1. **Rappresentazione duale**: Esplicita (simbolica) e implicita (rete neurale)
2. **Apprendimento bottom-up**: Estrarre regole esplicite dalla conoscenza implicita
3. **Apprendimento top-down**: Raffinare la conoscenza implicita usando regole esplicite
4. **Centrato sull'azione**: Focus sulla selezione delle azioni e sull'apprendimento
5. **Motivazione e metacognizione**: Guidano la selezione e la riflessione

### Panoramica dell'Architettura

```
┌────────────────────────────────────────────────┐
│  Architettura CLARION                          │
│                                                │
│  ┌──────────────────────────────────────┐    │
│  │  Sottosistema Metacognitivo          │    │
│  │  (Monitoraggio, controllo, ragion.)  │    │
│  └──────────────┬───────────────────────┘    │
│                 │                             │
│  ┌──────────────▼──────────────────────┐     │
│  │  Sottosistema Centrato sull'Azione  │     │
│  │                                     │     │
│  │  Livello Esplicito (Top):          │     │
│  │  ┌────────────────────────┐        │     │
│  │  │  Regole Simboliche     │        │     │
│  │  └────────────────────────┘        │     │
│  │           ▲ ▼                      │     │
│  │  Livello Implicito (Bottom):       │     │
│  │  ┌────────────────────────┐        │     │
│  │  │  Reti Neurali          │        │     │
│  │  └────────────────────────┘        │     │
│  └─────────────────────────────────────┘     │
│                                               │
│  ┌──────────────────────────────────────┐    │
│  │  Sottosistema Non-Centrato-Azione   │    │
│  │  (Percezione, memoria, conoscenza)   │    │
│  │  Anche struttura dual-level          │    │
│  └──────────────────────────────────────┘    │
│                                               │
│  ┌──────────────────────────────────────┐    │
│  │  Sottosistema Motivazionale          │    │
│  │  (Pulsioni, obiettivi, emozioni)     │    │
│  └──────────────────────────────────────┘    │
└────────────────────────────────────────────────┘
```

### Meccanismi Chiave

#### 1. Rappresentazione Duale

**Livello Bottom (Implicito)**:
- Reti neurali (tipicamente backpropagation)
- Elaborazione veloce, automatica, olistica
- Gestisce pattern, generalizzazione
- Inizialmente appreso dall'esperienza

**Livello Top (Esplicito)**:
- Regole simboliche (coppie condizione-azione)
- Elaborazione lenta, deliberata, analitica
- Gestisce ragionamento, pianificazione
- Può essere fornito o estratto dal bottom

**Integrazione**:
```
Input
  ↓
Entrambi i livelli elaborano in parallelo
  ↓
Gli output vengono combinati (pesati)
  ↓
Azione
```

#### 2. Estrazione di Regole (Apprendimento Bottom-Up)

Estrarre regole esplicite dalle reti neurali addestrate:

```
1. Eseguire NN su molti input, osservare output
2. Trovare regolarità nei mapping input-output
3. Generalizzare a regole simboliche
4. Aggiungere regole al livello top
```

Esempio:
```
NN impara: pattern ad alta dimensione → azione A
Estrarre regola: IF caratteristica_X > soglia THEN azione A
```

Questo rende la conoscenza implicita esplicita e spiegabile.

#### 3. Raffinamento delle Regole (Apprendimento Top-Down)

Usare regole esplicite per guidare l'addestramento della rete neurale:

```
1. Applicare regola esplicita
2. Osservare il risultato
3. Usare il risultato per addestrare NN
4. NN generalizza la regola a situazioni simili
```

Questo trasferisce la conoscenza strutturata al livello subsimbolico.

#### 4. Sistema Motivazionale

Guida il comportamento attraverso:
- **Pulsioni primarie**: Fame, sete, ecc. (per agenti incarnati)
- **Pulsioni secondarie**: Realizzazione, affiliazione, ecc.
- **Impostazione degli obiettivi**: Creare obiettivi espliciti dalle pulsioni
- **Modulazione emotiva**: Influenza la selezione e l'apprendimento

#### 5. Livello Metacognitivo

Monitora e controlla il sottosistema centrato sull'azione:
- **Monitoraggio**: Tracciare il progresso, rilevare errori
- **Controllo**: Selezionare strategie, allocare attenzione
- **Ragionamento**: Pianificare, risolvere problemi a livello superiore

### Punti di Forza per il Design di Agenti

1. ✅ **Ragionamento ibrido**: Combina approcci simbolici e neurali
2. ✅ **Spiegabilità**: Può estrarre regole dal comportamento neurale
3. ✅ **Flessibilità**: Usare esplicito o implicito a seconda delle necessità
4. ✅ **Bootstrapping**: Trasferire conoscenza tra livelli
5. ✅ **Metacognizione**: Monitoraggio e controllo espliciti di sé
6. ✅ **Motivazione**: Comportamento orientato agli obiettivi

### Limitazioni per gli Agenti LLM

1. ❌ **Architettura complessa**: Molti sottosistemi interagenti
2. ❌ **Addestramento duale**: Deve addestrare sia componenti simbolici che neurali
3. ❌ **Overhead di estrazione delle regole**: Computazionalmente costoso
4. ❌ **Validazione limitata**: Meno supporto empirico di SOAR/ACT-R
5. ❌ **Subsimbolico ≠ LLM**: Le reti neurali non sono modelli linguistici

### Principi Applicabili

1. **Ragionamento dual-level**: Combinare implicito veloce con esplicito lento
2. **Estrazione di regole**: Rendere espliciti i pattern appresi
3. **Metacognizione come sottosistema**: Monitoraggio e controllo separati
4. **La motivazione guida gli obiettivi**: Pulsione esplicita → gerarchia di obiettivi
5. **Apprendimento bidirezionale**: Trasferimento di conoscenza top-down e bottom-up

---

## Altre Architetture Influenti

### EPIC (Executive Process Interactive Control)

**Focus**: Interazione percettivo-motoria e multitasking

**Idee Chiave**:
- Processori percettivi e motori paralleli
- Il processore esecutivo (cognitivo) coordina
- Modelli di timing precisi per percezione e azione
- Interferenza nei task e limiti di capacità

**Applicabile**: Elaborazione multi-modale, esecuzione parallela, gestione delle risorse

### ICARUS

**Focus**: Comportamento reattivo e guidato da obiettivi nella robotica

**Idee Chiave**:
- Gerarchia percettivo-motoria
- Abilità reattive eseguite in parallelo
- Pianificazione mezzi-fini per il raggiungimento degli obiettivi
- Apprendimento integrato di abilità e concetti

**Applicabile**: Uso di strumenti, comportamento reattivo, controllo gerarchico

### Sigma

**Focus**: Architettura cognitiva unificata usando modelli grafici

**Idee Chiave**:
- Grafi fattoriali per tutte le rappresentazioni
- Inferenza probabilistica per tutti i processi
- Apprendimento come cambiamento della struttura del grafo
- Framework unificato per memoria, ragionamento, apprendimento

**Applicabile**: Ragionamento probabilistico, rappresentazione unificata, inferenza come computazione

---

## Applicabilità agli Agenti LLM

### Cosa si Trasferisce Bene

#### 1. Organizzazione della Memoria
- **Memoria episodica**: Memorizzare esperienze passate (prompt + risposte)
- **Memoria semantica**: Memorizzare fatti e conoscenze
- **Memoria procedurale**: Memorizzare pattern di successo (esempi few-shot, strategie)
- **Memoria di lavoro**: Finestra di contesto limitata

#### 2. Gestione degli Obiettivi
- **Stack di obiettivi**: Mantenere obiettivi gerarchici
- **Rilevamento degli impasse**: Riconoscere quando si è bloccati
- **Subgoaling**: Decomporre problemi
- **Rilevamento del completamento degli obiettivi**: Sapere quando si è finito

#### 3. Strutture di Decisione
- **Cicli di decisione**: Passi di ragionamento discreti
- **Selezione degli operatori**: Scegliere l'azione successiva
- **Selezione basata su utilità**: Valutare le opzioni
- **Risoluzione dei conflitti**: Gestire più possibilità

#### 4. Meccanismi di Apprendimento
- **Apprendimento episodico**: Ricordare successi/fallimenti passati
- **Chunking**: Memorizzare pattern di successo
- **Apprendimento dell'utilità**: Regolare le preferenze in base ai risultati
- **Riflessione**: Miglioramento metacognitivo

#### 5. Decomposizione Modulare
- **Separare le preoccupazioni**: Ragionamento vs. memoria vs. azione
- **Parallelo dove possibile**: Moduli indipendenti
- **Interfacce chiare**: Buffer/API tra moduli
- **Componibilità**: Combinare moduli per nuove capacità

### Cosa Non si Trasferisce

#### 1. Regole Simboliche
- ❌ Gli LLM non usano regole di produzione
- ❌ Il pattern matching è implicito nei pesi
- ❌ Le regole non possono essere codificate manualmente
- **Ma**: Possono usare prompt come "regole soft"

#### 2. Timing al Millisecondo
- ❌ Gli LLM impiegano secondi, non millisecondi
- ❌ Nessun controllo preciso del timing
- **Ma**: Possono modellare la latenza e ottimizzare

#### 3. Esecuzione Deterministica
- ❌ Gli LLM sono stocastici
- ❌ Stesso input → output diversi
- **Ma**: Possono usare temperatura e seeding

#### 4. Apprendimento Incrementale
- ❌ Gli LLM non possono aggiornare i pesi durante l'inferenza
- ❌ Nessun apprendimento online dei parametri
- **Ma**: Possono usare in-context learning

#### 5. Memoria Perfetta
- ❌ Gli LLM hanno limiti di contesto
- ❌ Non possono memorizzare una storia illimitata
- **Ma**: Possono usare sistemi di memoria esterni

### Approccio Ibrido: Principi Cognitivi + Capacità LLM

```
┌────────────────────────────────────────────┐
│  Agente Ibrido Cognitivo-LLM               │
│                                            │
│  ┌──────────────────────────────────┐    │
│  │  Livello di Controllo Cognitivo  │    │
│  │  (Ispirato a SOAR)               │    │
│  │  • Stack di obiettivi            │    │
│  │  • Rilevamento impasse           │    │
│  │  • Ciclo di decisione            │    │
│  └───────────┬──────────────────────┘    │
│              │                            │
│  ┌───────────▼──────────────────────┐    │
│  │  Sistemi di Memoria              │    │
│  │  (Ispirato a ACT-R)              │    │
│  │  • Episodica (vector store)      │    │
│  │  • Semantica (knowledge graph)   │    │
│  │  • Lavoro (context window)       │    │
│  └───────────┬──────────────────────┘    │
│              │                            │
│  ┌───────────▼──────────────────────┐    │
│  │  Motore di Ragionamento          │    │
│  │  (Basato su LLM)                 │    │
│  │  • Elaborazione linguaggio nat.  │    │
│  │  • Riconoscimento pattern        │    │
│  │  • Generazione                   │    │
│  └───────────┬──────────────────────┘    │
│              │                            │
│  ┌───────────▼──────────────────────┐    │
│  │  Livello Metacognitivo           │    │
│  │  (Ispirato a CLARION)            │    │
│  │  • Auto-monitoraggio             │    │
│  │  • Selezione strategia           │    │
│  │  • Riflessione                   │    │
│  └──────────────────────────────────┘    │
└────────────────────────────────────────────┘
```

---

## Architettura Ibrida Cognitiva-LLM

### Principi di Design

1. **LLM come substrato di ragionamento**: Usare LLM per ragionamento flessibile, non solo generazione di testo
2. **Struttura di controllo cognitivo**: Controllo simile a SOAR per comportamento sistematico
3. **Memoria strutturata**: Organizzazione della memoria simile a ACT-R con store esterni
4. **Elaborazione duale**: Divisione esplicito/implicito simile a CLARION (prompt/pesi)
5. **Metacognizione**: Auto-monitoraggio e selezione adattiva della strategia

### Architettura Principale

#### 1. Livello di Controllo (Ispirato a SOAR)

```typescript
interface ControlLayer {
  // Gestione obiettivi
  goalStack: Goal[]
  currentGoal: Goal

  // Ciclo di decisione
  async decisionCycle(): Promise<Decision> {
    // Input: Percepire lo stato
    const state = await this.perceiveState()

    // Proposta: Generare operatori candidati
    const operators = await this.proposeOperators(state)

    // Decisione: Selezionare il miglior operatore
    const selected = await this.selectOperator(operators)

    // Controllo impasse
    if (selected === null) {
      return this.handleImpasse(state)
    }

    // Applicazione: Eseguire operatore
    const result = await this.applyOperator(selected)

    // Output: Restituire decisione
    return result
  }

  // Gestione impasse
  async handleImpasse(state: State): Promise<Decision> {
    // Creare sottoobiettivo
    const subgoal = this.createSubgoal(state)
    this.goalStack.push(subgoal)

    // Ricorrere
    return this.decisionCycle()
  }

  // Creazione sottoobiettivo
  createSubgoal(state: State): Goal {
    // Analizzare perché è avvenuto l'impasse
    const impasseType = this.classifyImpasse(state)

    // Creare sottoobiettivo appropriato
    switch (impasseType) {
      case 'no-operators':
        return new Goal('find-operators', state)
      case 'tie':
        return new Goal('disambiguate', state)
      case 'conflict':
        return new Goal('resolve-conflict', state)
    }
  }
}
```

#### 2. Sistemi di Memoria (Ispirato a ACT-R)

```typescript
interface MemorySystem {
  // Memoria di lavoro (finestra di contesto)
  workingMemory: WorkingMemory

  // Memoria episodica (esperienze)
  episodicMemory: EpisodicMemory

  // Memoria semantica (fatti/concetti)
  semanticMemory: SemanticMemory

  // Memoria procedurale (pattern memorizzati)
  proceduralMemory: ProceduralMemory

  // Recupero basato su attivazione
  async retrieve(cue: Cue): Promise<Memory[]> {
    // Calcolare attivazione per tutte le memorie
    const memories = await this.getAllRelevantMemories(cue)
    const withActivation = memories.map(m => ({
      memory: m,
      activation: this.computeActivation(m, cue)
    }))

    // Ordinare per attivazione
    withActivation.sort((a, b) => b.activation - a.activation)

    // Restituire top-k sopra soglia
    return withActivation
      .filter(m => m.activation > THRESHOLD)
      .slice(0, K)
      .map(m => m.memory)
  }

  // Calcolo attivazione
  computeActivation(memory: Memory, cue: Cue): number {
    const baseLevel = this.baseActivation(memory)
    const spreading = this.spreadingActivation(memory, cue)
    const noise = this.activationNoise()
    return baseLevel + spreading + noise
  }

  // Attivazione a livello base (recency + frequenza)
  baseActivation(memory: Memory): number {
    const recency = Math.log(Date.now() - memory.timestamp)
    const frequency = Math.log(memory.accessCount + 1)
    return -DECAY * recency + FREQUENCY_WEIGHT * frequency
  }

  // Attivazione per diffusione (similarità semantica)
  spreadingActivation(memory: Memory, cue: Cue): number {
    const similarity = this.computeSimilarity(memory, cue)
    return SPREAD_WEIGHT * similarity
  }
}

interface WorkingMemory {
  // Capacità limitata (finestra di contesto)
  maxTokens: number
  contents: Chunk[]

  // Aggiungere alla memoria di lavoro (con rimozione se necessario)
  add(chunk: Chunk): void {
    if (this.willExceedCapacity(chunk)) {
      this.evictLeastRelevant()
    }
    this.contents.push(chunk)
  }

  // Politica di rimozione (basata su attivazione)
  evictLeastRelevant(): void {
    // Rimuovere chunk con attivazione più bassa
    const lowestActivation = this.contents
      .reduce((min, c) => c.activation < min.activation ? c : min)
    this.contents = this.contents.filter(c => c !== lowestActivation)
  }
}
```

#### 3. Motore di Ragionamento (Basato su LLM)

```typescript
interface ReasoningEngine {
  // LLM primario
  model: LanguageModel

  // Generare ragionamento per decisione
  async reason(state: State, goal: Goal): Promise<Reasoning> {
    // Costruire prompt con input strutturato
    const prompt = this.constructPrompt(state, goal)

    // Generare traccia di ragionamento
    const response = await this.model.generate(prompt, {
      temperature: 0.7,
      maxTokens: 1000,
      stop: ['</reasoning>']
    })

    // Analizzare output strutturato
    return this.parseReasoning(response)
  }

  // Costruire prompt da stato + memorie + obiettivo
  constructPrompt(state: State, goal: Goal): Prompt {
    return {
      system: this.systemPrompt,
      context: this.formatContext(state),
      memories: this.formatRelevantMemories(goal),
      goal: this.formatGoal(goal),
      instruction: "Genera ragionamento passo-passo"
    }
  }

  // Generare operatori (possibili azioni)
  async generateOperators(state: State): Promise<Operator[]> {
    const prompt = {
      instruction: "Genera azioni possibili",
      state: state,
      format: "Lista di operatori con precondizioni"
    }

    const response = await this.model.generate(prompt)
    return this.parseOperators(response)
  }

  // Valutare utilità dell'operatore
  async evaluateOperator(operator: Operator, goal: Goal): Promise<number> {
    const prompt = {
      instruction: "Valuta utilità operatore per obiettivo",
      operator: operator,
      goal: goal,
      format: "Punteggio 0-1 con giustificazione"
    }

    const response = await this.model.generate(prompt)
    return this.parseUtility(response)
  }
}
```

#### 4. Livello Metacognitivo (Ispirato a CLARION)

```typescript
interface MetacognitiveLayer {
  // Monitorare esecuzione
  async monitor(state: State, history: History): Promise<MetacognitiveState> {
    return {
      progress: this.assessProgress(state, history),
      confidence: this.assessConfidence(state),
      errors: this.detectErrors(state, history),
      efficiency: this.assessEfficiency(history)
    }
  }

  // Selezionare strategia
  async selectStrategy(metacogState: MetacognitiveState): Promise<Strategy> {
    // Se bassa confidenza, usare strategia più deliberata
    if (metacogState.confidence < 0.5) {
      return Strategy.TREE_SEARCH
    }

    // Se alta confidenza e l'efficienza conta, usare strategia veloce
    if (metacogState.confidence > 0.8 && this.timeLimited) {
      return Strategy.GREEDY
    }

    // Default: strategia reattiva
    return Strategy.REACT
  }

  // Riflettere sulle prestazioni
  async reflect(episode: Episode): Promise<Reflection> {
    const prompt = {
      instruction: "Rifletti sulle prestazioni dell'episodio",
      episode: episode,
      focus: ["successi", "fallimenti", "miglioramenti"]
    }

    const response = await this.model.generate(prompt)
    const reflection = this.parseReflection(response)

    // Memorizzare nella memoria episodica
    await this.memory.episodic.store(episode, reflection)

    return reflection
  }

  // Adattare in base alla riflessione
  async adapt(reflection: Reflection): Promise<void> {
    // Aggiornare preferenze di strategia
    if (reflection.strategyWasEffective) {
      this.strategyUtilities[reflection.strategy] += 0.1
    } else {
      this.strategyUtilities[reflection.strategy] -= 0.1
    }

    // Aggiornare euristiche
    for (const heuristic of reflection.usefulHeuristics) {
      this.heuristicWeights[heuristic] += 0.05
    }
  }
}
```

---

## Principi Chiave per l'Agente Ideale

### Dalle Architetture Cognitive

1. **Memoria di Lavoro con Capacità Limitata**
   - Finestra di contesto fissa
   - Gestione dei contenuti basata su attivazione
   - Politiche esplicite di refresh/rimozione

2. **Gestione Gerarchica degli Obiettivi**
   - Stack di obiettivi per decomposizione
   - Subgoaling guidato da impasse
   - Criteri chiari di raggiungimento degli obiettivi

3. **Memoria a Lungo Termine Strutturata**
   - Episodica: Esperienze passate
   - Semantica: Fatti e concetti
   - Procedurale: Pattern di successo
   - Recupero basato su attivazione

4. **Cicli di Decisione Espliciti**
   - Percepire → Proporre → Decidere → Applicare → Output
   - Confini di fase chiari
   - Esecuzione tracciabile

5. **Apprendimento dall'Esperienza**
   - Ricordare percorsi di successo (chunking)
   - Riflettere sui fallimenti (reflexion)
   - Regolare utilità in base ai risultati
   - Trasferire conoscenza tra episodi

6. **Monitoraggio Metacognitivo**
   - Tracciare progresso e confidenza
   - Rilevare errori e stati bloccati
   - Adattare strategie dinamicamente
   - Riflettere sulle prestazioni

7. **Architettura Parallela Modulare**
   - Processori indipendenti per diverse funzioni
   - Coordinamento attraverso stato condiviso
   - Interfacce chiare tra moduli
   - Capacità componibili

8. **Selezione Guidata dall'Utilità**
   - Selezione probabilistica delle azioni
   - Considerare costi e benefici
   - Trade-off razionali
   - Massimizzazione dell'utilità attesa

### Adattamenti per LLM

1. **Prompt come Regole Soft**
   - I prompt di sistema definiscono il comportamento
   - Esempi few-shot come memoria procedurale
   - Chain-of-thought come ragionamento strutturato

2. **Store di Memoria Esterni**
   - Database vettoriali per memoria semantica
   - Database a grafo per conoscenza
   - Log di episodi per esperienze
   - Finestra di contesto come memoria di lavoro

3. **Formati di Output Strutturati**
   - Forzare risposte JSON/XML
   - Analizzare in dati strutturati
   - Abilitare elaborazione programmatica
   - Mantenere tracciabilità

4. **In-Context Learning per Chunking**
   - Aggiungere esempi di successo al prompt
   - Costruire libreria few-shot
   - Memorizzare pattern efficaci
   - Trasferire tra task

5. **Streaming per Responsività**
   - Streammare token man mano che vengono generati
   - Risultati anticipati per generazioni lunghe
   - Elaborazione interrompibile
   - Raffinamento progressivo

6. **Model Routing per Efficienza**
   - Modelli veloci per task semplici
   - Modelli lenti/potenti quando necessario
   - Trade-off costo/qualità
   - Selezione dinamica

7. **Gestione Esplicita degli Errori**
   - Errori di parsing → retry
   - Rifiuti LLM → riformulare
   - Loop infiniti → circuit breaker
   - Limiti di risorse → degradazione elegante

### Sintesi: Principi Cognitivi-LLM

| Principio | Origine Arch. Cognitiva | Adattamento LLM |
|-----------|----------------------|----------------|
| Memoria di Lavoro Limitata | Buffer ACT-R | Finestra di contesto |
| Decomposizione Obiettivi | Impasse SOAR | Subgoaling guidato da prompt |
| Recupero basato su Attivazione | Diffusione ACT-R | Similarità vettoriale |
| Cicli di Decisione | Fasi SOAR | Loop di ragionamento esplicito |
| Chunking | Apprendimento SOAR | Esempi in-context |
| Memoria Procedurale | Produzioni ACT-R | Prompt few-shot |
| Memoria Episodica | Tutte le architetture | Episodi memorizzati |
| Metacognizione | CLARION | Auto-riflessione LLM |
| Moduli Paralleli | ACT-R, EPIC | Esecuzione asincrona |
| Selezione per Utilità | Subsimbolico ACT-R | Temperature + sampling |

---

## Conclusione

Le architetture cognitive forniscono una ricchezza di pattern di design comprovati per costruire agenti intelligenti. Sebbene gli LLM differiscano fondamentalmente dai sistemi simbolici, i principi chiave si trasferiscono:

1. **Strutturare la memoria** in episodica, semantica e procedurale
2. **Gestire gli obiettivi** gerarchicamente con stack espliciti
3. **Elaborare in cicli** con fasi chiare
4. **Imparare dall'esperienza** attraverso riflessione e chunking
5. **Monitorare metacognitivamente** per adattare strategie
6. **Decomporre modularmente** per sviluppo e test indipendenti

L'agente ideale dovrebbe essere un **ibrido**: controllo dell'architettura cognitiva + flessibilità LLM. Questo combina:
- **Prevedibilità** del flusso di controllo strutturato
- **Flessibilità** del ragionamento in linguaggio naturale
- **Tracciabilità** dello stato esplicito
- **Potenza** del pre-training su larga scala

I prossimi documenti specificheranno l'architettura precisa che realizza questi principi.

---

## Riferimenti

1. Laird, J. E. (2012). The Soar Cognitive Architecture. MIT Press.
2. Anderson, J. R. (2007). How Can the Human Mind Occur in the Physical Universe? Oxford University Press.
3. Sun, R. (2006). The CLARION cognitive architecture: Extending cognitive modeling to social simulation. In Sun, R. (Ed.), Cognition and Multi-Agent Interaction. Cambridge University Press.
4. Meyer, D. E., & Kieras, D. E. (1997). A computational theory of executive cognitive processes and multiple-task performance: Part 1. Basic mechanisms. Psychological Review, 104(1), 3–65.
5. Langley, P., & Choi, D. (2006). A unified cognitive architecture for physical agents. AAAI, 21, 1469–1474.
6. Rosenbloom, P. S., Demski, A., & Ustun, V. (2016). The Sigma cognitive architecture and system. AISB Quarterly, 136, 4–13.
7. Newell, A. (1990). Unified Theories of Cognition. Harvard University Press.
