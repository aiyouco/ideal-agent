# Architettura Core: I 3 Componenti Essenziali

> **"Everything should be made as simple as possible, but not simpler."** — Einstein

## Overview

Un agente funzionante richiede esattamente **3 componenti**:

1. **Execution Loop** - Il ciclo Perceive → Think → Act
2. **Memory** - Context window + Long-term storage
3. **Tools** - Interfaccia per azioni nel mondo

Niente di più, niente di meno.

## Diagramma Architetturale Completo

```
┌─────────────────────────────────────────────────────────┐
│                     AGENT                               │
│                                                         │
│  ┌───────────────────────────────────────────────────┐ │
│  │           1. EXECUTION LOOP                       │ │
│  │                                                   │ │
│  │   Input (Task)                                    │ │
│  │       ↓                                           │ │
│  │   Perceive ─────→ Memory.retrieve(task)          │ │
│  │       ↓                  │                        │ │
│  │   Think  ←───────────────┘                        │ │
│  │       │                                           │ │
│  │       │  (LLM generates response with tools)     │ │
│  │       ↓                                           │ │
│  │   Act ───────→ Tools.execute(tool_call)          │ │
│  │       │               │                           │ │
│  │       │               ↓                           │ │
│  │       │        Memory.store(result)              │ │
│  │       ↓                                           │ │
│  │   Output / Loop back to Perceive                 │ │
│  └───────────────────────────────────────────────────┘ │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────┐  │
│  │  2. MEMORY   │  │   3. TOOLS   │  │ Infra Layers│  │
│  │              │  │              │  │             │  │
│  │ - Context    │  │ - Registry   │  │ - Logging   │  │
│  │ - Long-term  │  │ - Execute    │  │ - Tracing   │  │
│  │ - Retrieve   │  │ - Validate   │  │ - Security  │  │
│  └──────────────┘  └──────────────┘  └─────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## Componente 1: Execution Loop

### Concetto

Il loop di esecuzione è il "cuore" dell'agente. Implementa un ciclo semplice:

```
while not task_complete:
    1. Perceive: Raccogliere informazioni rilevanti
    2. Think: Ragionare sulla prossima azione
    3. Act: Eseguire azioni (o return output)
```

### Implementazione Minimale

```python
class ExecutionLoop:
    def __init__(self, llm, memory, tools):
        self.llm = llm
        self.memory = memory
        self.tools = tools

    def run(self, task: str, max_iterations: int = 10) -> str:
        """
        Esegue il task fino al completamento o max iterations.
        """
        for i in range(max_iterations):
            # PERCEIVE: Recupera contesto rilevante
            context = self.memory.retrieve(task)

            # THINK: LLM ragiona e decide azione
            response = self.llm.generate(
                system=self.get_system_prompt(),
                messages=[
                    {"role": "user", "content": task},
                    {"role": "context", "content": context}
                ],
                tools=self.tools.available()
            )

            # ACT: Esegui tool calls o ritorna output
            if response.tool_calls:
                for call in response.tool_calls:
                    result = self.tools.execute(call)
                    self.memory.store({
                        "type": "tool_execution",
                        "tool": call.name,
                        "input": call.args,
                        "output": result,
                        "timestamp": now()
                    })

            # Check se task è completo
            if response.finished or not response.tool_calls:
                return response.content

        return "Max iterations reached. Task incomplete."

    def get_system_prompt(self) -> str:
        return """
        You are a helpful AI agent. You can use tools to accomplish tasks.

        When given a task:
        1. Break it down into steps if needed
        2. Use available tools to execute steps
        3. Verify results before proceeding
        4. Return final answer when complete

        Think step-by-step and be precise.
        """
```

**That's it**. Questo è l'intero execution loop in ~40 righe.

### Perché Funziona

- **Perceive**: Il context window dell'LLM è limitato. Memory retrieval porta solo informazioni rilevanti.
- **Think**: L'LLM usa il suo ragionamento nativo (chain-of-thought implicito) per decidere.
- **Act**: Tool execution permette azioni concrete nel mondo.

Nessun bisogno di:
- Goal manager esplicito (LLM gestisce goal implicitamente)
- Planner separato (LLM pianifica naturalmente)
- State machine complessa (il loop è lo state machine)

### Estensioni Opzionali

Per casi più avanzati:

```python
class AdvancedExecutionLoop(ExecutionLoop):
    def run(self, task: str, max_iterations: int = 10) -> str:
        """Extended con observability e error handling"""

        with trace("agent_execution", task=task) as t:
            for i in range(max_iterations):
                try:
                    with t.span(f"iteration_{i}"):
                        # Perceive
                        context = self.memory.retrieve(task)
                        t.log("context_size", len(context))

                        # Think
                        response = self.llm.generate(...)
                        t.log("llm_response", response)

                        # Act
                        if response.tool_calls:
                            results = self.execute_tools_safely(
                                response.tool_calls
                            )
                            t.log("tool_results", results)

                        # Check completion
                        if self.is_complete(response):
                            t.log("status", "completed")
                            return response.content

                except Exception as e:
                    t.log("error", str(e))
                    if not self.is_retriable(e):
                        raise

            t.log("status", "max_iterations_reached")
            return "Task incomplete after max iterations"
```

Ma il core rimane lo stesso: **Perceive → Think → Act**.

## Componente 2: Memory

### Concetto

La memoria permette all'agente di:
- Ricordare conversazioni passate (context)
- Richiamare esperienze rilevanti (long-term)
- Imparare da successi/fallimenti

### Architettura Memoria

```
Memory System
├── Working Memory (Context Window)
│   ├── Capacity: 8K-128K tokens
│   ├── Lifetime: Current session
│   └── Content: Recent messages, active task info
│
└── Long-Term Memory (Persistent)
    ├── Storage: Vector DB + Metadata
    ├── Lifetime: Indefinite
    └── Content: Past conversations, tool results, learnings
```

### Implementazione Minimale

```python
class UnifiedMemory:
    def __init__(self, vector_db, context_window_size=100000):
        self.vector_db = vector_db  # e.g., Pinecone, Weaviate
        self.context = []  # Working memory
        self.context_window_size = context_window_size

    def retrieve(self, query: str, limit: int = 5) -> list:
        """
        Recupera memorie rilevanti per la query.
        Combina working memory + long-term memory.
        """
        # Get from long-term memory
        long_term = self.vector_db.search(
            query=query,
            limit=limit
        )

        # Get from working memory (recent context)
        working = self.context[-limit:]

        # Combine (deduplicate if needed)
        return working + long_term

    def store(self, memory: dict):
        """
        Memorizza in both working e long-term.
        """
        # Add to working memory
        self.context.append(memory)

        # Trim if exceeds window
        self.trim_context()

        # Store in long-term
        self.vector_db.insert(
            text=json.dumps(memory),
            metadata=memory.get("metadata", {}),
            embedding=self.embed(memory)
        )

    def trim_context(self):
        """Mantiene context entro window size"""
        total_tokens = sum(
            estimate_tokens(m) for m in self.context
        )

        while total_tokens > self.context_window_size:
            # Remove oldest (or least important)
            removed = self.context.pop(0)
            total_tokens -= estimate_tokens(removed)

    def embed(self, memory: dict) -> list[float]:
        """Crea embedding per similarity search"""
        text = json.dumps(memory)
        return embedding_model.encode(text)
```

### Cosa Memorizzare?

Memorizza eventi significativi:

```python
# Tool execution
memory.store({
    "type": "tool_execution",
    "tool": "search_web",
    "input": "latest AI news",
    "output": "...",
    "timestamp": now(),
    "success": True
})

# User interaction
memory.store({
    "type": "interaction",
    "user_message": "Build me a todo app",
    "agent_response": "I'll create a todo app with...",
    "timestamp": now()
})

# Learning/Reflection
memory.store({
    "type": "learning",
    "insight": "When building web apps, always start with HTML structure",
    "source": "episode_123",
    "confidence": 0.8
})
```

### Perché NON servono 4 tipi separati?

**Memoria Episodica vs Semantica vs Procedurale**: Nella pratica, la distinzione è sfumata.

Un semplice vector store con good metadata può servire tutti i casi:

```python
# "Episodica" - cosa è successo
memory.store({
    "type": "episode",
    "content": "Last time I built a web app, I used Flask"
})

# "Semantica" - fatti
memory.store({
    "type": "fact",
    "content": "Flask is a Python web framework"
})

# "Procedurale" - come fare
memory.store({
    "type": "procedure",
    "content": "To build a Flask app: 1) Install flask, 2) Create app.py..."
})

# All retrieved via same mechanism:
results = vector_db.search("How to build a web app?")
# Returns mix of episodes, facts, procedures - all relevant!
```

Il metadata "type" permette filtering se serve, ma non richiede architetture separate.

## Componente 3: Tools

### Concetto

Tools permettono all'agente di **agire** nel mondo:
- Read/write files
- Search web
- Execute code
- Call APIs
- Control browser
- etc.

### Interfaccia Tool Standard

```python
class Tool:
    """Base interface per tutti i tools"""

    name: str
    description: str
    parameters: dict  # JSON Schema

    def execute(self, **kwargs) -> Any:
        """Esegue il tool con parametri dati"""
        raise NotImplementedError

    def validate(self, **kwargs) -> bool:
        """Valida parametri prima dell'esecuzione"""
        raise NotImplementedError
```

### Esempio Tools

```python
class SearchWeb(Tool):
    name = "search_web"
    description = "Search the web for information"
    parameters = {
        "query": {"type": "string", "required": True},
        "limit": {"type": "integer", "default": 5}
    }

    def execute(self, query: str, limit: int = 5):
        # Call search API
        results = search_api.search(query, limit=limit)
        return results

class ReadFile(Tool):
    name = "read_file"
    description = "Read contents of a file"
    parameters = {
        "path": {"type": "string", "required": True}
    }

    def execute(self, path: str):
        with open(path, 'r') as f:
            return f.read()

class ExecuteCode(Tool):
    name = "execute_code"
    description = "Execute Python code in a sandbox"
    parameters = {
        "code": {"type": "string", "required": True}
    }

    def execute(self, code: str):
        # Execute in sandbox
        result = sandbox.run(code, timeout=30)
        return result.output
```

### Tool Registry

```python
class ToolRegistry:
    def __init__(self):
        self.tools = {}

    def register(self, tool: Tool):
        """Registra un nuovo tool"""
        self.tools[tool.name] = tool

    def available(self) -> list[dict]:
        """Return tool definitions per LLM function calling"""
        return [
            {
                "name": tool.name,
                "description": tool.description,
                "parameters": tool.parameters
            }
            for tool in self.tools.values()
        ]

    def execute(self, tool_call: dict) -> Any:
        """Esegue tool call dal LLM"""
        tool_name = tool_call["name"]
        tool_args = tool_call["arguments"]

        # Get tool
        tool = self.tools.get(tool_name)
        if not tool:
            raise ValueError(f"Unknown tool: {tool_name}")

        # Validate
        if not tool.validate(**tool_args):
            raise ValueError(f"Invalid arguments for {tool_name}")

        # Execute
        return tool.execute(**tool_args)
```

### Safe Execution

Per produzione, add safety:

```python
class SecureToolRegistry(ToolRegistry):
    def execute(self, tool_call: dict) -> Any:
        """Execute con security checks"""

        # 1. Validate input
        validated = self.validate_input(tool_call)

        # 2. Check permissions
        if not self.has_permission(tool_call["name"]):
            raise PermissionError()

        # 3. Execute in sandbox
        result = self.sandbox_execute(validated)

        # 4. Validate output
        sanitized = self.sanitize_output(result)

        # 5. Log for audit
        self.audit_log(tool_call, result)

        return sanitized

    def sandbox_execute(self, tool_call: dict):
        """Execute con resource limits"""
        tool = self.tools[tool_call["name"]]

        with ResourceLimits(max_time=30, max_memory=1GB):
            return tool.execute(**tool_call["arguments"])
```

## Infrastruttura di Supporto

Oltre ai 3 componenti core, serve infrastruttura minima:

### Observability

```python
class Trace:
    """Semplice distributed tracing"""

    def __init__(self, name: str):
        self.name = name
        self.start = time.time()
        self.spans = []
        self.logs = []

    def span(self, name: str):
        """Crea nested span"""
        return Trace(f"{self.name}.{name}")

    def log(self, key: str, value: Any):
        """Log evento"""
        self.logs.append({
            "timestamp": time.time(),
            "key": key,
            "value": value
        })

    def end(self):
        """Finalizza trace"""
        self.duration = time.time() - self.start
        # Send to tracing backend (e.g., OpenTelemetry)
        send_to_backend(self)
```

### Configuration

```python
@dataclass
class AgentConfig:
    """Configurazione centrale"""

    # LLM
    llm_provider: str = "anthropic"
    llm_model: str = "claude-3-5-sonnet"
    llm_temperature: float = 0.0

    # Memory
    memory_backend: str = "pinecone"
    context_window: int = 100000

    # Execution
    max_iterations: int = 10
    timeout: int = 300  # seconds

    # Safety
    sandbox_enabled: bool = True
    allowed_tools: list[str] = None
```

## Putting It All Together

```python
class Agent:
    """L'agente completo"""

    def __init__(self, config: AgentConfig):
        # Initialize components
        self.llm = LLM(config.llm_provider, config.llm_model)
        self.memory = UnifiedMemory(
            vector_db=VectorDB(config.memory_backend),
            context_window_size=config.context_window
        )
        self.tools = ToolRegistry()

        # Register tools
        self.tools.register(SearchWeb())
        self.tools.register(ReadFile())
        self.tools.register(ExecuteCode())
        # ... register more tools

        # Execution loop
        self.loop = ExecutionLoop(self.llm, self.memory, self.tools)

    def run(self, task: str) -> str:
        """Entry point principale"""
        with trace("agent_run") as t:
            t.log("task", task)

            result = self.loop.run(
                task=task,
                max_iterations=self.config.max_iterations
            )

            t.log("result", result)
            return result

# Usage
config = AgentConfig()
agent = Agent(config)

result = agent.run("Search for latest AI news and summarize")
print(result)
```

## Performance Caratteristiche

| Metrica | Target | Come Raggiunto |
|---------|--------|----------------|
| Setup time | <1 min | Minimal dependencies |
| First response | <2s | Fast LLM + context caching |
| Memory retrieval | <100ms | Vector DB indexed search |
| Tool execution | Variable | Depends on tool |
| End-to-end (simple) | <5s | Optimized loop |
| End-to-end (complex) | <2min | Multi-iteration with tools |

## Scalabilità

### Vertical Scaling

Single instance può gestire:
- 10-100 concurrent users
- 1000+ tools registered
- Millioni di memorie in long-term storage

### Horizontal Scaling

Componenti sono **stateless** (eccetto memory):
- Execution Loop: Stateless, scale infinitamente
- Tools: Stateless, scale infinitamente
- Memory: Backed by scalable DB (Pinecone, etc.)

Quindi: Deploy N instances dietro load balancer.

## Estensibilità

### Adding Capabilities = Adding Tools

```python
# Nuova capability: Vision
class AnalyzeImage(Tool):
    name = "analyze_image"
    description = "Analyze an image and describe contents"
    parameters = {"image_url": {"type": "string", "required": True}}

    def execute(self, image_url: str):
        return vision_model.analyze(image_url)

# Register
agent.tools.register(AnalyzeImage())

# Ora l'agente può analizzare immagini!
agent.run("What's in this image? https://example.com/image.jpg")
```

**Nessuna modifica al core.** Solo nuovo tool.

### Swapping Components

```python
# Swap LLM provider
agent.llm = LLM("openai", "gpt-4")

# Swap memory backend
agent.memory = UnifiedMemory(vector_db=Weaviate())

# Swap tools
agent.tools = CustomToolRegistry()
```

Tutto modular e swappable.

## Testing

### Unit Tests

```python
def test_execution_loop():
    # Mock dependencies
    llm = MockLLM()
    memory = MockMemory()
    tools = MockTools()

    loop = ExecutionLoop(llm, memory, tools)

    # Test
    result = loop.run("test task")

    # Assertions
    assert llm.generate_called
    assert memory.retrieve_called
    assert result == "expected output"
```

### Integration Tests

```python
def test_agent_e2e():
    agent = Agent(test_config)

    result = agent.run("Search for AI news")

    assert "AI" in result
    assert agent.memory.context  # Memory was used
```

## Conclusione

**3 componenti. ~500 LOC. Production-ready.**

Questa è l'architettura ideale perché:

1. **Simple** - Chiunque può capire in <30 min
2. **Fast** - Implementare MVP in <1 settimana
3. **Extensible** - Nuove capability senza riscritture
4. **Performant** - Latenza sub-2s per query semplici
5. **Debuggable** - Traces chiare mostrano ogni step

Non serve più di questo per iniziare. Il resto emerge quando serve.

---

**Next**: [06-implementazione-pratica.md](06-implementazione-pratica.md) - Codice completo e runnable →
