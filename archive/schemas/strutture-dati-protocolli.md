# Strutture Dati e Protocolli

**Stato del Documento:** 🟢 Completo
**Ultimo Aggiornamento:** 2025-11-10
**Autore:** Kilo Code

## Indice

1. [Panoramica](#panoramica)
2. [Strutture Dati Principali](#strutture-dati-principali)
3. [Schemi JSON](#schemi-json)
4. [Protocolli di Comunicazione](#protocolli-di-comunicazione)
5. [Formati di Serializzazione](#formati-di-serializzazione)
6. [Protocolli di Rete](#protocolli-di-rete)
7. [Validazione degli Schemi](#validazione-degli-schemi)

---

## Panoramica

Questo documento definisce tutte le strutture dati e i protocolli di comunicazione utilizzati nell'intero sistema dell'agente ideale. Tutte le strutture sono progettate per:

1. **Serializzabilità**: JSON, Protocol Buffers, MessagePack
2. **Versionamento**: Evoluzione dello schema senza rompere i client
3. **Validazione**: Applicazione automatica dello schema
4. **Tracciabilità**: Tutte le strutture includono metadati
5. **Efficienza**: Ottimizzate per storage e trasmissione

---

## Strutture Dati Principali

### Goal

Schema JSON completo:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://agent.example.com/schemas/goal.json",
  "title": "Goal",
  "description": "A goal to be achieved by the agent",
  "type": "object",
  "required": ["id", "description", "type", "status", "createdAt"],
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid",
      "description": "Unique goal identifier"
    },
    "description": {
      "type": "string",
      "minLength": 1,
      "maxLength": 10000,
      "description": "Natural language description of goal"
    },
    "type": {
      "type": "string",
      "enum": ["achievement", "maintenance", "avoidance", "exploration"],
      "description": "Type of goal"
    },
    "parent": {
      "type": ["string", "null"],
      "format": "uuid",
      "description": "Parent goal ID (null if top-level)"
    },
    "children": {
      "type": "array",
      "items": {
        "type": "string",
        "format": "uuid"
      },
      "description": "Child goal IDs"
    },
    "status": {
      "type": "string",
      "enum": ["pending", "active", "blocked", "completed", "failed", "abandoned"],
      "description": "Current goal status"
    },
    "successCriteria": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/Criterion"
      },
      "description": "Criteria for goal achievement"
    },
    "constraints": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/Constraint"
      },
      "description": "Constraints on goal achievement"
    },
    "createdAt": {
      "type": "integer",
      "description": "Creation timestamp (Unix milliseconds)"
    },
    "completedAt": {
      "type": ["integer", "null"],
      "description": "Completion timestamp (null if not completed)"
    },
    "priority": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Goal priority (0-1)"
    },
    "tags": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "Tags for categorization"
    }
  },
  "definitions": {
    "Criterion": {
      "type": "object",
      "required": ["type", "description"],
      "properties": {
        "type": {
          "type": "string",
          "enum": ["state_property", "observable", "user_confirmation", "test_passes", "invariant"]
        },
        "description": {
          "type": "string"
        },
        "value": {
          "description": "Expected value (type-dependent)"
        }
      }
    },
    "Constraint": {
      "type": "object",
      "required": ["type"],
      "properties": {
        "type": {
          "type": "string",
          "enum": ["resource", "time", "quality", "safety"]
        },
        "value": {
          "description": "Constraint value (type-dependent)"
        }
      }
    }
  }
}
```

### Episode

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://agent.example.com/schemas/episode.json",
  "title": "Episode",
  "description": "Complete record of task execution",
  "type": "object",
  "required": ["id", "goal", "outcome", "duration", "resourcesUsed", "timestamp"],
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "goal": {
      "$ref": "goal.json"
    },
    "plan": {
      "$ref": "plan.json"
    },
    "execution": {
      "$ref": "#/definitions/ExecutionResult"
    },
    "outcome": {
      "type": "object",
      "required": ["success", "type"],
      "properties": {
        "success": {
          "type": "boolean"
        },
        "type": {
          "type": "string",
          "enum": ["success", "partial_success", "failure", "timeout", "budget_exceeded", "cancelled"]
        },
        "result": {
          "description": "Task result (type-dependent)"
        },
        "error": {
          "type": "object",
          "properties": {
            "message": { "type": "string" },
            "code": { "type": "string" },
            "stack": { "type": "string" }
          }
        },
        "confidence": {
          "type": "number",
          "minimum": 0,
          "maximum": 1
        }
      }
    },
    "duration": {
      "type": "integer",
      "minimum": 0,
      "description": "Duration in milliseconds"
    },
    "resourcesUsed": {
      "$ref": "#/definitions/ResourceUsage"
    },
    "trace": {
      "$ref": "trace.json"
    },
    "reflection": {
      "$ref": "reflection.json"
    },
    "timestamp": {
      "type": "integer",
      "description": "Episode timestamp"
    },
    "tags": {
      "type": "array",
      "items": { "type": "string" }
    },
    "embedding": {
      "type": "array",
      "items": { "type": "number" },
      "description": "Episode embedding vector"
    }
  },
  "definitions": {
    "ResourceUsage": {
      "type": "object",
      "properties": {
        "tokens": { "type": "integer", "minimum": 0 },
        "llmCalls": { "type": "integer", "minimum": 0 },
        "toolCalls": { "type": "integer", "minimum": 0 },
        "wallTime": { "type": "integer", "minimum": 0 },
        "cost": { "type": "number", "minimum": 0 }
      }
    }
  }
}
```

### Plan

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://agent.example.com/schemas/plan.json",
  "title": "Plan",
  "description": "Structured execution plan",
  "type": "object",
  "required": ["id", "goalId", "steps"],
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "goalId": {
      "type": "string",
      "format": "uuid"
    },
    "steps": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/PlanStep"
      },
      "minItems": 1
    },
    "dependencies": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/Dependency"
      }
    },
    "estimatedCost": {
      "$ref": "#/definitions/ResourceEstimate"
    },
    "strategy": {
      "type": "string",
      "enum": ["greedy", "react", "tree_search", "plan_execute", "iterative"]
    },
    "createdAt": {
      "type": "integer"
    }
  },
  "definitions": {
    "PlanStep": {
      "type": "object",
      "required": ["id", "name", "type", "action"],
      "properties": {
        "id": { "type": "string", "format": "uuid" },
        "name": { "type": "string" },
        "type": {
          "type": "string",
          "enum": ["reasoning", "tool_use", "subgoal", "decision", "validation", "wait"]
        },
        "action": {
          "type": "object",
          "required": ["type"],
          "properties": {
            "type": { "type": "string" },
            "parameters": { "type": "object" },
            "timeout": { "type": "integer" }
          }
        },
        "preconditions": {
          "type": "array",
          "items": { "type": "object" }
        },
        "postconditions": {
          "type": "array",
          "items": { "type": "object" }
        },
        "dependencies": {
          "type": "array",
          "items": { "type": "string", "format": "uuid" }
        }
      }
    },
    "Dependency": {
      "type": "object",
      "required": ["from", "to", "type"],
      "properties": {
        "from": { "type": "string", "format": "uuid" },
        "to": { "type": "string", "format": "uuid" },
        "type": {
          "type": "string",
          "enum": ["required", "optional", "data"]
        }
      }
    },
    "ResourceEstimate": {
      "type": "object",
      "properties": {
        "tokens": { "type": "integer" },
        "llmCalls": { "type": "integer" },
        "duration": { "type": "integer" },
        "cost": { "type": "number" }
      }
    }
  }
}
```

### Tool

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://agent.example.com/schemas/tool.json",
  "title": "Tool",
  "description": "Tool definition and metadata",
  "type": "object",
  "required": ["id", "name", "version", "description", "inputSchema", "outputSchema"],
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "name": {
      "type": "string",
      "pattern": "^[a-z][a-z0-9_]*$",
      "description": "Tool name (lowercase, underscores)"
    },
    "version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$",
      "description": "Semantic version"
    },
    "description": {
      "type": "string",
      "minLength": 10,
      "maxLength": 1000
    },
    "category": {
      "type": "string",
      "enum": [
        "file_system", "process", "network",
        "database", "api", "search",
        "code_execution", "compiler", "debugger",
        "browser", "ui_automation",
        "llm", "vision", "embedding",
        "math", "text_processing", "data_transform"
      ]
    },
    "inputSchema": {
      "type": "object",
      "description": "JSON Schema for input validation"
    },
    "outputSchema": {
      "type": "object",
      "description": "JSON Schema for output validation"
    },
    "capabilities": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/Capability"
      }
    },
    "requirements": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/Requirement"
      }
    },
    "properties": {
      "$ref": "#/definitions/ToolProperties"
    },
    "documentation": {
      "type": "string"
    },
    "examples": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/ToolExample"
      }
    },
    "estimatedLatency": {
      "type": "integer",
      "minimum": 0,
      "description": "Estimated latency in milliseconds"
    },
    "estimatedCost": {
      "type": "number",
      "minimum": 0,
      "description": "Estimated cost in dollars"
    },
    "reliability": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Historical reliability score"
    }
  },
  "definitions": {
    "Capability": {
      "type": "object",
      "required": ["type", "description"],
      "properties": {
        "type": {
          "type": "string",
          "enum": ["read", "write", "execute", "query", "transform", "generate", "validate"]
        },
        "description": {
          "type": "string"
        }
      }
    },
    "Requirement": {
      "type": "object",
      "required": ["type", "value"],
      "properties": {
        "type": {
          "type": "string",
          "enum": ["permission", "dependency", "resource", "credential", "environment"]
        },
        "value": {},
        "optional": {
          "type": "boolean",
          "default": false
        }
      }
    },
    "ToolProperties": {
      "type": "object",
      "properties": {
        "idempotent": { "type": "boolean", "default": false },
        "deterministic": { "type": "boolean", "default": false },
        "sideEffects": { "type": "boolean", "default": true },
        "stateful": { "type": "boolean", "default": false },
        "async": { "type": "boolean", "default": false },
        "streaming": { "type": "boolean", "default": false },
        "cacheable": { "type": "boolean", "default": false },
        "retriable": { "type": "boolean", "default": true },
        "timeout": {
          "type": "integer",
          "minimum": 1000,
          "maximum": 300000,
          "default": 30000
        }
      }
    },
    "ToolExample": {
      "type": "object",
      "required": ["description", "input", "output"],
      "properties": {
        "description": { "type": "string" },
        "input": { "type": "object" },
        "output": {}
      }
    }
  }
}
```

### Memory

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://agent.example.com/schemas/memory.json",
  "title": "Memory",
  "description": "Base memory structure",
  "type": "object",
  "required": ["id", "type", "content", "activation", "timestamp"],
  "properties": {
    "id": {
      "type": "string",
      "format": "uuid"
    },
    "type": {
      "type": "string",
      "enum": ["episodic", "semantic", "procedural"]
    },
    "content": {
      "description": "Memory content (type-specific structure)"
    },
    "activation": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Current activation level"
    },
    "accessCount": {
      "type": "integer",
      "minimum": 0,
      "description": "Number of times accessed"
    },
    "lastAccess": {
      "type": "integer",
      "description": "Last access timestamp"
    },
    "timestamp": {
      "type": "integer",
      "description": "Creation timestamp"
    },
    "tags": {
      "type": "array",
      "items": { "type": "string" }
    },
    "embedding": {
      "type": "array",
      "items": { "type": "number" },
      "description": "Embedding vector for similarity search"
    }
  }
}
```

### Trace

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://agent.example.com/schemas/trace.json",
  "title": "ExecutionTrace",
  "description": "Distributed trace following OpenTelemetry spec",
  "type": "object",
  "required": ["traceId", "rootSpanId", "spans", "startTime"],
  "properties": {
    "traceId": {
      "type": "string",
      "pattern": "^[0-9a-f]{32}$",
      "description": "128-bit trace ID (hex)"
    },
    "rootSpanId": {
      "type": "string",
      "pattern": "^[0-9a-f]{16}$",
      "description": "64-bit root span ID (hex)"
    },
    "spans": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/Span"
      }
    },
    "startTime": {
      "type": "integer",
      "description": "Trace start time (Unix nanoseconds)"
    },
    "endTime": {
      "type": "integer",
      "description": "Trace end time (Unix nanoseconds)"
    },
    "duration": {
      "type": "integer",
      "description": "Total duration (nanoseconds)"
    },
    "status": {
      "type": "string",
      "enum": ["ok", "error", "unset"]
    }
  },
  "definitions": {
    "Span": {
      "type": "object",
      "required": ["spanId", "traceId", "name", "startTime"],
      "properties": {
        "spanId": {
          "type": "string",
          "pattern": "^[0-9a-f]{16}$"
        },
        "traceId": {
          "type": "string",
          "pattern": "^[0-9a-f]{32}$"
        },
        "parentSpanId": {
          "type": ["string", "null"],
          "pattern": "^[0-9a-f]{16}$"
        },
        "name": {
          "type": "string"
        },
        "kind": {
          "type": "string",
          "enum": ["internal", "server", "client", "producer", "consumer"]
        },
        "startTime": {
          "type": "integer"
        },
        "endTime": {
          "type": "integer"
        },
        "attributes": {
          "type": "object",
          "additionalProperties": true
        },
        "events": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "timestamp": { "type": "integer" },
              "name": { "type": "string" },
              "attributes": { "type": "object" }
            }
          }
        },
        "status": {
          "type": "object",
          "properties": {
            "code": {
              "type": "string",
              "enum": ["ok", "error", "unset"]
            },
            "message": {
              "type": "string"
            }
          }
        }
      }
    }
  }
}
```

---

## Protocolli di Comunicazione

### Protocollo Request-Response

#### Richiesta Standard

```json
{
  "protocol": "agent/v1",
  "request": {
    "id": "req_abc123",
    "method": "execute",
    "params": {
      "goal": { ... },
      "budget": { ... }
    },
    "metadata": {
      "userId": "user_456",
      "sessionId": "session_789",
      "timestamp": 1699564800000
    }
  }
}
```

#### Risposta Standard

```json
{
  "protocol": "agent/v1",
  "response": {
    "id": "req_abc123",
    "result": {
      "episode": { ... }
    },
    "metadata": {
      "duration": 145000,
      "tokensUsed": 45000,
      "cost": 2.34
    }
  }
}
```

#### Risposta di Errore

```json
{
  "protocol": "agent/v1",
  "response": {
    "id": "req_abc123",
    "error": {
      "code": "EXECUTION_FAILED",
      "message": "Tool execution timeout",
      "data": {
        "toolId": "tool_123",
        "timeout": 30000,
        "elapsed": 31000
      }
    }
  }
}
```

---

### Protocollo Streaming

Basato su Server-Sent Events (SSE):

#### Tipi di Eventi

```typescript
interface StreamEvent {
  id: string
  event: StreamEventType
  data: any
  retry?: number
}

enum StreamEventType {
  STARTED = 'started',
  PROGRESS = 'progress',
  OUTPUT = 'output',
  COMPLETED = 'completed',
  ERROR = 'error'
}
```

#### Formati degli Eventi

**Started**:
```
event: started
id: evt_1
data: {"requestId":"req_123","timestamp":1699564800000}

```

**Progress**:
```
event: progress
id: evt_2
data: {"stepsCompleted":2,"stepsTotal":5,"percentComplete":40}

```

**Output** (streaming tokens):
```
event: output
id: evt_3
data: {"chunk":"def fibonacci(n):\n","sequence":1}

```

**Completed**:
```
event: completed
id: evt_999
data: {"episode":{...},"trace":{...}}

```

**Error**:
```
event: error
id: evt_500
data: {"code":"TIMEOUT","message":"Execution timeout"}

```

---

### Protocollo di Esecuzione degli Strumenti

#### Richiesta Strumento

```json
{
  "protocol": "tool/v1",
  "request": {
    "id": "exec_abc123",
    "tool": "file_reader",
    "version": "1.0.0",
    "parameters": {
      "path": "/path/to/file.txt"
    },
    "context": {
      "requestId": "req_456",
      "userId": "user_789",
      "permissions": ["file:read"],
      "budget": {
        "timeout": 30000,
        "maxMemory": 536870912
      }
    }
  }
}
```

#### Risposta Strumento

```json
{
  "protocol": "tool/v1",
  "response": {
    "id": "exec_abc123",
    "success": true,
    "output": {
      "content": "File contents...",
      "encoding": "utf-8",
      "size": 1024
    },
    "metadata": {
      "duration": 125,
      "resourcesUsed": {
        "cpu": 0.05,
        "memory": 1048576,
        "io": 1024
      },
      "cacheHit": false
    }
  }
}
```

---

## Formati di Serializzazione

### JSON (Predefinito)

**Pro**:
- Leggibile dall'umano
- Supporto universale
- Auto-descrittivo

**Contro**:
- Verboso
- Dimensione maggiore
- Parsing più lento

**Utilizzare Per**:
- Risposte API
- File di configurazione
- Output di debug

**Esempio**:
```json
{
  "id": "goal_123",
  "description": "Create function",
  "type": "achievement"
}
```

### MessagePack (Ottimizzato)

**Pro**:
- Formato binario compatto
- Più veloce di JSON
- Supporta dati binari

**Contro**:
- Non leggibile dall'umano
- Richiede libreria

**Utilizzare Per**:
- RPC interno
- Storage cache
- Dati ad alto volume

**Confronto Dimensioni**:
- JSON: 1000 bytes
- MessagePack: ~400 bytes (60% più piccolo)

### Protocol Buffers (Alte Prestazioni)

**Pro**:
- Dimensione minima
- Parsing più veloce
- Compatibilità forward/backward
- Generazione di codice

**Contro**:
- Richiede compilazione dello schema
- Non leggibile dall'umano

**Utilizzare Per**:
- Servizi gRPC
- Comunicazione ad alta frequenza
- Protocolli binari

**Esempio di Schema**:
```protobuf
message Goal {
  string id = 1;
  string description = 2;
  GoalType type = 3;
  repeated Criterion success_criteria = 4;
  int64 created_at = 5;
}
```

---

## Protocolli di Rete

### HTTP/REST

**Protocollo**: HTTP/1.1 o HTTP/2
**Formato**: JSON
**Content-Type**: `application/json`

**Headers**:
```http
POST /v1/tasks HTTP/1.1
Host: api.agent.example.com
Authorization: Bearer <token>
Content-Type: application/json
Accept: application/json
X-Request-ID: req_123
X-Trace-ID: trace_456
Content-Length: 1234

{...}
```

### WebSocket

**Protocollo**: WebSocket (RFC 6455)
**Formato**: JSON
**Upgrade**: Da HTTP

**Handshake**:
```http
GET /v1/stream HTTP/1.1
Host: api.agent.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

**Messaggi**:
```json
{"type":"execute","data":{...}}
{"type":"progress","data":{...}}
{"type":"complete","data":{...}}
```

### gRPC

**Protocollo**: HTTP/2
**Formato**: Protocol Buffers
**Content-Type**: `application/grpc+proto`

**Definizione del Servizio**:
```protobuf
service AgentService {
  rpc Execute(ExecuteRequest) returns (ExecuteResponse);
  rpc StreamExecute(StreamExecuteRequest) returns (stream ExecuteProgress);
}
```

**Metadata**:
```
authorization: Bearer <token>
x-request-id: req_123
x-trace-id: trace_456
```

---

## Validazione degli Schemi

### Pipeline di Validazione

```typescript
interface SchemaValidator {
  /**
   * Valida i dati rispetto allo schema
   * @param data - Dati da validare
   * @param schema - JSON Schema
   * @returns Risultato della validazione
   */
  validate(data: any, schema: JSONSchema): ValidationResult

  /**
   * Compila lo schema per validazioni ripetute più veloci
   * @param schema - JSON Schema
   * @returns Validatore compilato
   */
  compile(schema: JSONSchema): CompiledValidator
}

interface ValidationResult {
  valid: boolean
  errors: ValidationError[]
  warnings: ValidationWarning[]
}

interface ValidationError {
  path: string
  message: string
  expected: any
  actual: any
  schemaPath: string
}

type CompiledValidator = (data: any) => ValidationResult
```

### Validazione Runtime

```typescript
// Esempio: Valida goal a runtime
import Ajv from 'ajv'
import goalSchema from './schemas/goal.json'

const ajv = new Ajv()
const validateGoal = ajv.compile(goalSchema)

function ensureValidGoal(goal: any): Goal {
  const valid = validateGoal(goal)

  if (!valid) {
    throw new ValidationError(
      'Invalid goal',
      validateGoal.errors || []
    )
  }

  return goal as Goal
}
```

---

## Definizioni Protocol Buffers

### File .proto Completo

```protobuf
syntax = "proto3";

package agent.v1;

option go_package = "github.com/example/agent/proto/v1";

// Goal Types
enum GoalType {
  GOAL_TYPE_UNSPECIFIED = 0;
  ACHIEVEMENT = 1;
  MAINTENANCE = 2;
  AVOIDANCE = 3;
  EXPLORATION = 4;
}

enum GoalStatus {
  GOAL_STATUS_UNSPECIFIED = 0;
  PENDING = 1;
  ACTIVE = 2;
  BLOCKED = 3;
  COMPLETED = 4;
  FAILED = 5;
  ABANDONED = 6;
}

// Goal message
message Goal {
  string id = 1;
  string description = 2;
  GoalType type = 3;
  string parent = 4;
  repeated string children = 5;
  GoalStatus status = 6;
  repeated Criterion success_criteria = 7;
  repeated Constraint constraints = 8;
  int64 created_at = 9;
  int64 completed_at = 10;
  double priority = 11;
  repeated string tags = 12;
}

message Criterion {
  string type = 1;
  string description = 2;
  string value = 3;
}

message Constraint {
  string type = 1;
  string value = 2;
}

// Resource Budget
message ResourceBudget {
  int32 max_tokens = 1;
  int32 max_llm_calls = 2;
  int32 max_wall_time = 3;
  double max_cost = 4;
  int32 max_depth = 5;
}

// Episode
message Episode {
  string id = 1;
  Goal goal = 2;
  Plan plan = 3;
  ExecutionResult execution = 4;
  Outcome outcome = 5;
  int64 duration = 6;
  ResourceUsage resources_used = 7;
  ExecutionTrace trace = 8;
  Reflection reflection = 9;
  int64 timestamp = 10;
  repeated string tags = 11;
  repeated double embedding = 12;
}

message Plan {
  string id = 1;
  string goal_id = 2;
  repeated PlanStep steps = 3;
  repeated Dependency dependencies = 4;
  ResourceEstimate estimated_cost = 5;
  int64 created_at = 6;
}

message PlanStep {
  string id = 1;
  string name = 2;
  string type = 3;
  Action action = 4;
  repeated string dependencies = 5;
}

message Action {
  string type = 1;
  map<string, string> parameters = 2;
  int32 timeout = 3;
}

// Tool
message Tool {
  string id = 1;
  string name = 2;
  string version = 3;
  string description = 4;
  string category = 5;
  string input_schema = 6;  // JSON Schema come stringa
  string output_schema = 7;
  repeated Capability capabilities = 8;
  ToolProperties properties = 9;
  int32 estimated_latency = 10;
  double estimated_cost = 11;
  double reliability = 12;
}

message Capability {
  string type = 1;
  string description = 2;
}

message ToolProperties {
  bool idempotent = 1;
  bool deterministic = 2;
  bool side_effects = 3;
  bool cacheable = 4;
  bool retriable = 5;
  int32 timeout = 6;
}

// gRPC Service
service AgentService {
  // Esegue un obiettivo
  rpc Execute(ExecuteRequest) returns (ExecuteResponse);

  // Stream del progresso di esecuzione
  rpc StreamExecute(StreamExecuteRequest) returns (stream ExecuteProgress);

  // Ottiene lo stato dell'attività
  rpc GetTaskStatus(GetTaskStatusRequest) returns (GetTaskStatusResponse);

  // Annulla l'attività
  rpc CancelTask(CancelTaskRequest) returns (CancelTaskResponse);

  // Operazioni di memoria
  rpc RetrieveMemory(RetrieveMemoryRequest) returns (RetrieveMemoryResponse);
  rpc StoreMemory(StoreMemoryRequest) returns (StoreMemoryResponse);

  // Operazioni degli strumenti
  rpc ListTools(ListToolsRequest) returns (ListToolsResponse);
  rpc ExecuteTool(ExecuteToolRequest) returns (ExecuteToolResponse);
}

message ExecuteRequest {
  Goal goal = 1;
  ResourceBudget budget = 2;
  map<string, string> context = 3;
}

message ExecuteResponse {
  string task_id = 1;
  Episode episode = 2;
}

message ExecuteProgress {
  string request_id = 1;
  string status = 2;
  int32 steps_completed = 3;
  int32 steps_total = 4;
  double percent_complete = 5;
  string current_step = 6;
}
```

---

## Definizioni di Tipo (TypeScript)

### Esportazione Completa dei Tipi

```typescript
// Riesporta tutti i tipi per un facile import
export type {
  // Core
  Goal,
  Plan,
  Episode,
  Memory,
  Tool,

  // Enums
  GoalType,
  GoalStatus,
  MemoryType,
  ToolCategory,

  // Results
  Outcome,
  ExecutionResult,
  ToolResult,

  // Resources
  ResourceBudget,
  ResourceUsage,

  // Tracing
  Trace,
  Span,

  // Interfaces
  ReasoningEngine,
  MemorySystem,
  ToolRegistry,

  // Utilities
  UUID,
  Timestamp,
  Embedding
}

// Type guards
export function isGoal(obj: any): obj is Goal {
  return (
    typeof obj === 'object' &&
    typeof obj.id === 'string' &&
    typeof obj.description === 'string' &&
    ['achievement', 'maintenance', 'avoidance', 'exploration'].includes(obj.type)
  )
}

export function isEpisode(obj: any): obj is Episode {
  return (
    typeof obj === 'object' &&
    typeof obj.id === 'string' &&
    isGoal(obj.goal) &&
    typeof obj.duration === 'number'
  )
}
```

---

## Evoluzione degli Schemi

### Strategia di Versionamento

**Regola 1**: Non rompere mai i client esistenti

**Modifiche Consentite** (non-breaking):
- Aggiungere nuovi campi opzionali
- Aggiungere nuovi valori enum (con gestione predefinita)
- Allentare la validazione (es. aumentare maxLength)
- Aggiungere nuovi endpoint

**Modifiche Proibite** (breaking):
- Rimuovere campi
- Rinominare campi
- Cambiare tipi di campo
- Rendere obbligatorio un campo opzionale
- Rimuovere valori enum
- Rendere più stretta la validazione

### Esempio di Migrazione

**Schema V1**:
```json
{
  "id": "string",
  "description": "string"
}
```

**Schema V2** (retrocompatibile):
```json
{
  "id": "string",
  "description": "string",
  "priority": "number (optional, default: 0.5)",
  "tags": "array (optional, default: [])"
}
```

**Client V1** → **Server V2**: ✅ Funziona (nuovi campi opzionali)
**Client V2** → **Server V1**: ⚠️ Nuovi campi ignorati

---

## Conclusione

Questo documento fornisce specifiche complete di strutture dati e protocolli:

1. **Schemi JSON**: Tutti i principali tipi di dati con regole di validazione
2. **Protocol Buffers**: Formato binario per RPC ad alte prestazioni
3. **Protocolli di Comunicazione**: REST, WebSocket, gRPC
4. **Validazione**: Applicazione degli schemi a runtime
5. **Versionamento**: Strategia di evoluzione per modifiche breaking
6. **Type Safety**: Definizioni TypeScript e type guards

Tutte le specifiche sono:
- ✅ Leggibili dalla macchina (possono generare codice, docs, test)
- ✅ Validate (conformità allo schema verificata)
- ✅ Versionate (compatibilità backward mantenuta)
- ✅ Auto-documentanti (gli schemi includono descrizioni)
- ✅ Testabili (possono validare rispetto agli schemi)

Queste specifiche abilitano:
- Generazione automatica del codice client/server
- Generazione documentazione API (Swagger UI)
- Contract testing
- Sviluppo type-safe
- Interoperabilità con altri sistemi

---

## Riferimenti

1. JSON Schema Specification 2020-12. https://json-schema.org/
2. OpenAPI Specification 3.1.0. https://spec.openapis.org/oas/latest.html
3. Protocol Buffers Language Guide. https://protobuf.dev/
4. gRPC Protocol. https://grpc.io/docs/what-is-grpc/
5. Server-Sent Events. https://html.spec.whatwg.org/multipage/server-sent-events.html
6. Semantic Versioning 2.0.0. https://semver.org/
7. Ajv JSON Schema Validator. https://ajv.js.org/
