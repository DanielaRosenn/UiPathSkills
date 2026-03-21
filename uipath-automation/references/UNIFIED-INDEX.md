# Unified UiPath Studio knowledge map (`uipath-automation`)

**There is one skill:** `uipath-automation`. The former **uipath-master** pack (everything UiPath–Studio related) is **merged into this skill** as the files below—not a separate Cursor skill.

Use this page as the **single table of contents** when you need any Studio topic.

## How it combines

| Part | Role |
|------|------|
| **[SKILL.md](../SKILL.md)** | Entry point: orchestration steps, canvas/planning rules, regression smoke hints, minimal modern examples; org-specific HITL/SALES02 in [org-overlays/sales02-hitl-patterns.md](org-overlays/sales02-hitl-patterns.md). |
| **This index** | Full map of all reference material. |
| **[orchestration/](orchestration/README.md)** | **Merged master pack**: topic modules (activity areas, REFramework, deploy, debugging, expressions, …). |
| **Other `references/*.md`** | Focused primers + official doc links (invoke-code primer, connectors, project structure, legacy activity catalog). |

**Read order for a new task:** [SKILL.md](../SKILL.md) → classify topic → open the module(s) below → follow links to official docs in [official-documentation-index.md](official-documentation-index.md).

---

## A. Orchestration modules (full coverage — merged master pack)

### Core workflow & quality

| File | Topics |
|------|--------|
| [orchestration/xaml-generator.md](orchestration/xaml-generator.md) | Workflow types, project compatibility, structure, XAML design |
| [orchestration/best-practices.md](orchestration/best-practices.md) | Naming, ST-* analyzer, security, project layout |
| [orchestration/error-handling.md](orchestration/error-handling.md) | Try/Catch, retries, GEH, exception types |
| [orchestration/expressions.md](orchestration/expressions.md) | VB/C# expressions, LINQ, Regex, strings/dates |
| [orchestration/invoke-code.md](orchestration/invoke-code.md) | Invoke Code patterns (snippet depth) |
| [orchestration/debugging.md](orchestration/debugging.md) | Studio debugging |
| [orchestration/deploy-publish.md](orchestration/deploy-publish.md) | Publish, Orchestrator, Git in Studio |
| [orchestration/code-reviewer.md](orchestration/code-reviewer.md) | Review checklist (also see `uipath-code-reviewer` skill) |

### Architecture patterns

| File | Topics |
|------|--------|
| [orchestration/reframework.md](orchestration/reframework.md) | REFramework lifecycle (deep: `uipath-reframework` skill) |
| [orchestration/long-running.md](orchestration/long-running.md) | LRW, suspend/resume (deep: long-running / persistence skills) |
| [orchestration/coded-automations.md](orchestration/coded-automations.md) | CodedWorkflow, C# patterns |

### Activity area modules (`orchestration/activities/`)

| File | Topics |
|------|--------|
| [orchestration/activities/ui-automation.md](orchestration/activities/ui-automation.md) | Selectors, Object Repository, modern UI |
| [orchestration/activities/excel-office.md](orchestration/activities/excel-office.md) | Excel, Outlook, mail, DataTable |
| [orchestration/activities/web-api.md](orchestration/activities/web-api.md) | HTTP, REST, JSON, WebAPI |
| [orchestration/activities/database.md](orchestration/activities/database.md) | DB, queries |
| [orchestration/activities/system-files.md](orchestration/activities/system-files.md) | Files, folders, paths |
| [orchestration/activities/orchestrator-activities.md](orchestration/activities/orchestrator-activities.md) | Queues, assets, Orchestrator |
| [orchestration/activities/developer.md](orchestration/activities/developer.md) | Invoke Method, Python Scope, scripts |

**Routing shortcuts:** [orchestration/README.md](orchestration/README.md)

---

## B. Supporting references (primers & links)

| File | Topics |
|------|--------|
| [official-documentation-index.md](official-documentation-index.md) | Curated docs.uipath.com URLs |
| [invoke-code-and-expressions.md](invoke-code-and-expressions.md) | Invoke Code official behavior + link to orchestration deep dive |
| [connectors-and-integration.md](connectors-and-integration.md) | Integration Service orientation |
| [coded-automations.md](coded-automations.md) | Long C# / coded automation guide; for **patterns**, prefer [orchestration/coded-automations.md](orchestration/coded-automations.md) first |
| [activities.md](activities.md) | **XAML catalog** (many activities with sample XML); **narrative** Studio guidance is in `orchestration/activities/*.md` — read each topic once |
| [xaml-syntax.md](xaml-syntax.md) | XAML structure |
| [selectors.md](selectors.md) | Selectors |
| [project-structure.md](project-structure.md) | `project.json`, layout |
| [org-overlays/sales02-hitl-patterns.md](org-overlays/sales02-hitl-patterns.md) | Optional HITL HTTP + SALES02-style production XAML (not generic Studio defaults) |

---

## C. Delegation (other Cursor skills — not duplicate masters)

| Need | Skill |
|------|--------|
| Canvas → plan | `uipath-workflow-planning` |
| REFramework project | `uipath-reframework` |
| Code review automation | `uipath-code-reviewer` |
| Integration Service depth | `uipath-integration-service` |
| Data Service entities | `uipath-data-service` |
| Document Understanding | `uipath-document-understanding` |
| Persistence / HITL depth | `uipath-persistence-activities` |

Everything else Studio-side defaults to **`uipath-automation`** using the modules above.
