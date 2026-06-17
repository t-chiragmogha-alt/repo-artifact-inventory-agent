# Design Decisions — Universal Repository Artifact Inventory Agent

This document explains the key design choices behind the agent and how they support reliable, framework-agnostic repository analysis.

---

## 1. Why Technology Detection Comes First

**Decision:** Detect language, capabilities, build tools, and package managers before searching for artifacts.

**Reasoning:**
- Artifact naming is not portable. A `*Controller` in one framework is a `*Resource` in another; Python uses `views.py`, Go uses `handlers/`.
- Capability detection ("this code registers HTTP routes") is stable across frameworks; label detection ("this is Spring Boot") is not.
- Build manifests reveal module boundaries in monorepos, which determines where to search for entry points and artifacts.

**Alternative considered:** Start with artifact grep patterns per framework (e.g., `@RestController`, `@app.route`). Rejected because it fails on unfamiliar stacks and produces false negatives.

**Outcome:** The agent produces accurate inventories on polyglot monorepos (e.g., Java + Python + React) without prior knowledge of the stack.

---

## 2. Why Evidence Is Required

**Decision:** Every finding must include a file path and an evidence snippet. Findings without evidence are omitted.

**Reasoning:**
- Inventory exercises are audit tasks. Reviewers must verify claims independently.
- LLM-generated inventories can hallucinate file counts or paths. Requiring evidence forces the agent to read the repository.
- Evidence snippets (route path, class declaration, schema title, registration call) make findings actionable for onboarding and refactoring.

**Alternative considered:** Summary-only reports with counts. Rejected because counts without paths cannot be validated.

**Outcome:** Reports are traceable. A reviewer can open any cited file and confirm the classification.

---

## 3. Why Confidence Scoring Exists

**Decision:** Assign High, Medium, or Low confidence using a signal checklist (S1–S8).

**Reasoning:**
- Repositories use inconsistent naming. `UserHandler` might be an HTTP handler, a message consumer, or a utility.
- Not all artifacts have explicit wiring. A service class may be identifiable by shape but lack a traced import from a controller.
- Confidence scoring communicates certainty without hiding ambiguous findings.

**Signal design:**
| Signal | Weight |
|--------|--------|
| S4 — Direct wiring (route registration, DI, scheduler) | Strong |
| S5 — Import chain | Strong |
| S6 — Config reference | Strong |
| S3 — Export shape | Moderate |
| S1/S2 — Directory/naming | Weak |
| S7 — Framework decorator | Supporting only |
| S8 — Contradictory role | Negative |

**Outcome:** Readers know which findings are proven (High) and which need review (Medium/Low).

---

## 4. Why Verified and Inferred Findings Are Separated

**Decision:** Only High-confidence findings that pass verification gates appear in **Verified Findings**. Others go to **Inferred Findings**.

**Reasoning:**
- Mixing proven and guessed artifacts creates false confidence. Reviewers cannot tell which entries are safe to rely on.
- Inferred findings are still valuable — they highlight candidates for manual review.
- The B1 exercise requires rigor: an inventory should distinguish observation from proof.

**Verification gate (summary):**
1. Source file was read.
2. Confidence is High.
3. Behavioral role is confirmed.
4. At least one of S4, S5, S6 contributed (not annotation-only).

**Outcome:** Verified sections are trustworthy for automation and onboarding; Inferred sections flag follow-up work.

---

## 5. How the Agent Adapts to Different Frameworks

**Decision:** Use a universal behavioral taxonomy with framework labels as secondary summaries.

### Adaptation mechanism

| Concern | Framework-specific approach (avoided) | Universal approach (used) |
|---------|--------------------------------------|---------------------------|
| HTTP routes | Find `@GetMapping`, `@app.route` | Find path registration + handler signatures |
| Services | Find `@Service` | Find business logic modules imported by handlers |
| Repositories | Find `@Repository` | Find data-access modules with CRUD/query methods |
| Models | Find `@Entity` | Find schema definitions, structs, or plain data classes |
| DTOs | Find `*Dto` suffix | Find request/response types at API boundaries |
| Jobs | Find `@Scheduled` | Find scheduler/cron registration |
| Events | Find `@KafkaListener` | Find subscribe/publish calls and broker config |
| Frontend | Find `@Component` | Find router config → page → component chain |

### Monorepo adaptation

- Partition by manifest/workspace unit before discovery.
- Inventory each unit independently; rollup in summary.
- Trace cross-unit edges (shared schemas, internal packages, API clients).

### Event-driven adaptation

- Extend artifact types: event schemas, handlers, sagas, projections, DLQ handlers.
- Document producer → topic → consumer flows when traceable.

### Frontend adaptation

- Extend artifact types: pages, routes, layouts, components, hooks, stores, API clients.
- Require router wiring for page-level High confidence.

**Outcome:** The same skill works on Spring Boot, Express, NestJS, FastAPI, Django, Go, .NET, Rust, and polyglot monorepos without maintaining per-framework rule sets.

---

## 6. Additional Design Choices

### Capability before framework label

Framework names appear in the Technology Stack table for readability but are derived from capabilities + manifest + entry-point evidence, not assumed from file extensions.

### Wiring is proof

Registration calls (`app.use`, `jersey().register`, `urlpatterns`, `router.get`) outweigh naming conventions. This is the single strongest signal for Controllers/Routes.

### Default exclusions

Generated and vendored trees are excluded by default to prevent inflated artifact counts from `target/` or `node_modules/`.

### No guessing policy

If a candidate cannot be confirmed after reading the file, it is downgraded, moved to Inferred, or listed in Gaps — never reported as Verified.

---

## Summary

| Design choice | Problem it solves |
|---------------|-------------------|
| Technology detection first | Wrong search patterns on unfamiliar stacks |
| Evidence required | Hallucinated or unverifiable findings |
| Confidence scoring | Ambiguous naming and partial wiring |
| Verified vs Inferred split | False confidence in mixed-quality inventories |
| Behavioral taxonomy | Framework lock-in and annotation assumptions |

These decisions align the agent with the B1 Repo Artifact Inventory requirement: a **verified, evidence-backed catalog** of repository artifacts that a reviewer can audit without running the application.
