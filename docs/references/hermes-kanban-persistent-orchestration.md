# Hermes Kanban — Persistent Orchestration Research

> **Research Date:** 2026-06-22
> **Provenance:** VERIFIED against Hermes Agent SKILL.md v2.1.0 (installed) + GitHub issue tracker + community sources

---

## 1. Architecture Overview

The Hermes Kanban is a **durable SQLite-backed task board** shared across all Hermes profiles. It enables multiple named agents to collaborate on work without fragile in-memory coordination.

**Core principle:** The Kanban board is the **single source of truth** for task state. Workers never own lifecycle truth — they only act on what the board says.

### Why Kanban Wins (hermes-agent.ai, 2026-05-15):
1. **Durable** — survives restarts and crashes (SQLite on disk)
2. **Parallel** — many agents work simultaneously on different tasks
3. **Observable** — full history of task state transitions
4. **Recoverable** — dispatcher reclaims stale claims automatically
5. **Isolated** — each worker gets its own context and tools

---

## 2. Database Schema

### Tables:
| Table | Purpose |
|-------|---------|
| `tasks` | Main task board — id, goal, status, assignee, artifacts |
| `task_runs` | Execution history per task (id: TEXT PRIMARY KEY, AUTOINCREMENT) |
| `kanban_notify_subs` | Notification subscriptions (last_event_id: TEXT) |

### Task Status Enum:
```
ready → claimed → blocked / completed / archived
```

### Key Columns in `tasks` table:
- `id` — unique task identifier
- `goal` — task description
- `status` — ready | claimed | blocked | completed | archived
- `assignee` — profile name assigned to execute
- `artifacts` — file paths / logs produced by worker
- `session_id` — added in later schema versions (migration issue #28844)
- `spawn_failures` — consecutive spawn failure counter (migration issue #20842)

### Schema Migration Notes (GitHub Issues):
- **#28554:** `CREATE INDEX on session_id` fails when table already exists
- **#35096:** `task_runs.id` uses TEXT PRIMARY KEY, not INTEGER
- **#20842:** `spawn_failures` column added in update — dispatcher must handle missing column
- **#28844:** `idx_tasks_session_id` created before column exists in older DBs

---

## 3. Dispatcher Architecture

The dispatcher runs **inside the gateway** by default (`kanban.dispatch_in_gateway: true`).

### Core Responsibilities:
1. **Poll** the `tasks` table for entries with status `ready`
2. **Atomically claim** tasks (ready → claimed) to prevent race conditions
3. **Spawn** the assigned worker profile for each claimed task
4. **Monitor** worker liveness and reclaim stale/failed tasks
5. **Auto-block** tasks after `failure_limit` consecutive spawn failures

### Dispatch Lifecycle:
1. **Polling:** Query `tasks WHERE status = 'ready'`
2. **Atomic Claim:** `UPDATE tasks SET status = 'claimed' WHERE id = ? AND status = 'ready'`
3. **Worker Spawn:** Spawn assigned profile with `HERMES_KANBAN_TASK` + `HERMES_KANBAN_BOARD` env vars
4. **Monitoring:** Track worker liveness via heartbeats
5. **Failure Handling:** Increment `spawn_failures`, auto-block after `failure_limit` (default: 2)

### Zombie Task Reaping:
- Detection: Task in `claimed` status beyond timeout threshold without completion
- Reclamation: Revert to `ready` or mark as `failed`

### Dispatcher Configuration:
| Config Key | Default | Purpose |
|-----------|---------|---------|
| `kanban.dispatch_in_gateway` | `true` | Run dispatcher inside gateway process |
| `kanban.failure_limit` | `2` | Consecutive spawn failures before auto-block |

---

## 4. Worker Lifecycle (6 Steps)

```
orient → work → heartbeat → block/complete
```

### Step 1: Orient
- Worker receives `HERMES_KANBAN_TASK` environment variable
- Worker calls `kanban_show` to read task details

### Step 2: Work
- Worker executes the task using available tools
- Worker produces artifacts (files, logs, reports)

### Step 3: Heartbeat
- Worker calls `kanban_heartbeat` to signal liveness
- Dispatcher detects dead workers via missing heartbeats

### Step 4a: Complete
- Worker calls `kanban_complete`
- Task status: `claimed` → `completed`

### Step 4b: Block
- Worker calls `kanban_block` when it cannot proceed
- Task status: `claimed` → `blocked`
- Orchestrator is notified and must intervene

---

## 5. Environment Isolation

### Injected Environment Variables:
| Variable | Purpose |
|----------|---------|
| `HERMES_KANBAN_TASK` | Task ID — worker can only interact with this task |
| `HERMES_KANBAN_BOARD` | Board identifier — pins worker to specific board |

### Toolset Restriction:
Workers receive focused subset: `kanban_show`, `kanban_complete`, `kanban_block`, `kanban_heartbeat`, `kanban_comment`, `kanban_create`, `kanban_link`

Workers do NOT have: `kanban_list`, `kanban_unblock`

### Context Pollution Prevention (GitHub #26730):
If worker's system prompt is polluted with parent session context, the worker may ignore the Kanban lifecycle. Mitigation: `KANBAN_GUIDANCE` block is **auto-injected** into every worker's system prompt.

---

## 6. Profile Separation

### Orchestrator Profile:
- Full kanban toolset: `kanban_list`, `kanban_create`, `kanban_link`, `kanban_unblock`, `kanban_comment`
- Does NOT execute implementation work — only decomposes, routes, and monitors

### Worker Profile:
- Restricted kanban toolset: `kanban_show`, `kanban_complete`, `kanban_block`, `kanban_heartbeat`, `kanban_comment`, `kanban_create`, `kanban_link`
- Cannot fan out or route unrelated work
- Cannot mutate unrelated tasks

### Normal Sessions:
- Zero `kanban_*` schema footprint unless explicitly configured

---

## 7. Orchestrator-Dispatcher Boundary

| Orchestrator Responsibility | Dispatcher Responsibility |
|-----------------------------|---------------------------|
| Create tasks | Atomic claim |
| Assign profiles | Spawn workers |
| Set dependencies | Detect zombies |
| Monitor board | Auto-block on repeated failures |
| Intervene on blocks | Reclaim stale claims |
| Synthesize artifacts | Monitor worker liveness |

**Key insight:** The orchestrator does NOT directly spawn workers — it creates tasks with assignee profiles and lets the dispatcher handle spawning.

---

## 8. Toolset Gating Matrix

| Tool | Orchestrator | Worker | Normal Session |
|------|:---:|:---:|:---:|
| `kanban_list` | ✅ | ❌ | ❌ |
| `kanban_create` | ✅ | ✅ | ❌ |
| `kanban_show` | ✅ | ✅ | ❌ |
| `kanban_complete` | ❌ | ✅ | ❌ |
| `kanban_block` | ❌ | ✅ | ❌ |
| `kanban_unblock` | ✅ | ❌ | ❌ |
| `kanban_link` | ✅ | ✅ | ❌ |
| `kanban_comment` | ✅ | ✅ | ❌ |
| `kanban_heartbeat` | ❌ | ✅ | ❌ |

---

## 9. Backend Portability

Workers whose terminal tool points at a remote backend (Docker / Modal / Singularity / SSH) must use the `kanban_*` toolset (injected into schema) rather than CLI commands, since `hermes` is not installed in the container and `~/.hermes/kanban.db` is not mounted.

---

## References

- Hermes Agent SKILL.md v2.1.0 (installed locally) — "Kanban (multi-agent work queue)" section
- GitHub: NousResearch/hermes-agent issues #20842, #26730, #28554, #35096
- hermes-agent.ai blog: "Multi-Agent Workflows" (2026-05-15)
- hermes-agent.ai/features/multi-agent (v0.6.0)
- Reddit: r/hermesagent (2026-05)
- TechTimes: "SQLite Beats Cloud Queues for AI Agent Orchestration" (2026-05-30)
- DEV Community: "Building a Durable Message Queue on SQLite" (2026)
