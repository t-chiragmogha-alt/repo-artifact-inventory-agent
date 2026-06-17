# Cursor Agent Prompt — Universal Repository Artifact Inventory

Copy and paste the prompt below into Cursor Agent chat. Replace `{REPO_PATH}` with the absolute or workspace-relative path to the repository you want to analyze.

---

## Prompt

```
Use the Universal Repository Artifact Inventory skill from skills/universal-repo-artifact-inventory.md.

Analyze this repository: {REPO_PATH}

Follow the workflow defined in the skill (Phases 0–7).

Generate a complete Repository Artifact Inventory report with the following sections:

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

Requirements:
- Do not guess. Every finding must be discovered by reading or searching the repository.
- For every finding, provide: File Path, Evidence, Confidence (High / Medium / Low).
- Separate Verified Findings from Inferred Findings.
- Apply confidence scoring rules (signals S1–S8) and verification gates from the skill.
- Exclude generated and vendored directories (target/, node_modules/, dist/, build/, vendor/) unless I specify otherwise.
- If the repository is a monorepo, partition into units first, then inventory each unit and provide rollup totals.
- If messaging or frontend capabilities are detected, include event flows and frontend route trees as applicable.
```

---

## Example (OpenMetadata)

```
Use the Universal Repository Artifact Inventory skill from skills/universal-repo-artifact-inventory.md.

Analyze this repository: /path/to/OpenMetadata

Follow the workflow defined in the skill.

Generate:
Technology Stack, Repository Structure, Architecture Overview,
Controllers / Routes, Services, Interfaces, Repositories, Models,
Entities, DTOs, Jobs, Workers, Consumers, Producers, Configurations,
Utilities, Tests, Main Entry Points, Architecture Summary.

For every finding provide: File Path, Evidence, Confidence.
Do not guess. Separate verified findings from inferred findings.
```

---

## Optional Flags

Add any of these lines to the prompt when needed:

| Flag | When to use |
|------|-------------|
| `Include generated code in scope.` | When `target/` or `dist/` output should be scanned |
| `Focus on the {module-name} module only.` | For partial monorepo analysis |
| `Save the report to examples/{repo-name}/repo-artifact-inventory-report.md` | To write output to a file |
| `Include frontend artifact types (pages, components, hooks, stores).` | When UI inventory is required |

---

## Setup in Cursor

1. Clone or open the `repo-artifact-inventory-agent` project in Cursor.
2. Copy `skills/universal-repo-artifact-inventory.md` to one of:
   - **Project rule:** `.cursor/rules/universal-repo-artifact-inventory.md`
   - **Personal skill:** `~/.cursor/skills/universal-repo-artifact-inventory/SKILL.md`
3. Paste the prompt above into Agent chat.
4. Review the generated report; spot-check High-confidence findings against source files.
