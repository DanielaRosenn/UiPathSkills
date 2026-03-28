---
name: uipath-workflow-planning
tags: [uipath-dev, planning, canvas, sdd]
description: Plan UiPath workflow implementation from canvas/SDD — MANDATORY before any UiPath codegen. Workflow-type agnostic. Run first, then route to uipath-automation, uipath-longrunning-workflow, uipath-bpmn-maestro, coded-workflow-builder, or other target skill. Triggers on "uipath-workflow-planning", "generate XAML from canvas", "build UiPath from canvas", "workflow plan", "solution flow", or when canvas/SDD is present.
---

# UiPath Workflow Planning (Canvas to Code Bridge)

**MANDATORY STEP** before generating **any** UiPath-related implementation (Studio XAML, coded automation, or Maestro-aligned specs). This skill extracts and validates requirements from canvas or SDD; it does **not** assume a specific project template — the downstream skill depends on the automation type.

## When to Use

This skill is **AUTOMATICALLY TRIGGERED** before codegen skills such as:
- `uipath-automation`
- `uipath-longrunning-workflow` (Studio Long Running Automation / ProcessDiagram)
- `coded-workflow-builder`
- Any workflow where canvas/SDD defines arguments and flow (including inputs for `uipath-bpmn-maestro` when translating model to delivery)

**NEVER** generate UiPath implementation without first completing this planning step when a canvas or structured SDD exists (or obtain an equivalent written execution plan from the user).

## Purpose

Prevents the common error of generating workflows with incorrect or missing arguments by:
1. Extracting ALL data fields from canvas nodes
2. Mapping data flow between workflows
3. Identifying visibility rules (LIMITED vs FULL VIEW)
4. Validating argument types and sources
5. Creating a workflow execution plan

## Core Workflow

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  1. LOCATE      │────>│  2. EXTRACT     │────>│  3. MAP DATA    │
│  Canvas File    │     │  Node Details   │     │  Flow           │
└─────────────────┘     └─────────────────┘     └─────────────────┘
                                                        │
                                                        ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  6. GENERATE    │<────│  5. CREATE      │<────│  4. IDENTIFY    │
│  Code           │     │  Execution Plan │     │  Visibility     │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Step 1: Locate Canvas File

**ALWAYS** search for canvas/flow documentation first:

```
project/
├── docs/
│   ├── solution-flow.json      # Primary canvas file
│   ├── business-flow.json      # Business process flow
│   └── technical-flow.json     # Technical implementation
├── canvas/
│   └── *.json                  # Alternative location
└── *.canvas.json               # Root level canvas
```

Search patterns:
```bash
# Find canvas files
**/solution-flow*.json
**/*canvas*.json
**/*flow*.json
```

## Step 2: Extract Node Details

For each node in the canvas, extract:

### Form/Approval Nodes (type: "form" or "human")

```json
{
  "id": "n8",
  "label": "Sales Rep Form",
  "drillDown": {
    "type": "form",
    "title": "Sales Rep Approval Form (LIMITED VIEW)",
    "visibilityLevel": "LIMITED",
    "hiddenFields": ["profitabilityMargin", "managerChain"],
    "fields": [
      {"name": "accountName", "label": "Customer Account", "source": "SF Connector", "editable": false},
      {"name": "maxCapPercent", "label": "Maximum Cap %", "source": "Sales Rep Input", "editable": true}
    ],
    "inputArguments": [
      {"name": "in_ProcessId", "type": "String", "source": "Queue Item (QuoteId)"},
      {"name": "in_CustomerName", "type": "String", "source": "Queue Item"}
    ],
    "outputArguments": [
      {"name": "out_ApprovalId", "type": "String"},
      {"name": "out_MaxCapPercent", "type": "Decimal"}
    ]
  }
}
```

### API/Service Nodes (type: "api")

```json
{
  "id": "n5",
  "label": "Bob HR API",
  "drillDown": {
    "type": "api",
    "method": "POST",
    "endpoint": "https://api.hibob.com/v1/people/search",
    "outputFields": ["bobEmployeeId", "name", "email", "title", "isCRO"]
  }
}
```

### DMN/Decision Nodes (type: "dmn")

```json
{
  "id": "n7",
  "label": "Policy Check",
  "drillDown": {
    "type": "dmn",
    "inputs": [
      {"name": "quoteTerm", "type": "number"},
      {"name": "maxCapPercent", "type": "number"},
      {"name": "cpiInclusion", "type": "boolean"},
      {"name": "acv", "type": "number"}
    ],
    "outputs": [
      {"name": "withinPolicy", "type": "boolean"},
      {"name": "requiresFinance", "type": "boolean"}
    ]
  }
}
```

#### DMN Output Guidelines

**Simplified outputs**: Use boolean outputs for easier consumption:
- `withinPolicy` (boolean): `true` if request meets policy, `false` otherwise
- `requiresFinance` (boolean): `true` if Finance approval needed

#### DMN Boolean Guidelines

**CRITICAL**: Boolean values in DMN must use `true`/`false`, never `"Yes"`/`"No"`:

| Context | Correct | Incorrect |
|---------|---------|-----------|
| DMN XML `<inputEntry>` | `<text>true</text>` | `<text>Yes</text>` |
| DMN XML `<outputEntry>` | `<text>false</text>` | `<text>No</text>` |
| Canvas JSON rules | `"withinPolicy": true` | `"withinPolicy": "true"` |
| Form field defaults | `"Default: true"` | `"Default: Yes"` |

### Queue Nodes (type: "queue")

```json
{
  "id": "n3",
  "label": "Queue: RenewalPriceCommitment",
  "drillDown": {
    "type": "queue",
    "dataUploaded": {
      "SpecificContent": {
        "quoteId": "{from SF}",
        "accountName": "{from SF}",
        "salesRepEmail": "{from SF}",
        "acv": "{from SF}"
      }
    }
  }
}
```

## Step 3: Map Data Flow

Create a data flow matrix showing how data moves between nodes:

| Data Field | Source Node | Target Nodes | Type | Notes |
|------------|-------------|--------------|------|-------|
| quoteId | n3 (Queue) | n8, n10, n13, n15, n17 | String | Reference ID |
| accountName | n2 (SF) | All approval forms | String | Customer name |
| profitabilityMargin | n2 (SF) | n10, n13, n15, n17 | Decimal | HIDDEN from n8 |
| managerChainJSON | n5 (Bob API) | n10, n13, n15 | String | JSON array |
| policyStatus | n7 (DMN) | n10, n13, n15, n17 | String | "WithinPolicy" or "OutOfPolicy" |
| maxCapPercent | n8 (Sales Rep) | n7, n10, n13, n15, n17 | Decimal | User input |

## Step 4: Identify Visibility Rules

### Visibility Levels

| Level | Description | Hidden Fields |
|-------|-------------|---------------|
| LIMITED | Sales Rep view | profitabilityMargin, managerChain, priorApprovals |
| FULL | Manager/RevOps view | None - sees everything |
| FULL + CHAIN EDITOR | RevOps only | None + can edit managerChain |
| FULL + POLICY | Finance only | None + sees policyViolationReason |

### Per-Workflow Visibility Matrix

| Workflow | Visibility | Sees Profitability | Sees Chain | Edits Chain | Sees Prior Approvals |
|----------|------------|-------------------|------------|-------------|---------------------|
| ApprovalFlow_SalesRep | LIMITED | NO | NO | NO | NO |
| ApprovalFlow_RevOps | FULL + CHAIN EDITOR | YES | YES | YES | NO |
| ApprovalFlow_Manager | FULL | YES | YES | NO | YES |
| ApprovalFlow_CRO | FULL | YES | YES | NO | YES |
| ApprovalFlow_Finance | FULL + POLICY | YES | YES | NO | YES |

## Step 5: Create Execution Plan

Generate a structured execution plan. Name the **main entry workflow** and **project type** for the target automation (examples: `Main.xaml` for REFramework, `Main-Queue.xaml` for Long Running Automation, attended entry workflow, etc.).

```markdown
## Workflow Execution Plan

### Example — Long Running Automation (Studio)

### Main entry: Main-Queue.xaml
Type: Long-Running ProcessDiagram (example)
Entry: Queue trigger (example)

### Execution Sequence:
1. InitAllSettingsJSON.xaml
   - Input: None
   - Output: Config (Dictionary)

2. getTransaction.xaml
   - Input: Config
   - Output: TransactionItem (QueueItem)

3. GetManagerHierarchy.xaml (Services/)
   - Input: in_SalesRepEmail (from Queue), in_BobAPIUrl, in_BobAPICredentials
   - Output: out_ManagerChainJSON, out_CROEmail, out_CROName, out_ManagerCount

4. ApprovalFlow_SalesRep.xaml (Workflows/)
   - Visibility: LIMITED
   - Input: in_ProcessId, in_Title, in_CustomerName, in_QuoteId, in_ACV, in_TCV, in_QuoteTerm, in_Config
   - Output: out_ApprovalId, out_ResponseToken, out_MaxCapPercent, out_CpiInclusion, out_BusinessJustification

5. PolicyCheck.xaml (Services/)
   - Input: in_QuoteTerm (Int32), in_MaxCapPercent (Decimal), in_CpiInclusion (Boolean), in_ACV (Decimal)
   - Output: out_PolicyStatus, out_RequiresFinance, out_PolicyViolationReason

6. ApprovalFlow_RevOps.xaml (Workflows/)
   - Visibility: FULL + CHAIN EDITOR
   - Input: All from SalesRep + in_ProfitabilityMargin, in_ManagerChainJSON, in_PolicyStatus
   - Output: out_ApprovalId, out_ResponseToken, out_ModifiedManagerChainJSON, out_RevOpsComments

... (continue for all workflows)
```

## Step 6: Argument Specification

For each workflow, generate complete argument specifications:

```markdown
### ApprovalFlow_SalesRep.xaml

#### Input Arguments
| Name | Type | Source | Required |
|------|------|--------|----------|
| in_ProcessId | String | TransactionItem.Reference | Yes |
| in_Title | String | "Sales Rep Approval - Renewal Price Commitment" | Yes |
| in_NotificationChannel | String | Config("NotificationChannel") | Yes |
| in_ApproverEmail | String | TransactionItem.SpecificContent("salesRepEmail") | Yes |
| in_CustomerName | String | TransactionItem.SpecificContent("accountName") | Yes |
| in_QuoteId | String | TransactionItem.SpecificContent("quoteId") | Yes |
| in_ACV | String | TransactionItem.SpecificContent("acv") | Yes |
| in_TCV | String | TransactionItem.SpecificContent("tcv") | Yes |
| in_QuoteTerm | Int32 | CInt(TransactionItem.SpecificContent("quoteTerm")) | Yes |
| in_CallBackURL | String | Config("CallbackWebhookURL") | Yes |
| in_Config | Dictionary(String, Object) | Config | Yes |

#### Output Arguments
| Name | Type | Description |
|------|------|-------------|
| out_ApprovalId | String | HITL Platform approval ID |
| out_ResponseToken | String | Token for webhook callback |
| out_MaxCapPercent | Decimal | User-entered max cap percentage |
| out_CpiInclusion | Boolean | User selection for CPI |
| out_BusinessJustification | String | User-entered justification |
| out_RenewalCommitmentTerm | Int32 | User-entered renewal term |
```

## Output: Planning Document

Generate a `workflow-plan.md` file in the project docs folder:

```markdown
# Workflow Implementation Plan

Generated: {timestamp}
Canvas Source: project/docs/solution-flow.json

## 1. Data Flow Summary
[Data flow matrix]

## 2. Visibility Rules
[Visibility matrix]

## 3. Workflow Specifications

### 3.1 ApprovalFlow_SalesRep
[Complete argument specification]

### 3.2 ApprovalFlow_RevOps
[Complete argument specification]

... (all workflows)

## 4. Execution Sequence
[Ordered list with dependencies]

## 5. Validation Checklist
- [ ] All canvas fields mapped to arguments
- [ ] Visibility rules applied correctly
- [ ] Data types match canvas specification
- [ ] Output arguments capture all user inputs
- [ ] Prior approvals chain maintained
```

## Integration with Other Skills

Use the same **`workflow-plan.md`** (or equivalent) as the handoff artifact. Adjust steps 3–4 to the **target** automation.

### Before uipath-automation
```
1. Run uipath-workflow-planning skill
2. Generate workflow-plan.md
3. Pass plan to uipath-automation
4. Generate XAML (or project structure) with correct arguments
```

### Before uipath-longrunning-workflow (Studio Long Running Automation)
```
1. Run uipath-workflow-planning skill
2. Extract ProcessDiagram-oriented sequence from canvas (TaskNode chain / approvals)
3. Map gateways, loops, and rejection paths from the plan
4. Generate or extend Main entry workflow (e.g. Main-Queue.xaml) with invokes — only after plan is complete
```

### Before uipath-bpmn-maestro (Maestro / web BPMN)
```
1. Run uipath-workflow-planning skill to normalize process steps, roles, and data from canvas/SDD
2. Pass structured narrative and data contracts to Maestro-oriented work (modeling in Automation Cloud / web modeler)
3. Do not confuse with Studio desktop XAML — implementation in Studio uses uipath-longrunning-workflow or uipath-automation when coding
```

### Before coded-workflow-builder
```
1. Run uipath-workflow-planning skill
2. Extract C# class structure from canvas
3. Map method signatures from arguments
4. Generate .cs files with correct parameters
```

## Quality Gates

**DO NOT proceed to code generation if:**

- [ ] Canvas file not found
- [ ] Missing drillDown data for form nodes
- [ ] Unresolved data sources
- [ ] Visibility rules not defined
- [ ] Argument types not specified

---

## CRITICAL: Canvas-to-Workflow Structure Validation

After planning, validate that the **implemented workflow structure matches the canvas flow** for the **chosen target**. The detailed example below is oriented to **Studio Long Running Automation** (e.g. `Main-Queue.xaml`, gateways, human steps). For **REFramework**, map to transaction states and `Process.xaml`; for **attended**, to the relevant entry workflow; for **Maestro-only** delivery, validate against the process model steps and SLAs instead of XAML filenames.

**Example target (Long Running Automation in Studio):** before generating `Main-Queue.xaml` (or the agreed main LRW file), check alignment as follows.

### Step 1: Extract Canvas Flow Sequence

From `solution-flow.json`, extract the ordered node sequence by following edges:

```
Canvas Nodes (from edges):
n1 (Quote Action) → n2 (Extract SF) → n3 (Queue) → n4 (Get Transaction)
→ n5 (Bob HR API) → n6 (Build Manager Chain) → n7 (Policy Check)
→ n8 (Sales Rep Form) → n9 (Approved?) 
  → YES: n10 (RevOps Form) → n11 (Approved?)
    → YES: n12 (Update Chain) → n13 (Manager Approval) → n14 (More Managers?)
      → YES: loop back to n13
      → NO: n15 (CRO Approval) → n16 (Finance Required?)
        → YES: n17 (Finance Approval) → n18 (Update SF)
        → NO: n18 (Update SF)
    → NO: n21 (Delete Transaction) → n22 (Rejected)
  → NO: n21 (Delete Transaction) → n22 (Rejected)
→ n18 (Update SF) → n19 (Send Confirmation) → n20 (Complete)
```

### Step 2: Map Canvas Nodes to Workflow Elements

| Canvas Node | Type | Workflow Element | Notes |
|-------------|------|------------------|-------|
| n1 (Quote Action) | start-event | External trigger | Not in Main-Queue |
| n2 (Extract SF) | service-task | External (Dispatcher) | Not in Main-Queue |
| n3 (Queue) | orchestrator | GetTransactionItem | Entry point |
| n4 (Get Transaction) | service-task | InvokeWorkflow: getTransaction.xaml | |
| n5 (Bob HR API) | api | InvokeWorkflow: GetManagerHierarchy.xaml | |
| n6 (Build Manager Chain) | service-task | Part of GetManagerHierarchy | |
| n7 (Policy Check) | dmn | InvokeWorkflow: PolicyCheck.xaml | **ORDER MATTERS** |
| n8 (Sales Rep Form) | user-task | InvokeWorkflow: ApprovalFlow_SalesRep.xaml | |
| n9 (Approved?) | gateway | **GatewayNode** or **If condition** | **MISSING** |
| n10 (RevOps Form) | user-task | InvokeWorkflow: ApprovalFlow_RevOps.xaml | |
| n11 (Approved?) | gateway | **GatewayNode** or **If condition** | **MISSING** |
| n12 (Update Chain) | service-task | Assign activity | **MISSING** |
| n13 (Manager Approval) | user-task | InvokeWorkflow: ApprovalFlow_Manager.xaml | **NEEDS LOOP** |
| n14 (More Managers?) | gateway | **GatewayNode** with loop | **MISSING** |
| n15 (CRO Approval) | user-task | InvokeWorkflow: ApprovalFlow_CRO.xaml | |
| n16 (Finance Required?) | gateway | **GatewayNode** or **If condition** | **MISSING** |
| n17 (Finance Approval) | user-task | InvokeWorkflow: ApprovalFlow_Finance.xaml | **CONDITIONAL** |
| n18 (Update SF) | service-task | InvokeWorkflow: UpdateSalesforce.xaml | **MISSING** |
| n19 (Send Confirmation) | service-task | InvokeWorkflow: SendConfirmation.xaml | **MISSING** |
| n20 (Complete) | end-event | EndNode | |
| n21 (Delete Transaction) | service-task | InvokeWorkflow: DeleteTransaction.xaml | **MISSING** |
| n22 (Rejected) | end-event | EndNode (rejection path) | **MISSING** |

### Step 3: Validate Structure Alignment

**CRITICAL CHECKS:**

1. **Node Order**: Verify the sequence matches canvas edges
   - Canvas: Policy Check (n7) → Sales Rep Form (n8)
   - Workflow: Sales Rep Approval → Policy Check ❌ **WRONG ORDER**

2. **Decision Gateways**: Every canvas decision node needs a workflow gateway
   - n9 (Approved?) → **MISSING** in workflow
   - n11 (Approved?) → **MISSING** in workflow
   - n14 (More Managers?) → **MISSING** in workflow
   - n16 (Finance Required?) → **MISSING** in workflow

3. **Loops**: Canvas loops must be implemented
   - n14 → n13 loop → **MISSING** in workflow

4. **Conditional Paths**: Canvas conditional branches must exist
   - Finance approval only if requiresFinance=true → **MISSING** in workflow

5. **Rejection Path**: Canvas rejection flow must be implemented
   - n21 (Delete Transaction) → n22 (Rejected) → **MISSING** in workflow

6. **Post-Approval Steps**: All finalization steps must exist
   - n18 (Update SF) → **MISSING** in workflow
   - n19 (Send Confirmation) → **MISSING** in workflow

### Step 4: Generate Validation Report

```markdown
## Canvas-to-Workflow Validation Report

### Structure Alignment: ❌ FAILED

| Check | Status | Details |
|-------|--------|---------|
| Node Order | ❌ | Policy Check should be BEFORE Sales Rep Form |
| Decision Gateways | ❌ | 4 missing gateways (n9, n11, n14, n16) |
| Manager Loop | ❌ | No loop for multiple managers |
| Finance Conditional | ❌ | Finance always runs, should be conditional |
| Rejection Path | ❌ | No rejection handling |
| Update SF | ❌ | Missing UI automation step |
| Send Confirmation | ❌ | Missing notification step |

### Required Changes:

1. **Reorder nodes**: Move PolicyCheck BEFORE SalesRepApproval
2. **Add GatewayNodes**: After each approval for Approved? decision
3. **Implement Manager Loop**: For Each manager in chain (except CRO)
4. **Add Finance Conditional**: If v_RequiresFinance Then FinanceApproval
5. **Add Rejection Path**: GatewayNode → DeleteTransaction → EndNode(Rejected)
6. **Add UpdateSalesforce**: UI automation workflow
7. **Add SendConfirmation**: Email notification workflow
```

### Step 5: Corrected Workflow Structure

```
Main-Queue.xaml (ProcessDiagram):

Start → InitAllSettings → GetTransaction → GetManagerHierarchy
→ PolicyCheck (BEFORE approvals!)
→ SalesRepApproval → Gateway(Approved?)
  → YES: RevOpsApproval → Gateway(Approved?)
    → YES: UpdateChain → ForEach(Manager in Chain except CRO)
      → ManagerApproval → Gateway(Approved?)
        → YES: Continue loop
        → NO: DeleteTransaction → End(Rejected)
    → After loop: CROApproval → Gateway(Approved?)
      → YES: Gateway(FinanceRequired?)
        → YES: FinanceApproval → Gateway(Approved?)
          → YES: UpdateSalesforce → SendConfirmation → SetStatus → End
          → NO: DeleteTransaction → End(Rejected)
        → NO: UpdateSalesforce → SendConfirmation → SetStatus → End
      → NO: DeleteTransaction → End(Rejected)
    → NO: DeleteTransaction → End(Rejected)
  → NO: DeleteTransaction → End(Rejected)
```

## Common Errors Prevented

| Error | Cause | Prevention |
|-------|-------|------------|
| Wrong argument names | Not reading canvas | Extract from drillDown.inputArguments |
| Missing visibility rules | Hardcoded values | Check drillDown.visibilityLevel |
| Wrong data types | Guessing types | Use canvas field.type specification |
| Missing fields in forms | Incomplete extraction | Extract ALL drillDown.fields |
| Broken data flow | Not mapping sources | Create data flow matrix |

## Template: Planning Checklist

```markdown
## Pre-Generation Checklist

### Canvas Analysis
- [ ] Located solution-flow.json
- [ ] Extracted all node drillDown data
- [ ] Identified all form nodes
- [ ] Identified all API nodes
- [ ] Identified all DMN nodes

### Data Mapping
- [ ] Created data flow matrix
- [ ] Mapped all queue SpecificContent fields
- [ ] Traced data from source to all consumers
- [ ] Identified computed/derived fields

### Visibility Analysis
- [ ] Identified visibility level per workflow
- [ ] Listed hidden fields per visibility level
- [ ] Verified LIMITED view restrictions
- [ ] Verified FULL view inclusions

### Argument Specification
- [ ] All input arguments defined with types
- [ ] All output arguments defined with types
- [ ] Sources specified for each input
- [ ] Destinations specified for each output

### Ready for Code Generation
- [ ] workflow-plan.md generated
- [ ] All checklist items complete
- [ ] No unresolved dependencies
```
