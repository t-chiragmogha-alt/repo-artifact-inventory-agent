# Methodology — Universal Repository Artifact Inventory

This document describes the step-by-step workflow used by the Universal Repository Artifact Inventory agent to analyze unfamiliar codebases and produce verified artifact inventories.

## Overview

```
Repository
    → Technology Detection
    → Architecture Discovery
    → Framework Detection
    → Artifact Discovery
    → Evidence Collection
    → Confidence Scoring
    → Verification
    → Inventory Generation
```

Each step builds on the previous one. Findings are not reported until evidence is collected and confidence is scored.

---

## Step 1: Repository Reconnaissance

**Input:** Repository root path.

**Actions:**
- Scan root directory and one level of major subdirectories.
- Identify manifests (`pom.xml`, `package.json`, `go.mod`, `Cargo.toml`, `pyproject.toml`, etc.).
- Locate entry points (`main`, `index`, `Application`, `manage.py`).
- Identify config directories and deployment artifacts.
- Detect monorepo signals (workspaces, multi-module POM, Nx, Turborepo).

**Output:** Scope definition, exclusion list, partition plan.

**Rationale:** Without reconnaissance, the agent risks scanning generated code, misidentifying the primary language, or treating a monorepo as a flat project. Partitioning early prevents duplicated or missed artifacts.

---

## Step 2: Technology Detection

**Input:** Reconnaissance results per partition unit.

**Actions:**
- Infer **language** from file extensions and manifests.
- Detect **runtime capabilities** (HTTP server, SPA, persistence, messaging, CLI, auth).
- Identify **build tool** and **package manager** from lockfiles and config.
- Record evidence file for each dimension.

**Output:** Stack profile table with evidence and confidence per row.

**Rationale:** Technology detection establishes the vocabulary for discovery. Capabilities (e.g., "serves HTTP") are detected before framework labels (e.g., "Spring Boot") so the agent does not assume a specific stack.

---

## Step 3: Architecture Discovery

**Input:** Stack profile and directory layout.

**Actions:**
- Map module tree with one-line responsibility per directory.
- Classify architecture type: monolith, modular monolith, microservices, layered, hexagonal, event-driven, CQRS, serverless, frontend-only.
- Identify boundaries: API/UI, application logic, persistence, messaging, shared kernel.
- Document cross-cutting concerns: middleware, DI, global error handlers.

**Output:** Architecture overview, module tree, optional event-flow or route-tree sketches.

**Rationale:** Architecture discovery provides context for artifact roles. A file named `Handler` may be an HTTP handler, message handler, or utility depending on which layer it sits in.

---

## Step 4: Framework Detection

**Input:** Capabilities + dependency manifests + entry-point source.

**Actions:**
- Map capabilities to framework names as **secondary labels**.
- Confirm framework usage in entry-point imports and bootstrap code.
- Note multiple frameworks in polyglot or monorepo layouts.

**Output:** Framework row in stack profile (e.g., Dropwizard + Jersey, React + Vite, Django, FastAPI).

**Rationale:** Framework names help readers but are not used as the primary classification mechanism. Detection is confirmed in source, not inferred from README alone.

---

## Step 5: Artifact Discovery

**Input:** Architecture map and universal artifact taxonomy.

**Actions:**
- Glob and search for candidates by **behavioral role**, not annotations.
- Open each candidate file; confirm or reject classification.
- Trace wiring: registration, import chains, config references.
- Apply role disambiguation (`*Handler`, `*Manager`, `*Model`).
- Extend taxonomy for event-driven (producers, consumers, schemas) and frontend (pages, components, hooks) when applicable.

**Artifact types discovered:**

| Category | Types |
|----------|-------|
| API layer | Controllers / Routes |
| Logic layer | Services, Interfaces |
| Data layer | Repositories, Models, Entities, DTOs |
| Async layer | Jobs, Workers, Consumers, Producers |
| Infrastructure | Configurations, Utilities |
| Quality | Tests |
| Bootstrap | Main Entry Points |

**Output:** Raw candidate list with provisional types.

**Rationale:** Universal behavioral definitions allow the same workflow to work on Java, Python, Go, TypeScript, Rust, and .NET without hard-coded annotation rules.

---

## Step 6: Evidence Collection

**Input:** Artifact candidates from Step 5.

**Actions:**
- For each candidate, record:
  - **Name** — class, module, or file stem
  - **Type** — artifact category
  - **File Path** — repo-relative path
  - **Evidence** — quote or description: export, route path, registration line, schema title, import chain
  - **Signals** — S1–S8 checklist from the skill

**Output:** Finding records with evidence snippets.

**Rationale:** Evidence makes the inventory auditable. A reviewer can open the cited file and confirm the finding without trusting the agent's summary.

---

## Step 7: Confidence Scoring

**Input:** Evidence records with signals S1–S8.

**Actions:**
- Apply scoring rules:
  - **High** — S4 or S6, or (S3 + S5 + S1/S2), no S8
  - **Medium** — S3 + S1 or S2, without S4–S6
  - **Low** — S1 or S2 only, or contradictory evidence
- Apply mandatory downgrades (annotation-only, generated code, dead code).

**Output:** Confidence level per finding.

**Rationale:** Not all naming conventions are reliable. Confidence scoring quantifies certainty and drives the Verified vs Inferred split.

---

## Step 8: Verification

**Input:** Scored findings.

**Actions:**
- Apply universal verification gate (file read, High confidence, role confirmed, not annotation-only).
- Apply per-type minimum proof (e.g., Controller requires route registration).
- PASS → **Verified Findings**
- FAIL → **Inferred Findings** (document verification gap)
- REJECT → omit or note in Gaps section

**Output:** Verified and Inferred finding sets.

**Rationale:** Verification prevents false positives from appearing as proven facts. Inferred findings remain visible for human review.

---

## Step 9: Inventory Generation

**Input:** Verified and Inferred findings, stack profile, architecture map.

**Actions:**
- Assemble final report sections:
  1. Technology Stack
  2. Repository Structure
  3. Architecture Overview
  4. Artifact tables (Verified, then Inferred)
  5. Main Entry Points
  6. Architecture Summary (counts, patterns, gaps)
- For monorepos: per-unit subtables + rollup totals.
- Include methodology notes and exclusions.

**Output:** Complete Repository Artifact Inventory report.

**Rationale:** A consistent report structure allows reviewers to compare inventories across repositories and satisfies the B1 exercise deliverable format.

---

## Exclusions (Default)

| Path | Reason |
|------|--------|
| `node_modules/` | Third-party dependencies |
| `vendor/` | Vendored code |
| `target/`, `dist/`, `build/` | Compiled/generated output |
| `.git/` | Version control metadata |
| `coverage/`, `__pycache__/` | Tooling artifacts |

---

## Quality Gates

Before delivering the report, the agent confirms:

- [ ] Monorepo partitioned (if applicable)
- [ ] Stack profile has evidence for every row
- [ ] No finding without file path and evidence
- [ ] Verified findings pass verification gates
- [ ] Summary counts match detail tables
- [ ] No High-confidence finding based on annotation alone
