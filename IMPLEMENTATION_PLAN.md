# MCP Platform Implementation Plan

## Context
This plan derives from `ARCHITECTURE_RECOMMENDATION.md`.
Scope targets an HTTP MCP gateway over Tailnet transport with worker services for browser, task, sandbox, memory capabilities.

## Delivery Rules
- Every task has one narrow outcome.
- Every task title avoids the word `and`.
- Every task lists explicit prerequisites.
- Parallel groups include only safe concurrent items.

## Milestones
- M0: Foundation readiness
- M1: Gateway alpha
- M2: Capability beta
- M3: Production hardening

## Dependency Legend
- `[]` no prerequisite
- `[Txx]` prerequisite task IDs

## Workstream A: Program Setup

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T01 | Create product requirement brief | [] | Approved scope brief |
| T02 | Define nonfunctional target list | [T01] | NFR catalog |
| T03 | Define service boundary map | [T01] | Context map |
| T04 | Define ownership matrix | [T03] | RACI table |
| T05 | Create risk register | [T02] | Risk log |

## Workstream B: Repository Bootstrap

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T06 | Initialize monorepo layout | [T03] | Base folders |
| T07 | Configure TypeScript project references | [T06] | Build graph |
| T08 | Configure lint profile | [T06] | Lint rules |
| T09 | Configure formatter profile | [T06] | Format rules |
| T10 | Configure commit hook pipeline | [T08,T09] | Local quality gate |
| T11 | Create CI workflow skeleton | [T07,T10] | CI pipeline draft |

## Workstream C: Platform Contracts

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T12 | Define MCP envelope schema | [T03] | Shared type package |
| T13 | Define error code taxonomy | [T12] | Error spec |
| T14 | Define job state model | [T12] | State machine spec |
| T15 | Define event payload schema | [T14] | Event contract |
| T16 | Publish API style guide | [T12,T13] | Interface guide |

## Workstream D: Infrastructure Baseline

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T17 | Provision Tailnet node set | [T02] | Reachable nodes |
| T18 | Define ACL policy file | [T17] | Tailnet ACL |
| T19 | Provision PostgreSQL instance | [T02] | Database endpoint |
| T20 | Provision Redis instance | [T02] | Cache endpoint |
| T21 | Provision object storage bucket | [T02] | Artifact bucket |
| T22 | Provision vector store cluster | [T02] | Vector endpoint |
| T23 | Provision secret manager paths | [T02] | Secret namespace |

## Workstream E: Gateway Service

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T24 | Scaffold gateway service package | [T07,T12] | Service skeleton |
| T25 | Implement health endpoint | [T24] | Liveness route |
| T26 | Implement auth token verifier | [T24,T23] | Auth middleware |
| T27 | Implement request validator | [T24,T12] | Schema guard |
| T28 | Implement tool registry loader | [T24,T16] | Registry loader |
| T29 | Implement worker routing module | [T24,T28] | Router layer |
| T30 | Implement idempotency key store | [T24,T20] | Idempotency module |
| T31 | Implement rate limit middleware | [T24,T20] | Rate control |
| T32 | Implement audit log writer | [T24,T19] | Audit writer |
| T33 | Implement stream response adapter | [T24,T15] | Stream transport |
| T34 | Implement cancel request endpoint | [T24,T14] | Cancel API |

## Workstream F: Queue Runtime

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T35 | Create queue topic catalog | [T14] | Queue map |
| T36 | Implement enqueue client | [T35,T20] | Producer client |
| T37 | Implement dequeue worker base | [T35,T20] | Consumer base |
| T38 | Implement retry policy module | [T37] | Retry policy |
| T39 | Implement dead letter handler | [T38,T19] | DLQ handler |
| T40 | Implement heartbeat emitter | [T37,T15] | Heartbeat events |

## Workstream G: Browser Worker

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T41 | Scaffold browser worker package | [T07,T12] | Worker skeleton |
| T42 | Integrate Playwright runtime | [T41] | Browser runtime |
| T43 | Implement browser session pool | [T42,T20] | Session manager |
| T44 | Implement page open tool | [T41,T42] | `browser.open` |
| T45 | Implement interaction tool | [T41,T42] | `browser.act` |
| T46 | Implement extraction tool | [T41,T42] | `browser.extract` |
| T47 | Implement trace capture tool | [T41,T42,T21] | `browser.trace` |
| T48 | Enforce domain policy filter | [T44,T45,T46] | URL guard |

## Workstream H: Task Worker

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T49 | Scaffold task worker package | [T07,T12] | Worker skeleton |
| T50 | Create task schema migration | [T49,T19] | DB migration |
| T51 | Implement task create tool | [T49,T50] | `task.create` |
| T52 | Implement task update tool | [T49,T50] | `task.update` |
| T53 | Implement task list tool | [T49,T50] | `task.list` |
| T54 | Implement dependency graph tool | [T49,T50] | `task.dep_graph` |
| T55 | Implement next action tool | [T49,T54] | `task.next_action` |

## Workstream I: Sandbox Worker

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T56 | Scaffold sandbox worker package | [T07,T12] | Worker skeleton |
| T57 | Build container image allowlist | [T56] | Policy config |
| T58 | Implement sandbox create tool | [T56,T57] | `sandbox.create` |
| T59 | Implement sandbox exec tool | [T56,T58] | `sandbox.exec` |
| T60 | Implement sandbox snapshot tool | [T56,T58,T21] | `sandbox.snapshot` |
| T61 | Implement sandbox destroy tool | [T56,T58] | `sandbox.destroy` |
| T62 | Implement ttl cleanup scheduler | [T56,T61] | Reaper job |

## Workstream J: Memory Worker

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T63 | Scaffold memory worker package | [T07,T12] | Worker skeleton |
| T64 | Create memory schema migration | [T63,T19] | DB migration |
| T65 | Implement memory store tool | [T63,T64,T22] | `memory.store` |
| T66 | Implement memory search tool | [T63,T64,T22] | `memory.search` |
| T67 | Implement memory timeline tool | [T63,T64] | `memory.timeline` |
| T68 | Implement memory compact job | [T63,T64] | `memory.compact` |

## Workstream K: Run Orchestration

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T69 | Create run schema migration | [T19,T14] | DB migration |
| T70 | Implement run start tool | [T69,T24] | `run.start` |
| T71 | Implement run heartbeat tool | [T69,T40,T24] | `run.heartbeat` |
| T72 | Implement run resume tool | [T69,T24] | `run.resume` |
| T73 | Implement run cancel tool | [T69,T34,T24] | `run.cancel` |

## Workstream L: Security Controls

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T74 | Create RBAC role catalog | [T04,T16] | Role matrix |
| T75 | Implement RBAC authorizer | [T74,T24] | Authz middleware |
| T76 | Enable mTLS worker channel | [T17,T24] | Mutual TLS |
| T77 | Configure signed audit digest | [T32,T23] | Tamper signal |
| T78 | Enable secret rotation job | [T23] | Rotation automation |

## Workstream M: Observability

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T79 | Define telemetry naming spec | [T02] | Metric naming |
| T80 | Integrate OpenTelemetry sdk | [T24,T41,T49,T56,T63] | Trace export |
| T81 | Export Prometheus metrics | [T24,T79] | Metrics endpoint |
| T82 | Create Grafana dashboard pack | [T81] | Dashboard set |
| T83 | Define alert rule catalog | [T82,T79] | Alert rules |

## Workstream N: Test Matrix

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T84 | Create contract test suite | [T12,T24] | Contract tests |
| T85 | Create gateway integration suite | [T29,T30,T31,T33] | Gateway tests |
| T86 | Create browser worker suite | [T44,T45,T46,T47] | Browser tests |
| T87 | Create task worker suite | [T51,T52,T53,T54,T55] | Task tests |
| T88 | Create sandbox worker suite | [T58,T59,T60,T61,T62] | Sandbox tests |
| T89 | Create memory worker suite | [T65,T66,T67,T68] | Memory tests |
| T90 | Create resilience chaos suite | [T38,T39,T40,T73] | Failure tests |

## Workstream O: Release Engineering

| ID | Task | Prerequisites | Output |
|---|---|---|---|
| T91 | Create container build pipeline | [T11,T24,T41,T49,T56,T63] | Build jobs |
| T92 | Create staging deploy pipeline | [T91,T17,T18] | Staging rollout |
| T93 | Create production deploy pipeline | [T92,T83] | Production rollout |
| T94 | Create rollback runbook | [T93] | Recovery guide |
| T95 | Create incident response runbook | [T83] | Incident guide |

## Safe Parallel Groups

### Group P1
- T01, T17, T19, T20, T21, T22, T23

### Group P2
- T06, T12, T35, T79

### Group P3
- T24, T41, T49, T56, T63, T69

### Group P4
- T42, T50, T57, T64, T74

### Group P5
- T44, T45, T46, T51, T52, T53, T58, T65, T66

### Group P6
- T47, T54, T59, T67, T70, T80

### Group P7
- T48, T55, T60, T68, T71, T81, T84

### Group P8
- T61, T62, T72, T73, T82, T85, T86, T87, T89

### Group P9
- T75, T76, T77, T78, T83, T88, T90, T91

### Group P10
- T92, T93, T94, T95

## Critical Path
T01 -> T03 -> T12 -> T24 -> T28 -> T29 -> T85 -> T91 -> T92 -> T93

## Completion Criteria
- Gateway tools pass contract validation.
- Worker tools pass integration coverage target.
- Tailnet ACL blocks unauthorized caller identity.
- Run resume succeeds after gateway restart.
- Sandbox ttl cleanup leaves zero expired runtime.
- Alert rules fire during injected fault scenario.
