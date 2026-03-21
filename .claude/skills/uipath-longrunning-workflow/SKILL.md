---
name: uipath-longrunning-workflow
tags: [uipath-dev, long-running, studio, hitl, persistence, processdiagram]
description: UiPath Studio (desktop) Long Running Automation — ProcessDiagram XAML (upa:ProcessDiagram), Flowchart Builder designer, supportsPersistence, HITL, queues. Mandatory root pattern x:Class="LongRunningWorkflow" on entry XAML; project.json with FlowchartBuilder + Persistence + FormActivityLibrary + FormActivityLibrary.Contracts; missing-activity troubleshooting; uipathcli analyze. For Maestro or web BPMN use uipath-bpmn-maestro. Triggers on "Long Running Automation", "Main-Queue", "ProcessDiagram", "LongRunningWorkflow", "Flowchart Builder", "blank canvas", "Studio long running workflow", "approval process with persistence", "missing dependencies", "QueueItemData", "unresolved activity".
---

# UiPath Long Running Workflow (Studio Desktop)

Generate **UiPath Studio** long-running automations using **`upa:ProcessDiagram`** (Long Running Workflow designer / Flowchart Builder activities) with HITL Platform integration for human-in-the-loop processes. This is the **Windows Studio project** model, not the Maestro web modeler.

**Production Reference**: This skill is based on the production-validated `SALES02_RenewalPriceCommitment` project at `C:\UiPath\SALES02_RenewalPriceCommitment`.

## Not this skill

| Use another skill | When |
|-------------------|------|
| **`uipath-bpmn-maestro`** | Process modeling in **Maestro** / **Studio Web** ([bpmn.uipath.com](https://bpmn.uipath.com/)), [Maestro docs](https://docs.uipath.com/maestro/automation-cloud/latest), BPMN import for **cloud** process design — not desktop `.xaml` / `project.json` codegen. |
| **`uipath-automation`** | Standard sequences/flowcharts, transactional automations **without** Long Running Automation / ProcessDiagram as the main entry. |

## Official documentation (read-first)

Use these together: **Studio** explains the designer and project setup; **Activities** lists every Flowchart Builder activity, properties, and examples.

| Doc | URL |
|-----|-----|
| **Long Running Workflows** (Studio user guide) | [docs.uipath.com/studio/standalone/latest/user-guide/long-running-workflows](https://docs.uipath.com/studio/standalone/latest/user-guide/long-running-workflows) |
| **Flowchart Builder Activities** (Workflow Activities pack) | [docs.uipath.com/activities/other/latest/workflow/flowchart-builder-activities](https://docs.uipath.com/activities/other/latest/workflow/flowchart-builder-activities) |
| **Missing or Invalid Activities** (Output panel, dependency tree, Repair, Manage Packages) | [docs.uipath.com/studio/standalone/2023.10/user-guide/unresolved-activity](https://docs.uipath.com/studio/standalone/2023.10/user-guide/unresolved-activity) |

From the Studio guide (align implementation and explanations with this product wording):

- **Designer**: Long Running Workflow uses a **dedicated canvas**; activities appear under **Activities → Long Running Workflow** (plus **Recommended** for roots that can be **Change Type**’d into other elements).
- **New file**: Ribbon **New → Long Running Workflow**, or start from Backstage **Long Running Automation** template (**Start** tab *New from template* or **Templates** tab).
- **Dependencies** (default for a Long Running Workflow): **`UiPath.FlowchartBuilder.Activities`** (mandatory—do not remove) and **`UiPath.System.Activities`**.
- **BPMN import**: Explorer **Import Files →** `*.bpmn` creates a long running workflow; unsupported BPMN shapes show a **warning** on the node and validation errors until **Change Type** fixes them. (This is Studio file import, not Maestro cloud modeling.)
- **Manual Trigger** is **always present by default**, can be **moved** but **not deleted**; use **Change Type** to switch it to **On a Schedule** or **On App Trigger** where appropriate.
- **Process Mining**: nodes can be tracked for mining/analytics (see Studio doc).
- **Resilience / errors** (product terms): **Detached Error Handler**, **Error Boundary Event**, **Error End Event**—map to XAML patterns in this skill (`BoundaryNode`, `EventSubProcessNode`, Throw/Terminate).
- **Canvas hygiene**: **Change color** (Default, Green, Blue, Red, Yellow, Violet) for readability; optional for generated XAML but valid in Studio.

Activity **categories** in the Flowchart Builder docs match how to think about the diagram: **Triggers**, **Sequences** (task placeholders), **Subprocesses**, **Gateways**, **Events** (catch/intermediate; **Error Handler** attaches to Sequence or Subprocess), **End**. Nested details (Manual Trigger, Sequence → Human Approval, Resume after Delay, etc.) are in the linked activity pages.

## QUICK REFERENCE: Dependencies by activity (add to `project.json` when you add activities)

| Activity / type | NuGet package |
|-----------------|---------------|
| `upa:ProcessDiagram`, Flowchart Builder nodes, **Subprocess** | `UiPath.FlowchartBuilder.Activities` |
| Log Message, Assign, Sequence, Invoke Workflow File, **Add Queue Item** (system) | `UiPath.System.Activities` |
| **Add Queue Item And Get Reference**, **Wait For Queue Item And Resume**, `QueueItemData`, Form/App task persistence | `UiPath.Persistence.Activities` |
| **Form contracts** (transitive dep of Persistence 1.8.x — declare explicitly in `project.json`) | `UiPath.FormActivityLibrary` + `UiPath.FormActivityLibrary.Contracts` (e.g. `[2.0.8]`, same pattern as production LRW projects) |
| HTTP / Integration Service connectors | `UiPath.WebAPI.Activities` + often `UiPath.IntegrationService.Activities` (and connector-specific bundles) |

**Always apply** [Dependency hygiene and validation (mandatory)](#dependency-hygiene-and-validation-mandatory) when editing `project.json` or when Studio shows missing types or activities.

## Dependency hygiene and validation (mandatory)

This section is **part of the skill contract** for generated LRW projects: follow it on every change that touches **`UiPath.Persistence.Activities`**, **`QueueItemData`**, or **`Add Queue Item And Get Reference`**.

### 1. Persistence and Form packages (do not skip)

**`UiPath.Persistence.Activities` (e.g. 1.8.x)** declares a dependency on **`UiPath.FormActivityLibrary.Contracts`**. If those packages are only pulled transitively, NuGet may log an **“approximate best match”** for Contracts in **`.local/nuget.cache`**, which correlates with **missing or invalid activities** and **`UnknownType`** on **`QueueItemData`** at design time.

**Rule:** Whenever **`UiPath.Persistence.Activities`** is in `project.json`, **also declare explicitly** (same pattern as production **SALES02**):

- `UiPath.FormActivityLibrary` — e.g. `"[2.0.8]"`
- `UiPath.FormActivityLibrary.Contracts` — e.g. `"[2.0.8]"`

Then **Manage Dependencies → Restore** (or reopen the project). If problems persist with a stale cache, close Studio, delete the project **`.local`** folder once, and reopen.

### 2. Missing or invalid activities in Studio

Use the product workflow: [Missing or Invalid Activities](https://docs.uipath.com/studio/standalone/2023.10/user-guide/unresolved-activity) — read **Output** (red lines name the package), inspect the **dependency tree** on the project root, **Repair** invalid dependencies, install missing versions, restore again.

### 3. CLI governance analyze before handoff / CI

- Install **official [uipathcli](https://github.com/UiPath/uipathcli/releases)** (the `uipath` binary that supports **`studio package analyze`**). The **Python** `pip install uipath` CLI is **different** and does **not** implement `studio package …`.
- **Close Studio** (or at least close the project) before running **`uipath studio package analyze`** — the analyzer uses Studio’s local DB and fails if the project is open (**“already opened in another Studio instance”**).
- Cross-check with **`uipath-cli-git`** and **[UiPath CLI reference](../_shared/uipath-cli.md)** (`.claude/skills/_shared/` — same path with a Cursor **junction** to this repo), and with Studio **Workflow Analyzer** for rule coverage.

## QUICK REFERENCE: Critical XAML Rules

**Before generating ANY LRW XAML, remember these rules:**

### Root workflow file (Long Running entry with **ProcessDiagram**)

Studio binds the **Long Running Workflow** / Flowchart Builder **canvas** to the **root** `.xaml` declaration. Generated entry workflows **must** match the **Long Running Automation** template and production **SALES02** (`Main-Queue.xaml`), not a generic sequence-style root.

| Requirement | Description |
|---------------|-------------|
| **`x:Class="LongRunningWorkflow"`** on the root `<Activity>` | Production LRW files use this **class name** for the top-level workflow. Using the **project name** as `x:Class` (e.g. `LrwSkillTest`) is a common mistake and can cause the **wrong designer** or a **blank / non-rendering** Flowchart Builder surface. **File name** (e.g. `Main.xaml`) may still differ; **class name** stays `LongRunningWorkflow`. |
| **Default input arguments** on the root `<Activity>` as **attributes** | e.g. `this:LongRunningWorkflow.in_FolderName="Shared"` — same pattern as Studio-generated LRW. Prefer this over long `this:ClassName.prop` / `<InArgument><Literal>` **child blocks** under the root (those are easy to get wrong and differ from the template). |
| **`PresentationFramework`** in `TextExpression.ReferencesForImplementation` | Included in working LRW projects; supports WPF types used in diagram view state (`av:Point`, etc.). |
| **`UiPath.Persistence.Activities.Queue`** in `TextExpression.NamespacesForImplementation` | Declare alongside `UiPath.Persistence.Activities` when using **`QueueItemData`** / queue persistence activities. |

**Validated in Studio:** This pattern matches production **`SALES02`** (`Main-Queue.xaml`: `x:Class="LongRunningWorkflow"`, attribute defaults on `<Activity>`). An additional end-to-end sample used to confirm the **Flowchart Builder** surface loads correctly: workspace folder **`cursor_projects/LrwSkillTest`** (`Main.xaml` + `project.json` + `Workflows/PreFlight_Check.xaml`). Use it as a structural reference; project name may differ from the root **class** name.

**Minimal root `<Activity>` shape (first line only — ellipsis indicates xmlns attributes Studio emits):**

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="LongRunningWorkflow"
  this:LongRunningWorkflow.in_FolderName="Shared"
  xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:this="clr-namespace:" … />
```

After `x:Members`, follow with `sap:VirtualizedContainerService`, `TextExpression.*`, then a single **`upa:ProcessDiagram`** (not a top-level `Sequence`).

| Rule | Description |
|------|-------------|
| **Inline Nodes** | ALL nodes (EventNode, TaskNode, DecisionNode, EndNode) MUST be defined INLINE via nested `.Next` chains |
| **x:Reference Purpose** | `x:Reference` at end of ProcessDiagram only REFERENCES existing inline nodes - does NOT define new nodes |
| **Shared EndNode** | Define EndNode INLINE in one branch, use `<x:Reference>` in other branches to point to it |
| **Single Next** | Each TaskNode can only have ONE `<upa:TaskNode.Next>` element |
| **IdRef Required** | Every node MUST have `sap2010:WorkflowViewState.IdRef` attribute |
| **ViewState Required** | Every node MUST have `sap:WorkflowViewStateService.ViewState` with ShapeLocation, ShapeSize, ConnectorLocation |
| **Subprocess Structure** | Subprocess = TaskNode with `NodeType="Task.Subprocess"` containing embedded ProcessDiagram |
| **Webhook Pattern** | ApprovalFlow → Subprocess (webhook wait) → DecisionNode (route based on decision) |

**Common Errors:**
- `'Nodes' property has already been set` → Nodes defined both inline AND separately (see Pitfall 13b)
- `'Next' property has already been set` → TaskNode has two `<upa:TaskNode.Next>` elements
- `Value cannot be null` → Missing `sap2010:WorkflowViewState.IdRef` or `x:Reference`
- `ErrorActivity` → Missing namespace/assembly references for Integration Service connectors
- **`InvalidCastException: Literal`1[...] to type Argument`** → For the **LRW root** workflow, prefer **`this:LongRunningWorkflow.argumentName="value"`** attributes on `<Activity>` (template style). If using child elements, wrap literals in `<InArgument><Literal/></InArgument>`, not a bare `<Literal>` under `this:ClassName.propertyName`
- **Missing dependency / unresolved activities** → [Dependency hygiene and validation (mandatory)](#dependency-hygiene-and-validation-mandatory); Studio doc: [Missing or Invalid Activities](https://docs.uipath.com/studio/standalone/2023.10/user-guide/unresolved-activity).

**Pre-finalize (CI / handoff):** See **section 3** in [Dependency hygiene and validation (mandatory)](#dependency-hygiene-and-validation-mandatory) — **`uipath studio package analyze`** via **uipathcli**, Studio closed, plus Workflow Analyzer.

## CRITICAL: Canvas Context Required (MANDATORY)

**BEFORE generating ANY long-running workflow code, you MUST:**

0. **Do not generate any XAML until planning is complete.** If the user has or references a canvas/solution flow, run **`workflow-planning`** first and obtain `workflow-plan.md` or equivalent. **Greenfield process** (no canvas): still produce an execution plan (sequence, gateways, I/O per workflow)—use **`workflow-planning`**’s checklist style or a short written plan—before generating `Main-Queue.xaml` / ProcessDiagram.

```
┌─────────────────────────────────────────────────────────────────┐
│  MANDATORY WORKFLOW                                              │
│                                                                  │
│  1. FIND CANVAS ──> 2. RUN PLANNING ──> 3. GENERATE CODE        │
│     solution-flow.json    workflow-planning     this skill       │
└─────────────────────────────────────────────────────────────────┘
```

### Step 1: Locate Canvas File

```bash
# Search patterns
**/solution-flow*.json
**/*canvas*.json
**/docs/*.json
```

### Step 2: Extract from Canvas

For EACH approval workflow node, extract:

| Canvas Field | Maps To |
|--------------|---------|
| `drillDown.inputArguments` | Workflow `<x:Members>` input arguments |
| `drillDown.outputArguments` | Workflow `<x:Members>` output arguments |
| `drillDown.fields` | Form schema and adaptive card FactSet |
| `drillDown.visibilityLevel` | Which fields to show/hide |
| `drillDown.hiddenFields` | Fields to EXCLUDE from this workflow |

### Step 3: Create Data Flow Matrix

| Data Field | Source | Target Workflows | Type |
|------------|--------|------------------|------|
| quoteId | Queue | All approvals | String |
| profitabilityMargin | SF | RevOps, Manager, CRO, Finance (NOT SalesRep) | Decimal |
| managerChainJSON | Bob API | RevOps, CRO | String |

### Step 4: Validate Before Generation

- [ ] Canvas file located and read
- [ ] ALL node drillDown.inputArguments extracted
- [ ] ALL node drillDown.outputArguments extracted
- [ ] Visibility rules applied (LIMITED vs FULL)
- [ ] Data types match canvas specification
- [ ] TaskNode chain matches canvas edge flow

**DO NOT PROCEED if canvas data is missing. Do not generate Main-Queue.xaml or any ProcessDiagram XAML until the plan exists.**

### Integration with `workflow-planning`

**`workflow-planning`** is workflow-type agnostic; this skill consumes its **Studio LRW–specific** outputs (ProcessDiagram sequence, gateways, subprocesses, argument contracts).

| Situation | What to do |
|-----------|------------|
| **Canvas / SDD present** | Run **`workflow-planning`** first. Use `workflow-plan.md` (or equivalent): execution sequence, per-workflow arguments, visibility, canvas-to-structure validation. Then generate or edit XAML here. |
| **From scratch (greenfield)** | No `solution-flow.json`: still **plan first**—minimal execution sequence + argument tables (same artifacts **`workflow-planning`** would produce). Do not invent `Main-Queue.xaml` without that plan. |
| **Starting from the official template** | **Long Running Automation** template / clone is a **starting skeleton** only—replace placeholders using the plan; do not treat the template as a substitute for argument/data-flow definition. |

Before generating long-running workflow code:

1. **Run `workflow-planning`** when any structured inputs exist (canvas, SDD, or pasted node specs).
2. **Use its output** – `workflow-plan.md` or the same structured data (execution sequence, argument specs per workflow, visibility matrix).
3. **Validate alignment** – TaskNode order matches intended flow; every decision/gateway has a branch; rejection and post-approval steps exist where required.

If no canvas exists and the user cannot run a full extract, require an **execution sequence and argument list per invoked workflow** before generating XAML.

---

## CRITICAL: Template Source

**ALWAYS** base long-running workflows on the official template (Studio **Long Running Automation** template matches the same model as in [Studio – Long Running Workflows](https://docs.uipath.com/studio/standalone/latest/user-guide/long-running-workflows)):

- **GitHub**: https://github.com/cato-networks-IT/PROCESSNAME_LongRunningAutomationTemplate
- **Local Clone**: Clone the template first, then customize

```powershell
git clone https://github.com/cato-networks-IT/PROCESSNAME_LongRunningAutomationTemplate.git
```

The template provides **project shape, packages, and a working ProcessDiagram entry**—not the business plan. Pair it with **`workflow-planning`** output when building a real process.

## Architecture Overview

Long-running workflows in Studio use a **ProcessDiagram** canvas (BPMN-inspired shapes; see [Studio LRW docs](https://docs.uipath.com/studio/standalone/latest/user-guide/long-running-workflows)) with:

1. **`upa:ProcessDiagram`** - Process diagram root (NOT a plain top-level `Sequence` for the main LRW file)
2. **`supportsPersistence: true`** - Enables workflow persistence
3. **Event-driven triggers** - Email, Manual, Queue, Webhook
4. **Task nodes** (`upa:TaskNode`) - Task shapes on the diagram
5. **Boundary error handlers** - Attached error handling
6. **Detached Error Handler** - Global exception handling subprocess

```
┌─────────────────────────────────────────────────────────────────┐
│                    upa:ProcessDiagram                           │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐  │
│  │  Start   │───>│  Init    │───>│  Task    │───>│   End    │  │
│  │  Event   │    │ Settings │    │  Node    │    │  Event   │  │
│  └──────────┘    └──────────┘    └────┬─────┘    └──────────┘  │
│                                       │                         │
│                                  ┌────▼─────┐                   │
│                                  │ Boundary │                   │
│                                  │  Error   │                   │
│                                  └──────────┘                   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │            Detached Error Handler (Subprocess)          │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## When to Use

- "Create a long-running workflow" / **Long Running Automation** in **UiPath Studio**
- "Build an approval process with persistence" (desktop project)
- "Main-Queue" / `upa:ProcessDiagram` / `supportsPersistence`
- "Build queue-based automation" with suspend-resume
- Canvas export with approval nodes **implemented in Studio**
- Multi-step approval chains in **Studio** XAML

**Not for:** Maestro-only or [bpmn.uipath.com](https://bpmn.uipath.com/) modeling without Studio implementation — use **`uipath-bpmn-maestro`**.

---

## Foundations (from LRW best practices)

### What is a Long Running Workflow?
An LRW is a UiPath process that **suspends** at persistence points (human approval, wait for message, delay), freeing the robot until resume. It spans hours or days and supports human-in-the-loop.

**Package**: `UiPath.FlowchartBuilder.Activities` (v1.1.0+) for new canvas; `UiPath.Persistence.Activities` for persistence activities.  
**Template**: Studio Backstage → Templates → **Long Running Automation**.  
**Project setting**: `supportsPersistence: true` in project.json.  
**Runs in**: Session 0 (unattended background).

### How this skill generates XAML
This skill generates **ProcessDiagram XAML** (`upa:ProcessDiagram`) based on the GitHub template. The XAML uses BPMN-style element names (EventNode, TaskNode, DecisionNode, etc.). Studio 2024.10+ introduced RPA-friendly names on the visual canvas (Manual Trigger, Human Approval, etc.) but the underlying XAML structure is the same. The terminology table below maps both.

### Node terminology (Canvas UI vs XAML)
Studio 2024.10+ shows RPA-friendly names on the canvas; the generated XAML uses BPMN-style element names. Both refer to the same nodes:

| Canvas name (RPA) | XAML element / NodeType | Category |
|-------------------|-------------------------|----------|
| Manual Trigger | `upa:EventNode` + `StartEvent.Interrupting.None` | Trigger |
| On a Schedule | `upa:EventNode` + Timer Start | Trigger |
| On App Trigger | `upa:EventNode` + `StartEvent.Interrupting.Message` | Trigger |
| Sequence | `upa:TaskNode` + `Task.None` | Task |
| Invoke Workflow | `upa:TaskNode` + `Task.Service` | Task |
| Activity | `upa:TaskNode` + `Task.Send` | Task |
| Human Approval | `upa:TaskNode` + `Task.User` (contains Create App/Form Task + Wait) | Task |
| Business Rule | `upa:TaskNode` + `Task.BusinessRule` | Task |
| Subprocess | `upa:TaskNode` + `Task.Subprocess` (embedded ProcessDiagram) | Task |
| Wait for Trigger | `upa:TaskNode` + `Task.Receive` or webhook subprocess | Event |
| Resume after Delay | Timer Intermediate Catch / Resume After Delay activity | Event |
| Wait for Message | `upa:EventNode` + `IntermediateEvent.Catch.Message` | Event |
| Decision | `upa:DecisionNode` | Gateway |
| Switch | Switch activity inside TaskNode | Gateway |
| Split / Merge | Parallel branches (Split → Merge) | Gateway |
| Error Handler | `upa:BoundaryNode` + `BoundaryEvent.Interrupting.Error` | Error |
| Detached Error Handler | `upa:EventSubProcessNode` + `StartEvent.Interrupting.Error` | Error |
| End | `upa:EndNode` + `EndEvent.None` | End |
| Throw | Throw activity (propagates to handler) | End |
| Terminate | Terminate activity (hard stop) | End |

### Serialization rules (critical)
All variables **in scope across a persistence/suspension point** must be serializable.

**Serializable**: String, Boolean, Int32, Double, DateTime, Array, DataTable, Dictionary, JObject, GenericValue.  
**Not serializable**: SecureString, HttpClient, IWebDriver, browser/UI handles, custom non-serializable types.

**Rule**: Any variable in the same scope as a Human Approval (webhook wait), Wait for Message, or Resume after Delay must be serializable. Keep non-serializable work in invoked workflows that finish before the suspension point.

### For Each + Human Approval
Do **not** place a Human Approval (or any Wait/Resume) directly inside a loop. Split:

- **Wrong**: For Each item → Human Approval (inside loop).
- **Right**: For Each item → Invoke Workflow "ProcessOneItem.xaml" → inside that workflow, Human Approval (or webhook subprocess) then resume.

For multiple managers, use the **Manager Approval Loop** pattern (one subprocess with internal loop and x:Reference), not a For Each over approvals.

### Resume after Delay (timed wait)
For any wait that must survive robot restart, use **Resume After Delay** from `UiPath.Persistence.Activities` (or the equivalent node in the template), not the standard `Delay` activity. Standard Delay does not persist; after restart the remaining time is lost.

### Error handling: when to use which
| Need | Use |
|------|-----|
| Handle error for one specific TaskNode | **BoundaryNode** (Error Handler) on that node |
| Handle a type of error from anywhere | **Detached Error Handler** (EventSubProcessNode) |
| Re-throw so Orchestrator marks job faulted | Handler → InvokeWorkflow ExceptionHandler → Throw / End |

### Variable naming (optional but clear)
For variables that cross persistence points, consider: `p_` (persisted), `temp_` (temporary, not across suspend), `decision_` (approval decision per step), `taskData_` (task output).

### Quick node selection guide

**Starting the process:**
- Runs manually / on demand → **Manual Trigger** (EventNode, StartEvent.Interrupting.None)
- Runs on a schedule (daily, hourly, cron) → **On a Schedule** (Timer Start)
- Triggered by external event (email, record created, queue item) → **On App Trigger** (Message Start)

**Doing automated work:**
- Run an existing `.xaml` workflow → **Invoke Workflow** (Task.Service)
- Call a single activity (send email, HTTP request) → **Activity** (Task.Send)
- Group steps on the same canvas → **Subprocess** (Task.Subprocess)

**Waiting / suspending (robot freed):**
- Wait for human to approve in Action Center → **Human Approval** (Task.User with Create App/Form Task + Wait)
- Wait for external event (email reply, Salesforce update, webhook) → **Wait for Trigger** or **Wait for Message**
- Wait a fixed duration or until a date/time → **Resume after Delay**

**Routing logic:**
- True/False branch → **Decision** (DecisionNode)
- Multiple cases based on a value → **Switch**
- Parallel paths → **Split + Merge**

**Ending the process:**
- Normal completion → **End** (EndNode)
- Throw exception (to be caught by handler) → **Throw**
- Kill entire process immediately → **Terminate**

**Error handling:**
- Catch error on one specific node → **Error Handler** (BoundaryNode on that node)
- Catch error type from anywhere → **Detached Error Handler** (EventSubProcessNode)

---

## Project Structure (From Template)

```
{project_name}/
├── project.json                    # supportsPersistence: true
├── Main.xaml                       # Email-triggered entry point
├── Main-Queue.xaml                 # Queue-based entry point
├── Framework/
│   ├── InitAllSettingsJSON.xaml    # Load config and assets
│   ├── ExceptionHandler.xaml       # Exception handling
│   ├── TakeScreenshot.xaml         # Screenshot utility
│   ├── KillAllProcesses.xaml       # Process cleanup
│   ├── InitAllApplications.xaml    # App initialization
│   └── CloseAllApplications.xaml   # App cleanup
├── Orchestrator/
│   ├── getTransaction.xaml         # Get queue item
│   └── uploadFailedQueueItem.xaml  # Mark queue item failed
├── Workflows/                      # Custom workflow files
│   ├── ApprovalFlow_SalesRep.xaml
│   ├── ApprovalFlow_RevOps.xaml
│   └── ...
├── Services/                       # API integration workflows
│   ├── GetManagerHierarchy.xaml
│   └── PolicyCheck.xaml
├── Data/
│   └── Config.json                 # Configuration settings
└── sendSlackExceptionMessage.xaml  # Slack notification
```

## Project Configuration

### project.json (CRITICAL Settings - From SALES02 Production)

Use **exact versions** from production reference (SALES02 at `C:\UiPath\SALES02_RenewalPriceCommitment`). Wrong versions cause missing activity or ErrorActivity.

**Persistence + Form:** Any LRW project that includes **`UiPath.Persistence.Activities`** must list **`UiPath.FormActivityLibrary`** and **`UiPath.FormActivityLibrary.Contracts`** explicitly in `dependencies` (see [Dependency hygiene and validation (mandatory)](#dependency-hygiene-and-validation-mandatory)). The JSON example below includes them.

```json
{
  "name": "SALES02_RenewalPriceCommitment",
  "projectId": "39568a59-4fdb-4b66-96f8-faefb61c0220",
  "description": "Approval process for Renewal Price Commitment",
  "main": "Main-Queue.xaml",
  "dependencies": {
    "UiPath.Excel.Activities": "[3.3.1]",
    "UiPath.FlowchartBuilder.Activities": "[1.1.2]",
    "UiPath.FormActivityLibrary": "[2.0.8]",
    "UiPath.FormActivityLibrary.Contracts": "[2.0.8]",
    "UiPath.HttpWebhook.IntegrationService.Activities": "[5.0.0-preview]",
    "UiPath.IntegrationService.Activities": "[1.23.0]",
    "UiPath.MicrosoftOffice365.Activities": "[3.5.0-preview]",
    "UiPath.Persistence.Activities": "[1.8.1]",
    "UiPath.System.Activities": "[25.10.3]",
    "UiPath.Testing.Activities": "[25.10.0]",
    "UiPath.UIAutomation.Activities": "[25.10.19]",
    "UiPath.WebAPI.Activities": "[2.3.2]"
  },
  "schemaVersion": "4.0",
  "studioVersion": "25.10.6.0",
  "projectVersion": "1.0.71",
  "runtimeOptions": {
    "autoDispose": false,
    "netFrameworkLazyLoading": false,
    "isPausable": true,
    "isAttended": false,
    "requiresUserInteraction": false,
    "supportsPersistence": true,
    "workflowSerialization": "NewtonsoftJson",
    "excludedLoggedData": ["Private:*", "*password*"],
    "executionType": "Workflow",
    "readyForPiP": false,
    "startsInPiP": false,
    "mustRestoreAllDependencies": true,
    "pipType": "ChildSession",
    "robotVersion": "25.10.0"
  },
  "designOptions": {
    "projectProfile": "Developement",
    "outputType": "Process",
    "libraryOptions": { "privateWorkflows": [] },
    "processOptions": { "ignoredFiles": [] },
    "fileInfoCollection": [],
    "saveToCloud": false
  },
  "arguments": {
    "input": [
      {
        "name": "in_FolderName",
        "type": "System.String",
        "hasDefault": true,
        "defaultValue": "Sales/SALES02_RenewalPriceCommitment"
      }
    ],
    "output": []
  },
  "expressionLanguage": "VisualBasic",
  "entryPoints": [
    {
      "filePath": "Main-Queue.xaml",
      "uniqueId": "1278c2c7-eaea-4f7f-9220-8179385e137d",
      "input": [{"name": "in_FolderName", "type": "System.String", "hasDefault": true}],
      "output": []
    }
  ],
  "isTemplate": false,
  "targetFramework": "Windows"
}
```

**CRITICAL Settings:**
- `supportsPersistence: true` - Enables long-running behavior
- `"projectProfile": "Developement"` - UiPath's typo, must be spelled this way
- `UiPath.Persistence.Activities` - Required for queue operations
- `UiPath.HttpWebhook.IntegrationService.Activities` - Required for webhook persistence
- `UiPath.IntegrationService.Activities` - Required for connector activities
- `in_FolderName` argument - Used by InitAllSettingsJSON to load assets from correct Orchestrator folder

## CRITICAL: XAML Namespace and Activity Tag Rules

**ALWAYS** use these rules to avoid `ErrorActivity` and parse errors:

1. **Single-line Activity tag** – The entire opening `<Activity ...>` tag with all `xmlns` must be on one line. Multi-line breaks cause namespace/parse errors in UiPath Studio.

2. **Required namespaces** – Include at least: `mc`, `sap`, `sap2010`, `scg`, `sco`, `s`, `ui`, `x`, `upa`, `upas`. For approval/HTTP/email workflows add: `njl`, `uwah`, `umam`, `umame`, `umamm`, `usau`, `uwahm`. For webhook subprocesses add: `isactr`, `uiascb`, `uiascb1`.

3. **Critical activity mappings** – Use the correct activity tags; wrong ones load as ErrorActivity:

| Purpose | CORRECT | WRONG |
|---------|---------|-------|
| HTTP POST/GET | `uwah:NetHttpRequest` | `ui:HttpClient` |
| Send email | `umam:SendMailConnections` | `ui:SendOutlookMailMessage` |
| Parse JSON | `ui:DeserializeJson x:TypeArguments="njl:JObject"` | `ui:DeserializeJson` alone |
| Queue item | `upaq:AddQueueItemAndGetReference` | `ui:AddQueueItem` |
| Main diagram | `upa:ProcessDiagram` | `<Sequence>` for long-running |

4. **Quotes in expressions** – In attribute values, use `&quot;` for string literals (e.g. `Condition="[x = &quot;Approved&quot;]"`). Unescaped `"` in VB expressions causes XML parse errors.

5. **InvokeCode** – Do **not** set `Language="VisualBasic"`. Omit the `Language` attribute; UiPath defaults to VB. Explicit `Language="VisualBasic"` can cause `NetLanguage` errors.

6. **Avoid `<` in string literals** – Inside `<ui:InvokeCode.Code>`, do not use `<` or `<=` in strings (e.g. "terms <= 24 months"). Use wording like "24 months or less" to avoid XML parsing issues.

## XAML Structure (ProcessDiagram)

### CRITICAL: Node Definition Rules

**ALL nodes MUST be defined INLINE** via nested Next chains. The `x:Reference` elements at the end only REFERENCE existing nodes - they do NOT define new nodes.

```
ProcessDiagram Structure:
├── ProcessDiagram.Variables (optional)
├── ProcessDiagram.StartNode → x:Reference to first EventNode
├── EventNode (INLINE - contains entire flow via .Next chains)
│   └── EventNode.Next → TaskNode → TaskNode.Next → ... → EndNode
├── x:Reference (references to inline nodes - NOT definitions)
└── ProcessDiagram.EventSubProcesses (optional - for Detached Error Handler)
```

### Main-Queue.xaml Pattern

The Main-Queue.xaml uses `upa:ProcessDiagram` with ALL nodes defined inline:

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="LongRunningWorkflow" 
  xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" 
  xmlns:av="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:upa="clr-namespace:UiPath.Process.Activities;assembly=UiPath.Process.Activities" 
  xmlns:upas="clr-namespace:UiPath.Process.Activities.Shared;assembly=UiPath.Process.Activities"
  ...>
  
  <upa:ProcessDiagram DisplayName="Long Running Workflow">
    <upa:ProcessDiagram.Variables>
      <Variable x:TypeArguments="scg:Dictionary(x:String, x:Object)" Name="Config" />
      <Variable x:TypeArguments="ui:QueueItem" Name="TransactionItem" />
    </upa:ProcessDiagram.Variables>
    
    <!-- StartNode references the first inline EventNode -->
    <upa:ProcessDiagram.StartNode>
      <x:Reference>__ReferenceID0</x:Reference>
    </upa:ProcessDiagram.StartNode>
    
    <!-- ALL NODES DEFINED INLINE via .Next chains -->
    <upa:EventNode x:Name="__ReferenceID0" DisplayName="Manual Trigger">
      <upa:EventNode.Behavior>
        <upa:StartBehavior>
          <upa:StartBehavior.DesignerMetadata>
            <upas:DesignerMetadata NodeType="StartEvent.Interrupting.None" />
          </upa:StartBehavior.DesignerMetadata>
        </upa:StartBehavior>
      </upa:EventNode.Behavior>
      
      <!-- Entire flow nested via .Next chains -->
      <upa:EventNode.Next>
        <upa:TaskNode x:Name="__ReferenceID1" DisplayName="InitAllSettings">
          <!-- InvokeWorkflowFile inside -->
          <upa:TaskNode.Next>
            <upa:TaskNode x:Name="__ReferenceID2" DisplayName="Process Task">
              <upa:TaskNode.Next>
                <!-- EndNode defined INLINE - not as separate element -->
                <upa:EndNode x:Name="__ReferenceID3" DisplayName="End" />
              </upa:TaskNode.Next>
            </upa:TaskNode>
          </upa:TaskNode.Next>
        </upa:TaskNode>
      </upa:EventNode.Next>
    </upa:EventNode>
    
    <!-- x:Reference ONLY references inline nodes - does NOT define them -->
    <x:Reference>__ReferenceID1</x:Reference>
    <x:Reference>__ReferenceID2</x:Reference>
    <x:Reference>__ReferenceID3</x:Reference>
    
    <!-- Detached Error Handler (optional) -->
    <upa:ProcessDiagram.EventSubProcesses>
      <upa:EventSubProcessNode DisplayName="Detached Error Handler">
        <!-- Global error handling subprocess -->
      </upa:EventSubProcessNode>
    </upa:ProcessDiagram.EventSubProcesses>
    
  </upa:ProcessDiagram>
</Activity>
```

### Shared Node Pattern (for Decision branches pointing to same End)

When multiple branches need to reach the same EndNode:
1. Define the EndNode INLINE in ONE branch (e.g., True branch)
2. Use `<x:Reference>` in other branches to point to it

```xml
<upa:DecisionNode x:Name="__ReferenceID5">
  <upa:DecisionNode.True>
    <upa:TaskNode x:Name="__ReferenceID6">
      <upa:TaskNode.Next>
        <!-- EndNode defined INLINE here -->
        <upa:EndNode x:Name="__ReferenceID8" DisplayName="End" />
      </upa:TaskNode.Next>
    </upa:TaskNode>
  </upa:DecisionNode.True>
  <upa:DecisionNode.False>
    <upa:TaskNode x:Name="__ReferenceID7">
      <upa:TaskNode.Next>
        <!-- Reference to same EndNode - NOT a new definition -->
        <x:Reference>__ReferenceID8</x:Reference>
      </upa:TaskNode.Next>
    </upa:TaskNode>
  </upa:DecisionNode.False>
</upa:DecisionNode>
```

### Key BPMN Node Types

| Node Type | XAML Element | DesignerMetadata NodeType | Description |
|-----------|--------------|---------------------------|-------------|
| Manual Trigger | `upa:EventNode` | `StartEvent.Interrupting.None` | Manual start event |
| Email/App Trigger | `upa:EventNode` | `StartEvent.Interrupting.Message` | Message-based start |
| Invoke Workflow | `upa:TaskNode` | `Task.Service` | Calls external .xaml workflow |
| Activity | `upa:TaskNode` | `Task.Send` | Single activity (Log, HTTP, etc.) |
| Sequence | `upa:TaskNode` | `Task.None` | Container for multiple activities |
| Subprocess | `upa:TaskNode` | `Task.Subprocess` | Embedded ProcessDiagram |
| Wait for Trigger | `upa:TaskNode` | `Task.Receive` | Waits for external event |
| Human Approval | `upa:TaskNode` | `Task.User` | Action Center task |
| Business Rule | `upa:TaskNode` | `Task.BusinessRule` | DMN rule evaluation |
| Decision | `upa:DecisionNode` | N/A (uses Condition attr) | True/False gateway |
| End Event | `upa:EndNode` | `EndEvent.None` | Normal completion |
| Boundary Error | `upa:BoundaryNode` | `BoundaryEvent.Interrupting.Error` | Attached error handler |
| Detached Error | `upa:EventSubProcessNode` | `StartEvent.Interrupting.Error` | Global error handler |

## Adding Custom Workflows - Complete XAML Patterns

### 1. Manual Trigger (Start Event)

```xml
<upa:EventNode x:Name="__ReferenceID0" DisplayName="Manual Trigger" sap:VirtualizedContainerService.HintSize="40,40" sap2010:WorkflowViewState.IdRef="EventNode_1">
  <upa:EventNode.Behavior>
    <upa:StartBehavior>
      <upa:StartBehavior.DesignerMetadata>
        <upas:DesignerMetadata NodeType="StartEvent.Interrupting.None" />
      </upa:StartBehavior.DesignerMetadata>
    </upa:StartBehavior>
  </upa:EventNode.Behavior>
  <sap:WorkflowViewStateService.ViewState>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <av:Point x:Key="ShapeLocation">90,200</av:Point>
      <av:Size x:Key="ShapeSize">40,40</av:Size>
      <av:PointCollection x:Key="ConnectorLocation">130,220 170,220</av:PointCollection>
    </scg:Dictionary>
  </sap:WorkflowViewStateService.ViewState>
  <upa:EventNode.Next>
    <!-- First TaskNode -->
  </upa:EventNode.Next>
</upa:EventNode>
```

### 2. Invoke Workflow (Task.Service)

```xml
<upa:TaskNode x:Name="__ReferenceID1" DisplayName="InitAllSettings" sap:VirtualizedContainerService.HintSize="120,80" sap2010:WorkflowViewState.IdRef="TaskNode_1">
  <upa:TaskNode.Behavior>
    <upa:NodeBehavior>
      <upa:NodeBehavior.DesignerMetadata>
        <upas:DesignerMetadata NodeType="Task.Service" />
      </upa:NodeBehavior.DesignerMetadata>
    </upa:NodeBehavior>
  </upa:TaskNode.Behavior>
  <sap:WorkflowViewStateService.ViewState>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Boolean x:Key="IsExpanded">True</x:Boolean>
      <av:Point x:Key="ShapeLocation">170,180</av:Point>
      <av:Size x:Key="ShapeSize">120,80</av:Size>
      <av:PointCollection x:Key="ConnectorLocation">290,220 330,220</av:PointCollection>
    </scg:Dictionary>
  </sap:WorkflowViewStateService.ViewState>
  <ui:InvokeWorkflowFile ArgumentsVariable="{x:Null}" ContinueOnError="{x:Null}" DisplayName="InitAllSettings" sap:VirtualizedContainerService.HintSize="120,80" sap2010:WorkflowViewState.IdRef="InvokeWorkflowFile_1" UnSafe="False" WorkflowFileName="Framework\InitAllSettingsJSON.xaml">
    <ui:InvokeWorkflowFile.Arguments>
      <OutArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)" x:Key="out_Config">[Config]</OutArgument>
      <InArgument x:TypeArguments="x:String" x:Key="in_ConfigFileJson">Data\Config.json</InArgument>
    </ui:InvokeWorkflowFile.Arguments>
    <sap:WorkflowViewStateService.ViewState>
      <scg:Dictionary x:TypeArguments="x:String, x:Object" />
    </sap:WorkflowViewStateService.ViewState>
  </ui:InvokeWorkflowFile>
  <upa:TaskNode.Next>
    <!-- Next TaskNode -->
  </upa:TaskNode.Next>
</upa:TaskNode>
```

### 3. Activity / Log Message (Task.Send)

```xml
<upa:TaskNode x:Name="__ReferenceID2" DisplayName="Log Process Start" sap:VirtualizedContainerService.HintSize="120,80" sap2010:WorkflowViewState.IdRef="TaskNode_2">
  <upa:TaskNode.Behavior>
    <upa:NodeBehavior>
      <upa:NodeBehavior.DesignerMetadata>
        <upas:DesignerMetadata NodeType="Task.Send" />
      </upa:NodeBehavior.DesignerMetadata>
    </upa:NodeBehavior>
  </upa:TaskNode.Behavior>
  <sap:WorkflowViewStateService.ViewState>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Boolean x:Key="IsExpanded">True</x:Boolean>
      <av:Point x:Key="ShapeLocation">290,180</av:Point>
      <av:Size x:Key="ShapeSize">120,80</av:Size>
      <av:PointCollection x:Key="ConnectorLocation">410,220 450,220</av:PointCollection>
    </scg:Dictionary>
  </sap:WorkflowViewStateService.ViewState>
  <ui:LogMessage DisplayName="Log Process Start" sap:VirtualizedContainerService.HintSize="120,80" sap2010:WorkflowViewState.IdRef="LogMessage_1" Level="Info" Message="[&quot;Process Started - ID: &quot; + processId]">
    <sap:WorkflowViewStateService.ViewState>
      <scg:Dictionary x:TypeArguments="x:String, x:Object" />
    </sap:WorkflowViewStateService.ViewState>
  </ui:LogMessage>
  <upa:TaskNode.Next>
    <!-- Next TaskNode -->
  </upa:TaskNode.Next>
</upa:TaskNode>
```

### 4. Sequence (Task.None)

```xml
<upa:TaskNode x:Name="__ReferenceID3" DisplayName="Process Data Sequence" sap:VirtualizedContainerService.HintSize="120,80" sap2010:WorkflowViewState.IdRef="TaskNode_3">
  <upa:TaskNode.Behavior>
    <upa:NodeBehavior>
      <upa:NodeBehavior.DesignerMetadata>
        <upas:DesignerMetadata NodeType="Task.None" />
      </upa:NodeBehavior.DesignerMetadata>
    </upa:NodeBehavior>
  </upa:TaskNode.Behavior>
  <sap:WorkflowViewStateService.ViewState>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Boolean x:Key="IsExpanded">True</x:Boolean>
      <av:Point x:Key="ShapeLocation">450,180</av:Point>
      <av:Size x:Key="ShapeSize">120,80</av:Size>
      <av:PointCollection x:Key="ConnectorLocation">570,220 610,220</av:PointCollection>
    </scg:Dictionary>
  </sap:WorkflowViewStateService.ViewState>
  <Sequence DisplayName="Process Data Sequence" sap:VirtualizedContainerService.HintSize="120,80" sap2010:WorkflowViewState.IdRef="Sequence_1">
    <sap:WorkflowViewStateService.ViewState>
      <scg:Dictionary x:TypeArguments="x:String, x:Object">
        <x:Boolean x:Key="IsExpanded">True</x:Boolean>
      </scg:Dictionary>
    </sap:WorkflowViewStateService.ViewState>
    <Assign DisplayName="Set Variable" sap:VirtualizedContainerService.HintSize="416,175" sap2010:WorkflowViewState.IdRef="Assign_1">
      <Assign.To>
        <OutArgument x:TypeArguments="x:String">[myVariable]</OutArgument>
      </Assign.To>
      <Assign.Value>
        <InArgument x:TypeArguments="x:String">["Value"]</InArgument>
      </Assign.Value>
    </Assign>
    <ui:LogMessage DisplayName="Log" sap:VirtualizedContainerService.HintSize="416,175" sap2010:WorkflowViewState.IdRef="LogMessage_2" Level="Info" Message="[myVariable]" />
  </Sequence>
  <upa:TaskNode.Next>
    <!-- Next TaskNode -->
  </upa:TaskNode.Next>
</upa:TaskNode>
```

### 5. Subprocess (Task.Subprocess) with Embedded ProcessDiagram

```xml
<upa:TaskNode x:Name="__ReferenceID4" DisplayName="Subprocess - Approval Handler" sap2010:WorkflowViewState.IdRef="TaskNode_4">
  <upa:TaskNode.Behavior>
    <upa:NodeBehavior>
      <upa:NodeBehavior.DesignerMetadata>
        <upas:DesignerMetadata NodeType="Task.Subprocess" />
      </upa:NodeBehavior.DesignerMetadata>
    </upa:NodeBehavior>
  </upa:TaskNode.Behavior>
  <sap:WorkflowViewStateService.ViewState>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <av:Point x:Key="ShapeLocation">610,180</av:Point>
      <av:Size x:Key="ShapeSize">120,80</av:Size>
      <x:String x:Key="NodeColor">Yellow</x:String>
      <av:PointCollection x:Key="ConnectorLocation">730,220 770,220</av:PointCollection>
    </scg:Dictionary>
  </sap:WorkflowViewStateService.ViewState>
  
  <!-- Embedded ProcessDiagram -->
  <upa:ProcessDiagram DisplayName="Subprocess - Approval Handler" sap:VirtualizedContainerService.HintSize="600,300" sap2010:WorkflowViewState.IdRef="ProcessDiagram_2">
    <upa:ProcessDiagram.Variables>
      <Variable x:TypeArguments="x:String" Name="subprocessVar" />
    </upa:ProcessDiagram.Variables>
    <sap:WorkflowViewStateService.ViewState>
      <scg:Dictionary x:TypeArguments="x:String, x:Object">
        <x:Boolean x:Key="IsExpanded">False</x:Boolean>
      </scg:Dictionary>
    </sap:WorkflowViewStateService.ViewState>
    <upa:ProcessDiagram.StartNode>
      <x:Reference>__ReferenceID10</x:Reference>
    </upa:ProcessDiagram.StartNode>
    
    <!-- Subprocess Start -->
    <upa:EventNode x:Name="__ReferenceID10" DisplayName="Subprocess Start" sap:VirtualizedContainerService.HintSize="40,40" sap2010:WorkflowViewState.IdRef="EventNode_10">
      <upa:EventNode.Behavior>
        <upa:StartBehavior>
          <upa:StartBehavior.DesignerMetadata>
            <upas:DesignerMetadata NodeType="StartEvent.Interrupting.None" />
          </upa:StartBehavior.DesignerMetadata>
        </upa:StartBehavior>
      </upa:EventNode.Behavior>
      <sap:WorkflowViewStateService.ViewState>
        <scg:Dictionary x:TypeArguments="x:String, x:Object">
          <av:Point x:Key="ShapeLocation">80,100</av:Point>
          <av:Size x:Key="ShapeSize">40,40</av:Size>
          <av:PointCollection x:Key="ConnectorLocation">120,120 160,120</av:PointCollection>
        </scg:Dictionary>
      </sap:WorkflowViewStateService.ViewState>
      <upa:EventNode.Next>
        <!-- Subprocess TaskNodes -->
        <upa:TaskNode x:Name="__ReferenceID11" DisplayName="Subprocess Task" sap:VirtualizedContainerService.HintSize="120,80" sap2010:WorkflowViewState.IdRef="TaskNode_10">
          <!-- ... task content ... -->
          <upa:TaskNode.Next>
            <upa:EndNode x:Name="__ReferenceID12" DisplayName="Subprocess End" sap:VirtualizedContainerService.HintSize="40,40" sap2010:WorkflowViewState.IdRef="EndNode_10">
              <upa:EndNode.Behavior>
                <upa:EndBehavior>
                  <upa:EndBehavior.DesignerMetadata>
                    <upas:DesignerMetadata NodeType="EndEvent.None" />
                  </upa:EndBehavior.DesignerMetadata>
                </upa:EndBehavior>
              </upa:EndNode.Behavior>
            </upa:EndNode>
          </upa:TaskNode.Next>
        </upa:TaskNode>
      </upa:EventNode.Next>
    </upa:EventNode>
    
    <!-- Subprocess x:References -->
    <x:Reference>__ReferenceID11</x:Reference>
    <x:Reference>__ReferenceID12</x:Reference>
  </upa:ProcessDiagram>
  
  <upa:TaskNode.Next>
    <!-- Next TaskNode after subprocess -->
  </upa:TaskNode.Next>
</upa:TaskNode>
```

### 6. Wait for Trigger (Task.Receive)

```xml
<upa:TaskNode x:Name="__ReferenceID5" DisplayName="Wait for Response" sap:VirtualizedContainerService.HintSize="120,80" sap2010:WorkflowViewState.IdRef="TaskNode_5">
  <upa:TaskNode.Behavior>
    <upa:NodeBehavior>
      <upa:NodeBehavior.DesignerMetadata>
        <upas:DesignerMetadata NodeType="Task.Receive" />
      </upa:NodeBehavior.DesignerMetadata>
    </upa:NodeBehavior>
  </upa:TaskNode.Behavior>
  <sap:WorkflowViewStateService.ViewState>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Boolean x:Key="IsExpanded">True</x:Boolean>
      <av:Point x:Key="ShapeLocation">320,100</av:Point>
      <av:Size x:Key="ShapeSize">120,80</av:Size>
      <av:PointCollection x:Key="ConnectorLocation">440,140 480,140</av:PointCollection>
    </scg:Dictionary>
  </sap:WorkflowViewStateService.ViewState>
  <!-- Content: typically a Delay or Wait activity -->
  <Sequence DisplayName="Wait for Trigger" sap:VirtualizedContainerService.HintSize="120,80" sap2010:WorkflowViewState.IdRef="Sequence_5">
    <ui:LogMessage DisplayName="Log Waiting" Level="Info" Message="[&quot;Waiting for external trigger...&quot;]" />
    <!-- For actual webhook: use ConnectorPersistenceActivity -->
  </Sequence>
  <upa:TaskNode.Next>
    <!-- Next TaskNode -->
  </upa:TaskNode.Next>
</upa:TaskNode>
```

### 7. Decision Node (Gateway)

```xml
<upa:DecisionNode x:Name="__ReferenceID6" Condition="[isApproved]" DisplayName="Is Approved?" sap:VirtualizedContainerService.HintSize="60,60" sap2010:WorkflowViewState.IdRef="DecisionNode_1">
  <sap:WorkflowViewStateService.ViewState>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Boolean x:Key="IsExpanded">True</x:Boolean>
      <av:Point x:Key="ShapeLocation">770,190</av:Point>
      <av:Size x:Key="ShapeSize">60,60</av:Size>
      <av:PointCollection x:Key="TrueConnector">830,220 870,220</av:PointCollection>
      <av:PointCollection x:Key="FalseConnector">800,250 800,350 870,350</av:PointCollection>
    </scg:Dictionary>
  </sap:WorkflowViewStateService.ViewState>
  <upa:DecisionNode.True>
    <!-- TaskNode for approved path -->
  </upa:DecisionNode.True>
  <upa:DecisionNode.False>
    <!-- TaskNode for rejected path OR x:Reference to shared handler -->
    <x:Reference>__ReferenceID_RejectionHandler</x:Reference>
  </upa:DecisionNode.False>
</upa:DecisionNode>
```

**Note**: For string comparisons, use XML escaping: `Condition="[decision = &quot;Approved&quot;]"`

### 8. Boundary Error Handler

```xml
<upa:TaskNode x:Name="__ReferenceID7" DisplayName="Risky Operation" sap:VirtualizedContainerService.HintSize="120,80" sap2010:WorkflowViewState.IdRef="TaskNode_7">
  <upa:TaskNode.Behavior>
    <upa:NodeBehavior>
      <upa:NodeBehavior.DesignerMetadata>
        <upas:DesignerMetadata NodeType="Task.Service" />
      </upa:NodeBehavior.DesignerMetadata>
    </upa:NodeBehavior>
  </upa:TaskNode.Behavior>
  <sap:WorkflowViewStateService.ViewState>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Boolean x:Key="IsExpanded">True</x:Boolean>
      <av:Point x:Key="ShapeLocation">450,180</av:Point>
      <av:Size x:Key="ShapeSize">120,80</av:Size>
      <av:PointCollection x:Key="ConnectorLocation">570,220 610,220</av:PointCollection>
    </scg:Dictionary>
  </sap:WorkflowViewStateService.ViewState>
  
  <!-- Boundary Error Handler attached to this TaskNode -->
  <upa:TaskNode.BoundaryNodes>
    <upa:BoundaryNode x:Name="__ReferenceID20" DisplayName="Error Handler" sap:VirtualizedContainerService.HintSize="40,40" sap2010:WorkflowViewState.IdRef="BoundaryNode_1">
      <upa:BoundaryNode.Behavior>
        <upa:CatchMostSpecificErrorBehavior>
          <upa:CatchMostSpecificErrorBehavior.DesignerMetadata>
            <upas:DesignerMetadata NodeType="BoundaryEvent.Interrupting.Error" />
          </upa:CatchMostSpecificErrorBehavior.DesignerMetadata>
        </upa:CatchMostSpecificErrorBehavior>
      </upa:BoundaryNode.Behavior>
      <sap:WorkflowViewStateService.ViewState>
        <scg:Dictionary x:TypeArguments="x:String, x:Object">
          <av:Point x:Key="ShapeLocation">530,240</av:Point>
          <av:Size x:Key="ShapeSize">40,40</av:Size>
          <av:PointCollection x:Key="ConnectorLocation">550,280 550,350 590,350</av:PointCollection>
        </scg:Dictionary>
      </sap:WorkflowViewStateService.ViewState>
      <upa:BoundaryNode.Next>
        <upa:TaskNode x:Name="__ReferenceID21" DisplayName="Handle Error" sap:VirtualizedContainerService.HintSize="120,80" sap2010:WorkflowViewState.IdRef="TaskNode_20">
          <upa:TaskNode.Behavior>
            <upa:NodeBehavior>
              <upa:NodeBehavior.DesignerMetadata>
                <upas:DesignerMetadata NodeType="Task.Service" />
              </upa:NodeBehavior.DesignerMetadata>
            </upa:NodeBehavior>
          </upa:TaskNode.Behavior>
          <ui:LogMessage DisplayName="Log Error" Level="Error" Message="[&quot;Error caught: &quot; + exception.Message]" />
          <upa:TaskNode.Next>
            <x:Reference>__ReferenceID_End</x:Reference>
          </upa:TaskNode.Next>
        </upa:TaskNode>
      </upa:BoundaryNode.Next>
    </upa:BoundaryNode>
  </upa:TaskNode.BoundaryNodes>
  
  <!-- Main task content -->
  <ui:InvokeWorkflowFile WorkflowFileName="RiskyOperation.xaml" />
  
  <upa:TaskNode.Next>
    <!-- Next TaskNode (normal path) -->
  </upa:TaskNode.Next>
</upa:TaskNode>
```

### 9. Detached Error Handler (EventSubProcessNode)

```xml
<upa:ProcessDiagram.EventSubProcesses>
  <upa:EventSubProcessNode x:Name="__ReferenceID30" DisplayName="Detached Error Handler" sap2010:WorkflowViewState.IdRef="EventSubProcessNode_1">
    <sap:WorkflowViewStateService.ViewState>
      <scg:Dictionary x:TypeArguments="x:String, x:Object">
        <av:Point x:Key="ShapeLocation">50,500</av:Point>
        <av:Size x:Key="ShapeSize">500,150</av:Size>
        <x:Boolean x:Key="IsExpanded">True</x:Boolean>
      </scg:Dictionary>
    </sap:WorkflowViewStateService.ViewState>
    <upa:ProcessDiagram DisplayName="Detached Error Handler" sap:VirtualizedContainerService.HintSize="500,150" sap2010:WorkflowViewState.IdRef="ProcessDiagram_3">
      <sap:WorkflowViewStateService.ViewState>
        <scg:Dictionary x:TypeArguments="x:String, x:Object">
          <x:Boolean x:Key="IsExpanded">True</x:Boolean>
        </scg:Dictionary>
      </sap:WorkflowViewStateService.ViewState>
      <upa:ProcessDiagram.StartNode>
        <x:Reference>__ReferenceID31</x:Reference>
      </upa:ProcessDiagram.StartNode>
      
      <!-- Error Start Event -->
      <upa:EventNode x:Name="__ReferenceID31" DisplayName="Error Start" sap:VirtualizedContainerService.HintSize="40,40" sap2010:WorkflowViewState.IdRef="EventNode_30">
        <upa:EventNode.Behavior>
          <upa:StartBehavior>
            <upa:StartBehavior.DesignerMetadata>
              <upas:DesignerMetadata NodeType="StartEvent.Interrupting.Error" />
            </upa:StartBehavior.DesignerMetadata>
          </upa:StartBehavior>
        </upa:EventNode.Behavior>
        <sap:WorkflowViewStateService.ViewState>
          <scg:Dictionary x:TypeArguments="x:String, x:Object">
            <av:Point x:Key="ShapeLocation">80,50</av:Point>
            <av:Size x:Key="ShapeSize">40,40</av:Size>
            <av:PointCollection x:Key="ConnectorLocation">120,70 160,70</av:PointCollection>
          </scg:Dictionary>
        </sap:WorkflowViewStateService.ViewState>
        <upa:EventNode.Next>
          <upa:TaskNode x:Name="__ReferenceID32" DisplayName="Log Global Error" sap:VirtualizedContainerService.HintSize="120,80" sap2010:WorkflowViewState.IdRef="TaskNode_30">
            <upa:TaskNode.Behavior>
              <upa:NodeBehavior>
                <upa:NodeBehavior.DesignerMetadata>
                  <upas:DesignerMetadata NodeType="Task.Send" />
                </upa:NodeBehavior.DesignerMetadata>
              </upa:NodeBehavior>
            </upa:TaskNode.Behavior>
            <ui:LogMessage Level="Error" Message="[&quot;GLOBAL ERROR: &quot; + exceptionText]" />
            <upa:TaskNode.Next>
              <upa:EndNode x:Name="__ReferenceID33" DisplayName="Error Handler End" sap:VirtualizedContainerService.HintSize="40,40" sap2010:WorkflowViewState.IdRef="EndNode_30">
                <upa:EndNode.Behavior>
                  <upa:EndBehavior>
                    <upa:EndBehavior.DesignerMetadata>
                      <upas:DesignerMetadata NodeType="EndEvent.None" />
                    </upas:DesignerMetadata>
                  </upa:EndBehavior>
                </upa:EndNode.Behavior>
              </upa:EndNode>
            </upa:TaskNode.Next>
          </upa:TaskNode>
        </upa:EventNode.Next>
      </upa:EventNode>
      
      <!-- Detached Handler x:References -->
      <x:Reference>__ReferenceID32</x:Reference>
      <x:Reference>__ReferenceID33</x:Reference>
    </upa:ProcessDiagram>
  </upa:EventSubProcessNode>
</upa:ProcessDiagram.EventSubProcesses>
```

### 10. End Node

```xml
<upa:EndNode x:Name="__ReferenceID9" DisplayName="End" sap:VirtualizedContainerService.HintSize="40,40" sap2010:WorkflowViewState.IdRef="EndNode_1">
  <upa:EndNode.Behavior>
    <upa:EndBehavior>
      <upa:EndBehavior.DesignerMetadata>
        <upas:DesignerMetadata NodeType="EndEvent.None" />
      </upa:EndBehavior.DesignerMetadata>
    </upa:EndBehavior>
  </upa:EndNode.Behavior>
  <sap:WorkflowViewStateService.ViewState>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Boolean x:Key="IsExpanded">True</x:Boolean>
      <av:Point x:Key="ShapeLocation">1030,200</av:Point>
      <av:Size x:Key="ShapeSize">40,40</av:Size>
    </scg:Dictionary>
  </sap:WorkflowViewStateService.ViewState>
</upa:EndNode>
```

### 11. Workflow Arguments (x:Members)

```xml
<x:Members>
  <x:Property Name="in_ProcessId" Type="InArgument(x:String)" />
  <x:Property Name="in_CustomerName" Type="InArgument(x:String)" />
  <x:Property Name="in_Config" Type="InArgument(scg:Dictionary(x:String, x:Object))" />
  <x:Property Name="out_FinalStatus" Type="OutArgument(x:String)" />
  <x:Property Name="out_ApprovalId" Type="OutArgument(x:String)" />
  <x:Property Name="io_TransactionItem" Type="InOutArgument(ui:QueueItem)" />
</x:Members>
```

## HITL Platform Integration

### HTTP Webhook Persistence Pattern (PRODUCTION VALIDATED)

This is the **primary pattern** used in production (SALES02) for HITL approval workflows. It uses `ConnectorPersistenceActivity` to wait for HTTP webhook callbacks.

**Flow Pattern:**
```
ApprovalFlow (sends request) → Webhook Subprocess (waits) → DecisionNode (routes)
```

#### Complete Webhook Subprocess Pattern

```xml
<!-- Step 1: Send Approval Request -->
<upa:TaskNode x:Name="__ReferenceID_ApprovalFlow" DisplayName="ApprovalFlow_CRO">
  <upa:TaskNode.Behavior>
    <upa:NodeBehavior>
      <upa:NodeBehavior.DesignerMetadata>
        <upas:DesignerMetadata NodeType="Task.Service" />
      </upa:NodeBehavior.DesignerMetadata>
    </upa:NodeBehavior>
  </upa:TaskNode.Behavior>
  <ui:InvokeWorkflowFile DisplayName="ApprovalFlow_CRO" WorkflowFileName="ApprovalFlows\ApprovalFlow_CRO.xaml">
    <ui:InvokeWorkflowFile.Arguments>
      <InArgument x:TypeArguments="x:String" x:Key="in_ApproverEmail">[CROEmail]</InArgument>
      <InArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)" x:Key="in_Config">[Config]</InArgument>
    </ui:InvokeWorkflowFile.Arguments>
  </ui:InvokeWorkflowFile>
  
  <upa:TaskNode.Next>
    <!-- Step 2: Webhook Subprocess waits for callback -->
    <upa:TaskNode x:Name="__ReferenceID_WebhookSubprocess" DisplayName="Subprocess - Run Webhook CRO Approval">
      <upa:TaskNode.Behavior>
        <upa:NodeBehavior>
          <upa:NodeBehavior.DesignerMetadata>
            <upas:DesignerMetadata NodeType="Task.Subprocess" />
          </upa:NodeBehavior.DesignerMetadata>
        </upa:NodeBehavior>
      </upa:TaskNode.Behavior>
      <sap:WorkflowViewStateService.ViewState>
        <scg:Dictionary x:TypeArguments="x:String, x:Object">
          <x:String x:Key="NodeColor">Yellow</x:String>
        </scg:Dictionary>
      </sap:WorkflowViewStateService.ViewState>
      
      <!-- Embedded ProcessDiagram for webhook wait -->
      <upa:ProcessDiagram DisplayName="Subprocess - Run Webhook CRO Approval">
        <sap:WorkflowViewStateService.ViewState>
          <scg:Dictionary x:TypeArguments="x:String, x:Object">
            <x:Boolean x:Key="IsExpanded">False</x:Boolean>
          </scg:Dictionary>
        </sap:WorkflowViewStateService.ViewState>
        <upa:ProcessDiagram.StartNode>
          <x:Reference>__ReferenceID_SubStart</x:Reference>
        </upa:ProcessDiagram.StartNode>
        
        <upa:EventNode x:Name="__ReferenceID_SubStart" DisplayName="Manual Trigger">
          <upa:EventNode.Behavior>
            <upa:StartBehavior>
              <upa:StartBehavior.DesignerMetadata>
                <upas:DesignerMetadata NodeType="StartEvent.Interrupting.None" />
              </upa:StartBehavior.DesignerMetadata>
            </upa:StartBehavior>
          </upa:EventNode.Behavior>
          <upa:EventNode.Next>
            <!-- CRITICAL: ConnectorPersistenceActivity waits for webhook -->
            <upa:EventNode x:Name="__ReferenceID_WebhookWait" DisplayName="Wait for an Event on HTTP Webhook and Resume">
              <upa:EventNode.Behavior>
                <upa:IntermediateBehavior>
                  <upa:IntermediateBehavior.DesignerMetadata>
                    <upas:DesignerMetadata NodeType="IntermediateEvent.Catch.Message" />
                  </upa:IntermediateBehavior.DesignerMetadata>
                </upa:IntermediateBehavior>
              </upa:EventNode.Behavior>
              <isactr:ConnectorPersistenceActivity 
                Configuration="[BASE64_CONFIG]" 
                ConnectionId="[GUID]" 
                DisplayName="Wait for an Event on HTTP Webhook and Resume" 
                UiPathActivityTypeId="c2fe9db2-8e73-3771-a3e0-f5c7e7b2a930">
                <isactr:ConnectorPersistenceActivity.FieldObjects>
                  <isactr:FieldObject Name="_Event" Type="FieldArgument">
                    <isactr:FieldObject.Value>
                      <OutArgument x:TypeArguments="uiascb:HTTP_WEBHOOK_Trigger">[event_output]</OutArgument>
                    </isactr:FieldObject.Value>
                  </isactr:FieldObject>
                  <isactr:FieldObject Name="out_Event_20_ID" Type="FieldArgument">
                    <isactr:FieldObject.Value>
                      <OutArgument x:TypeArguments="x:String">[eventID]</OutArgument>
                    </isactr:FieldObject.Value>
                  </isactr:FieldObject>
                </isactr:ConnectorPersistenceActivity.FieldObjects>
              </isactr:ConnectorPersistenceActivity>
              
              <upa:EventNode.Next>
                <upa:TaskNode DisplayName="Get Trigger Event Output">
                  <!-- ConnectorActivity to get trigger output -->
                  <upa:TaskNode.Next>
                    <upa:TaskNode DisplayName="Log Message">
                      <ui:LogMessage Level="Info" Message="[triggerOutput._Event.request_body.ToString]" />
                      <upa:TaskNode.Next>
                        <upa:TaskNode DisplayName="Deserialize Response">
                          <ui:InvokeWorkflowFile WorkflowFileName="Approval_DeserializeWebHook_CRO.xaml">
                            <ui:InvokeWorkflowFile.Arguments>
                              <InArgument x:TypeArguments="x:String" x:Key="in_webHookRequestBody">[triggerOutput._Event.request_body.ToString]</InArgument>
                              <OutArgument x:TypeArguments="x:String" x:Key="out_Decision">[decision_CRO]</OutArgument>
                            </ui:InvokeWorkflowFile.Arguments>
                          </ui:InvokeWorkflowFile>
                          <upa:TaskNode.Next>
                            <upa:EndNode DisplayName="End" />
                          </upa:TaskNode.Next>
                        </upa:TaskNode>
                      </upa:TaskNode.Next>
                    </upa:TaskNode>
                  </upa:TaskNode.Next>
                </upa:TaskNode>
              </upa:EventNode.Next>
            </upa:EventNode>
          </upa:EventNode.Next>
        </upa:EventNode>
      </upa:ProcessDiagram>
      
      <upa:TaskNode.Next>
        <!-- Step 3: Decision based on approval response -->
        <upa:DecisionNode Condition="[decision_CRO = &quot;Approved&quot;]" DisplayName="CRO Decision">
          <upa:DecisionNode.True>
            <!-- Continue to next step or end -->
          </upa:DecisionNode.True>
          <upa:DecisionNode.False>
            <!-- Handle rejection -->
            <upa:TaskNode DisplayName="Handle CRO Rejection">
              <ui:InvokeWorkflowFile WorkflowFileName="DeleteTransaction.xaml" />
            </upa:TaskNode>
          </upa:DecisionNode.False>
        </upa:DecisionNode>
      </upa:TaskNode.Next>
    </upa:TaskNode>
  </upa:TaskNode.Next>
</upa:TaskNode>
```

#### Required Variables for Webhook Pattern

**Complete Production Variable Set (from SALES02 Main-Queue.xaml):**

```xml
<upa:ProcessDiagram.Variables>
  <!-- Core workflow variables -->
  <Variable x:TypeArguments="scg:Dictionary(x:String, x:Object)" Name="Config" />
  <Variable x:TypeArguments="ui:QueueItem" Name="TransactionItem" />
  <Variable x:TypeArguments="x:String" Name="exceptionText" />
  <Variable x:TypeArguments="s:Exception" Name="exception" />
  
  <!-- Webhook event variables (shared across all approval subprocesses) -->
  <Variable x:TypeArguments="uiascb2:HTTP_WEBHOOK_Trigger" Name="WebHookEvent" />
  <Variable x:TypeArguments="uiascb:HTTP_WEBHOOK_Trigger" Name="event_output" />
  <Variable x:TypeArguments="x:String" Name="eventID" />
  <Variable x:TypeArguments="uiascb1:GetTriggerEventObject_Retrieve" Name="triggerOutput" />
  <Variable x:TypeArguments="njl:JObject" Name="deserializedJson" />
  
  <!-- Queue/Transaction data -->
  <Variable x:TypeArguments="upaq:QueueItemData" Name="QueueItem" />
  <Variable x:TypeArguments="njl:JObject" Name="retrievedQuote" />
  <Variable x:TypeArguments="x:String" Name="opportunityId" />
  <Variable x:TypeArguments="x:String" Name="QuoteID" />
  <Variable x:TypeArguments="x:String" Name="QuoteTerm" />
  <Variable x:TypeArguments="x:String" Name="QuoteStatus" />
  <Variable x:TypeArguments="x:String" Name="QuoteNumber" />
  <Variable x:TypeArguments="x:String" Name="QuoteName" />
  
  <!-- Deal data -->
  <Variable x:TypeArguments="x:String" Name="acv" />
  <Variable x:TypeArguments="x:String" Name="tcv" />
  <Variable x:TypeArguments="x:String" Name="accountName" />
  <Variable x:TypeArguments="x:String" Name="profitabilityMargin" />
  <Variable x:TypeArguments="x:String" Name="OpportunityName" />
  <Variable x:TypeArguments="x:String" Name="CustomerName" />
  
  <!-- Sales rep data -->
  <Variable x:TypeArguments="x:String" Name="salesRepName" />
  <Variable x:TypeArguments="x:String" Name="salesRepEmail" />
  <Variable x:TypeArguments="x:String" Name="salesRepId" />
  <Variable x:TypeArguments="x:String" Name="approverEmail" />
  
  <!-- Manager hierarchy data -->
  <Variable x:TypeArguments="x:String" Name="ManagerChainJSON" />
  <Variable x:TypeArguments="x:String" Name="CROEmail" />
  <Variable x:TypeArguments="x:String" Name="CROName" />
  <Variable x:TypeArguments="x:Int32" Name="ManagerCount" />
  
  <!-- Policy and approval data -->
  <Variable x:TypeArguments="x:String" Name="PolicyStatus" />
  <Variable x:TypeArguments="x:Boolean" Name="RequiresFinance" />
  <Variable x:TypeArguments="x:String" Name="maxCapPercent" />
  <Variable x:TypeArguments="x:String" Name="cpiInclusion" />
  <Variable x:TypeArguments="x:String" Name="BusinessJustification" />
  <Variable x:TypeArguments="x:Int32" Name="RenewalCommitmentTerm" />
  
  <!-- Decision variables per approval step -->
  <Variable x:TypeArguments="x:String" Name="decision_SalesRep" />
  <Variable x:TypeArguments="x:String" Name="decision_RevOps" />
  <Variable x:TypeArguments="x:String" Name="decision_managerLoop" />
  
  <!-- Final status tracking -->
  <Variable x:TypeArguments="x:Boolean" Name="AllApproved" />
  <Variable x:TypeArguments="x:String" Name="FinalStatus" />
  <Variable x:TypeArguments="x:String" Name="PriorApprovalsJSON" />
  <Variable x:TypeArguments="x:String" Name="LastApprovalId" />
</upa:ProcessDiagram.Variables>
```

**Variable Naming Conventions:**
- `decision_*` - Stores approval decision from each stage (e.g., "Approved", "Rejected")
- `*JSON` - Serialized JSON strings for complex data (ManagerChainJSON, PriorApprovalsJSON)
- `event_output`, `eventID`, `triggerOutput` - Webhook response handling (shared across subprocesses)
- `deserializedJson` - Parsed webhook response body

#### Required Namespaces for Webhook Activities

**CRITICAL**: The namespace declarations must match your Integration Service connector assemblies. These are dynamically generated based on your UiPath Cloud connection.

**Production Pattern (from SALES02):**
```xml
xmlns:isactr="http://schemas.uipath.com/workflow/integration-service-activities/isactr"
xmlns:uiascb="clr-namespace:UiPath.IntegrationService.Activities.SWEntities.C692BE26E04_HTTP_WEBHOOK_Trigger.Bundle;assembly=C692BE26E04_HTTP_WEB.tBAt61mTt9C124J7Z4qb5as1"
xmlns:uiascb1="clr-namespace:UiPath.IntegrationService.Activities.SWEntities.C692BE26E04_GetTriggerEventObject_Retrieve.Bundle;assembly=C692BE26E04_GetTrigg.OiA5RWVqVJ1iWTu52m8t1c2"
xmlns:uiascb2="clr-namespace:UiPath.IntegrationService.Activities.SWEntities.CB8DF0985A7_HTTP_WEBHOOK_Trigger.Bundle;assembly=CB8DF0985A7_HTTP_WEB.4FGZ73QjaUJ1YEDiMJZYCp1"
```

**Note**: The assembly names like `C692BE26E04_HTTP_WEB.tBAt61mTt9C124J7Z4qb5as1` are generated by UiPath when you configure the HTTP Webhook connector in Integration Service. Copy these from a working workflow in your project.

#### Complete Activity Tag Opening (Production Reference)

The `<Activity>` opening tag must be on a single line with ALL namespaces:

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="LongRunningWorkflow" this:LongRunningWorkflow.in_FolderName="Sales/SALES02_RenewalPriceCommitment" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:av="http://schemas.microsoft.com/winfx/2006/xaml/presentation" xmlns:isactr="http://schemas.uipath.com/workflow/integration-service-activities/isactr" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:njl="clr-namespace:Newtonsoft.Json.Linq;assembly=Newtonsoft.Json" xmlns:s="clr-namespace:System;assembly=System.Private.CoreLib" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:uiascb="clr-namespace:UiPath.IntegrationService.Activities.SWEntities.C692BE26E04_HTTP_WEBHOOK_Trigger.Bundle;assembly=C692BE26E04_HTTP_WEB.tBAt61mTt9C124J7Z4qb5as1" xmlns:uiascb1="clr-namespace:UiPath.IntegrationService.Activities.SWEntities.C692BE26E04_GetTriggerEventObject_Retrieve.Bundle;assembly=C692BE26E04_GetTrigg.OiA5RWVqVJ1iWTu52m8t1c2" xmlns:uiascb2="clr-namespace:UiPath.IntegrationService.Activities.SWEntities.CB8DF0985A7_HTTP_WEBHOOK_Trigger.Bundle;assembly=CB8DF0985A7_HTTP_WEB.4FGZ73QjaUJ1YEDiMJZYCp1" xmlns:uiascb3="clr-namespace:UiPath.IntegrationService.Activities.SWEntities.C91937DC93E_Quote.Bundle;assembly=C91937DC93E_Quote.0mfOP2XRVJS1ZZ13mrIkgr3" xmlns:uico="http://schemas.uipath.com/workflow/activities/contracts" xmlns:upa="clr-namespace:UiPath.Process.Activities;assembly=UiPath.Process.Activities" xmlns:upaq="clr-namespace:UiPath.Persistence.Activities.Queue;assembly=UiPath.Persistence.Activities" xmlns:upas="clr-namespace:UiPath.Process.Activities.Shared;assembly=UiPath.Process.Activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
```

### Approval Workflow Pattern (Simple - No Webhook Wait)

Each approval step is an **InvokeWorkflowFile** inside a **TaskNode**:

```xml
<upa:TaskNode DisplayName="Sales Rep Approval">
  <ui:InvokeWorkflowFile WorkflowFileName="Workflows\ApprovalFlow_SalesRep.xaml">
    <ui:InvokeWorkflowFile.Arguments>
      <InArgument x:TypeArguments="x:String" x:Key="in_ProcessId">[ProcessId]</InArgument>
      <InArgument x:TypeArguments="x:String" x:Key="in_CustomerName">[CustomerName]</InArgument>
      <InArgument x:TypeArguments="x:String" x:Key="in_HITL_API_URL">[Config("HITL_API_URL").ToString]</InArgument>
      <OutArgument x:TypeArguments="x:String" x:Key="out_ApprovalId">[ApprovalId]</OutArgument>
    </ui:InvokeWorkflowFile.Arguments>
  </ui:InvokeWorkflowFile>
</upa:TaskNode>
```

### HTTP Request Pattern (in Approval Workflows)

```xml
<uwah:NetHttpRequest 
  AuthenticationType="None" 
  ContinueOnError="True" 
  DisplayName="HTTP POST to HITL API" 
  Headers="[new Dictionary(Of String, String) From { { &quot;Content-Type&quot;, &quot;application/json&quot; }, { &quot;x-api-key&quot;, in_HITL_API_Key } }]" 
  Method="POST" 
  RequestBodyType="Text" 
  RequestUrl="[in_HITL_API_URL + &quot;/api/v1/approvals&quot;]" 
  Result="[responseContent]" 
  RetryCount="3" 
  RetryPolicyType="Basic" 
  TextPayload="[RequestBody]" 
  TextPayloadContentType="[&quot;application/json&quot;]" 
  TextPayloadEncoding="[&quot;UTF-8&quot;]">
</uwah:NetHttpRequest>
```

**CRITICAL**: Always use `AuthenticationType="None"` and pass auth via Headers.

## Config.json Structure

```json
{
  "Settings": {
    "HITL_API_URL": "http://HitlAp-HitlA-faNGMOpTUFLe-988756144.us-east-1.elb.amazonaws.com",
    "Bob_API_URL": "https://api.hibob.com",
    "NotificationChannel": "both",
    "CallbackWebhookURL": "https://cloud.uipath.com/catonetworks/webhooks/..."
  },
  "Constants": {
    "QueueName": "RenewalPriceCommitment",
    "MaxRetries": 3
  },
  "AnalyticsData": {}
}
```

## Canvas-Driven Workflow Generation

### Mapping Canvas Nodes to TaskNodes

Each canvas node becomes a `upa:TaskNode` in Main-Queue.xaml:

| Canvas Node Type | TaskNode DesignerMetadata | Workflow Location |
|------------------|---------------------------|-------------------|
| `type: "queue"` | `Task.Service` | Orchestrator/ |
| `type: "api"` | `Task.Service` | Services/ |
| `type: "dmn"` | `Task.Service` | Services/ |
| `type: "form"` / `icon: "human"` | `Task.User` | Workflows/ |
| `shape: "decision"` | Gateway (If/Switch) | Inline in Main |

### Approval Workflow Arguments from Canvas

**ALWAYS extract arguments from canvas `drillDown.inputArguments` and `drillDown.outputArguments`:**

```json
// Canvas node example
{
  "id": "n8",
  "label": "Sales Rep Form",
  "drillDown": {
    "visibilityLevel": "LIMITED",
    "hiddenFields": ["profitabilityMargin", "managerChain"],
    "inputArguments": [
      {"name": "in_ProcessId", "type": "String", "source": "Queue Item (QuoteId)"},
      {"name": "in_CustomerName", "type": "String", "source": "Queue Item"},
      {"name": "in_ACV", "type": "String", "source": "Queue Item (ACV)"}
    ],
    "outputArguments": [
      {"name": "out_ApprovalId", "type": "String"},
      {"name": "out_MaxCapPercent", "type": "Decimal"}
    ]
  }
}
```

**Generated XAML `<x:Members>`:**
```xml
<x:Members>
  <!-- From canvas inputArguments -->
  <x:Property Name="in_ProcessId" Type="InArgument(x:String)" />
  <x:Property Name="in_CustomerName" Type="InArgument(x:String)" />
  <x:Property Name="in_ACV" Type="InArgument(x:String)" />
  <!-- From canvas outputArguments -->
  <x:Property Name="out_ApprovalId" Type="OutArgument(x:String)" />
  <x:Property Name="out_MaxCapPercent" Type="OutArgument(x:Decimal)" />
</x:Members>
```

### Visibility Rules from Canvas

| Canvas visibilityLevel | What to Include | What to Exclude |
|------------------------|-----------------|-----------------|
| `LIMITED` | Basic deal info only | profitabilityMargin, managerChain, priorApprovals |
| `FULL` | All deal info + profitability | Nothing |
| `FULL + CHAIN EDITOR` | All + editable managerChain | Nothing |
| `FULL + POLICY` | All + policyViolationReason | Nothing |

### Form Fields to Adaptive Card Mapping

Canvas `drillDown.fields` → Adaptive Card FactSet:

```json
// Canvas fields
"fields": [
  {"name": "accountName", "label": "Customer Account", "visible": true},
  {"name": "quoteId", "label": "Quote ID", "visible": true},
  {"name": "profitabilityMargin", "visible": false}  // HIDDEN for LIMITED
]
```

**Generated FactSet (only visible fields):**
```json
{
  "type": "FactSet",
  "facts": [
    {"title": "Customer Account", "value": "{{accountName}}"},
    {"title": "Quote ID", "value": "{{quoteId}}"}
  ]
}
```

---

## Quality Checklist

Before completing workflow generation:

### Canvas / planning validation (MANDATORY)
- [ ] **CRITICAL**: Canvas file located and read, or workflow-plan.md / execution plan available
- [ ] **CRITICAL**: ALL drillDown.inputArguments (or plan equivalent) extracted for each workflow
- [ ] **CRITICAL**: ALL drillDown.outputArguments (or plan equivalent) extracted for each workflow
- [ ] **CRITICAL**: Visibility rules applied (hiddenFields respected)
- [ ] **CRITICAL**: Data types match canvas/plan specification
- [ ] **CRITICAL**: TaskNode order matches canvas/plan flow; gateways and rejection paths present

### Technical Validation
- [ ] **CRITICAL**: Project uses `upa:ProcessDiagram` (NOT regular Sequence)
- [ ] **CRITICAL**: `supportsPersistence: true` in project.json
- [ ] **CRITICAL**: Based on PROCESSNAME_LongRunningAutomationTemplate
- [ ] Framework files present (InitAllSettingsJSON, ExceptionHandler, etc.)
- [ ] Orchestrator files present (getTransaction, uploadFailedQueueItem)
- [ ] Config.json configured with HITL Platform settings
- [ ] All TaskNodes have proper DesignerMetadata NodeType
- [ ] Boundary error handlers on critical tasks
- [ ] Detached Error Handler subprocess configured
- [ ] HTTP requests use `AuthenticationType="None"` with Headers

## Common Pitfalls to Avoid

### 1. Using Regular Sequence Instead of ProcessDiagram
**WRONG**: `<Sequence DisplayName="Main">`
**RIGHT**: `<upa:ProcessDiagram DisplayName="Long Running Workflow">`

### 2. Missing supportsPersistence
**WRONG**: `"supportsPersistence": false`
**RIGHT**: `"supportsPersistence": true`

### 3. Using AuthenticationType="Basic"
**WRONG**: `AuthenticationType="Basic"` (causes XML parsing error)
**RIGHT**: `AuthenticationType="None"` with `Authorization` header

### 4. Not Using Template Structure
**WRONG**: Creating from scratch
**RIGHT**: Clone PROCESSNAME_LongRunningAutomationTemplate and customize

### 5. Missing Framework Files
**WRONG**: Only Main.xaml
**RIGHT**: Include all Framework/, Orchestrator/, Data/ files

### 6. Wrong Escaping in Adaptive Card Action.Http Body
**WRONG**: `\\""` (double backslash) - Card will NOT render
**RIGHT**: `\""` (single backslash) - Working pattern

### 7. Using HTTP URL for Adaptive Card Actions
**WRONG**: `[in_Config("HITLApiUrl").ToString & "/api/v1/approvals/adaptive-card/" & token]`
**RIGHT**: `["https://djun97l419cdy.cloudfront.net/api/v1/approvals/adaptive-card/" & token]`

Microsoft Outlook Actionable Messages REQUIRE HTTPS URLs. HTTP URLs will fail with "Target URL scheme 'http' is not allowed".

### 8. Wrong Language Attribute in InvokeCode
**WRONG**: `Language="VisualBasic"` (causes "VisualBasic is not a valid value for NetLanguage")
**RIGHT**: Omit the `Language` attribute; UiPath defaults to VB.

### 9. Using < or <= in String Literals Inside InvokeCode
**WRONG**: `violations.Add("Cap % exceeds 5% for terms <= 24 months")`
**RIGHT**: `violations.Add("Cap % exceeds 5% for terms 24 months or less")`

The `<` character in string literals inside `<ui:InvokeCode.Code>` is parsed as XML, causing "Name cannot begin with the '=' character" errors.

### 10. Webhook Subprocess Structure (CRITICAL)
**WRONG**: Placing `ConnectorPersistenceActivity` directly in main ProcessDiagram
**RIGHT**: Wrap webhook wait in a Subprocess TaskNode with embedded ProcessDiagram

**Production Pattern:**
```
Main ProcessDiagram
  └── TaskNode (ApprovalFlow - sends request)
        └── TaskNode (Subprocess - NodeType="Task.Subprocess")
              └── ProcessDiagram (embedded)
                    └── EventNode (Start)
                          └── EventNode (ConnectorPersistenceActivity - waits)
                                └── TaskNode (Get Trigger Event Output)
                                      └── TaskNode (Deserialize)
                                            └── EndNode
              └── DecisionNode (check decision)
```

### 11. Missing TextExpression.ReferencesForImplementation
**WRONG**: Only including `TextExpression.NamespacesForImplementation` without `TextExpression.ReferencesForImplementation`
**RIGHT**: Include both sections in every XAML workflow; copy references from a working template to avoid "ErrorActivity: This activity is missing or could not be loaded".

### 12. Using ForEach with JArray/JToken
**WRONG**: `ForEach x:TypeArguments="njl:JToken"` over a JArray (can cause ErrorActivity or runtime issues)
**RIGHT**: Use a `While` loop with an index and `CType(myJArray(index), JObject)` for reliable JArray iteration.

### 13. Missing x:Reference Nodes
**WRONG**: Only defining nodes inline without x:Reference declarations
**RIGHT**: Every node referenced in the ProcessDiagram needs an x:Reference at the end

```xml
<upa:ProcessDiagram>
  <!-- Node definitions -->
  <upa:EventNode x:Name="__ReferenceID0">...</upa:EventNode>
  
  <!-- REQUIRED: x:Reference for each node -->
  <x:Reference>__ReferenceID1</x:Reference>
  <x:Reference>__ReferenceID2</x:Reference>
</upa:ProcessDiagram>
```

### 13b. Nodes Property Already Set Error (CRITICAL)
**Error**: `'Nodes' property has already been set on 'ProcessDiagram'`

**Cause**: Defining nodes BOTH as separate elements AND inline. ProcessDiagram can only have nodes defined ONE way.

**WRONG**: Mixing inline and separate node definitions
```xml
<upa:ProcessDiagram>
  <!-- Inline node -->
  <upa:EventNode x:Name="__ReferenceID0">
    <upa:EventNode.Next>...</upa:EventNode.Next>
  </upa:EventNode>
  
  <!-- WRONG: Separate node definition - causes duplicate Nodes property -->
  <upa:EndNode x:Name="__ReferenceID9" DisplayName="End" />
  
  <x:Reference>__ReferenceID0</x:Reference>
</upa:ProcessDiagram>
```

**RIGHT**: ALL nodes inline, x:Reference only references existing nodes
```xml
<upa:ProcessDiagram>
  <upa:EventNode x:Name="__ReferenceID0">
    <upa:EventNode.Next>
      <upa:TaskNode x:Name="__ReferenceID1">
        <upa:TaskNode.Next>
          <!-- EndNode defined INLINE -->
          <upa:EndNode x:Name="__ReferenceID2" />
        </upa:TaskNode.Next>
      </upa:TaskNode>
    </upa:EventNode.Next>
  </upa:EventNode>
  <!-- x:Reference only REFERENCES - does not define -->
  <x:Reference>__ReferenceID1</x:Reference>
  <x:Reference>__ReferenceID2</x:Reference>
</upa:ProcessDiagram>
```

### 14. JSON Formatting in API Payloads
**WRONG**: `priorApprovals.ToString()` (produces pretty-printed JSON with newlines)
**RIGHT**: `priorApprovals.ToString(Newtonsoft.Json.Formatting.None)` (compact JSON)

**Common Error**: `BadRequest` with `SyntaxError: Bad control character in string literal in JSON`

### 15. DecisionNode Condition Syntax
**WRONG**: `Condition="decision_CRO = Approved"` (missing quotes and brackets)
**RIGHT**: `Condition="[decision_CRO = &quot;Approved&quot;]"` (proper VB syntax with XML escaping)

**Alternative for complex conditions:**
```xml
<upa:DecisionNode.Condition>
  <VisualBasicValue x:TypeArguments="x:Boolean" ExpressionText="decision_CRO = &quot;Approved&quot;" />
</upa:DecisionNode.Condition>
```

### 16. Duplicate Rejection Handlers (Maintainability)
**WRONG**: Multiple `DeleteTransaction` TaskNodes for each rejection path
**RIGHT**: Single shared rejection handler or reference to common workflow

**Problem**: Each approval stage (SalesRep, RevOps, Manager, CRO, Finance) having its own inline DeleteTransaction call creates:
- Code duplication
- Maintenance burden
- Inconsistent behavior if one is updated but others aren't

**Solution**: Use a single `DeleteTransaction.xaml` workflow with parameters for rejection stage, and route all rejection paths to it.

### 17. Subprocess ProcessDiagram Naming
**WRONG**: Inconsistent naming like `ProcessDiagram_5`, `ProcessDiagram_ManagerLoop`
**RIGHT**: Consistent pattern like `Subprocess - Run Webhook {ApprovalStage} Approval`

**Production Pattern:**
- `Subprocess - Run Webhook SalesRep Approval`
- `Subprocess - Run Webhook Rev Ops Approval`
- `Subprocess - Run Webhook Manager Approval`
- `Subprocess - Run Webhook CRO Approval`
- `Subprocess - Run Webhook Finance Approval`

### 18. Manager Approval Loop Pattern (PRODUCTION VALIDATED)
**CRITICAL**: Manager approvals require a loop pattern because there can be multiple managers in the chain.

**Production Implementation (from SALES02 Main-Queue.xaml):**

The Manager Loop is implemented as a **Subprocess TaskNode** containing:
1. **Local Variables** for loop state (managersArray, currentIndex, priorApprovals, loopComplete, wasRejected)
2. **InterruptibleWhile** activity for the loop (NOT a standard While or ForEach)
3. **Flowchart** inside the loop body for webhook handling

**Structure:**
```
TaskNode (SubProcess - Run ManagerLoopApproval, NodeType="Task.Subprocess")
  └── ProcessDiagram (with local Variables)
        └── EventNode (Start)
              └── TaskNode (Init Manager Loop Vars - Sequence)
                    └── TaskNode (InterruptibleWhile - Loop Through Managers)
                          └── Body: Sequence
                                └── Get Current Manager from managersArray[currentIndex]
                                └── InvokeWorkflowFile (ApprovalFlow_Manager.xaml)
                                └── Flowchart (subprocess for webhook)
                                      └── FlowStep (ConnectorPersistenceActivity)
                                      └── FlowStep (Get Trigger Event Output)
                                      └── FlowStep (Deserialize)
                                      └── FlowStep (Assign outputs)
                                └── Update priorApprovals, increment currentIndex
                                └── Check if rejected or complete
                    └── DecisionNode (wasRejected?)
                          ├── True: x:Reference to DeleteTransaction
                          └── False: Continue to CRO
```

**Key Variables (defined in subprocess ProcessDiagram.Variables):**
```xml
<upa:ProcessDiagram.Variables>
  <Variable x:TypeArguments="njl:JArray" Name="managersArray" />
  <Variable x:TypeArguments="x:Int32" Name="currentIndex" />
  <Variable x:TypeArguments="njl:JArray" Name="priorApprovals" />
  <Variable x:TypeArguments="x:Boolean" Name="loopComplete">
    <Variable.Default><Literal x:TypeArguments="x:Boolean" /></Variable.Default>
  </Variable>
  <Variable x:TypeArguments="x:Boolean" Name="wasRejected">
    <Variable.Default><Literal x:TypeArguments="x:Boolean" /></Variable.Default>
  </Variable>
  <Variable x:TypeArguments="x:String" Name="RejectionComments" />
  <Variable x:TypeArguments="njl:JObject" Name="currentManager" />
  <Variable x:TypeArguments="x:String" Name="managerEmail" />
  <Variable x:TypeArguments="x:String" Name="managerName" />
  <Variable x:TypeArguments="x:Int32" Name="managerLevel" />
</upa:ProcessDiagram.Variables>
```

**InterruptibleWhile Pattern:**
```xml
<ui:InterruptibleWhile DisplayName="Loop Through Managers" 
  sap2010:WorkflowViewState.IdRef="InterruptibleWhile_1">
  <ui:InterruptibleWhile.Condition>
    <VisualBasicValue x:TypeArguments="x:Boolean" 
      ExpressionText="currentIndex &lt; managersArray.Count AndAlso Not loopComplete AndAlso Not wasRejected" />
  </ui:InterruptibleWhile.Condition>
  <ui:InterruptibleWhile.Body>
    <Sequence DisplayName="Body">
      <!-- Process current manager -->
      <Assign DisplayName="Get Current Manager">
        <Assign.To><OutArgument x:TypeArguments="njl:JObject">[currentManager]</OutArgument></Assign.To>
        <Assign.Value><InArgument x:TypeArguments="njl:JObject">[CType(managersArray(currentIndex), JObject)]</InArgument></Assign.Value>
      </Assign>
      <!-- Extract email, name, level -->
      <!-- Send approval request -->
      <!-- Wait for webhook (Flowchart with ConnectorPersistenceActivity) -->
      <!-- Process response, update priorApprovals -->
      <!-- Increment index or set wasRejected/loopComplete -->
    </Sequence>
  </ui:InterruptibleWhile.Body>
</ui:InterruptibleWhile>
```

**Key Points:**
- Use `InterruptibleWhile` (NOT standard While) for persistence compatibility
- Use `Flowchart` inside the loop body for the webhook wait pattern
- Track `priorApprovals` as JArray to pass to each subsequent manager
- Use `loopComplete` and `wasRejected` flags to control loop exit
- Initialize with empty check: `If managersArray.Count = 0 Then loopComplete = True`

### 19. File System Caching with OneDrive
**ISSUE**: When working with files in OneDrive-synced folders, file reads may return stale content.

**Symptoms:**
- `Read` tool shows different content than `Shell` commands
- Line counts don't match
- Recent edits not visible

**Solutions:**
1. Use `[System.IO.File]::ReadAllText()` in PowerShell for accurate reads
2. Add `Start-Sleep -Seconds 2` after writes before reading
3. Use `Copy-Item -Force` followed by sleep for reliable overwrites

### 20. Webhook Subprocess Must Be Inside TaskNode
**WRONG**: ProcessDiagram directly as child of another ProcessDiagram
**RIGHT**: ProcessDiagram wrapped inside TaskNode with `NodeType="Task.Subprocess"`

### 21. TaskNode nesting and closing tags
**WRONG**: Mismatched or unclosed `<upa:TaskNode.Next>` when chaining TaskNodes (causes "start tag does not match end tag").
**RIGHT**: Each TaskNode must close its `<upa:TaskNode.Next>` before `</upa:TaskNode>`. Pattern: open TaskNode → Behavior → content → TaskNode.Next → (next node) → close TaskNode.Next → close TaskNode.

```xml
<!-- CORRECT Structure -->
<upa:TaskNode DisplayName="Subprocess - Run Webhook CRO Approval">
  <upa:TaskNode.Behavior>
    <upa:NodeBehavior>
      <upa:NodeBehavior.DesignerMetadata>
        <upas:DesignerMetadata NodeType="Task.Subprocess" />
      </upa:NodeBehavior.DesignerMetadata>
    </upa:NodeBehavior>
  </upa:TaskNode.Behavior>
  <sap:WorkflowViewStateService.ViewState>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:String x:Key="NodeColor">Yellow</x:String>  <!-- Yellow indicates subprocess -->
    </scg:Dictionary>
  </sap:WorkflowViewStateService.ViewState>
  <upa:ProcessDiagram DisplayName="Subprocess - Run Webhook CRO Approval">
    <!-- Subprocess content -->
  </upa:ProcessDiagram>
</upa:TaskNode>
```

### 22. Duplicate TaskNode.Next Property (CRITICAL)
**WRONG**: Adding a second `<upa:TaskNode.Next>` to a TaskNode that already has one
**RIGHT**: A TaskNode can only have ONE `<upa:TaskNode.Next>` property

**Error**: `'Next' property has already been set on 'TaskNode'`

**WRONG Pattern:**
```xml
<upa:TaskNode x:Name="__ReferenceID39">
  <ui:InvokeWorkflowFile ... />
  <upa:TaskNode.Next>
    <upa:DecisionNode x:Name="__ReferenceID40" ... />
  </upa:TaskNode.Next>
  <!-- ERROR: Second TaskNode.Next -->
  <upa:TaskNode.Next>
    <upa:TaskNode x:Name="__ReferenceID50" ... />
  </upa:TaskNode.Next>
</upa:TaskNode>
```

**RIGHT Pattern** - Chain through DecisionNode.True:
```xml
<upa:TaskNode x:Name="__ReferenceID39">
  <ui:InvokeWorkflowFile ... />
  <upa:TaskNode.Next>
    <upa:DecisionNode x:Name="__ReferenceID40" Condition="[decision = &quot;Approved&quot;]">
      <upa:DecisionNode.True>
        <upa:TaskNode x:Name="__ReferenceID50" ... />
      </upa:DecisionNode.True>
      <upa:DecisionNode.False>
        <!-- Rejection handling -->
      </upa:DecisionNode.False>
    </upa:DecisionNode>
  </upa:TaskNode.Next>
</upa:TaskNode>
```

### 23. sap2010:WorkflowViewState.IdRef Required for Designer
**WRONG**: TaskNode or DecisionNode without `sap2010:WorkflowViewState.IdRef` attribute
**RIGHT**: All nodes that appear in the designer need unique IdRef values

**Error**: `Value cannot be null` in `SetFlowElementModelItem` or `FlowDecisionBehavior.AddToDesigner`

**Pattern:**
```xml
<upa:TaskNode x:Name="__ReferenceID50" DisplayName="ManagerApprovalFlow" 
  sap:VirtualizedContainerService.HintSize="120,80" 
  sap2010:WorkflowViewState.IdRef="TaskNode_20">  <!-- REQUIRED -->
```

**IdRef Naming Convention:**
- TaskNodes: `TaskNode_1`, `TaskNode_2`, etc.
- DecisionNodes: `DecisionNode_1`, `DecisionNode_2`, etc.
- EventNodes: `EventNode_1`, `EventNode_2`, etc.
- EndNodes: `EndNode_1`, `EndNode_2`, etc.
- InvokeWorkflowFile: `InvokeWorkflowFile_1`, etc.

### 24. x:Reference Declarations - CRITICAL STRUCTURE RULES
**Error**: `'Nodes' property has already been set on 'ProcessDiagram'`

This error occurs when nodes are defined BOTH inline AND as separate elements. The correct pattern is:

1. **ALL nodes must be defined INLINE** (nested within EventNode.Next → TaskNode.Next chains)
2. **x:Reference elements at the end** only REFERENCE nodes that were already defined inline - they do NOT define new nodes
3. **The StartNode uses x:Reference** to point to the first inline node

**WRONG - Defining nodes separately:**
```xml
<upa:ProcessDiagram>
  <upa:ProcessDiagram.StartNode>
    <x:Reference>__ReferenceID0</x:Reference>
  </upa:ProcessDiagram.StartNode>
  
  <!-- WRONG: Defining EventNode as separate element -->
  <upa:EventNode x:Name="__ReferenceID0" DisplayName="Start">
    ...
  </upa:EventNode>
  
  <!-- WRONG: Defining EndNode as separate element -->
  <upa:EndNode x:Name="__ReferenceID9" DisplayName="End">
    ...
  </upa:EndNode>
  
  <x:Reference>__ReferenceID0</x:Reference>
  <x:Reference>__ReferenceID9</x:Reference>
</upa:ProcessDiagram>
```

**RIGHT - All nodes inline, x:Reference only references:**
```xml
<upa:ProcessDiagram>
  <upa:ProcessDiagram.StartNode>
    <x:Reference>__ReferenceID0</x:Reference>
  </upa:ProcessDiagram.StartNode>
  
  <!-- All nodes defined INLINE via Next chains -->
  <upa:EventNode x:Name="__ReferenceID0" DisplayName="Start">
    <upa:EventNode.Next>
      <upa:TaskNode x:Name="__ReferenceID1" DisplayName="Task 1">
        <upa:TaskNode.Next>
          <upa:EndNode x:Name="__ReferenceID2" DisplayName="End">
            ...
          </upa:EndNode>
        </upa:TaskNode.Next>
      </upa:TaskNode>
    </upa:EventNode.Next>
  </upa:EventNode>
  
  <!-- x:Reference only REFERENCES inline nodes - does NOT define them -->
  <x:Reference>__ReferenceID1</x:Reference>
  <x:Reference>__ReferenceID2</x:Reference>
</upa:ProcessDiagram>
```

**Shared EndNode Pattern (for Decision branches):**
```xml
<upa:DecisionNode x:Name="__ReferenceID5">
  <upa:DecisionNode.True>
    <upa:TaskNode x:Name="__ReferenceID6">
      <upa:TaskNode.Next>
        <!-- EndNode defined inline in True branch -->
        <upa:EndNode x:Name="__ReferenceID8" DisplayName="End">
          ...
        </upa:EndNode>
      </upa:TaskNode.Next>
    </upa:TaskNode>
  </upa:DecisionNode.True>
  <upa:DecisionNode.False>
    <upa:TaskNode x:Name="__ReferenceID7">
      <upa:TaskNode.Next>
        <!-- Reference to same EndNode - NOT a new definition -->
        <x:Reference>__ReferenceID8</x:Reference>
      </upa:TaskNode.Next>
    </upa:TaskNode>
  </upa:DecisionNode.False>
</upa:DecisionNode>
```

### 25. ViewState ShapeLocation and ConnectorLocation
**CRITICAL**: For nodes to render correctly in the designer, they need proper ViewState with coordinates.

**Required ViewState elements:**
```xml
<sap:WorkflowViewStateService.ViewState>
  <scg:Dictionary x:TypeArguments="x:String, x:Object">
    <x:Boolean x:Key="IsExpanded">True</x:Boolean>
    <av:Point x:Key="ShapeLocation">1650,610</av:Point>  <!-- X,Y position -->
    <av:Size x:Key="ShapeSize">120,80</av:Size>          <!-- Width,Height -->
    <av:PointCollection x:Key="ConnectorLocation">1770,650 1810,650</av:PointCollection>  <!-- Line path to next node -->
  </scg:Dictionary>
</sap:WorkflowViewStateService.ViewState>
```

**DecisionNode ViewState (includes True/False connectors):**
```xml
<sap:WorkflowViewStateService.ViewState>
  <scg:Dictionary x:TypeArguments="x:String, x:Object">
    <x:Boolean x:Key="IsExpanded">True</x:Boolean>
    <av:Point x:Key="ShapeLocation">1390,200</av:Point>
    <av:Size x:Key="ShapeSize">60,60</av:Size>
    <av:PointCollection x:Key="TrueConnector">1450,230 1480,230</av:PointCollection>
    <av:PointCollection x:Key="FalseConnector">1420,260 1420,580 1330,580 1330,610</av:PointCollection>
  </scg:Dictionary>
</sap:WorkflowViewStateService.ViewState>
```

### 26. Shared Rejection Handler via x:Reference
**PATTERN**: Use `x:Reference` to point multiple rejection paths to a single DeleteTransaction TaskNode.

**Production Pattern from SALES02:**
```xml
<!-- Define DeleteTransaction once -->
<upa:TaskNode x:Name="__ReferenceID33" DisplayName="DeleteTransaction">
  <ui:InvokeWorkflowFile WorkflowFileName="Service\DeleteTransaction.xaml">
    <ui:InvokeWorkflowFile.Arguments>
      <InArgument x:TypeArguments="x:String" x:Key="in_RejectionStage" />
      <InArgument x:TypeArguments="x:String" x:Key="in_RejectorName" />
      <!-- ... other args ... -->
    </ui:InvokeWorkflowFile.Arguments>
  </ui:InvokeWorkflowFile>
  <upa:TaskNode.Next>
    <upa:EndNode x:Name="__ReferenceID35" DisplayName="End" />
  </upa:TaskNode.Next>
</upa:TaskNode>

<!-- Reference from multiple DecisionNode.False branches -->
<upa:DecisionNode x:Name="__ReferenceID45" Condition="[decision_SalesRep = &quot;Approved&quot;]">
  <upa:DecisionNode.True>...</upa:DecisionNode.True>
  <upa:DecisionNode.False>
    <x:Reference>__ReferenceID33</x:Reference>  <!-- Points to shared handler -->
  </upa:DecisionNode.False>
</upa:DecisionNode>

<upa:DecisionNode x:Name="__ReferenceID49" Condition="[decision_RevOps = &quot;Approved&quot;]">
  <upa:DecisionNode.True>...</upa:DecisionNode.True>
  <upa:DecisionNode.False>
    <x:Reference>__ReferenceID33</x:Reference>  <!-- Same shared handler -->
  </upa:DecisionNode.False>
</upa:DecisionNode>
```

## Complete Multi-Stage Approval Chain Pattern (PRODUCTION VALIDATED)

For workflows with multiple approval stages (e.g., SalesRep → RevOps → Manager(s) → CRO → Finance), follow this structure based on the production SALES02 implementation:

```
Main ProcessDiagram
├── EventNode (Start - Manual Trigger)
│     └── TaskNode (InitAllSettings - loads Config.json)
│           └── TaskNode (SF-GetQuote - Salesforce data extraction)
│                 └── TaskNode (Add to Queue)
│                       └── TaskNode (Subprocess - GetManagerHierarchy)
│                             └── TaskNode (SalesRepApprovalFlow)
│                                   └── TaskNode (Subprocess - Run Webhook SalesRep Approval)
│                                         └── TaskNode (EvaluateBusinessRule - Policy Check)
│                                               └── DecisionNode (decision_SalesRep = "Approved"?)
│                                                     ├── True: TaskNode (RevOpsApprovalFlow)
│                                                     │         └── TaskNode (Subprocess - Run Webhook RevOps Approval)
│                                                     │               └── DecisionNode (decision_RevOps = "Approved"?)
│                                                     │                     ├── True: TaskNode (SubProcess - Run ManagerLoopApproval)
│                                                     │                     │         └── DecisionNode (Manager Chain Result)
│                                                     │                     │               ├── True: TaskNode (CRO_ApprovalFlow)
│                                                     │                     │               │         └── DecisionNode (CRO Decision)
│                                                     │                     │               │               ├── True: TaskNode (Set Transaction Status) → EndNode
│                                                     │                     │               │               └── False: x:Reference → DeleteTransaction
│                                                     │                     │               └── False: x:Reference → DeleteTransaction
│                                                     │                     └── False: x:Reference → DeleteTransaction
│                                                     └── False: x:Reference → DeleteTransaction
│
├── TaskNode (DeleteTransaction - SHARED via x:Reference)
│     └── EndNode (End)
│
└── TaskNode (Set Transaction Status - marks queue item successful)
      └── x:Reference → EndNode
```

### Key Implementation Notes (from Production):

1. **Each approval stage follows the same pattern:**
   - TaskNode with `NodeType="Task.Service"` (calls ApprovalFlow_*.xaml to send request)
   - TaskNode with `NodeType="Task.Subprocess"` (embedded ProcessDiagram with ConnectorPersistenceActivity)
   - DecisionNode with `Condition="[decision_* = &quot;Approved&quot;]"`

2. **Rejection handling uses SHARED DeleteTransaction via x:Reference:**
   ```xml
   <!-- Define once -->
   <upa:TaskNode x:Name="__ReferenceID33" DisplayName="DeleteTransaction">
     <ui:InvokeWorkflowFile WorkflowFileName="Service\DeleteTransaction.xaml">
       <ui:InvokeWorkflowFile.Arguments>
         <InArgument x:TypeArguments="x:String" x:Key="in_RejectionStage" />
         <InArgument x:TypeArguments="x:String" x:Key="in_RejectorName" />
         <InArgument x:TypeArguments="x:String" x:Key="in_RejectorEmail" />
         <InArgument x:TypeArguments="x:String" x:Key="in_RejectionComments" />
       </ui:InvokeWorkflowFile.Arguments>
     </ui:InvokeWorkflowFile>
     <upa:TaskNode.Next>
       <upa:EndNode x:Name="__ReferenceID35" DisplayName="End" />
     </upa:TaskNode.Next>
   </upa:TaskNode>
   
   <!-- Reference from all rejection paths -->
   <upa:DecisionNode.False>
     <x:Reference>__ReferenceID33</x:Reference>
   </upa:DecisionNode.False>
   ```

3. **Manager loop is a Subprocess with InterruptibleWhile:**
   - Contains local variables (managersArray, currentIndex, priorApprovals, loopComplete, wasRejected)
   - Uses `InterruptibleWhile` activity for persistence compatibility
   - Flowchart inside loop body handles webhook wait
   - Tracks prior approvals as JArray for audit trail

4. **Subprocess webhook pattern (consistent across all stages):**
   ```
   Subprocess ProcessDiagram
     └── EventNode (Start)
           └── EventNode (ConnectorPersistenceActivity - Wait for HTTP Webhook)
                 └── TaskNode (Get Trigger Event Output - ConnectorActivity)
                       └── TaskNode (Log Message)
                             └── TaskNode (Deserialize - InvokeWorkflowFile)
                                   └── TaskNode (Sequence - Assign outputs)
                                         └── EndNode
   ```

5. **x:Reference declarations at end of ProcessDiagram:**
   ```xml
   <upa:ProcessDiagram>
     <!-- All node definitions -->
     
     <!-- REQUIRED: x:Reference for every x:Name -->
     <x:Reference>__ReferenceID33</x:Reference>
     <x:Reference>__ReferenceID34</x:Reference>
     <x:Reference>__ReferenceID35</x:Reference>
     <!-- ... all other references ... -->
   </upa:ProcessDiagram>
   ```

6. **ViewState coordinates for proper designer rendering:**
   - Each node needs `ShapeLocation` (X,Y position)
   - Each node needs `ShapeSize` (width,height - typically 120,80 for TaskNodes, 60,60 for DecisionNodes, 40,40 for EventNodes/EndNodes)
   - Connections need `ConnectorLocation` (PointCollection defining the line path)
   - DecisionNodes need `TrueConnector` and `FalseConnector` separately

## TextExpression Namespaces and References (CRITICAL)

The `TextExpression.NamespacesForImplementation` and `TextExpression.ReferencesForImplementation` sections are REQUIRED for VB expressions to compile. Missing entries cause "ErrorActivity" or expression evaluation failures.

### Key Namespaces Required for Approval Workflows

```xml
<TextExpression.NamespacesForImplementation>
  <sco:Collection x:TypeArguments="x:String">
    <!-- Core namespaces -->
    <x:String>System</x:String>
    <x:String>System.Collections.Generic</x:String>
    <x:String>System.Linq</x:String>
    
    <!-- UiPath namespaces -->
    <x:String>UiPath.Core</x:String>
    <x:String>UiPath.Core.Activities</x:String>
    <x:String>UiPath.Process.Activities</x:String>
    <x:String>UiPath.Process.Activities.Shared</x:String>
    <x:String>UiPath.Persistence.Activities.Queue</x:String>
    
    <!-- JSON handling (CRITICAL for webhook responses) -->
    <x:String>Newtonsoft.Json.Linq</x:String>
    <x:String>Newtonsoft.Json</x:String>
    
    <!-- Integration Service (webhook activities) -->
    <x:String>UiPath.IntegrationService.Activities.Runtime.Models</x:String>
    <x:String>UiPath.IntegrationService.Activities.SWEntities.C692BE26E04_HTTP_WEBHOOK_Trigger.Bundle</x:String>
    <x:String>UiPath.IntegrationService.Activities.SWEntities.C692BE26E04_GetTriggerEventObject_Retrieve.Bundle</x:String>
    
    <!-- Additional required -->
    <x:String>System.Dynamic</x:String>
    <x:String>System.Collections.Specialized</x:String>
    <x:String>System.Runtime.Serialization</x:String>
  </sco:Collection>
</TextExpression.NamespacesForImplementation>
```

### Key Assembly References

```xml
<TextExpression.ReferencesForImplementation>
  <sco:Collection x:TypeArguments="AssemblyReference">
    <!-- Core -->
    <AssemblyReference>System.Private.CoreLib</AssemblyReference>
    <AssemblyReference>System.Linq</AssemblyReference>
    <AssemblyReference>System.Collections</AssemblyReference>
    
    <!-- JSON (CRITICAL) -->
    <AssemblyReference>Newtonsoft.Json</AssemblyReference>
    
    <!-- UiPath -->
    <AssemblyReference>UiPath.System.Activities</AssemblyReference>
    <AssemblyReference>UiPath.Process.Activities</AssemblyReference>
    <AssemblyReference>UiPath.Persistence.Activities</AssemblyReference>
    <AssemblyReference>UiPath.IntegrationService.Activities.Runtime</AssemblyReference>
    
    <!-- Integration Service connector assemblies (copy from working workflow) -->
    <AssemblyReference>C692BE26E04_HTTP_WEB.tBAt61mTt9C124J7Z4qb5as1</AssemblyReference>
    <AssemblyReference>C692BE26E04_GetTrigg.OiA5RWVqVJ1iWTu52m8t1c2</AssemblyReference>
  </sco:Collection>
</TextExpression.ReferencesForImplementation>
```

**IMPORTANT**: The Integration Service assembly names (like `C692BE26E04_HTTP_WEB...`) are dynamically generated. Always copy these from a working workflow in your project rather than making them up.

---

## Reference Documentation

**Path convention:** This skill folder is `uipath-longrunning-workflow/` under `.claude/skills/` (spec-kit-ui **or** `%USERPROFILE%\.cursor\skills` when using a **directory junction**). Links below use paths **relative to `SKILL.md`**: `references/…` = same skill folder; `../_shared/…` = shared docs for all UiPath skills.

### Production Reference
- **SALES02 Production Project**: `C:\UiPath\SALES02_RenewalPriceCommitment` - The production-validated long-running workflow implementation
- **Main-Queue.xaml**: Complete multi-stage approval chain with webhook subprocesses
- **ApprovalFlows/**: Individual approval workflows with HITL Platform and Adaptive Card integration

### Template and production guidance
- **Template**: https://github.com/cato-networks-IT/PROCESSNAME_LongRunningAutomationTemplate
- **Production guide**: [references/production-template-guide.md](./references/production-template-guide.md) – common failure points, exact activity tags, project.json and Config patterns

### Persistence and events
- **Persistence activities**: [references/persistence-activities.md](./references/persistence-activities.md) – Create Form/App Task, Wait … And Resume, Resume After Delay, Queue/Job, webhook pattern
- **Events**: [references/events.md](./references/events.md) – Resume after Delay, Wait for Message (and webhook)
- **State serialization**: [references/state-serialization.md](./references/state-serialization.md) – serializable types, scope rules

### HITL Platform
- **Integration & API (shared)**: [../_shared/hitl-platform.md](../_shared/hitl-platform.md) – base URLs, auth, approvals payload, Adaptive Card HTTPS rules
- **Extended endpoints (skill)**: [references/hitl-platform-api.md](./references/hitl-platform-api.md) – detailed endpoint list (links back to shared doc)
- **Adaptive Cards**: [references/adaptive-card-templates.md](./references/adaptive-card-templates.md)

### Additional skill references (optional depth)
- [references/webhook-patterns.md](./references/webhook-patterns.md), [references/action-center-integration.md](./references/action-center-integration.md), [references/workflow-templates.md](./references/workflow-templates.md), [references/production-xaml-template.md](./references/production-xaml-template.md)

### UiPath (canonical links)
- [Long Running Workflows (Studio, latest)](https://docs.uipath.com/studio/standalone/latest/user-guide/long-running-workflows)
- [Flowchart Builder Activities (latest)](https://docs.uipath.com/activities/other/latest/workflow/flowchart-builder-activities) — Manual Trigger, Sequence, Gateways, Events, End, per-activity pages

---

## Troubleshooting Guide

### Error: "'Nodes' property has already been set on 'ProcessDiagram'"

**Cause**: Nodes are defined BOTH inline (nested in Next chains) AND as separate elements in the ProcessDiagram.

**Solution**: ALL nodes must be defined INLINE via EventNode.Next → TaskNode.Next chains. The `x:Reference` elements at the end only REFERENCE nodes - they do NOT define new nodes.

**Wrong:**
```xml
<upa:ProcessDiagram>
  <upa:EventNode x:Name="__ReferenceID0">
    <upa:EventNode.Next>...</upa:EventNode.Next>
  </upa:EventNode>
  <!-- WRONG: Separate node definition -->
  <upa:EndNode x:Name="__ReferenceID9" DisplayName="End" />
  <x:Reference>__ReferenceID0</x:Reference>
</upa:ProcessDiagram>
```

**Right:**
```xml
<upa:ProcessDiagram>
  <upa:EventNode x:Name="__ReferenceID0">
    <upa:EventNode.Next>
      <upa:TaskNode x:Name="__ReferenceID1">
        <upa:TaskNode.Next>
          <!-- EndNode defined INLINE in the chain -->
          <upa:EndNode x:Name="__ReferenceID2" />
        </upa:TaskNode.Next>
      </upa:TaskNode>
    </upa:EventNode.Next>
  </upa:EventNode>
  <!-- x:Reference only REFERENCES existing inline nodes -->
  <x:Reference>__ReferenceID1</x:Reference>
  <x:Reference>__ReferenceID2</x:Reference>
</upa:ProcessDiagram>
```

### Error: "'Next' property has already been set on 'TaskNode'"

**Cause**: A TaskNode has two `<upa:TaskNode.Next>` elements.

**Solution**: Each TaskNode can only have ONE `<upa:TaskNode.Next>`. Chain additional nodes through DecisionNode branches or sequential TaskNode.Next nesting.

**Wrong:**
```xml
<upa:TaskNode>
  <upa:TaskNode.Next>...</upa:TaskNode.Next>
  <upa:TaskNode.Next>...</upa:TaskNode.Next>  <!-- ERROR -->
</upa:TaskNode>
```

**Right:**
```xml
<upa:TaskNode>
  <upa:TaskNode.Next>
    <upa:DecisionNode>
      <upa:DecisionNode.True>...</upa:DecisionNode.True>
      <upa:DecisionNode.False>...</upa:DecisionNode.False>
    </upa:DecisionNode>
  </upa:TaskNode.Next>
</upa:TaskNode>
```

### Error: "Value cannot be null" in FlowchartDesigner

**Cause**: Missing `sap2010:WorkflowViewState.IdRef` attribute or missing `x:Reference` declaration.

**Solution**:
1. Ensure every node has `sap2010:WorkflowViewState.IdRef="NodeType_N"` attribute
2. Ensure every `x:Name="__ReferenceIDN"` has a corresponding `<x:Reference>__ReferenceIDN</x:Reference>` at the end of the ProcessDiagram

**Check:**
```powershell
# Find all x:Name declarations
Select-String -Path "Main-Queue.xaml" -Pattern 'x:Name="__ReferenceID\d+"' -AllMatches

# Find all x:Reference declarations  
Select-String -Path "Main-Queue.xaml" -Pattern '<x:Reference>__ReferenceID\d+</x:Reference>' -AllMatches

# Compare counts - they should match
```

### Error: "start tag does not match end tag"

**Cause**: Unclosed or mismatched XML tags, often in deeply nested TaskNode chains.

**Solution**:
1. Validate XML structure: `[xml](Get-Content "Main-Queue.xaml" -Raw)`
2. Check that every `<upa:TaskNode>` has a closing `</upa:TaskNode>`
3. Check that every `<upa:TaskNode.Next>` has a closing `</upa:TaskNode.Next>`

**Validation Script:**
```powershell
try {
    [xml]$xml = Get-Content "C:\UiPath\PROJECT\Main-Queue.xaml" -Raw
    Write-Host "XML is valid"
} catch {
    Write-Host "XML Error: $($_.Exception.Message)"
}
```

### Error: Workflow opens but nodes don't render correctly

**Cause**: Missing or incorrect ViewState coordinates.

**Solution**: Ensure each node has proper ViewState with ShapeLocation and ShapeSize:

```xml
<sap:WorkflowViewStateService.ViewState>
  <scg:Dictionary x:TypeArguments="x:String, x:Object">
    <x:Boolean x:Key="IsExpanded">True</x:Boolean>
    <av:Point x:Key="ShapeLocation">1650,610</av:Point>
    <av:Size x:Key="ShapeSize">120,80</av:Size>
    <av:PointCollection x:Key="ConnectorLocation">1770,650 1810,650</av:PointCollection>
  </scg:Dictionary>
</sap:WorkflowViewStateService.ViewState>
```

### Error: ErrorActivity for Integration Service activities

**Cause**: Missing or incorrect namespace/assembly references for Integration Service connectors.

**Solution**:
1. Copy the exact `xmlns:uiascb*` declarations from a working workflow
2. Copy the exact assembly references from `TextExpression.ReferencesForImplementation`
3. The assembly names are dynamically generated - don't make them up

### Best Practice: Backup Before Modifications

Always create a backup before making structural changes:

```powershell
Copy-Item "C:\UiPath\PROJECT\Main-Queue.xaml" "C:\UiPath\PROJECT\Main-Queue-backup.xaml"
```

### Best Practice: Incremental Changes

When adding new approval stages:
1. Add one stage at a time
2. Validate XML after each addition
3. Open in UiPath Studio to verify
4. Only proceed to next stage after current one works
