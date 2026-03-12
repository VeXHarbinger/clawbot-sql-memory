# ClawBot SQL Memory

SQL Server-based persistent memory system for OpenClaw AI agents.

## Components

### sql-connector
Generic SQL Server connectivity with retry, connection validation, and structured result parsing.

### sql-memory  
Semantic memory layer built on sql-connector. Provides:
- **Memories** — curated long-term storage with importance scoring
- **Task Queue** — agent work items with priority and retry logic
- **Activity Log** — event/audit trail
- **Knowledge Index** — domain-specific knowledge store
- **Sessions** — agent session tracking

### Infrastructure
- `agent_base.py` — OblioAgent base class for all agents
- `model_router.py` — AI model selection engine (Ollama + API)
- `task_executor.py` — Generic task execution wrapper
- `agent_reporter.py` — Agent reporting utilities

## Schema
All tables in `memory.*` schema on SQL Server.

## Installation
```bash
pip install PyPDF2  # for PDF processing
```

## Usage
```python
from infrastructure.sql_memory import get_memory
mem = get_memory('cloud')
mem.remember('facts', 'key', 'value', importance=5)
```

## License
MIT
