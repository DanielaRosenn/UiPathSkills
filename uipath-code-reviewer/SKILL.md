---
name: uipath-code-reviewer
tags: [uipath-dev, code-review, quality, best-practices]
description: Review UiPath automation code (XAML and C#) for quality issues, best practices violations, and potential bugs. Generates actionable fix suggestions that can be implemented by AI. Use when asked to review UiPath code, check workflow quality, audit automation, or find issues in XAML/C# workflows. Triggers on requests like "review my UiPath code", "check this workflow", "audit automation quality", "find issues in XAML", or "code review".
---

# UiPath Code Reviewer

Analyze UiPath automation code (XAML workflows and C# coded automations) for quality issues, best practices violations, and potential bugs. Generates structured, actionable fix suggestions that can be automatically implemented.

## Official Documentation Reference

This skill aligns with UiPath's official **Workflow Analyzer** rules:
- [UiPath Studio User Guide](https://docs.uipath.com/studio/standalone/latest/user-guide/introduction)
- [Workflow Analyzer](https://docs.uipath.com/studio/standalone/latest/user-guide/about-workflow-analyzer)
- [Naming Rules](https://docs.uipath.com/studio/standalone/latest/user-guide/naming-rules)
- [Design Best Practices](https://docs.uipath.com/studio/standalone/latest/user-guide/design-best-practices)
- [Security Rules](https://docs.uipath.com/studio/standalone/latest/user-guide/security-rules)
- [Coded Automations Best Practices](https://docs.uipath.com/studio/standalone/latest/user-guide/best-practices)

## When to Use

- "Review my UiPath code"
- "Check this workflow for issues"
- "Audit automation quality"
- "Find bugs in XAML"
- "Code review this automation"
- "What's wrong with this workflow?"
- "Improve my UiPath project"
- Before deploying to production
- After generating UiPath code with other skills

## Quick Reference

### Review Scope

| Category | What It Checks |
|----------|----------------|
| **Naming** | Variables, arguments, workflows, activities |
| **Error Handling** | Try-Catch coverage, exception types, logging |
| **Performance** | Unnecessary delays, inefficient loops, selectors |
| **Security** | Hardcoded credentials, sensitive data exposure |
| **Maintainability** | Complexity, modularity, documentation |
| **Best Practices** | UiPath standards, REFramework compliance |

### Severity Levels

| Level | Description | Action |
|-------|-------------|--------|
| **Critical** | Bugs, security issues, will cause failures | Must fix before deployment |
| **High** | Best practice violations, performance issues | Should fix |
| **Medium** | Code quality, maintainability concerns | Recommended to fix |
| **Low** | Style, minor improvements | Nice to have |
| **Info** | Suggestions, optimizations | Consider for future |

## Core Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  CODE REVIEW WORKFLOW                                           │
│                                                                 │
│  1. LOCATE FILES ──> 2. ANALYZE ──> 3. GENERATE REPORT         │
│     *.xaml, *.cs       Run checks     review-report.md          │
│     project.json       by category                              │
│                                                                 │
│  4. CREATE FIXES ──> 5. (OPTIONAL) APPLY                       │
│     fix-suggestions.json   Auto-implement                       │
└─────────────────────────────────────────────────────────────────┘
```

### Step 1: Locate UiPath Project Files

Search for files to review:

```bash
# Project configuration
**/project.json

# XAML workflows
**/*.xaml

# Coded automations
**/*.cs

# Config files
**/Config.xlsx
**/appsettings.json
```

### Step 2: Analyze Each File

For each file type, run the appropriate checks:

#### XAML Analysis Checklist

- [ ] **Namespace declarations** - All required namespaces present
- [ ] **Variable naming** - Follows convention (camelCase with type prefix)
- [ ] **Argument naming** - Follows convention (in_/out_/io_ prefix)
- [ ] **DisplayName** - All activities have meaningful DisplayNames
- [ ] **Error handling** - Try-Catch blocks around risky operations
- [ ] **Logging** - Appropriate LogMessage activities
- [ ] **Selectors** - Dynamic, not hardcoded; use wildcards appropriately
- [ ] **Hardcoded values** - No hardcoded credentials or paths
- [ ] **Activity configuration** - Timeouts, delays configured properly
- [ ] **Workflow complexity** - Not too many nested levels

#### C# Analysis Checklist

- [ ] **Namespace organization** - Proper using statements
- [ ] **Argument attributes** - Correct Direction attributes
- [ ] **Exception handling** - Try-catch with proper exception types
- [ ] **Logging** - Log() calls at appropriate points
- [ ] **Resource cleanup** - Using statements or proper disposal
- [ ] **Code complexity** - Methods not too long
- [ ] **LINQ usage** - Efficient queries, no N+1 patterns

#### Project Configuration Checklist

- [ ] **Dependencies** - Correct versions, no conflicts
- [ ] **Entry points** - Properly defined
- [ ] **Runtime options** - Appropriate for project type
- [ ] **Target framework** - Compatible with deployment environment

### Step 3: Generate Review Report

Create a structured report with all findings.

### Step 4: Create Fix Suggestions

Generate machine-readable fix suggestions for each issue.

### Step 5: (Optional) Apply Fixes

Use the fix suggestions to automatically implement corrections.

---

## UiPath Workflow Analyzer Rule Mapping

This skill checks rules that align with UiPath's built-in Workflow Analyzer. Here's the mapping:

### Official UiPath Naming Rules (ST-NMG-*)

| UiPath Rule | Description | Our Rule |
|-------------|-------------|----------|
| ST-NMG-001 | Variables Naming Convention | RULE-001 |
| ST-NMG-002 | Arguments Naming Convention | RULE-002 |
| ST-NMG-004 | Display Name Duplication | RULE-004 |
| ST-NMG-005 | Variable Overrides Variable | RULE-005 |
| ST-NMG-008 | Variable Length Exceeded | RULE-006 |
| ST-NMG-009 | Prefix DataTable Variables | RULE-007 |
| ST-NMG-011 | Prefix DataTable Arguments | RULE-008 |
| ST-NMG-012 | Argument Default Values | RULE-009 |

### Official UiPath Design Best Practices (ST-DBP-*)

| UiPath Rule | Description | Our Rule |
|-------------|-------------|----------|
| ST-DBP-002 | High Arguments Count | RULE-040 |
| ST-DBP-003 | Empty Catch Block | RULE-011 |
| ST-DBP-007 | Multiple Flowchart Layers | RULE-044 |
| ST-DBP-020 | Undefined Output Properties | RULE-056 |
| ST-DBP-021 | Hardcoded Timeout | RULE-032 |
| ST-DBP-023 | Empty Workflow | RULE-041 |

### Official UiPath Maintainability Rules (ST-MRD-*)

| UiPath Rule | Description | Our Rule |
|-------------|-------------|----------|
| ST-MRD-002 | Activity Name Defaults | RULE-004 |
| ST-MRD-007 | Nested If Clauses | RULE-044 |
| ST-MRD-008 | Empty Sequence | RULE-041 |
| ST-MRD-009 | Deeply Nested Activities | RULE-040 |

### Official UiPath Usage Rules (ST-USG-*)

| UiPath Rule | Description | Our Rule |
|-------------|-------------|----------|
| ST-USG-005 | Hardcoded Activity Properties | RULE-021, RULE-043 |
| ST-USG-009 | Unused Variables | RULE-055 |
| ST-USG-020 | Minimum Log Messages | RULE-050 |

### Official UiPath Security Rules (ST-SEC-*)

| UiPath Rule | Description | Our Rule |
|-------------|-------------|----------|
| ST-SEC-007 | SecureString Argument Usage | RULE-023 |
| ST-SEC-008 | SecureString Variable Usage | RULE-023 |
| ST-SEC-009 | SecureString Misusage | RULE-020 |

### Official UiPath Performance Rules (ST-PRR-*)

| UiPath Rule | Description | Our Rule |
|-------------|-------------|----------|
| ST-PRR-004 | Hardcoded Delay Activity | RULE-030 |

### Additional Custom Rules (Beyond Workflow Analyzer)

This skill also includes rules not covered by the built-in Workflow Analyzer:

| Our Rule | Description | Category |
|----------|-------------|----------|
| RULE-010 | Missing Try-Catch | Error Handling |
| RULE-012 | Generic Exception Only | Error Handling |
| RULE-013 | Missing Retry Logic | Error Handling |
| RULE-022 | Sensitive Data in Logs | Security |
| RULE-024 | SQL Injection Risk | Security |
| RULE-031 | Inefficient Loop | Performance |
| RULE-033 | Selector Too Specific | Performance |
| RULE-052 | Missing Input Validation | Best Practices |
| RULE-053 | HTTP Without Error Check | Best Practices |

---

## Review Rules Reference

### Naming Convention Rules

#### RULE-001: Variable Naming Convention

**Severity**: Medium  
**Category**: Naming

Variables should use camelCase with a type prefix.

| Type | Prefix | Example |
|------|--------|---------|
| String | str | `strFileName` |
| Int32 | int | `intCounter` |
| Boolean | bool | `boolIsValid` |
| DataTable | dt | `dtResults` |
| DataRow | row | `rowCurrent` |
| DateTime | dt | `dtStartTime` |
| Array | arr | `arrItems` |
| List | lst | `lstNames` |
| Dictionary | dict | `dictConfig` |
| Object | obj | `objResponse` |
| JObject | jo | `joApiResponse` |
| Exception | ex | `exCurrent` |

**Bad Examples**:
```xml
<Variable x:TypeArguments="x:String" Name="filename" />
<Variable x:TypeArguments="x:Int32" Name="Counter" />
<Variable x:TypeArguments="sd:DataTable" Name="data" />
```

**Good Examples**:
```xml
<Variable x:TypeArguments="x:String" Name="strFileName" />
<Variable x:TypeArguments="x:Int32" Name="intCounter" />
<Variable x:TypeArguments="sd:DataTable" Name="dtData" />
```

#### RULE-002: Argument Naming Convention

**Severity**: Medium  
**Category**: Naming

Arguments should use direction prefix followed by PascalCase.

| Direction | Prefix | Example |
|-----------|--------|---------|
| In | in_ | `in_FilePath` |
| Out | out_ | `out_Result` |
| InOut | io_ | `io_DataTable` |

**Bad Examples**:
```xml
<x:Property Name="FilePath" Type="InArgument(x:String)" />
<x:Property Name="result" Type="OutArgument(x:String)" />
```

**Good Examples**:
```xml
<x:Property Name="in_FilePath" Type="InArgument(x:String)" />
<x:Property Name="out_Result" Type="OutArgument(x:String)" />
```

#### RULE-003: Workflow File Naming

**Severity**: Low  
**Category**: Naming

Workflow files should use PascalCase and describe their purpose.

**Bad Examples**:
- `process_data.xaml`
- `workflow1.xaml`
- `test.xaml`

**Good Examples**:
- `ProcessTransaction.xaml`
- `InitAllSettings.xaml`
- `GetTransactionData.xaml`

#### RULE-004: Activity DisplayName

**Severity**: Medium  
**Category**: Naming

All activities should have descriptive DisplayNames.

**Bad Examples**:
```xml
<Assign DisplayName="Assign">
<ui:LogMessage DisplayName="Log Message">
<If DisplayName="If">
```

**Good Examples**:
```xml
<Assign DisplayName="Set Transaction Status">
<ui:LogMessage DisplayName="Log Process Start">
<If DisplayName="Check If File Exists">
```

---

### Error Handling Rules

#### RULE-010: Missing Try-Catch

**Severity**: High  
**Category**: Error Handling

Critical operations should be wrapped in Try-Catch blocks.

**Critical operations include**:
- HTTP requests
- Database operations
- File I/O
- Excel operations
- UI automation
- External API calls

**Bad Example**:
```xml
<Sequence DisplayName="Process Data">
  <ui:HttpClient Endpoint="[apiUrl]" Response="[response]" />
  <ui:ExcelWriteRange DataTable="[dtData]" />
</Sequence>
```

**Good Example**:
```xml
<TryCatch DisplayName="Process Data with Error Handling">
  <TryCatch.Try>
    <Sequence>
      <ui:HttpClient Endpoint="[apiUrl]" Response="[response]" />
      <ui:ExcelWriteRange DataTable="[dtData]" />
    </Sequence>
  </TryCatch.Try>
  <TryCatch.Catches>
    <Catch x:TypeArguments="s:Exception">
      <ActivityAction x:TypeArguments="s:Exception">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="s:Exception" Name="exception" />
        </ActivityAction.Argument>
        <ui:LogMessage Level="Error" Message="[exception.Message]" />
      </ActivityAction>
    </Catch>
  </TryCatch.Catches>
</TryCatch>
```

#### RULE-011: Empty Catch Block

**Severity**: Critical  
**Category**: Error Handling

Catch blocks should not be empty - they must log or handle the exception.

**Bad Example**:
```xml
<TryCatch.Catches>
  <Catch x:TypeArguments="s:Exception">
    <ActivityAction x:TypeArguments="s:Exception">
      <ActivityAction.Argument>
        <DelegateInArgument x:TypeArguments="s:Exception" Name="exception" />
      </ActivityAction.Argument>
      <!-- Empty! -->
    </ActivityAction>
  </Catch>
</TryCatch.Catches>
```

**Good Example**:
```xml
<TryCatch.Catches>
  <Catch x:TypeArguments="s:Exception">
    <ActivityAction x:TypeArguments="s:Exception">
      <ActivityAction.Argument>
        <DelegateInArgument x:TypeArguments="s:Exception" Name="exception" />
      </ActivityAction.Argument>
      <Sequence>
        <ui:LogMessage Level="Error" Message="[&quot;Error: &quot; + exception.Message]" />
        <Rethrow />
      </Sequence>
    </ActivityAction>
  </Catch>
</TryCatch.Catches>
```

#### RULE-012: Generic Exception Only

**Severity**: Medium  
**Category**: Error Handling

Catch specific exception types before generic Exception.

**Bad Example**:
```xml
<TryCatch.Catches>
  <Catch x:TypeArguments="s:Exception">
    <!-- Only catches generic Exception -->
  </Catch>
</TryCatch.Catches>
```

**Good Example**:
```xml
<TryCatch.Catches>
  <Catch x:TypeArguments="ui:BusinessRuleException">
    <!-- Handle business errors -->
  </Catch>
  <Catch x:TypeArguments="s:TimeoutException">
    <!-- Handle timeouts -->
  </Catch>
  <Catch x:TypeArguments="s:Exception">
    <!-- Handle everything else -->
  </Catch>
</TryCatch.Catches>
```

#### RULE-013: Missing Retry Logic

**Severity**: Medium  
**Category**: Error Handling

Transient operations (HTTP, UI) should use RetryScope.

**Bad Example**:
```xml
<ui:Click Target="[selector]" />
```

**Good Example**:
```xml
<ui:RetryScope NumberOfRetries="3" RetryInterval="00:00:02">
  <ui:RetryScope.ActivityBody>
    <ActivityAction>
      <ui:Click Target="[selector]" />
    </ActivityAction>
  </ui:RetryScope.ActivityBody>
</ui:RetryScope>
```

---

### Security Rules

#### RULE-020: Hardcoded Credentials

**Severity**: Critical  
**Category**: Security

Never hardcode passwords, API keys, or secrets.

**Bad Example**:
```xml
<Assign DisplayName="Set Password">
  <Assign.To><OutArgument x:TypeArguments="x:String">[strPassword]</OutArgument></Assign.To>
  <Assign.Value><InArgument x:TypeArguments="x:String">["MySecretPassword123!"]</InArgument></Assign.Value>
</Assign>
```

**Good Example**:
```xml
<ui:GetRobotCredential CredentialAssetName="SystemCredential" 
  Username="[strUsername]" Password="[secPassword]" />
```

#### RULE-021: Hardcoded File Paths

**Severity**: High  
**Category**: Security/Maintainability

Avoid hardcoded absolute paths - use config or relative paths.

**Bad Example**:
```xml
<ui:ReadTextFile FileName="C:\Users\John\Documents\input.txt" />
```

**Good Example**:
```xml
<ui:ReadTextFile FileName="[in_ConfigPath + &quot;\input.txt&quot;]" />
```

#### RULE-022: Sensitive Data in Logs

**Severity**: Critical  
**Category**: Security

Never log passwords, tokens, or PII.

**Bad Example**:
```xml
<ui:LogMessage Level="Info" Message="[&quot;Password: &quot; + strPassword]" />
```

**Good Example**:
```xml
<ui:LogMessage Level="Info" Message="Authentication successful" />
```

#### RULE-023: Missing Secure String

**Severity**: High  
**Category**: Security

Passwords should use SecureString, not String.

**Bad Example**:
```xml
<Variable x:TypeArguments="x:String" Name="strPassword" />
```

**Good Example**:
```xml
<Variable x:TypeArguments="s:Security.SecureString" Name="secPassword" />
```

---

### Performance Rules

#### RULE-030: Excessive Delays

**Severity**: Medium  
**Category**: Performance

Avoid hardcoded delays - use element waits instead.

**Bad Example**:
```xml
<Delay Duration="[TimeSpan.FromSeconds(10)]" DisplayName="Wait for page" />
<ui:Click Target="[selector]" />
```

**Good Example**:
```xml
<ui:WaitElementVanish Target="[loadingSelector]" Timeout="30000" />
<ui:Click Target="[selector]" />
```

#### RULE-031: Inefficient Loop

**Severity**: Medium  
**Category**: Performance

Use LINQ instead of loops for DataTable filtering.

**Bad Example**:
```xml
<ForEach x:TypeArguments="sd:DataRow" Values="[dtData.Rows]">
  <If Condition="[row(&quot;Status&quot;).ToString = &quot;Active&quot;]">
    <!-- Process -->
  </If>
</ForEach>
```

**Good Example**:
```xml
<Assign DisplayName="Filter Active Rows">
  <Assign.To><OutArgument x:TypeArguments="sd:DataTable">[dtFiltered]</OutArgument></Assign.To>
  <Assign.Value><InArgument x:TypeArguments="sd:DataTable">[dtData.AsEnumerable().Where(Function(r) r(&quot;Status&quot;).ToString = &quot;Active&quot;).CopyToDataTable()]</InArgument></Assign.Value>
</Assign>
```

#### RULE-032: Missing Timeout

**Severity**: High  
**Category**: Performance

UI activities should have explicit timeouts.

**Bad Example**:
```xml
<ui:Click Target="[selector]" />
```

**Good Example**:
```xml
<ui:Click Target="[selector]" Timeout="30000" />
```

#### RULE-033: Selector Too Specific

**Severity**: Medium  
**Category**: Performance/Maintainability

Selectors should not include volatile attributes.

**Bad Example**:
```xml
<ui:Target Selector="&lt;html title='Page - Session 12345'/&gt;&lt;webctrl idx='3'/&gt;" />
```

**Good Example**:
```xml
<ui:Target Selector="&lt;html title='Page*'/&gt;&lt;webctrl tag='BUTTON' id='submit'/&gt;" />
```

---

### Maintainability Rules

#### RULE-040: Workflow Too Complex

**Severity**: Medium  
**Category**: Maintainability

Workflows should not exceed recommended complexity limits.

| Metric | Limit | Recommendation |
|--------|-------|----------------|
| Activities | 50 | Split into sub-workflows |
| Nesting depth | 5 | Flatten or extract |
| Arguments | 10 | Group into objects |

#### RULE-041: Missing Documentation

**Severity**: Low  
**Category**: Maintainability

Workflows should have annotations explaining purpose.

**Bad Example**:
```xml
<Sequence DisplayName="Main">
  <!-- No annotation -->
</Sequence>
```

**Good Example**:
```xml
<Sequence DisplayName="Main">
  <sap2010:Annotation.AnnotationText>
    Main entry point for invoice processing.
    Input: in_InvoicePath - Path to invoice PDF
    Output: out_ProcessingResult - Success/Failure status
  </sap2010:Annotation.AnnotationText>
</Sequence>
```

#### RULE-042: Duplicate Code

**Severity**: Medium  
**Category**: Maintainability

Similar code blocks should be extracted to reusable workflows.

#### RULE-043: Magic Numbers

**Severity**: Low  
**Category**: Maintainability

Use named constants or config values instead of magic numbers.

**Bad Example**:
```xml
<If Condition="[intAmount > 10000]">
```

**Good Example**:
```xml
<If Condition="[intAmount > Config(&quot;ApprovalThreshold&quot;)]">
```

---

### Best Practice Rules

#### RULE-050: Missing Logging

**Severity**: Medium  
**Category**: Best Practices

Key workflow stages should have log messages.

**Required logging points**:
- Workflow start
- Workflow end
- Before/after external calls
- Exception handling
- Business decisions

#### RULE-051: Inconsistent Activity Naming

**Severity**: Low  
**Category**: Best Practices

Activity DisplayNames should follow a consistent pattern.

**Recommended patterns**:
- `Verb + Object`: "Read Config File", "Process Transaction"
- `Check + Condition`: "Check If Valid", "Check File Exists"
- `Set + Variable`: "Set Status", "Set Counter"

#### RULE-052: Missing Input Validation

**Severity**: High  
**Category**: Best Practices

Input arguments should be validated at workflow start.

**Bad Example**:
```xml
<Sequence DisplayName="Process File">
  <ui:ReadTextFile FileName="[in_FilePath]" />
</Sequence>
```

**Good Example**:
```xml
<Sequence DisplayName="Process File">
  <If Condition="[String.IsNullOrEmpty(in_FilePath)]">
    <If.Then>
      <Throw>
        <Throw.Exception>
          <InArgument x:TypeArguments="s:Exception">[New ArgumentException(&quot;in_FilePath is required&quot;)]</InArgument>
        </Throw.Exception>
      </Throw>
    </If.Then>
  </If>
  <ui:PathExists Path="[in_FilePath]" Exists="[boolExists]" />
  <If Condition="[Not boolExists]">
    <If.Then>
      <Throw>
        <Throw.Exception>
          <InArgument x:TypeArguments="s:Exception">[New FileNotFoundException(&quot;File not found: &quot; + in_FilePath)]</InArgument>
        </Throw.Exception>
      </Throw>
    </If.Then>
  </If>
  <ui:ReadTextFile FileName="[in_FilePath]" />
</Sequence>
```

#### RULE-053: HTTP Without Error Check

**Severity**: High  
**Category**: Best Practices

HTTP responses should check status code before processing.

**Bad Example**:
```xml
<ui:HttpClient Endpoint="[apiUrl]" Response="[response]" />
<ui:DeserializeJson JsonString="[response]" JsonObject="[joResult]" />
```

**Good Example**:
```xml
<ui:HttpClient Endpoint="[apiUrl]" Response="[response]" StatusCode="[statusCode]" />
<If Condition="[statusCode &lt;&gt; 200]">
  <If.Then>
    <Throw>
      <Throw.Exception>
        <InArgument x:TypeArguments="s:Exception">[New Exception(&quot;API call failed: &quot; + statusCode.ToString)]</InArgument>
      </Throw.Exception>
    </Throw>
  </If.Then>
</If>
<ui:DeserializeJson JsonString="[response]" JsonObject="[joResult]" />
```

---

## Output Formats

### Review Report Format (review-report.md)

```markdown
# UiPath Code Review Report

**Project**: {ProjectName}
**Date**: {ReviewDate}
**Reviewer**: AI Code Reviewer

## Summary

| Severity | Count |
|----------|-------|
| Critical | {count} |
| High | {count} |
| Medium | {count} |
| Low | {count} |
| Info | {count} |

## Findings

### Critical Issues

#### [RULE-020] Hardcoded Credentials in Main.xaml

**File**: Main.xaml
**Line**: 45
**Description**: Password is hardcoded in plain text
**Impact**: Security vulnerability - credentials exposed in source code
**Fix**: Use Orchestrator Credential Asset

**Current Code**:
```xml
<Assign DisplayName="Set Password">
  <Assign.Value><InArgument x:TypeArguments="x:String">["MyPassword123"]</InArgument></Assign.Value>
</Assign>
```

**Suggested Fix**:
```xml
<ui:GetRobotCredential CredentialAssetName="SystemCredential" 
  Username="[strUsername]" Password="[secPassword]" />
```

---

### High Issues
...

### Medium Issues
...
```

### Fix Suggestions Format (fix-suggestions.json)

```json
{
  "projectName": "InvoiceProcessor",
  "reviewDate": "2026-02-24",
  "totalIssues": 12,
  "fixes": [
    {
      "id": "FIX-001",
      "ruleId": "RULE-001",
      "severity": "medium",
      "file": "Main.xaml",
      "line": 23,
      "description": "Variable 'filename' should use naming convention",
      "currentCode": "<Variable x:TypeArguments=\"x:String\" Name=\"filename\" />",
      "suggestedCode": "<Variable x:TypeArguments=\"x:String\" Name=\"strFileName\" />",
      "autoFixable": true,
      "searchPattern": "Name=\"filename\"",
      "replacePattern": "Name=\"strFileName\""
    },
    {
      "id": "FIX-002",
      "ruleId": "RULE-010",
      "severity": "high",
      "file": "ProcessData.xaml",
      "line": 45,
      "description": "HTTP request missing Try-Catch",
      "currentCode": "<ui:HttpClient Endpoint=\"[apiUrl]\" />",
      "suggestedCode": "<!-- See full suggestion in review-report.md -->",
      "autoFixable": false,
      "manualSteps": [
        "Wrap the HTTP activity in a TryCatch",
        "Add appropriate Catch blocks",
        "Add logging in catch block"
      ]
    }
  ]
}
```

---

## Usage Examples

### Basic Review

```
User: Review my UiPath project at C:\Projects\InvoiceProcessor

AI: I'll analyze your UiPath project for code quality issues.

[Reads project.json, Main.xaml, and all workflow files]
[Applies all review rules]
[Generates review-report.md and fix-suggestions.json]

Found 12 issues:
- 2 Critical (hardcoded credentials, empty catch block)
- 3 High (missing try-catch, no input validation)
- 5 Medium (naming conventions, missing timeouts)
- 2 Low (missing annotations)

See review-report.md for details and fix suggestions.
```

### Review with Auto-Fix

```
User: Review and fix the naming convention issues in my workflow

AI: I'll review and automatically fix naming convention violations.

[Analyzes code]
[Identifies 8 naming issues]
[Applies auto-fixes for variables and arguments]

Fixed 8 naming issues:
- filename → strFileName
- counter → intCounter
- FilePath → in_FilePath
- result → out_Result
...
```

### Review Specific File

```
User: Check ProcessTransaction.xaml for error handling issues

AI: I'll focus on error handling in ProcessTransaction.xaml.

[Analyzes error handling patterns]
[Checks try-catch coverage]
[Validates exception handling]

Found 3 error handling issues:
1. Line 34: HTTP request without Try-Catch (High)
2. Line 67: Empty catch block (Critical)
3. Line 89: Missing retry logic for UI automation (Medium)
```

---

## Integration with Other Skills

### Post-Generation Review

After using `uipath-automation` or `coded-workflow-builder` skills:

```
1. Generate workflow with uipath-automation skill
2. Run uipath-code-reviewer on generated code
3. Apply fixes from fix-suggestions.json
4. Re-run review to confirm all issues resolved
```

### Pre-Deployment Checklist

Before deploying to Orchestrator:

```
1. Run full code review
2. Ensure no Critical or High issues
3. Document any accepted Medium/Low issues
4. Generate review-report.md for audit trail
```

---

## Quality Checklist

Before completing a code review:

- [ ] All XAML files analyzed
- [ ] All C# files analyzed (if coded automation)
- [ ] project.json validated
- [ ] All rule categories checked
- [ ] Review report generated
- [ ] Fix suggestions created
- [ ] Auto-fixable issues identified
- [ ] Manual fix steps documented

---

## Reference Documentation

### Shared References

- [Error Handling](../_shared/error-handling.md) - Error handling patterns
- [Logging Standards](../_shared/logging-standards.md) - Logging best practices
- [Project Configuration](../_shared/project-configuration.md) - project.json structure

### Code Reviewer References

- [Review Rules Catalog](references/review-rules-catalog.md) - Complete list of all rules
- [XAML Patterns](references/xaml-patterns.md) - Good vs bad XAML patterns
- [C# Patterns](references/csharp-patterns.md) - Good vs bad C# patterns
- [Auto-Fix Guide](references/auto-fix-guide.md) - How to apply automatic fixes
