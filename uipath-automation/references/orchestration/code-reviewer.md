# Module: Code Reviewer
**Covers:** Systematic review — ST-* rules, severity scoring, anti-patterns, review output format

---

## Review Output Format

```
## Code Review: [Workflow/Project Name]
**Standard:** UiPath Studio 2025.10 | **Date:** [today]

### Summary
[2-3 sentences: overall quality and biggest concern]

### Scorecard
| Category           | Score | Critical | Warnings |
|--------------------|-------|----------|----------|
| Naming conventions | /10   |          |          |
| Error handling     | /10   |          |          |
| Security           | /10   |          |          |
| Maintainability    | /10   |          |          |
| Performance        | /10   |          |          |
| Overall            | /10   |          |          |

### 🔴 Critical (fix before production)
**[C-001]** Title | Rule: ST-XXX-NNN | Location: file → activity
- Issue / Impact / Fix

### 🟡 Warnings (should fix)
### 🔵 Suggestions (nice to have)

### Top 3 Priority Actions
1. ...
```

---

## Full Review Checklist

### 1. Project Structure
- [ ] `Main.xaml` = orchestrator only — no business logic
- [ ] Folder structure: Framework/ Workflows/ Utilities/ Data/
- [ ] `GlobalExceptionHandler.xaml` exists and registered in `project.json`
- [ ] `Config.xlsx` in Data/ — no hardcoded values anywhere

### 2. Naming (ST-NMG-*)
- [ ] Arguments prefixed `in_`/`out_`/`io_` → **ST-NMG-002**
- [ ] Variables in camelCase → **ST-NMG-001**
- [ ] No default display names (`Assign 1`, `Sequence`) → **ST-NMG-003**
- [ ] Workflow files = VerbNoun.xaml → **ST-NMG-004**
- [ ] DataTable variables prefixed `dt`, booleans prefixed `is`/`has`

### 3. Error Handling
- [ ] No empty catch blocks → **ST-DBP-003** 🔴
- [ ] `BusinessRuleException` caught separately, NOT retried
- [ ] System exceptions logged AND rethrown
- [ ] `Finally` for resource cleanup (DB, files)
- [ ] Screenshots on UI automation failures

### 4. Security
- [ ] No hardcoded passwords / API keys → **ST-SEC-001** 🔴
- [ ] Credentials from Orchestrator Assets only
- [ ] `Private=True` on sensitive activities → **ST-SEC-002**
- [ ] No PII in log messages

### 5. Maintainability
- [ ] Workflows < 30 activities → **ST-DBP-004**
- [ ] Nesting depth < 4 levels → **ST-DBP-006**
- [ ] No duplicate logic
- [ ] Variables in innermost scope
- [ ] No unused variables → **ST-DBP-001**
- [ ] No unused arguments → **ST-DBP-002**

### 6. Performance
- [ ] No fixed `Delay` for UI waiting — use `WaitElementAppear`
- [ ] No positional `idx` selectors unless unavoidable
- [ ] `SimulateType`/`SimulateClick` for unattended
- [ ] DB filtering in SQL WHERE, not post-load

### 7. REFramework
- [ ] Business logic only in Process.xaml
- [ ] Init and Close are symmetric (same apps)
- [ ] `GetTransactionData` returns `Nothing` when queue empty
- [ ] `MaxRetryNumber` in Config.xlsx
- [ ] Transaction start/end logging

### 8. Long Running Workflow
- [ ] No DataTable/DataRow/MailMessage across Suspend → 🔴
- [ ] `NoPersistScope` wraps UI sessions
- [ ] All `WaitFor*` have Timeout
- [ ] Bookmark names unique per instance (include ID/GUID)

### 9. Coded Automations
- [ ] `[WorkflowArgument]` on all public properties
- [ ] `in_`/`out_`/`io_` naming preserved
- [ ] Services via `GetService<T>()` only
- [ ] No static mutable state

---

## Anti-Pattern Catalog

### 🔴 Empty Catch Block (ST-DBP-003)
```xml
<!-- WRONG: swallows exception silently -->
<Catch x:TypeArguments="s:Exception">
  <ActivityAction x:TypeArguments="s:Exception">
    <ActivityAction.Argument>
      <DelegateInArgument x:TypeArguments="s:Exception" Name="ex"/>
    </ActivityAction.Argument>
    <!-- empty! -->
  </ActivityAction>
</Catch>

<!-- RIGHT -->
<Catch x:TypeArguments="s:Exception">
  <ActivityAction x:TypeArguments="s:Exception">
    <ActivityAction.Argument>
      <DelegateInArgument x:TypeArguments="s:Exception" Name="ex"/>
    </ActivityAction.Argument>
    <Sequence>
      <ui:LogMessage Level="Error" Message="[&quot;Exception: &quot; + ex.Message]"/>
      <Rethrow/>
    </Sequence>
  </ActivityAction>
</Catch>
```

### 🔴 Hardcoded Credential (ST-SEC-001)
```xml
<!-- WRONG -->
<ui:TypeInto Text="&quot;MyPassword123!&quot;" Private="False"/>

<!-- RIGHT -->
<ui:GetCredential AssetName="&quot;AppCredential&quot;"
    Username="[username]" SecurePassword="[securePass]"/>
<ui:TypeInto Private="True"
    Text="[New System.Net.NetworkCredential(String.Empty, securePass).Password]"/>
```

### 🔴 Non-Serializable Type Across LRW Suspend
```xml
<!-- WRONG: DataTable in scope when SuspendWorkflow runs -->
<ui:SuspendWorkflow BookmarkName="[bookmark]"/>

<!-- RIGHT: serialize first -->
<ui:InvokeCode DisplayName="DT to JSON">
  <!-- json = JsonConvert.SerializeObject(dtOrders) -->
</ui:InvokeCode>
<ui:SuspendWorkflow BookmarkName="[bookmark]"/>
```

### 🟡 Fixed Delay Instead of WaitElementAppear
```xml
<!-- WRONG -->
<Delay Duration="[TimeSpan.FromSeconds(3)]"/>
<ui:Click Target.Selector="[btn]"/>

<!-- RIGHT -->
<ui:WaitElementAppear Target.Selector="[btn]" Timeout="[TimeSpan.FromSeconds(15)]"/>
<ui:Click Target.Selector="[btn]"/>
```

### 🟡 Default Display Names (ST-NMG-003)
```xml
<!-- WRONG -->
<Assign DisplayName="Assign"/>
<Sequence DisplayName="Sequence"/>

<!-- RIGHT -->
<Assign DisplayName="Set Invoice Total"/>
<Sequence DisplayName="Process Invoice Line Items"/>
```

### 🟡 Business Logic in Main.xaml
Main.xaml should only contain: state machine transitions, `InvokeWorkflowFile`, and `LogMessage`.
Move all data manipulation, UI interaction, and decision logic to Process.xaml or sub-workflows.

### 🟡 Swallowing BusinessRuleException in Process.xaml
```xml
<!-- WRONG: REF can't mark the transaction as BizEx failed -->
<Catch x:TypeArguments="ui:BusinessRuleException">
  <ActivityAction x:TypeArguments="ui:BusinessRuleException">
    <ActivityAction.Argument>
      <DelegateInArgument x:TypeArguments="ui:BusinessRuleException" Name="ex"/>
    </ActivityAction.Argument>
    <ui:LogMessage Message="[ex.Message]"/>
    <!-- no Rethrow — transaction will be marked Successful! -->
  </ActivityAction>
</Catch>

<!-- RIGHT: always rethrow from Process.xaml -->
<Sequence>
  <ui:LogMessage Message="[ex.Message]"/>
  <Rethrow/>
</Sequence>
```

### 🔵 Variable Scope Too Wide (ST-DBP-006 related)
```xml
<!-- WRONG: tempValue declared at root, only used in inner sequence -->
<Sequence.Variables>
  <Variable x:TypeArguments="x:String" Name="tempValue"/>
</Sequence.Variables>
...
<Sequence DisplayName="Inner Logic">
  <!-- tempValue used here only -->
</Sequence>

<!-- RIGHT: declare in innermost container -->
<Sequence DisplayName="Inner Logic">
  <Sequence.Variables>
    <Variable x:TypeArguments="x:String" Name="tempValue"/>
  </Sequence.Variables>
</Sequence>
```

---

## Scoring Guide

| Score | Meaning |
|---|---|
| 9–10 | Excellent — production-ready |
| 7–8 | Good — minor issues only |
| 5–6 | Acceptable — fix warnings before UAT |
| 3–4 | Poor — critical issues present |
| 1–2 | Critical — major rework needed |

**Overall verdict:** ≥8 approve | 6–7 approve with conditions | <6 rework required

---

## Quick Review Modes

- "Review naming only" → run checklist Section 2
- "Security audit" → run Section 4
- "Is this REFramework correct?" → Sections 7 + 3
- "Is this LRW-safe?" → Section 8 + 3
- "Full review" → all sections + scorecard
- "Review this XAML" → parse XML, identify activities, run full checklist
