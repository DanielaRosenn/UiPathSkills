# UiPath Code Review Rules Catalog

Complete reference of all code review rules with detection patterns and fix templates.

## Official UiPath Documentation

This catalog aligns with UiPath's official Workflow Analyzer rules:
- [Workflow Analyzer Overview](https://docs.uipath.com/studio/standalone/latest/user-guide/about-workflow-analyzer)
- [Naming Rules (ST-NMG-*)](https://docs.uipath.com/studio/standalone/latest/user-guide/naming-rules)
- [Design Best Practices (ST-DBP-*)](https://docs.uipath.com/studio/standalone/latest/user-guide/design-best-practices)
- [Maintainability Rules (ST-MRD-*)](https://docs.uipath.com/studio/standalone/latest/user-guide/maintainability-and-readability-rules)
- [Usage Rules (ST-USG-*)](https://docs.uipath.com/studio/standalone/latest/user-guide/usage-rules)
- [Security Rules (ST-SEC-*)](https://docs.uipath.com/studio/standalone/latest/user-guide/security-rules)
- [Performance Rules (ST-PRR-*)](https://docs.uipath.com/studio/standalone/latest/user-guide/performance-and-reusability-rules)

## Table of Contents

1. [Naming Rules (001-009)](#naming-rules)
2. [Error Handling Rules (010-019)](#error-handling-rules)
3. [Security Rules (020-029)](#security-rules)
4. [Performance Rules (030-039)](#performance-rules)
5. [Maintainability Rules (040-049)](#maintainability-rules)
6. [Best Practice Rules (050-059)](#best-practice-rules)
7. [XAML-Specific Rules (060-069)](#xaml-specific-rules)
8. [C#-Specific Rules (070-079)](#csharp-specific-rules)
9. [Project Configuration Rules (080-089)](#project-configuration-rules)

---

## Naming Rules

### RULE-001: Variable Naming Convention

| Property | Value |
|----------|-------|
| **ID** | RULE-001 |
| **UiPath Rule** | [ST-NMG-001](https://docs.uipath.com/studio/standalone/latest/user-guide/st-nmg-001-variables-naming-convention) |
| **Severity** | Medium |
| **Category** | Naming |
| **Auto-fixable** | Yes |
| **Applies to** | XAML |

**Description**: Variables should use camelCase with a type prefix (str, int, bool, dt, etc.)

**Detection Pattern (Regex)**:
```regex
<Variable\s+x:TypeArguments="([^"]+)"\s+Name="([^"]+)"
```

**Validation Logic**:
```javascript
function validateVariableName(type, name) {
  const prefixMap = {
    'x:String': 'str',
    'x:Int32': 'int',
    'x:Boolean': 'bool',
    'sd:DataTable': 'dt',
    'sd:DataRow': 'row',
    's:DateTime': 'dt',
    'x:Object': 'obj',
    'nj:JObject': 'jo',
    's:Exception': 'ex'
  };
  
  const expectedPrefix = prefixMap[type];
  if (!expectedPrefix) return true; // Unknown type, skip
  
  return name.startsWith(expectedPrefix) && 
         name.charAt(expectedPrefix.length) === name.charAt(expectedPrefix.length).toUpperCase();
}
```

**Fix Template**:
```json
{
  "searchPattern": "Name=\"{oldName}\"",
  "replacePattern": "Name=\"{newName}\"",
  "additionalReplacements": [
    {"search": "[{oldName}]", "replace": "[{newName}]"},
    {"search": "[{oldName}.", "replace": "[{newName}."},
    {"search": "[{oldName} ", "replace": "[{newName} "},
    {"search": "[{oldName}+", "replace": "[{newName}+"}
  ]
}
```

---

### RULE-002: Argument Naming Convention

| Property | Value |
|----------|-------|
| **ID** | RULE-002 |
| **UiPath Rule** | [ST-NMG-002](https://docs.uipath.com/studio/standalone/latest/user-guide/st-nmg-002-arguments-naming-convention) |
| **Severity** | Medium |
| **Category** | Naming |
| **Auto-fixable** | Yes |
| **Applies to** | XAML |

**Description**: Arguments should use direction prefix (in_, out_, io_) followed by PascalCase.

**Detection Pattern (Regex)**:
```regex
<x:Property\s+Name="([^"]+)"\s+Type="(In|Out|InOut)Argument\(([^)]+)\)"
```

**Validation Logic**:
```javascript
function validateArgumentName(name, direction) {
  const prefixMap = {
    'In': 'in_',
    'Out': 'out_',
    'InOut': 'io_'
  };
  
  const expectedPrefix = prefixMap[direction];
  return name.startsWith(expectedPrefix);
}
```

**Fix Template**:
```json
{
  "searchPattern": "Name=\"{oldName}\"",
  "replacePattern": "Name=\"{newName}\"",
  "additionalReplacements": [
    {"search": "x:Key=\"{oldName}\"", "replace": "x:Key=\"{newName}\""},
    {"search": "[{oldName}]", "replace": "[{newName}]"}
  ]
}
```

---

### RULE-003: Workflow File Naming

| Property | Value |
|----------|-------|
| **ID** | RULE-003 |
| **Severity** | Low |
| **Category** | Naming |
| **Auto-fixable** | No (requires file rename) |
| **Applies to** | XAML |

**Description**: Workflow files should use PascalCase and describe their purpose.

**Detection Pattern**:
- File name contains underscore
- File name is all lowercase
- File name is generic (workflow1, test, temp)

**Validation Logic**:
```javascript
function validateWorkflowFileName(fileName) {
  const badPatterns = [
    /^workflow\d*\.xaml$/i,
    /^test\d*\.xaml$/i,
    /^temp\d*\.xaml$/i,
    /_/,
    /^[a-z]+\.xaml$/
  ];
  
  return !badPatterns.some(pattern => pattern.test(fileName));
}
```

---

### RULE-004: Activity DisplayName

| Property | Value |
|----------|-------|
| **ID** | RULE-004 |
| **UiPath Rule** | [ST-NMG-004](https://docs.uipath.com/studio/standalone/latest/user-guide/st-nmg-004-display-name-duplication), [ST-MRD-002](https://docs.uipath.com/studio/standalone/latest/user-guide/st-mrd-002-activity-name-defaults) |
| **Severity** | Medium |
| **Category** | Naming |
| **Auto-fixable** | Partial |
| **Applies to** | XAML |

**Description**: All activities should have meaningful, descriptive DisplayNames.

**Detection Pattern (Regex)**:
```regex
DisplayName="(Assign|If|Sequence|ForEach|While|TryCatch|Catch|Switch|Flowchart)"
```

**Bad DisplayNames**:
- Same as activity type (e.g., "Assign", "If", "Sequence")
- Generic (e.g., "Activity 1", "Step 1")
- Empty

**Suggested Fix**: Generate descriptive name based on activity content.

---

### RULE-005: Consistent Casing

| Property | Value |
|----------|-------|
| **ID** | RULE-005 |
| **Severity** | Low |
| **Category** | Naming |
| **Auto-fixable** | Yes |
| **Applies to** | XAML, C# |

**Description**: Names should use consistent casing throughout the project.

---

## Error Handling Rules

### RULE-010: Missing Try-Catch

| Property | Value |
|----------|-------|
| **ID** | RULE-010 |
| **Severity** | High |
| **Category** | Error Handling |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Critical operations should be wrapped in Try-Catch blocks.

**Critical Operations (XAML)**:
```regex
<ui:HttpClient|<uwah:NetHttpRequest|<ui:ExcelReadRange|<ui:ExcelWriteRange|<ui:ExecuteQuery|<ui:Click|<ui:TypeInto|<ui:GetText
```

**Detection Logic**:
```javascript
function checkTryCatchCoverage(activity, ancestors) {
  const criticalActivities = [
    'HttpClient', 'NetHttpRequest', 
    'ExcelReadRange', 'ExcelWriteRange', 'UseExcelFile',
    'ExecuteQuery', 'ExecuteNonQuery',
    'Click', 'TypeInto', 'GetText', 'SetText'
  ];
  
  if (!criticalActivities.includes(activity.type)) return true;
  
  return ancestors.some(a => a.type === 'TryCatch');
}
```

**Manual Fix Steps**:
1. Wrap the activity in a TryCatch
2. Add Catch block for specific exception types
3. Add Catch block for generic Exception
4. Add logging in catch blocks
5. Consider adding Finally block for cleanup

---

### RULE-011: Empty Catch Block

| Property | Value |
|----------|-------|
| **ID** | RULE-011 |
| **UiPath Rule** | [ST-DBP-003](https://docs.uipath.com/studio/standalone/latest/user-guide/st-dbp-003-empty-catch-block) |
| **Severity** | Critical |
| **Category** | Error Handling |
| **Auto-fixable** | Partial |
| **Applies to** | XAML, C# |

**Description**: Catch blocks must not be empty - they should log or handle the exception.

**Detection Pattern (XAML)**:
```regex
<Catch[^>]*>\s*<ActivityAction[^>]*>\s*<ActivityAction\.Argument>\s*<DelegateInArgument[^/]*/>\s*</ActivityAction\.Argument>\s*</ActivityAction>\s*</Catch>
```

**Fix Template**:
```xml
<Catch x:TypeArguments="s:Exception">
  <ActivityAction x:TypeArguments="s:Exception">
    <ActivityAction.Argument>
      <DelegateInArgument x:TypeArguments="s:Exception" Name="exception" />
    </ActivityAction.Argument>
    <Sequence DisplayName="Handle Exception">
      <ui:LogMessage Level="Error" Message="[&quot;Error: &quot; + exception.Message]" />
      <Rethrow />
    </Sequence>
  </ActivityAction>
</Catch>
```

---

### RULE-012: Generic Exception Only

| Property | Value |
|----------|-------|
| **ID** | RULE-012 |
| **Severity** | Medium |
| **Category** | Error Handling |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Catch specific exception types before generic Exception.

**Detection Logic**:
```javascript
function checkCatchSpecificity(tryCatch) {
  const catches = tryCatch.catches;
  
  // Only one catch and it's generic Exception
  if (catches.length === 1 && catches[0].type === 's:Exception') {
    return false;
  }
  
  // Generic Exception should be last
  const genericIndex = catches.findIndex(c => c.type === 's:Exception');
  if (genericIndex !== -1 && genericIndex !== catches.length - 1) {
    return false;
  }
  
  return true;
}
```

**Recommended Catch Order**:
1. `ui:BusinessRuleException`
2. `s:TimeoutException`
3. `s:IO.IOException`
4. `s:Net.WebException`
5. `s:Exception` (last)

---

### RULE-013: Missing Retry Logic

| Property | Value |
|----------|-------|
| **ID** | RULE-013 |
| **Severity** | Medium |
| **Category** | Error Handling |
| **Auto-fixable** | No |
| **Applies to** | XAML |

**Description**: Transient operations should use RetryScope.

**Transient Operations**:
- HTTP requests
- UI automation (Click, TypeInto, GetText)
- Database connections
- File operations on network paths

---

### RULE-014: Missing Finally Block

| Property | Value |
|----------|-------|
| **ID** | RULE-014 |
| **Severity** | Low |
| **Category** | Error Handling |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: TryCatch with resource allocation should have Finally for cleanup.

---

### RULE-015: Swallowed Exception

| Property | Value |
|----------|-------|
| **ID** | RULE-015 |
| **Severity** | High |
| **Category** | Error Handling |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Exceptions should not be caught and ignored without logging.

---

## Security Rules

### RULE-020: Hardcoded Credentials

| Property | Value |
|----------|-------|
| **ID** | RULE-020 |
| **Severity** | Critical |
| **Category** | Security |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Never hardcode passwords, API keys, or secrets.

**Detection Patterns**:
```regex
# Password patterns
[Pp]assword\s*=\s*["'][^"']+["']
[Pp]wd\s*=\s*["'][^"']+["']

# API key patterns
[Aa]pi[Kk]ey\s*=\s*["'][^"']+["']
[Ss]ecret\s*=\s*["'][^"']+["']

# Connection string with password
Server=.*;Password=[^;]+

# Bearer token
Bearer\s+[A-Za-z0-9\-_]+\.[A-Za-z0-9\-_]+\.[A-Za-z0-9\-_]+
```

**Fix**: Use Orchestrator Credential Assets or GetSecret activity.

---

### RULE-021: Hardcoded File Paths

| Property | Value |
|----------|-------|
| **ID** | RULE-021 |
| **Severity** | High |
| **Category** | Security/Maintainability |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Avoid hardcoded absolute paths.

**Detection Pattern**:
```regex
[A-Z]:\\[^"'\]]+
/home/[^"'\]]+
/Users/[^"'\]]+
```

---

### RULE-022: Sensitive Data in Logs

| Property | Value |
|----------|-------|
| **ID** | RULE-022 |
| **Severity** | Critical |
| **Category** | Security |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Never log passwords, tokens, or PII.

**Detection Patterns**:
```regex
LogMessage.*[Pp]assword
LogMessage.*[Tt]oken
LogMessage.*[Ss]ecret
LogMessage.*SSN
LogMessage.*[Cc]redit[Cc]ard
```

---

### RULE-023: Missing Secure String

| Property | Value |
|----------|-------|
| **ID** | RULE-023 |
| **UiPath Rule** | [ST-SEC-007](https://docs.uipath.com/studio/standalone/latest/user-guide/st-sec-007-securestring-argument-usage), [ST-SEC-008](https://docs.uipath.com/studio/standalone/latest/user-guide/st-sec-008-securestring-variable-usage) |
| **Severity** | High |
| **Category** | Security |
| **Auto-fixable** | Yes |
| **Applies to** | XAML |

**Description**: Passwords should use SecureString, not String.

**Detection Pattern**:
```regex
<Variable[^>]*Name="[^"]*[Pp]assword[^"]*"[^>]*TypeArguments="x:String"
```

**Fix Template**:
```json
{
  "searchPattern": "x:TypeArguments=\"x:String\" Name=\"{name}\"",
  "replacePattern": "x:TypeArguments=\"s:Security.SecureString\" Name=\"{name}\""
}
```

---

### RULE-024: SQL Injection Risk

| Property | Value |
|----------|-------|
| **ID** | RULE-024 |
| **Severity** | Critical |
| **Category** | Security |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Use parameterized queries, not string concatenation.

**Detection Pattern**:
```regex
Sql="[^"]*\+[^"]*"
Sql="[^"]*&amp;[^"]*"
```

---

## Performance Rules

### RULE-030: Excessive Delays

| Property | Value |
|----------|-------|
| **ID** | RULE-030 |
| **UiPath Rule** | [ST-PRR-004](https://docs.uipath.com/studio/standalone/latest/user-guide/st-prr-004-hardcoded-delay-activity) |
| **Severity** | Medium |
| **Category** | Performance |
| **Auto-fixable** | No |
| **Applies to** | XAML |

**Description**: Avoid hardcoded delays - use element waits instead.

**Detection Pattern**:
```regex
<Delay\s+Duration="\[TimeSpan\.From(Seconds|Minutes)\(\d+\)\]"
```

**Threshold**: Delays > 5 seconds should be reviewed.

---

### RULE-031: Inefficient Loop

| Property | Value |
|----------|-------|
| **ID** | RULE-031 |
| **Severity** | Medium |
| **Category** | Performance |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Use LINQ instead of loops for DataTable filtering.

---

### RULE-032: Missing Timeout

| Property | Value |
|----------|-------|
| **ID** | RULE-032 |
| **UiPath Rule** | [ST-DBP-021](https://docs.uipath.com/studio/standalone/latest/user-guide/st-dbp-021-hardcoded-timeout) |
| **Severity** | High |
| **Category** | Performance |
| **Auto-fixable** | Yes |
| **Applies to** | XAML |

**Description**: UI activities should have explicit timeouts.

**Detection Pattern**:
```regex
<ui:(Click|TypeInto|GetText|SetText|ElementExists)[^>]*(?!Timeout)
```

**Fix Template**:
```json
{
  "searchPattern": "<ui:Click ",
  "replacePattern": "<ui:Click Timeout=\"30000\" "
}
```

---

### RULE-033: Selector Too Specific

| Property | Value |
|----------|-------|
| **ID** | RULE-033 |
| **Severity** | Medium |
| **Category** | Performance/Maintainability |
| **Auto-fixable** | No |
| **Applies to** | XAML |

**Description**: Selectors should not include volatile attributes.

**Volatile Attributes**:
- `idx` (index)
- Session IDs in title
- Timestamps
- Dynamic IDs

---

### RULE-034: Large DataTable in Memory

| Property | Value |
|----------|-------|
| **ID** | RULE-034 |
| **Severity** | Medium |
| **Category** | Performance |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Process large DataTables in batches.

---

### RULE-035: Unnecessary Excel Opens

| Property | Value |
|----------|-------|
| **ID** | RULE-035 |
| **Severity** | Medium |
| **Category** | Performance |
| **Auto-fixable** | No |
| **Applies to** | XAML |

**Description**: Multiple Excel operations should use a single scope.

---

## Maintainability Rules

### RULE-040: Workflow Too Complex

| Property | Value |
|----------|-------|
| **ID** | RULE-040 |
| **UiPath Rule** | [ST-DBP-002](https://docs.uipath.com/studio/standalone/latest/user-guide/st-dbp-002-high-arguments-count), [ST-MRD-009](https://docs.uipath.com/studio/standalone/latest/user-guide/st-mrd-009-deeply-nested-activities) |
| **Severity** | Medium |
| **Category** | Maintainability |
| **Auto-fixable** | No |
| **Applies to** | XAML |

**Thresholds**:
| Metric | Warning | Error |
|--------|---------|-------|
| Activities | 30 | 50 |
| Nesting depth | 4 | 6 |
| Arguments | 8 | 12 |
| Variables | 15 | 25 |

---

### RULE-041: Missing Documentation

| Property | Value |
|----------|-------|
| **ID** | RULE-041 |
| **Severity** | Low |
| **Category** | Maintainability |
| **Auto-fixable** | No |
| **Applies to** | XAML |

**Description**: Workflows should have annotations explaining purpose.

---

### RULE-042: Duplicate Code

| Property | Value |
|----------|-------|
| **ID** | RULE-042 |
| **Severity** | Medium |
| **Category** | Maintainability |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Similar code blocks should be extracted to reusable workflows.

---

### RULE-043: Magic Numbers

| Property | Value |
|----------|-------|
| **ID** | RULE-043 |
| **Severity** | Low |
| **Category** | Maintainability |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Use named constants or config values instead of magic numbers.

---

### RULE-044: Deep Nesting

| Property | Value |
|----------|-------|
| **ID** | RULE-044 |
| **UiPath Rule** | [ST-MRD-007](https://docs.uipath.com/studio/standalone/latest/user-guide/st-mrd-007-nested-if-clauses), [ST-DBP-007](https://docs.uipath.com/studio/standalone/latest/user-guide/st-dbp-007-multiple-flowchart-layers) |
| **Severity** | Medium |
| **Category** | Maintainability |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Avoid deeply nested control structures.

---

### RULE-045: Long Workflow

| Property | Value |
|----------|-------|
| **ID** | RULE-045 |
| **Severity** | Medium |
| **Category** | Maintainability |
| **Auto-fixable** | No |
| **Applies to** | XAML |

**Description**: Workflows with many activities should be split.

---

## Best Practice Rules

### RULE-050: Missing Logging

| Property | Value |
|----------|-------|
| **ID** | RULE-050 |
| **UiPath Rule** | [ST-USG-020](https://docs.uipath.com/studio/standalone/latest/user-guide/st-usg-020-minimum-log-messages) |
| **Severity** | Medium |
| **Category** | Best Practices |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Required Logging Points**:
- Workflow start
- Workflow end (success/failure)
- Before external API calls
- After external API calls
- Exception handling
- Business decisions

---

### RULE-051: Inconsistent Activity Naming

| Property | Value |
|----------|-------|
| **ID** | RULE-051 |
| **Severity** | Low |
| **Category** | Best Practices |
| **Auto-fixable** | No |
| **Applies to** | XAML |

**Recommended Patterns**:
- `Verb + Object`: "Read Config File"
- `Check + Condition`: "Check If Valid"
- `Set + Variable`: "Set Status"

---

### RULE-052: Missing Input Validation

| Property | Value |
|----------|-------|
| **ID** | RULE-052 |
| **Severity** | High |
| **Category** | Best Practices |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Input arguments should be validated at workflow start.

---

### RULE-053: HTTP Without Error Check

| Property | Value |
|----------|-------|
| **ID** | RULE-053 |
| **Severity** | High |
| **Category** | Best Practices |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: HTTP responses should check status code before processing.

---

### RULE-054: Missing Transaction Status

| Property | Value |
|----------|-------|
| **ID** | RULE-054 |
| **Severity** | High |
| **Category** | Best Practices |
| **Auto-fixable** | No |
| **Applies to** | XAML |

**Description**: Queue transactions should set status (Success/Failed).

---

### RULE-055: Unused Variables

| Property | Value |
|----------|-------|
| **ID** | RULE-055 |
| **UiPath Rule** | [ST-USG-009](https://docs.uipath.com/studio/standalone/latest/user-guide/st-usg-009-unused-variables) |
| **Severity** | Low |
| **Category** | Best Practices |
| **Auto-fixable** | Yes |
| **Applies to** | XAML, C# |

**Description**: Remove variables that are declared but never used.

---

### RULE-056: Unused Arguments

| Property | Value |
|----------|-------|
| **ID** | RULE-056 |
| **Severity** | Medium |
| **Category** | Best Practices |
| **Auto-fixable** | No |
| **Applies to** | XAML, C# |

**Description**: Remove arguments that are declared but never used.

---

## XAML-Specific Rules

### RULE-060: Missing Namespace

| Property | Value |
|----------|-------|
| **ID** | RULE-060 |
| **Severity** | High |
| **Category** | XAML |
| **Auto-fixable** | Yes |
| **Applies to** | XAML |

**Description**: Required namespaces must be declared.

---

### RULE-061: Invalid Selector

| Property | Value |
|----------|-------|
| **ID** | RULE-061 |
| **Severity** | High |
| **Category** | XAML |
| **Auto-fixable** | No |
| **Applies to** | XAML |

**Description**: Selectors must be valid XML and properly escaped.

---

### RULE-062: Missing WorkflowViewState

| Property | Value |
|----------|-------|
| **ID** | RULE-062 |
| **Severity** | Low |
| **Category** | XAML |
| **Auto-fixable** | Yes |
| **Applies to** | XAML |

**Description**: Activities should have WorkflowViewState.IdRef for designer.

---

### RULE-063: Multiline Activity Tag

| Property | Value |
|----------|-------|
| **ID** | RULE-063 |
| **Severity** | Critical |
| **Category** | XAML |
| **Auto-fixable** | Yes |
| **Applies to** | XAML |

**Description**: Opening Activity tag with namespaces must be on single line.

---

## C#-Specific Rules

### RULE-070: Missing Argument Attribute

| Property | Value |
|----------|-------|
| **ID** | RULE-070 |
| **Severity** | High |
| **Category** | C# |
| **Auto-fixable** | Yes |
| **Applies to** | C# |

**Description**: Workflow parameters need [Argument] attribute.

---

### RULE-071: Missing Using Statement

| Property | Value |
|----------|-------|
| **ID** | RULE-071 |
| **Severity** | Medium |
| **Category** | C# |
| **Auto-fixable** | Yes |
| **Applies to** | C# |

**Description**: Disposable resources should use using statement.

---

### RULE-072: Async Without Await

| Property | Value |
|----------|-------|
| **ID** | RULE-072 |
| **Severity** | High |
| **Category** | C# |
| **Auto-fixable** | No |
| **Applies to** | C# |

**Description**: Async methods should await async operations.

---

## Project Configuration Rules

### RULE-080: Missing Dependency

| Property | Value |
|----------|-------|
| **ID** | RULE-080 |
| **Severity** | High |
| **Category** | Configuration |
| **Auto-fixable** | Yes |
| **Applies to** | project.json |

**Description**: Required packages must be in dependencies.

---

### RULE-081: Outdated Dependency

| Property | Value |
|----------|-------|
| **ID** | RULE-081 |
| **Severity** | Medium |
| **Category** | Configuration |
| **Auto-fixable** | Yes |
| **Applies to** | project.json |

**Description**: Dependencies should use recent stable versions.

---

### RULE-082: Missing Entry Point

| Property | Value |
|----------|-------|
| **ID** | RULE-082 |
| **Severity** | Critical |
| **Category** | Configuration |
| **Auto-fixable** | No |
| **Applies to** | project.json |

**Description**: Project must have defined entry points.

---

### RULE-083: Invalid Target Framework

| Property | Value |
|----------|-------|
| **ID** | RULE-083 |
| **Severity** | High |
| **Category** | Configuration |
| **Auto-fixable** | No |
| **Applies to** | project.json |

**Description**: Target framework must be compatible with deployment.

---

### RULE-084: Missing Runtime Options

| Property | Value |
|----------|-------|
| **ID** | RULE-084 |
| **Severity** | Medium |
| **Category** | Configuration |
| **Auto-fixable** | Yes |
| **Applies to** | project.json |

**Description**: Runtime options should be explicitly configured.
