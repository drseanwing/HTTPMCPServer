# HTTP/TCP MCP Server over Tailscale for Long-Running Coding Agents

## Goals
- Expose MCP capabilities to Codex/Claude Code over a stable private network (Tailscale).
- Support long-running development work across large repositories.
- Separate high-risk/high-load tooling (browser automation, sandboxed execution) from orchestration.
- Persist state (tasks + memory + artifacts) across sessions and agent restarts.

## Recommended Architecture

### 1) Control Plane + Capability Workers
Use a **hub-and-spoke** design:

- **MCP Gateway (Control Plane)**
  - Single MCP endpoint exposed via HTTP + optional raw TCP.
  - Handles authn/authz, routing, rate limits, observability, tenancy/workspace isolation.
  - Aggregates tool schemas and forwards calls to workers.
- **Capability Workers (Spokes)**
  - Browser worker (Playwright/Puppeteer)
  - Task worker (Task Master-compatible)
  - Sandbox worker (ephemeral dockerized environments)
  - Memory worker (semantic + timeline memory)

This avoids one giant process and lets you scale workers independently.

### 2) Network & Access
- Put all services on a private Tailnet.
- Publish gateway via **Tailscale Serve/Funnel (private preferred)**, or direct node address.
- Use **mTLS or signed service tokens** between gateway and workers.
- Restrict worker ingress to gateway only (`tailscale ACLs`).

### 3) Persistence
Use a durable data layer:
- **PostgreSQL**: tasks, run metadata, audit logs, tool call history.
- **Redis**: queues, locks, short-lived execution state.
- **Object storage (S3/MinIO)**: screenshots, traces, build artifacts, logs.
- **Vector DB (pgvector/Qdrant/Weaviate)**: memory retrieval.

### 4) Execution Model
- Tool call enters gateway -> normalized request envelope -> queued to worker.
- Worker emits progress events and heartbeats.
- Gateway streams status back to MCP client where supported.
- All long jobs get resumable IDs and checkpoints.

### 5) Reliability Patterns
- Idempotency keys for tool calls.
- Job retries with dead-letter queue.
- Timeouts + cancellation tokens.
- Per-workspace quotas (CPU, RAM, browser contexts, containers).
- Circuit breakers around external APIs.

## Technology Recommendations

## Core Stack
- **Language**: TypeScript/Node.js for fastest MCP ecosystem integration.
- **Framework**: Fastify or NestJS for gateway APIs.
- **Transport**:
  - MCP over HTTP (primary)
  - Optional TCP bridge for legacy or low-overhead internal links.
- **Queue**: BullMQ (Redis) or NATS JetStream.
- **Container runtime**: Docker + gVisor/Kata isolation where possible.
- **Observability**: OpenTelemetry + Prometheus + Grafana + Loki.

## Why Node/TypeScript here?
- Strong MCP tooling ecosystem and SDK momentum.
- Native fit for Playwright/Puppeteer + many existing MCP repos.
- Faster iteration for integrating OSS modules you referenced.

## Module-by-Module Recommendation

### A) Browser Automation
**Requirement:** Playwright +/- Puppeteer

Recommended:
- Start with **Playwright-only** in v1 (better cross-browser primitives and tracing).
- Keep Puppeteer as optional adapter layer if specific Chrome CDP use-cases appear.

Build/Module approach:
- Use a dedicated browser worker with a context pool.
- Enable HAR/video/screenshot/tracing toggles by policy.
- Add domain allowlists and outbound network policies per workspace.

Candidate module baseline:
- Existing MCP browser server patterns + Playwright service extracted into worker process.

### B) Task Management
**Requirement:** claude-task-master or similar

Recommended:
- Reuse Task Master task model concepts (epics/tasks/subtasks, status transitions, dependencies).
- Store canonical state in PostgreSQL; expose MCP tools for CRUD + planning + execution checkpoints.
- Add “agent run linkage” (task <-> git branch <-> commit <-> CI status).

Build/Module approach:
- Either wrap task-master-compatible APIs, or reimplement minimal subset focused on:
  - backlog ingestion
  - dependency graph
  - sprint/run slices
  - progress summaries for LLM context compression

### C) Dockerized Sandbox Creation
**Requirement:** node-code-sandbox-mcp or similar

Recommended:
- Keep this as a separately deployed worker with strict security boundaries.
- Each sandbox is an ephemeral container/namespace with TTL + resource caps.
- Provide snapshot/export primitives to persist useful outputs.

Build/Module approach:
- Baseline from node-code-sandbox-mcp patterns.
- Add:
  - image allowlist
  - seccomp profile
  - filesystem quotas
  - egress controls
  - automatic cleanup controller

### D) LLM Memory
**Requirement:** supermemory or similar

Recommended:
- Hybrid memory model:
  - **Episodic memory** (what happened in runs) in Postgres.
  - **Semantic memory** (facts/snippets) in vector index.
  - **Working memory cache** (current run) in Redis.
- Use explicit retention policies and per-repo namespaces.

Build/Module approach:
- Start with supermemory-like ingestion/retrieval flow.
- Add tool outputs as first-class memory artifacts (task updates, screenshots, diffs, failures).
- Include “summarize and compact” jobs to keep contexts small and useful.

## Suggested MCP Tool Surface

Gateway-exposed tools should be stable and high-level:

- `task.create`, `task.update`, `task.list`, `task.dep_graph`, `task.next_action`
- `sandbox.create`, `sandbox.exec`, `sandbox.snapshot`, `sandbox.destroy`
- `browser.open`, `browser.act`, `browser.extract`, `browser.trace`
- `memory.store`, `memory.search`, `memory.timeline`, `memory.compact`
- `run.start`, `run.heartbeat`, `run.resume`, `run.cancel`

Keep worker-specific low-level tools private behind gateway routing.

## Security Model (Important)
- Tailnet ACL restricts client identities that can call gateway.
- Gateway enforces per-tool RBAC (read/write/exec privileges).
- Signed audit log of all tool invocations + artifact hashes.
- Secrets stored in Vault/SOPS; short-lived credentials for workers.
- Sandboxes and browser workers run as non-root, read-only base image when possible.

## Deployment Topology

### Minimum Viable (single host)
- 1 VM in tailnet
- gateway + all workers + postgres + redis
- good for pilot teams

### Production baseline
- gateway replicated (2+)
- workers autoscaled by queue depth
- managed Postgres/Redis
- object storage + vector DB managed service
- centralized observability stack

## Build vs Buy Guidance
- **Build custom gateway**: yes (you need unified policy, routing, and reliability semantics).
- **Adopt existing modules**: yes for each capability worker initial bootstrap.
- **Avoid deep fork early**: wrap upstream components first; fork only where reliability/security gaps appear.

## Phased Implementation Plan

### Phase 1 (2-4 weeks)
- Gateway skeleton + auth + tool registry
- Playwright worker + basic task store + simple memory search
- Tailnet deployment + ACLs

### Phase 2 (4-8 weeks)
- Sandbox worker hardening
- task dependency graph + run orchestration
- memory compaction and replay/timeline
- observability + SLOs

### Phase 3 (8-12 weeks)
- autoscaling + multi-tenant isolation
- policy engine (OPA/Cedar)
- advanced recovery and cost controls

## Opinionated Final Recommendation
For your goal (long-running Codex/Claude Code sessions on large codebases), the best path is:
1. **TypeScript gateway over HTTP MCP**, fronted by Tailscale.
2. **Dedicated workers** for browser/task/sandbox/memory instead of one process.
3. **Postgres + Redis + object storage + vector index** as core state backbone.
4. **Job-oriented, resumable execution model** with strong observability and audit trails.

This architecture keeps iteration speed high now while avoiding the scaling/security trap of a monolithic “all tools in one MCP process.”
