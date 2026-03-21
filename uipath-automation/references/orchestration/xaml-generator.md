# Module: Workflow Design
**Studio 2025.10 | Source: docs.uipath.com/studio**

---

## Workflow Types — When to Use Each

### Sequence
- **Use when**: activities follow each other linearly, single path, no complex branching
- **Best for**: UI automation steps (click → type → read), data processing pipelines, sub-workflows invoked by a parent
- **Rule**: preferred for most workflows — keep simple and focused on one task
- **Anti-pattern**: deeply nested If statements inside Sequences → move to Flowchart

### Flowchart
- **Use when**: multiple decision points, branching logic, If/ElseIf cascades, business rules
- **Best for**: main orchestration logic, process routing, decision-heavy business logic
- **Warning** (ST-DBP-007): avoid multiple nested Flowchart layers — keep to 1-2 levels
- **Anti-pattern**: using Flowcharts for linear steps → use Sequence instead

### State Machine
- **Use when**: process has distinct states with conditional transitions, can move back and forth
- **Best for**: REFramework main logic, order processing, approval workflows with multiple outcomes
- **Structure**: States → Transitions → Entry/Exit actions
- **Rule**: each state should have clear entry and exit conditions

### Long Running Workflow (LRW) — Studio 2025.10
- **Use when**: process spans hours/days, includes human approval steps, needs to suspend/resume
- **Best for**: HITL (Human-in-the-Loop) approvals, multi-day processes, **Studio** Long Running Automation (`upa:ProcessDiagram`). For **Maestro / cloud BPMN** modeling, use **`uipath-bpmn-maestro`**, not this pattern.
- **Key concept**: dedicated LRW canvas in Studio (Flowchart Builder activities); nodes include triggers, tasks, gateways, subprocesses
- **Create**: Design ribbon → New → Long Running Workflow
- **Critical**: variables across Suspend points MUST be serializable (see [long-running.md](long-running.md))
- **Codegen / project contract**: follow **[`uipath-longrunning-workflow`](../../../uipath-longrunning-workflow/SKILL.md)** for the entry `.xaml`, `project.json` packages, and designer troubleshooting — do not invent `ProcessDiagram` structure from this module alone

---

## Project Types

### Process
- Standard automation project
- Entry point: `Main.xaml`
- Can be attended or unattended

### Library
- Reusable activity packages
- Exports workflows as custom activities
- Use for shared business logic across projects

### Test Automation
- Test cases + test suites
- Integrates with UiPath Test Manager

### Template
- Reusable project scaffolds
- Local templates: stored in project's Templates folder
- Published templates: available after library publish

---

## Project Compatibility (2025.10)

| Compatibility | Runtime | Notes |
|---|---|---|
| Windows | .NET 8 | **Default for new projects** — use this |
| Cross-platform | .NET 8 | For Linux robot execution |
| Windows - Legacy | .NET Framework 4.7.2 | Legacy only — no new projects |

**Rule**: Always create Windows compatibility projects unless cross-platform execution is required.

---

## Project Structure Best Practices

```
MyProcess/
├── Main.xaml                    ← orchestrator only, no business logic
├── project.json
├── Data/
│   └── Config.xlsx              ← settings (REFramework)
├── Framework/                   ← REFramework internal workflows
│   ├── InitAllSettings.xaml
│   ├── InitAllApplications.xaml
│   ├── GetTransactionData.xaml
│   ├── SetTransactionStatus.xaml
│   └── CloseAllApplications.xaml
├── Processes/                   ← business logic workflows
│   ├── Process.xaml
│   └── [SubWorkflows].xaml
├── Tests/                       ← test cases
└── .gitignore                   ← built-in from 2025.10
```

**Rules**:
- `Main.xaml` = orchestrator only (calls other workflows, no business logic)
- Business logic lives in `Processes/` sub-workflows
- Framework utilities live in `Framework/`
- Max ~30 workflows per project — split into libraries if larger

---

## Variables

### Declaration
- Scope: declare at the smallest container that needs the variable
- Use Data Manager panel (2025.10 overhauled) for centralized management
- Types: String, Int32, Boolean, Double, DateTime, DataTable, GenericValue, Array, List, Dictionary, IEnumerable, Queue, DataRow, SecureString

### Naming (ST-NMG-001)
- camelCase: `customerName`, `totalAmount`, `isProcessed`
- DataTable prefix: `dt_CustomerData`, `dt_Results` (ST-NMG-009)
- No generic names: ~~`variable1`~~, ~~`temp`~~, ~~`data`~~

### Variable vs Argument
- **Variable**: local to one workflow file
- **Argument**: passes data between workflows (always use for Invoke Workflow File)

---

## Arguments

### Naming (ST-NMG-002)
- `in_` prefix: input arguments — `in_Config`, `in_FilePath`, `in_TransactionItem`
- `out_` prefix: output arguments — `out_Result`, `out_TransactionStatus`
- `io_` prefix: both in and out — `io_DataTable`, `io_TransactionData`

### Direction Rules
- PascalCase after prefix: `in_CustomerName`, `out_ProcessedCount`
- Set default values where appropriate (ST-NMG-012)
- Max ~5-7 arguments per workflow (ST-DBP-002: high argument count warning)

### Creating from JSON Schema (2025.10)
- Data Manager → Variables tab → Create from JSON Schema
- Useful for complex API response types

---

## Display Names

- Rule (ST-NMG-004): no duplicate display names in same workflow
- Make descriptive: "Read Customer Data from Excel" not "Read Range"
- Include context: "Validate Invoice Amount" not "If"
- Max ~40 chars for readability

---

## Annotations

- Right-click any activity → Add Annotation
- Required for: complex business logic, non-obvious decisions, external system interactions
- Dynamic resize in 2025.10 (flowcharts and LRW)

---

## Imports / Namespaces

Common namespaces to add in Imports panel:
```
System
System.IO
System.Data
System.Collections.Generic
System.Text.RegularExpressions
System.Linq
Newtonsoft.Json
Newtonsoft.Json.Linq
```

---

## Autopilot (2025.10)

- Type natural language in Add Activity search bar → double-click "Generate with Autopilot"
- Generates a Sequence preview with suggested activities
- Review and refine before accepting — treat as a scaffold, not final output
- Instructions saved as annotation for future editing

---

## Key Decision: Which Workflow Type?

```
Is the process linear (steps follow each other)?
  YES → Sequence
  NO  → Does it have complex branching / multiple outcomes?
          YES → Flowchart
          NO  → Does it have distinct states and transition conditions?
                  YES → State Machine (or REFramework)
                  NO  → Does it span hours/days or require human approval?
                          YES → Long Running Workflow
                          NO  → Sequence (default)
```
