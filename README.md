# clawbot-sql-memory

SQL Server-based persistent memory system for OpenClaw AI agents. Provides semantic memory, task queuing, knowledge indexing, and agent lifecycle management.

> ⚠️ **Work in Progress** — results may vary as the system evolves. API is not yet stable.

## Overview

`clawbot-sql-memory` is the memory backbone for the Oblio AI agent system. It replaces ad-hoc `.md` file storage with a structured, queryable SQL Server database.

## Schema: `memory.*`

| Table | Purpose | Typical Rows |
|---|---|---|
| `Memories` | Long-term curated memories with importance scoring | ~65+ |
| `TaskQueue` | Agent work items with priority, retry, model tracking | ~450+ |
| `ActivityLog` | Full audit trail of all agent events | ~1500+ |
| `KnowledgeIndex` | Domain knowledge (models, facts, reports) | ~389+ |
| `Sessions` | Agent session tracking | Growing |
| `TaskFeedback` | Quality ratings for completed tasks | Growing |
| `AgentState` | Live agent status snapshots | Growing |
| `DecisionLog` | Important decisions with rationale | Growing |

## Components

### `sql_memory.py` — Core Memory Module

```python
from infrastructure.sql_memory import SQLMemory, get_memory

# Connect to cloud instance
mem = get_memory('cloud')

# Store a memory
mem.remember('facts', 'model_routing', 'Use Ollama first, GPT-4o fallback', importance=4)

# Recall a memory
result = mem.recall('facts', 'model_routing')
print(result)

# Queue a task
task_id = mem.queue_task('nlp_agent', 'nlp_train', payload='{"source":"idle"}', priority=2)

# Log an event
mem.log_event('training_started', 'nlp_agent', 'NLP training triggered by idle check', '')

# Check connectivity
if mem.ping():
    print("Connected!")
```

### Memory Roll-up Schedule

```
daily     → weekly (every Sunday)
weekly    → monthly (1st of month)
monthly   → quarterly (Jan/Apr/Jul/Oct)
quarterly → yearly (January 1st)
yearly    → epoch (every 5 years — the big picture)
```

Each roll-up references source entries for full traceability.

### `agent_base.py` — OblioAgent Base Class

All agents must inherit from `OblioAgent`:

```python
from infrastructure.agent_base import OblioAgent

class MyAgent(OblioAgent):
    agent_name = "my_agent"
    task_types = ["my_task_type"]
    budget = "free"  # or "premium"

    def run_task(self, task: dict) -> str:
        # Your task logic here
        return "Task complete"
```

`OblioAgent` provides:
- `self.mem` — SQLMemory instance (cloud)
- `self.log` — Pre-configured logger
- `self.sqlcmd(query)` — Raw SQL execution
- `self.log_activity(event, description)` — ActivityLog writer
- `self.claim_task(id)` / `self.complete_task(id)` — Lifecycle with model logging
- `self.ollama_generate(prompt)` — Local Ollama inference

### `model_router.py` — AI Model Selection

Routes tasks to the optimal model based on task type and budget:

| Tier | Models | Use Case |
|---|---|---|
| Free/Local | `gemma3:4b`, `mistral:7b`, `codellama:7b` (Ollama) | Classification, simple generation, code |
| Free/Cloud | GPT-4o (Copilot) | Review, enrichment, validation |
| Premium | Claude Sonnet/Haiku | Architecture, complex reasoning, final review |

```python
from infrastructure.model_router import select_model

model = select_model('code', budget='free')
# → {"name": "codellama:7b", "provider": "ollama", ...}
```

## .env Configuration

```env
# Primary (used by sql_memory.py)
SQL_SERVER=SQL5112.site4now.net
SQL_PORT=1433
SQL_DATABASE=db_99ba1f_memory4oblio
SQL_USER=db_99ba1f_memory4oblio_admin
SQL_PASSWORD=your_password

# Cloud alias
SQL_CLOUD_SERVER=SQL5112.site4now.net
SQL_CLOUD_DATABASE=db_99ba1f_memory4oblio
SQL_CLOUD_USER=db_99ba1f_memory4oblio_admin
SQL_CLOUD_PASSWORD=your_password

# GitHub
GitHub_token=ghp_your_token_here
```

## Task Decomposer (Ralph Wiggum Loop)

Breaks complex tasks into subtasks using tiered models:

```python
from agents.task_decomposer import decompose

result = decompose("Implement idle training for all agents")
# → {"subtasks": [...], "complexity": "moderate", "model_used": "gemma3:4b"}
```

Pipeline:
1. **Ollama** classifies complexity and decomposes
2. **GPT-4o** (Copilot) enriches and validates
3. **Claude** handles ambiguous/architectural decisions
4. Subtasks queued to `memory.TaskQueue` automatically

## Related Repos

- [clawbot-sql-connector](https://github.com/VeXHarbinger/clawbot-sql-connector) — Low-level SQL transport
- [oblio-heart-and-soul](https://github.com/VeXHarbinger/oblio-heart-and-soul) — Full agent system
- [AI-UI](https://github.com/VeXHarbinger/AI-UI) — Dashboard UI

## License

MIT
