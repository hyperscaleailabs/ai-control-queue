# AI Control Queue

---

Author: Constantine (Kostyantyn) Gurnov
Org: Hyperscale.AI
Version: 0.1 May 2, 2026

---

A minimal stateful queue primitive for reliable AI systems.

This project demonstrates how AI-enabled workflows can separate probabilistic processing from deterministic state 
control using PostgreSQL, explicit state transitions, and append-only event history.

> The queue does not eliminate uncertainty; it decomposes uncertainty into observable work units.

## Current Status

Implemented:

- Dockerized PostgreSQL
- Queue item schema
- Queue event schema
- Initial sample queue item
- Traceable / auditable persistence layer

Next:

- Notebook-based transition logic
- Basic metrics
- Minimal FastAPI wrapper

## Why This Matters

The queue provides a deterministic control surface around work items moving through AI-assisted workflows.

It supports:

- traceability
- auditability
- observability
- recoverability
- idempotent processing patterns

The LLM or AI agent may propose actions, but the system owns state.

## Repo Structure

```text
ai-control-queue/
  README.md
  docker-compose.yml
  requirements.txt
  sql/
    001_schema.sql
  notebooks/
    analysis.ipynb
  docs/
    Whitepaper-Stateful-Queues-for-Reliable-AI-Systems.md
  src/
    app.py
  Dockerfile
  data/
```

## Setup macOS

### 1. Clone or enter repo

```bash
cd ai-control-queue
````


### 2. Create Python virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Start PostgreSQL

```bash
docker compose up -d
```

### 5. Verify container

```bash
docker ps
```

### 6. Connect to database

```bash
docker exec -it ai-control-queue-postgres psql -U ai_queue_user -d ai_queue
```

Inside psql:

```sql
\dt
```

Expected tables:

```text
queue_items
queue_events
```

Exit:

```sql
\q
```

## Sanity Insert

```bash
docker exec -it ai-control-queue-postgres psql -U ai_queue_user -d ai_queue
```

```sql
INSERT INTO queue_items (source, raw_data, state)
VALUES (
  'email',
  '{"from": "recruiter@example.com", "subject": "AI Architect Role"}',
  'INBOUND'
)
RETURNING *;
```

```sql
SELECT id, source, state, created_at
FROM queue_items;
```

## Teardown

Stop containers while preserving data:

```bash
docker compose down
```

Stop containers and delete PostgreSQL volume:

```bash
docker compose down -v
```

Remove Python virtual environment:

```bash
rm -rf .venv
```

## Data Persistence Note

This setup uses a Docker named volume:

```yaml
postgres_data:/var/lib/postgresql/data
```

On macOS, Docker stores this volume inside Docker Desktop's Linux VM. It is persistent across normal restarts but not directly visible in Finder.

Future extension:

* host bind mounts
* PostgreSQL dumps
* Borg backups
* restore drills
* replica setup

## Scope Boundaries

This project currently does not include:

* DAG engine
* workflow orchestration
* authentication
* production deployment
* UI
* backup / restore policy


---

