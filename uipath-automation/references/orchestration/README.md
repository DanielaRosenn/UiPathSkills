# Orchestration modules (master skill layer)

These files were merged from the **uipath-master** Studio developer pack. They are **progressive disclosure**: load only what the user’s task requires, in addition to the main [SKILL.md](../../SKILL.md).

**Version note:** Module headers may cite Studio **2025.10**; align with your installed Studio and [official-documentation-index.md](../official-documentation-index.md).

## Routing table — classify first, then read

| User asks about | Read this module |
|-----------------|------------------|
| Build workflow / generate XAML / Sequence, Flowchart, State Machine, structure | [xaml-generator.md](xaml-generator.md) |
| UI automation, selectors, Object Repository, Use Application/Browser | [activities/ui-automation.md](activities/ui-automation.md) |
| Excel, Outlook mail, SMTP, DataTable ops | [activities/excel-office.md](activities/excel-office.md) |
| HTTP, REST, JSON, WebAPI activities | [activities/web-api.md](activities/web-api.md) |
| Database, SQL, connections | [activities/database.md](activities/database.md) |
| Invoke Code (patterns inside snippet) | [invoke-code.md](invoke-code.md) |
| Invoke Method, Python Scope, scripts | [activities/developer.md](activities/developer.md) |
| File system, paths, zip | [activities/system-files.md](activities/system-files.md) |
| Queues, assets, Orchestrator activities | [activities/orchestrator-activities.md](activities/orchestrator-activities.md) |
| REFramework lifecycle | [reframework.md](reframework.md) |
| Long-running, suspend/resume, serialization; **main LRW / `ProcessDiagram` file** | [long-running.md](long-running.md) + [**uipath-longrunning-workflow**](../../../uipath-longrunning-workflow/SKILL.md) |
| Coded workflows / CodedWorkflow | [coded-automations.md](coded-automations.md) |
| Try/Catch, retries, GEH | [error-handling.md](error-handling.md) |
| Naming, Workflow Analyzer ST-* | [best-practices.md](best-practices.md) |
| Code review checklist | [code-reviewer.md](code-reviewer.md) |
| Publish, Git, Orchestrator deploy | [deploy-publish.md](deploy-publish.md) |
| Expressions (VB/C#), LINQ, Regex | [expressions.md](expressions.md) |
| Debugging in Studio | [debugging.md](debugging.md) |

**Multi-module:** Combine rows when the task spans domains (e.g. REFramework + Python → `reframework.md` + `activities/developer.md` + `xaml-generator.md`).

## Delegation to other Cursor skills

Do not duplicate full verticals; use dedicated skills when the user needs depth:

| Topic | Skill |
|-------|--------|
| Canvas → arguments / workflow plan | `uipath-workflow-planning` |
| REFramework project from scratch | `uipath-reframework` |
| Code review automation | `uipath-code-reviewer` |
| Integration Service connectors | `uipath-integration-service` |
| Data Service entities | `uipath-data-service` |
| Document Understanding pipelines | `uipath-document-understanding` |
| Persistence / Action Center–style HITL | `uipath-persistence-activities` |
| Studio desktop LRW / `ProcessDiagram` / Flowchart Builder | **`uipath-longrunning-workflow`** — [SKILL.md](../../../uipath-longrunning-workflow/SKILL.md) (org template optional; `LongRunningWorkflow` root class + dependencies) |

This **uipath-automation** skill remains the **default entry** for Studio XAML, `project.json`, and cross-cutting patterns.
