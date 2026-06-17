---
name: universal-repo-artifact-inventory
description: >-
  Framework-agnostic analysis of unfamiliar repositories — backend, frontend,
  monorepo, and event-driven — producing a verified artifact inventory with
  evidence, confidence scores, and verification gates. Use for repo inventory,
  artifact inventory, codebase catalog, architecture discovery, or Repo Artifact
  Inventory exercises.
alwaysApply: false
---

# Universal Repository Artifact Inventory

Analyze any unfamiliar repository and produce a verified inventory of artifacts. Classify by **observable role and wiring**, not framework labels, annotations, or filename conventions alone.

## When to Activate

- **Repo inventory**, **artifact inventory**, **codebase map**, or **Repo Artifact Inventory** exercise
- Onboarding to an unfamiliar backend, frontend, monorepo, or event-driven codebase
- User asks what routes, services, components, consumers, or tests exist

## Core Principles

1. **Capability over framework** — Detect what the code *does* (serves HTTP, persists data, publishes events) before naming Spring, React, etc.
2. **Wiring is proof** — Registration, import chains, and config references outweigh naming and decorators.
3. **Verify before report** — Apply verification gates; only **High** confidence findings go in Verified.
4. **Partition before scan** — In monorepos, identify workspace units first; inventory each unit independently, then rollup.
5. **Read the repo** — Search, glob, and open files. Never guess counts or paths.
6. **Scope honestly** — Exclude generated and vendored trees unless the user requests otherwise.

Default exclusions: `node_modules/`, `vendor/`, `target/`, `dist/`, `build/`, `.git/`, `coverage/`, `.next/`, `.nuxt/`, `__pycache__/`.

## Workflow

```
Inventory Progress:
- [ ] Phase 0: Partition (monorepo / multi-app detection)
- [ ] Phase 1: Reconnaissance
- [ ] Phase 2: Capability & Stack Detection
- [ ] Phase 3: Architecture Classification
- [ ] Phase 4: Artifact Discovery
- [ ] Phase 5: Confidence Scoring
- [ ] Phase 6: Verification
- [ ] Phase 7: Final Report
```

---

## Phase 0: Partition (Monorepo & Multi-App)

Run this **before** artifact discovery when multiple deployable or independently buildable units may exist.

### 0.1 Monorepo Detection Signals

| Tool / pattern | Evidence files |
|----------------|----------------|
| npm/yarn/pnpm workspaces | `package.json` → `workspaces`; `pnpm-workspace.yaml` |
| Nx | `nx.json`, `project.json` |
| Turborepo | `turbo.json` |
| Lerna | `lerna.json` |
| Bazel | `WORKSPACE`, `BUILD.bazel` |
| Gradle multi-project | `settings.gradle` → `include` |
| Maven multi-module | parent `pom.xml` → `<modules>` |
| Go workspace | `go.work` |
| Rust workspace | `Cargo.toml` → `[workspace]` |
| .NET solution | `*.sln` with multiple `*.csproj` |
| Polyglot layout | Multiple root-level apps (`apps/`, `services/`, `packages/`) |

### 0.2 Partition Procedure

1. **List workspace units** — Each app, service, lib, or package with its own manifest or `project.json`.
2. **Classify unit kind**:

   | Kind | Indicators |
   |------|------------|
   | **App / service** | Own entry point, deploy config, `main`/`start` script, Dockerfile |
   | **Shared library** | Published/consumed internally; no standalone entry point |
   | **Tooling / config package** | ESLint, TS config, codegen only — label separately, skip deep artifact scan |
   | **Frontend app** | `index.html`, `vite.config`, `next.config`, `src/App.tsx`, etc. |
   | **Backend app** | HTTP server entry, API manifest, `cmd/*/main.go`, etc. |

3. **Record unit metadata**: name, path, kind, manifest path, detected language.
4. **Run Phases 1–6 per unit** (apps and services get full inventory; shared libs get type-appropriate subset).
5. **Map cross-unit edges** — Internal package imports, shared event schemas, API client → backend coupling.

### 0.3 Monorepo Report Structure

- **Section 2 (Stack Profile)** — One row per unit + rollup summary row.
- **Sections 4–5 (Findings)** — Group as `{unit-name} / {artifact-type}`.
- **Section 6 (Summary)** — Per-unit subtables + monorepo totals.
- **Section 3 (Structure)** — Workspace tree showing apps vs packages.

---

## Phase 1: Reconnaissance

Per partition unit, scan root and one level of major directories.

| Signal | Universal indicators |
|--------|---------------------|
| **Manifests** | Any build/dependency file at unit root |
| **Entry points** | Files that bootstrap runtime: `main`, `index`, `app`, `server`, `Program`, `manage` |
| **Config** | `config/`, `settings`, `.env*`, `application.*`, `docker-compose*`, k8s manifests |
| **Source roots** | `src/`, `app/`, `lib/`, `internal/`, `cmd/`, `pkg/`, `components/` |
| **Test roots** | `test/`, `tests/`, `__tests__/`, `*_test.*`, `*.test.*`, `*.spec.*` |
| **Infra** | `terraform/`, `helm/`, `.github/workflows/`, `deploy/` |

---

## Phase 2: Capability & Stack Detection

Detect **capabilities** first, then map to framework names as secondary labels. Each dimension needs independent evidence.

### 2.1 Language

Infer from file extensions and manifests in the unit. Multi-language units are common (e.g., TS frontend + Go backend in one repo) — list each.

### 2.2 Runtime Capabilities (Framework-Agnostic)

| Capability | What proves it |
|------------|----------------|
| **HTTP API server** | Route/path registration, request handler signatures, listen/bind on port, OpenAPI/Swagger artifact |
| **Server-rendered web** | Template rendering, HTML response paths, SSR entry |
| **SPA / client UI** | Client mount root, router config, `index.html` + bundled JS entry |
| **CLI** | Argument parsing, `command` subcommands, stdin/stdout orchestration |
| **Persistence / ORM** | DB client imports, migration dirs, query builder usage, schema definitions |
| **Caching** | Redis/memcached client, cache get/set wrappers |
| **Messaging / events** | Broker client SDK, topic/queue config, subscribe/publish calls |
| **Background execution** | Scheduler config, queue workers, separate worker entry points |
| **Auth** | Token/session middleware, identity provider config, login/logout flows |

**Framework label** (Spring Boot, Express, FastAPI, React, etc.) is a *summary* derived from capabilities + dependency names — never the primary classification input.

### 2.3 Build Tool & Package Manager

Infer from lockfiles and tool config at unit root. Record evidence file for each.

### 2.4 Architecture Type

| Type | Structural evidence (need ≥2 signals) |
|------|-------------------------------------|
| **Monolith** | Single deployable, one primary entry point |
| **Modular monolith** | Bounded modules/packages, one deployable |
| **Distributed / microservices** | Multiple deployables, per-service manifests or compose/k8s services |
| **Layered** | Repeated vertical slice: interface → logic → data access |
| **Hexagonal / clean** | `domain`/`application`/`infrastructure`/`adapters` or ports-and-adapters layout |
| **Event-driven** | See § Event-Driven Architecture below |
| **CQRS** | Separate command and query paths/models with evidence of both |
| **Serverless** | Function-per-handler layout, `serverless.*`, Lambda/Cloud Function entry |
| **Frontend-only** | No server entry; build targets static/client bundle |

Architecture type is usually **Inferred** unless deployment config confirms it.

---

## Phase 3: Architecture Classification

Per unit:

1. Draw module tree with one-line responsibility per directory.
2. Identify boundaries: API/UI layer, application logic, persistence, messaging, shared kernel.
3. Note cross-cutting: middleware, interceptors, error handlers, DI graphs, global state.
4. For event-driven units: map event flow (producer → topic → consumer → side effect).
5. For frontend units: map route tree, layout hierarchy, data-fetch layer.

---

## Event-Driven Architecture

When messaging capability is detected, extend discovery beyond generic Consumer/Producer.

### Detection Signals (Any Broker)

- Broker config: connection strings, topic/queue names in env, YAML, or code
- SDK usage: publish/subscribe/send/receive/ack/nack calls
- Infra: Kafka, RabbitMQ, SQS, SNS, NATS, Pulsar, Redis streams, Azure Service Bus, Google Pub/Sub
- Patterns: outbox table, dead-letter queue, retry/backoff, idempotency keys, saga/orchestration dirs

### Event-Driven Artifact Types

| Type | Role | Discovery signals |
|------|------|-------------------|
| **Event / Message schema** | Contract for payload shape | Avro/Protobuf/JSON schema files; shared `events/` package; typed event classes |
| **Producer** | Emits events to broker or bus | `publish`, `emit`, `send`, `produce`; outbox relay |
| **Consumer** | Subscribes and processes | `subscribe`, `consume`, `onMessage`, `handle`, receiver loop |
| **Event handler** | Pure handler function for one event type | Registered in consumer/router; switches on event type/name |
| **Saga / orchestrator** | Multi-step distributed workflow | `saga`, `orchestrat`, compensation handlers, process manager |
| **Projection / read model** | CQRS view built from events | `projection`, `read_model`, event-applied state updates |
| **Dead-letter handler** | Failed message path | DLQ config, `deadLetter`, poison handling |
| **Topic / queue binding** | Named channel wiring | Topic constants, binding declarations, subscription config |

### Event Flow Documentation

In the report, add an **Event Flow** subsection when applicable:

```
{Producer} → [{topic/queue}] → {Consumer} → {Handler} → {side effect: DB, API, etc.}
```

Only include flows traceable in source or config.

---

## Frontend Repository Support

When the unit is a client UI (SPA, SSR UI, mobile-web, or hybrid), use these artifact types in addition to or instead of backend types.

### Frontend Artifact Types

| Type | Role | Discovery signals |
|------|------|-------------------|
| **Page / View / Screen** | Route-level UI | Files under `pages/`, `views/`, `screens/`, `routes/`; default export tied to router path |
| **Route definition** | Client navigation mapping | Router config (`createBrowserRouter`, `Route`, `path:`) |
| **Layout** | Shared shell around pages | `layout`, `Layout.tsx`, nested router outlets |
| **Component** | Reusable UI unit | `components/`; exported function/class returning JSX/template |
| **Container / smart component** | UI wired to data/effects | Fetches data, uses store, passes props to children |
| **Hook / composable** | Reusable stateful logic | `use*` (React), `use*` composables (Vue), custom hooks dir |
| **Store / state slice** | Client global state | Redux slice, Zustand store, Pinia store, NgRx, context providers with state |
| **API client / data layer** | Backend communication | `api/`, fetch/axios wrappers, React Query hooks, tRPC client, GraphQL client |
| **Form / validation schema** | Input validation | Zod/Yup/schema objects tied to forms |
| **Style module** | Scoped styling | CSS modules, styled-components, Tailwind component wrappers |
| **Test** | Component/unit/e2e tests | Testing Library, Cypress, Playwright, Vitest/Jest specs |

### Frontend Discovery Notes

- **Component ≠ Page** — A file in `components/` is not a Page unless router maps a URL to it.
- **Hooks ≠ Services** — Classify as Hook unless they orchestrate domain logic shared with backend.
- **API client ≠ Controller** — Client-side fetch wrappers are API clients; server-side route handlers are Controllers/Routes.
- Trace: `route config → page → components → api client` for High-confidence UI wiring.

---

## Phase 4: Artifact Discovery (Universal Taxonomy)

Discover by **role + wiring**. Open every candidate file; confirm or reject.

### Backend & Shared Artifact Types

| Type | Behavioral definition | Structural hints (weak alone) |
|------|----------------------|------------------------------|
| **Controller / Route** | Maps HTTP (or RPC) paths to handlers | `routes/`, `handlers/`, `api/`, path + method registration |
| **Service** | Business logic orchestration, no direct transport | Called by controllers/handlers; domain operations |
| **Interface / Port** | Contract without implementation | `interface`, `protocol`, `trait`, abstract signatures only |
| **Repository** | Data access abstraction | CRUD/query methods wrapping storage client |
| **Model** | Domain data shape | Plain structs/classes; no persistence metadata |
| **Entity** | Persistence-mapped domain object | Table/collection mapping, ORM metadata, `entities/` |
| **DTO** | Boundary transfer object | Request/response/input/output types; validation only |
| **Job** | Time-triggered or scheduled task | Cron/scheduler registration |
| **Worker** | Long-running or queue-driven processor | Separate process entry; poll/execute loop |
| **Consumer** | Inbound message subscription | Broker receive + handler dispatch |
| **Producer** | Outbound message emission | Broker send/publish |
| **Configuration** | Runtime wiring and env | Config modules, bootstrap, feature flags |
| **Utility** | Stateless helpers | No domain orchestration; generic helpers |
| **Test** | Automated verification | Test frameworks, `*_test`, `*.spec`, `*.test` |

### Discovery Procedure

1. Glob by directory role and export patterns — not by framework decorators.
2. Search for **wiring verbs**: `register`, `mount`, `use`, `listen`, `subscribe`, `map`, `route`, `handle`.
3. Read file — confirm behavioral role.
4. Trace at least one hop: who imports this? what registers it?
5. Deduplicate re-exports; record canonical path.
6. Assign provisional confidence (Phase 5) and run verification (Phase 6).

### Role Disambiguation

| Ambiguous name | Resolve by |
|----------------|------------|
| `*Handler` | HTTP route handler → Controller; message callback → Event handler; generic try/catch → Utility |
| `*Manager` | Orchestrates lifecycle → Service; thread pool only → Worker/Config |
| `*Helper` / `*Utils` | Default Utility unless wired as Service |
| `*Model` in UI repo | Domain model vs view model — check for JSX/render → Component |

---

## Phase 5: Confidence Scoring Rules

Assign confidence using a **signal checklist**. Count qualifying signals.

### 5.1 Evidence Signals

| Signal | Code | Description |
|--------|------|-------------|
| **S1 — Directory role** | Structural | File in a directory whose name matches artifact role (`services/`, `consumers/`, `components/`) |
| **S2 — Naming convention** | Weak structural | Filename or symbol contains role term (`UserService`, `OrderRepository`) |
| **S3 — Export shape** | Behavioral | Exported type matches role (handler fn, repository interface, React component) |
| **S4 — Direct wiring** | Strong behavioral | Registered in router, DI container, module, broker binding, or scheduler |
| **S5 — Import chain** | Strong behavioral | Traced ≥1 hop: caller role matches expected layer (controller imports service) |
| **S6 — Config reference** | Strong behavioral | Named in config: route table, topic binding, job schedule, env key |
| **S7 — Framework decorator** | Supporting only | Annotation/decorator present — counts only combined with S3–S6 |
| **S8 — Contradictory role** | Negative | Evidence suggests a different role — cancels S2 |

### 5.2 Confidence Levels

| Level | Rule |
|-------|------|
| **High** | S4 or S6 present, OR (S3 + S5 + at least one of S1/S2), AND no S8 |
| **Medium** | S3 + one of S1/S2, without S4/S5/S6, AND no S8 |
| **Low** | S1 or S2 only, OR ambiguous S3, OR partial/conflicting evidence |

### 5.3 Mandatory Downgrades

| Condition | Action |
|-----------|--------|
| S8 contradictory role detected | Cap at **Low**; explain conflict |
| S7 is the only signal | Cap at **Medium**; never **High** |
| Generated/codegen file | Cap at **Medium** unless S4/S6 traces to hand-written registration |
| Re-export / barrel file | Confidence applies to canonical implementation file |
| Test fixture or mock | Classify as **Test**, not domain artifact |
| Dead code (no importers, unregistered) | **Low** + flag in Gaps |

### 5.4 Stack Profile Confidence

Apply same signal logic to Language, Framework, Build Tool, Package Manager, Architecture:

- **High** — Manifest + entry-point import/usage agree
- **Medium** — Manifest only, or entry point only
- **Low** — Inferred from file extensions or README

---

## Phase 6: Verification Rules

Verification is the gate between discovery and the **Verified Findings** section. A finding is **Verified** only if it passes all applicable gates.

### 6.1 Universal Verification Gate

A finding is **Verified** when ALL of the following hold:

1. **File read** — Source was opened; evidence snippet is accurate.
2. **Confidence High** — Meets Phase 5 rules.
3. **Role confirmed** — Behavioral definition in Phase 4 is satisfied.
4. **Not annotation-only** — S4, S5, or S6 contributed; S7 alone is insufficient.
5. **Path is in scope** — Not excluded generated/vendor path.

### 6.2 Per-Type Verification Requirements

| Type | Minimum proof (in addition to universal gate) |
|------|-----------------------------------------------|
| **Controller / Route** | S4: path + verb registered, OR framework-neutral route table entry |
| **Service** | S5: imported and invoked by controller/handler/job, OR S4: DI registration |
| **Interface / Port** | S3: contract-only surface; ≥1 implementation exists elsewhere |
| **Repository** | S3: data-access methods + storage client usage |
| **Model / Entity / DTO** | S3: field schema visible; Entity also shows persistence mapping |
| **Job** | S4 or S6: schedule/cron/beat registration |
| **Worker** | S4: worker entry or queue processor registration |
| **Consumer** | S4 or S6: subscription/binding to topic/queue |
| **Producer** | S4 or S6: publish call site wired to topic/queue, or outbox relay |
| **Event handler** | S4: dispatched from consumer/router by event type/name |
| **Configuration** | S6: loaded at bootstrap or referenced by multiple modules |
| **Utility** | S3 only acceptable; S5 must show no controller/service wiring |
| **Page / Route (UI)** | S4: path in client router config |
| **Component** | S3: renders UI; S5 if imported by page/layout |
| **Hook / Store / API client** | S5: imported by component/page/container |
| **Test** | S3: test runner API (`describe`, `it`, `test`, `@Test`) |

### 6.3 Verification Procedure (Per Candidate)

```
1. Collect signals S1–S8
2. Compute confidence (Phase 5)
3. Check per-type minimum proof (§6.2)
4. PASS → Verified Findings
5. FAIL → Inferred Findings (include failure reason in Evidence)
6. REJECT → Omit from inventory; note in Gaps if noteworthy
```

### 6.4 Rejection Criteria (Do Not List)

- Re-export with no logic — point to canonical file instead
- Empty stub or placeholder with no wiring
- Comment-only or string-match hit without symbol
- Duplicate of already-listed artifact (keep one canonical entry)

### 6.5 Cross-Unit Verification (Monorepos)

- Shared lib artifact: verify exports used by ≥1 app (S5 across packages) for High confidence.
- Shared event schema: verify referenced by producer and/or consumer in any unit.
- Frontend API client: verify endpoint paths match a discovered backend route when both units are in scope.

---

## Phase 7: Final Report

### Required Deliverable Sections (B1)

The final report must include all of the following:

1. Technology Stack
2. Repository Structure
3. Architecture Overview
4. Controllers / Routes
5. Services
6. Interfaces
7. Repositories
8. Models
9. Entities
10. DTOs
11. Jobs
12. Workers
13. Consumers
14. Producers
15. Configurations
16. Utilities
17. Tests
18. Main Entry Points
19. Architecture Summary

Each artifact finding: **File Path**, **Evidence**, **Confidence**. Separate **Verified Findings** from **Inferred Findings**.

```markdown
# Repository Artifact Inventory

**Repository:** {repo-name}
**Analyzed:** {date}
**Scope:** {root, units scanned, exclusions}
**Repo shape:** {monolith | monorepo — N units | polyglot}

---

## 1. Executive Summary

{Language(s), primary capabilities, architecture, scale, notable patterns.}

---

## 2. Stack Profile

### 2.1 Rollup

| Dimension | Value | Evidence | Confidence |
|-----------|-------|----------|------------|

### 2.2 Per-Unit (monorepos)

| Unit | Kind | Language | Capabilities | Framework (label) | Build | Package Mgr | Confidence |
|------|------|----------|--------------|-------------------|-------|-------------|------------|

---

## 3. Repository Structure

{Workspace tree: apps / packages / services with one-line purpose.}

### 3.1 Event Flows (if applicable)

| Flow | Evidence |
|------|----------|

### 3.2 Frontend Route Tree (if applicable)

| Path | Page/Component | Layout | Evidence |
|------|----------------|--------|----------|

---

## 4. Verified Findings

{Group: `{unit} / {artifact-type}` — table per type.}

| Name | File Path | Evidence | Signals |
|------|-----------|----------|---------|
| | | | e.g. S3,S4,S5 |

---

## 5. Inferred Findings

| Name | File Path | Evidence | Confidence | Verification gap |
|------|-----------|----------|------------|-------------------|
| | | | Medium/Low | e.g. missing S4 route registration |

---

## 6. Inventory Summary

### 6.1 Per-Unit

| Unit | Controllers | Services | … | Tests | Verified | Inferred |
|------|-------------|----------|---|-------|----------|----------|

### 6.2 Monorepo Totals

| Artifact Type | Verified | Inferred | Total |
|---------------|----------|----------|-------|
| … | | | |
| **Total** | | | |

---

## 7. Gaps & Ambiguities

- Unresolved candidates, dead code, conflicting roles, unscanned units

---

## 8. Methodology Notes

- Partitions, tools, exclusions, limitations
```

---

## Quality Checklist

- [ ] Phase 0 partition complete for monorepos; each app/service inventoried
- [ ] Capabilities detected before framework labels
- [ ] Event flows documented when messaging capability exists
- [ ] Frontend artifact types used for UI units
- [ ] Every finding has signals documented (S1–S8)
- [ ] Verified section passes §6.1 and §6.2 gates
- [ ] No **High** finding relies on S7 alone
- [ ] Summary counts match detail tables
- [ ] Cross-unit references noted for shared packages

---

## Anti-Patterns

| Do not | Do instead |
|--------|------------|
| Inventory entire monorepo as one flat tree | Partition into units first (Phase 0) |
| Label framework from `package.json` alone | Confirm capability in entry-point source |
| Classify by decorator/annotation only | Require S4–S6 wiring |
| Treat every `*Controller.*` as a route handler | Verify path registration (§6.2) |
| Treat every `*Handler` as HTTP | Disambiguate message vs HTTP (Role Disambiguation) |
| List barrel re-exports as artifacts | Trace to canonical implementation |
| Merge frontend Components and Pages | Require router wiring for Pages |
| Count topic names without consumers/producers | Trace full flow or mark Inferred |
| Skip empty artifact types | Include in summary with count 0 |

---

## Example Evidence

**Verified Route (High — S3,S4,S5):**
> `apps/api/src/routes/users.ts` exports `GET /users`; mounted in `apps/api/src/server.ts` via `app.use('/users', usersRouter)`.

**Verified Consumer (High — S4,S6):**
> `services/billing/src/consumers/invoice-paid.ts` subscribes to topic `invoice.paid` in `kafka.config.yml`; handler updates subscription state.

**Inferred Service (Medium — S2,S3; gap: no S5):**
> `packages/core/src/AccountService.ts` exports business methods; naming and shape match Service; no import from a discovered controller in scanned units.

**Verified Page (High — S3,S4):**
> `apps/web/src/pages/Settings.tsx` mapped to `/settings` in `apps/web/src/router.tsx` `Route` definition.

**Rejected:**
> `libs/shared/src/utils/controller-helpers.ts` — formatting helpers only; no route registration (Utility, not Controller).
