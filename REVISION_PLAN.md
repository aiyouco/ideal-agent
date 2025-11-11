# Piano di Revisione: Da Implementazione a Ricerca/Design

## Problema Identificato

La documentazione attuale contiene troppi dettagli implementativi (snippet di codice, esempi di implementazione) quando dovrebbe rimanere a livello di ricerca e design architetturale.

## Obiettivo della Revisione

Trasformare la documentazione in una **ricerca architetturale di alto livello** che:
- Presenta principi e pattern di design
- Analizza trade-off e decisioni architetturali
- Confronta approcci alternativi
- Fornisce framework concettuali
- **NON include** codice implementativo

## Stato Corrente

### ✅ Completato
- README.md → Riscritto a livello ricerca (no codice)
- 06-implementazione-pratica.md → Rimosso completamente

### ⏳ Da Rivedere
- 00-visione-principi.md → Rimuovere snippet Python, mantenere solo pattern concettuali
- 01-architettura-core.md → Rimuovere implementazioni, mantenere solo architettura

### 📝 Da Creare
- 02-decisioni-tradeoff.md → Architecture Decision Records formali
- 03-comparazione-sistemi.md → Analisi comparativa sistemica

## Principi per la Revisione

### ❌ Evitare
- Snippet di codice implementativo
- Dettagli di API specifiche
- Esempi di utilizzo concreti
- Tutorial e how-to
- Best practices implementative

### ✅ Include
- Diagrammi architetturali
- Pattern di design concettuali
- Analisi trade-off
- Decision rationale
- Comparative analysis
- Formal specifications (quando appropriato)
- Metriche di valutazione architetturale

## Approccio Corretto

### Invece di:
```python
class Agent:
    def run(self, task):
        context = self.memory.retrieve(task)
        response = self.llm.generate(...)
```

### Usare:
```
ARCHITETTURA: Execution Loop

Pattern: Control Cycle
Componenti: Perceive → Reason → Act

Specifiche:
- Perceive: Retrieval contestuale da memoria
- Reason: Generazione decisionale via LLM
- Act: Esecuzione tool-based
  
Trade-off:
- Loop esplicito vs. recursive calls
- Decisione: Loop esplicito per observability

Alternative considerate:
1. ReAct pattern (singola chiamata)
2. Multi-agent orchestration  
3. Event-driven architecture

Scelta: Loop esplicito per controllo e tracciabilità
```

## Timeline Revisione

1. **Immediate** (oggi): 
   - Rivedere 00-visione-principi.md
   - Rivedere 01-architettura-core.md
   
2. **Short-term** (domani):
   - Creare 02-decisioni-tradeoff.md
   - Creare 03-comparazione-sistemi.md

3. **Final** (chiusura):
   - Review completa
   - Commit finale
   - Push

## Output Atteso

Documentazione che può essere:
- Presentata in conferenza accademica
- Usata come design specification per team
- Base per discussion architetturale
- Riferimento per decisioni di design

**NON**:
- Tutorial implementativo
- Code walkthrough
- Implementation guide
