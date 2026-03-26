---
name: sql-memory
version: 2.0.0-alpha
status: alpha
description: "Semantic memory layer for OpenClaw agents. Use when: (1) persisting agent memories with importance scoring, (2) hierarchical memory rollups (daily→weekly→monthly→yearly), (3) queuing tasks for agents, (4) logging activity and audit trails, (5) managing knowledge bases with semantic search. Provides remember/recall/search/queue_task/log_event/add_todo APIs. Built on sql-connector. Requires SQL Server schema setup — see README. ALPHA: use at your own risk, API may change."
---

# SQL Memory Skill
> Semantic memory layer for OpenClaw agents

## Overview

Persistent SQL Server-backed memory for OpenClaw agents. Wraps the sql-connector skill with agent-friendly operations: remember, recall, search, task queue, activity logging, todos, and hierarchical rollups (daily → weekly → monthly → yearly).

## Dependencies

Install sql-connector first:

```bash
clawhub install sql-connector
clawhub install sql-memory
```

## Quick Start

```python
from sql_memory import SQLMemory, get_memory

mem = get_memory('cloud')   # or 'local'

# Store a memory
mem.remember('facts', 'user_timezone', 'User is in EST/EDT', importance=7, tags='user,prefs')

# Recall it
entry = mem.recall('facts', 'user_timezone')   # → 'User is in EST/EDT'

# Search across all memories
results = mem.search_memories('timezone')

# Queue a task for an agent
task_id = mem.queue_task('my_agent', 'process_data', payload='{"source":"api"}', priority=3)

# Log an event
mem.log_event(event_type='task_started', agent='my_agent', description='Processing began')

# Add a todo
todo_id = mem.add_todo('Fix the login bug', priority=2, tags='bug,auth')
mem.complete_todo(todo_id)

# Connectivity check
mem.ping()   # → True
```

## API Reference

### Memory

- `remember(category, key, content, importance=3, tags='')` — Store or update a memory
- `recall(category, key)` — Retrieve most recent active entry → string or None
- `search_memories(query, limit=20)` — Full-text search across content, tags, key_name
- `recall_recent(n=10)` — Most recent N memories across all categories
- `forget(category, key)` — Soft-delete (marks is_active=0)

### Task Queue

- `queue_task(agent, task_type, payload='{}', priority=5)` — Add a task → task_id
- `get_pending_tasks(agent, task_types, limit=10)` — Fetch pending tasks
- `claim_task(task_id)` — Mark as processing
- `complete_task(task_id, result='')` — Mark as completed
- `fail_task(task_id, error, retry_count, max_retries=3)` — Fail or re-queue

### Activity Logging

- `log_event(event_type, agent='', description='', metadata='', importance=3)` — Write to ActivityLog
- `get_recent_activity(since_hours=24, agent=None)` — Query recent events

### Todos

- `add_todo(title, project='', priority=5, tags='', due_date=None)` — Create todo → id
- `complete_todo(todo_id)` — Mark done
- `update_todo(todo_id, **fields)` — Update: title, project, priority, status, tags, due_date
- `delete_todo(todo_id)` — Hard delete

### Knowledge Index

- `store_knowledge(domain, topic, summary='', file_path='', tags='')` — Upsert knowledge entry
- `search_knowledge(domain, keyword='')` — Search by domain + keyword
- `get_recent_knowledge(n=10)` — Most recently updated entries

## Importance Scale

- **1–2** — Ephemeral, can archive (old workspace files, debug notes)
- **3–4** — Context, nice-to-know (routine task completions)
- **5–6** — Standard operational (significant events)
- **7–8** — Important milestone (architecture decisions)
- **9** — Critical (system design choices)
- **10** — Permanent (core identity, values, golden rules)

## Memory Rollup Schedule

Hierarchical compression keeps long-term memory manageable:

- Daily entries → rolled up weekly (every Sunday)
- Weekly → monthly (1st of month)
- Monthly → yearly (January 1st)

Each rollup preserves source references for traceability.

## .env Setup

Same pattern as sql-connector:

```env
SQL_local_server=10.0.0.110
SQL_local_database=YourDatabase
SQL_local_user=your_user
SQL_local_password=your_password

SQL_cloud_server=yourserver.database.windows.net
SQL_cloud_database=your_cloud_db
SQL_cloud_user=your_cloud_user
SQL_cloud_password=your_cloud_password
```

## Schema Setup

Run the included setup script, or paste the DDL from the README into SSMS/Azure Data Studio.

```bash
python3 setup_schema.py
```

## Architecture

```
Agents
  └── SQLMemory   ← remember/recall/queue/log/todo
        └── SQLConnector  ← retry, parameterized SQL (pymssql)
              └── SQL Server
```

## Related

- [clawbot-sql-connector](https://github.com/VeXHarbinger/clawbot-sql-connector) — transport layer
- [oblio-heart-and-soul](https://github.com/VeXHarbinger/oblio-heart-and-soul) — full reference implementation

## License

MIT
