# Universal Repository Artifact Inventory Agent

A **framework-agnostic Cursor agent** that analyzes unfamiliar repositories and produces **verified, evidence-backed artifact inventories**. Built for the **B1 Repo Artifact Inventory** exercise.

![Agent overview](screenshots/repo-analysis.png)

---

## Purpose

Onboarding to a new codebase is slow when architecture is undocumented or spread across conventions. This agent:

1. **Detects** the technology stack and architecture without assuming a specific framework.
2. **Discovers** repository artifacts (routes, services, repositories, models, jobs, tests, and more).
3. **Collects evidence** for every finding (file path, snippet, wiring).
4. **Scores confidence** and separates **Verified** from **Inferred** findings.
5. **Generates** a structured inventory report suitable for review and onboarding.

The agent does **not** guess. It reads the repository using search, glob, and file inspection — the same steps a human reviewer would take.

---

## B1 Repo Artifact Inventory — Problem Statement

The **B1 Repo Artifact Inventory** exercise requires analyzing an unfamiliar repository and producing a catalog of its structural artifacts with proof.

### Requirements

| Requirement | How this agent satisfies it |
|-------------|----------------------------|
| Analyze an unfamiliar repository | Skill workflow starts with reconnaissance; no prior stack knowledge assumed |
| Detect technology stack | Phase 2: language, capabilities, build tool, package manager |
| Discover architecture | Phase 3: module tree, boundaries, architecture classification |
| Inventory artifacts | Phase 4: controllers, services, repositories, models, DTOs, jobs, workers, consumers, producers, configs, utilities, tests |
| Provide evidence per finding | Every row includes file path + evidence snippet |
| Confidence scoring | S1–S8 signal checklist → High / Medium / Low |
| Verified vs inferred separation | Phase 6 verification gates before reporting |
| Professional deliverable | Structured report (see `examples/openmetadata/`) |

### Deliverable sections

The agent generates all required sections:

- Technology Stack
- Repository Structure
- Architecture Overview
- Controllers / Routes · Services · Interfaces · Repositories
- Models · Entities · DTOs
- Jobs · Workers · Consumers · Producers
- Configurations · Utilities · Tests
- Main Entry Points
- Architecture Summary

---

## Supported Technologies

The agent is **framework-agnostic**. It detects capabilities first, then maps to framework labels.

### Backend (verified approach)

| Stack | Detection method |
|-------|------------------|
| Java (Spring Boot, Dropwizard, JAX-RS) | Manifest + HTTP registration + entry point |
| Node.js (Express, NestJS) | `package.json` + route/handler wiring |
| Python (FastAPI, Django, Flask) | `pyproject.toml` / `requirements.txt` + router/urlpatterns |
| Go (Gin, Echo, Chi, stdlib) | `go.mod` + `HandleFunc` / router groups |
| .NET (ASP.NET Core) | `*.csproj` + `MapControllers` / minimal API |
| Rust (Actix, Axum, Rocket) | `Cargo.toml` + route macros / router builder |

### Frontend

| Stack | Detection method |
|-------|------------------|
| React, Vue, Angular, Svelte | Client mount entry + router config |
| Next.js, Nuxt | SSR/SSG entry + `pages/` or `app/` router |

### Other patterns

| Pattern | Support |
|---------|---------|
| Monorepos (Maven, npm workspaces, Nx, Turborepo, Go/Rust workspaces) | Phase 0 partition + per-unit inventory |
| Event-driven (Kafka, RabbitMQ, SQS, in-process buses) | Extended artifact types + event flow docs |
| Polyglot repos | Per-unit stack profile and rollup summary |

---

## Repository Structure

```
repo-artifact-inventory-agent/
│
├── README.md                          # This file
│
├── skills/
│   └── universal-repo-artifact-inventory.md   # Reusable Cursor skill (Phases 0–7)
│
├── prompts/
│   └── cursor-agent-prompt.md         # Copy-paste prompt to invoke the agent
│
├── docs/
│   ├── methodology.md                 # Step-by-step workflow and rationale
│   └── design-decisions.md            # Why evidence, confidence, and verification matter
│
├── examples/
│   └── openmetadata/
│       └── repo-artifact-inventory-report.md  # Full sample inventory (4,400+ lines)
│
└── screenshots/
    ├── README.md                      # Screenshot capture instructions
    ├── cursor-skill.png               # Placeholder — replace before submission
    ├── inventory-output.png           # Placeholder — replace before submission
    └── repo-analysis.png              # Placeholder — replace before submission
```

---

## How to Use in Cursor

### 1. Install the skill

Copy the skill to your Cursor project or personal skills folder:

```bash
# Option A: Project rule (recommended for this repo)
mkdir -p .cursor/rules
cp skills/universal-repo-artifact-inventory.md .cursor/rules/

# Option B: Personal skill
mkdir -p ~/.cursor/skills/universal-repo-artifact-inventory
cp skills/universal-repo-artifact-inventory.md ~/.cursor/skills/universal-repo-artifact-inventory/SKILL.md
```

### 2. Open the target repository

Open the repository you want to analyze in Cursor (or ensure the agent has access to its path).

### 3. Run the agent prompt

Copy the prompt from [`prompts/cursor-agent-prompt.md`](prompts/cursor-agent-prompt.md) and replace `{REPO_PATH}` with your repository path.

Example:

```
Use the Universal Repository Artifact Inventory skill from skills/universal-repo-artifact-inventory.md.

Analyze this repository: /path/to/your-repo

Follow the workflow defined in the skill ...
```

### 4. Review the output

- **Verified Findings** — safe to rely on for onboarding and documentation.
- **Inferred Findings** — candidates needing manual review.
- Spot-check High-confidence items by opening the cited file paths.

![Inventory output](screenshots/inventory-output.png)

---

## Example Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Clone repo-artifact-inventory-agent                         │
│  2. Install skill → .cursor/rules/                              │
│  3. Open target repo in Cursor                                  │
│  4. Paste prompt from prompts/cursor-agent-prompt.md            │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  Agent Phase 0: Partition monorepo (if applicable)              │
│  Agent Phase 1: Reconnaissance (manifests, entry points)        │
│  Agent Phase 2: Technology & capability detection               │
│  Agent Phase 3: Architecture classification                     │
│  Agent Phase 4: Artifact discovery (read files, trace wiring)   │
│  Agent Phase 5: Confidence scoring (S1–S8)                      │
│  Agent Phase 6: Verification gates                              │
│  Agent Phase 7: Generate inventory report                       │
└────────────────────────────┬────────────────────────────────────┘
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│  Deliverable: Technology Stack + Architecture + Artifact tables   │
│  Each finding: File Path | Evidence | Confidence                │
│  Verified / Inferred sections separated                         │
└─────────────────────────────────────────────────────────────────┘
```

### Example: OpenMetadata

A complete inventory of the [OpenMetadata](https://github.com/open-metadata/OpenMetadata) repository is included:

**[`examples/openmetadata/repo-artifact-inventory-report.md`](examples/openmetadata/repo-artifact-inventory-report.md)**

Highlights from that run:

| Metric | Count |
|--------|-------|
| JAX-RS Resources (Controllers) | 118 |
| Repositories | 114 |
| Entity JSON Schemas (Models) | 367 |
| API JSON Schemas (DTOs) | 169 |
| Tests | 2,810 |
| Verified route handlers | 115 |

---

## How This Satisfies B1

| B1 criterion | Agent capability |
|--------------|------------------|
| Works on unfamiliar repos | No hard-coded framework rules; capability-based detection |
| Technology stack | Stack profile table with evidence per dimension |
| Architecture discovery | Module tree, layer boundaries, event flows |
| Artifact inventory | 17+ artifact categories with behavioral definitions |
| Evidence per finding | Mandatory file path + snippet; no finding without proof |
| Confidence | High / Medium / Low via S1–S8 signals |
| Verification | Separate Verified vs Inferred; annotation-only findings capped |
| Reproducible | Skill + prompt + methodology documented |
| Demonstrable | OpenMetadata example with 4,400+ lines of enumerated findings |

---

## Documentation

| Document | Description |
|----------|-------------|
| [`docs/methodology.md`](docs/methodology.md) | Step-by-step workflow with rationale for each phase |
| [`docs/design-decisions.md`](docs/design-decisions.md) | Why technology-first, evidence, confidence, and verification |
| [`skills/universal-repo-artifact-inventory.md`](skills/universal-repo-artifact-inventory.md) | Full agent skill (Phases 0–7, scoring, report template) |
| [`prompts/cursor-agent-prompt.md`](prompts/cursor-agent-prompt.md) | Reusable invocation prompt |

---

## Future Enhancements

| Enhancement | Description |
|-------------|-------------|
| **JSON / SARIF export** | Machine-readable inventory for CI pipelines and dashboards |
| **Diff mode** | Compare inventories across branches or commits |
| **Coverage scoring** | Percentage of source files classified vs unclassified |
| **Mermaid diagrams** | Auto-generate architecture and event-flow diagrams |
| **CI GitHub Action** | Run inventory on PR and comment summary |
| **Custom artifact types** | User-defined types via skill configuration |
| **IDE plugin** | Navigate from inventory row to source file |
| **Multi-repo rollup** | Aggregate inventories across a service catalog |

---

## Screenshots

Replace placeholder images in `screenshots/` before submission. See [`screenshots/README.md`](screenshots/README.md) for capture instructions.

| File | Shows |
|------|-------|
| `cursor-skill.png` | Skill installed in Cursor |
| `repo-analysis.png` | Agent analyzing a repository |
| `inventory-output.png` | Generated inventory report |

![Cursor skill](screenshots/cursor-skill.png)

---

## License

This agent skill and documentation are provided for educational and exercise submission purposes. The OpenMetadata example inventory analyzes the [OpenMetadata](https://github.com/open-metadata/OpenMetadata) open-source project (Apache License 2.0).

---

## Quick Start

```bash
git clone <your-fork-of-this-repo>
cd repo-artifact-inventory-agent
cp skills/universal-repo-artifact-inventory.md .cursor/rules/
# Open Cursor → paste prompt from prompts/cursor-agent-prompt.md
```

For questions about workflow or design, see [`docs/methodology.md`](docs/methodology.md) and [`docs/design-decisions.md`](docs/design-decisions.md).
