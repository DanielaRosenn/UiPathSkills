# Auto-Fix Guide

How to automatically apply fix suggestions from code reviews.

## Overview

The UiPath Code Reviewer generates `fix-suggestions.json` containing structured fix information. This guide explains how to apply these fixes automatically or manually.

## Fix Suggestion Structure

```json
{
  "id": "FIX-001",
  "ruleId": "RULE-001",
  "severity": "medium",
  "file": "Main.xaml",
  "line": 23,
  "column": 5,
  "description": "Variable 'filename' should use naming convention",
  "category": "naming",
  "currentCode": "<Variable x:TypeArguments=\"x:String\" Name=\"filename\" />",
  "suggestedCode": "<Variable x:TypeArguments=\"x:String\" Name=\"strFileName\" />",
  "autoFixable": true,
  "fixType": "replace",
  "searchPattern": "Name=\"filename\"",
  "replacePattern": "Name=\"strFileName\"",
  "additionalReplacements": [
    {"search": "[filename]", "replace": "[strFileName]"},
    {"search": "[filename.", "replace": "[strFileName."}
  ],
  "manualSteps": null
}
```

## Fix Types

### 1. Simple Replace (`fixType: "replace"`)

Direct string replacement in the file.

**Example**: Variable naming fix

```json
{
  "fixType": "replace",
  "searchPattern": "Name=\"filename\"",
  "replacePattern": "Name=\"strFileName\"",
  "additionalReplacements": [
    {"search": "[filename]", "replace": "[strFileName]"}
  ]
}
```

**Application**:
```javascript
function applySimpleReplace(fileContent, fix) {
  let result = fileContent.replace(fix.searchPattern, fix.replacePattern);
  
  if (fix.additionalReplacements) {
    for (const replacement of fix.additionalReplacements) {
      result = result.replaceAll(replacement.search, replacement.replace);
    }
  }
  
  return result;
}
```

### 2. Insert (`fixType: "insert"`)

Insert new content at a specific location.

**Example**: Add missing timeout

```json
{
  "fixType": "insert",
  "insertAfter": "<ui:Click ",
  "insertContent": "Timeout=\"30000\" "
}
```

### 3. Wrap (`fixType: "wrap"`)

Wrap existing content with new structure.

**Example**: Add Try-Catch

```json
{
  "fixType": "wrap",
  "targetStart": "<ui:HttpClient",
  "targetEnd": "/>",
  "wrapBefore": "<TryCatch DisplayName=\"Handle HTTP Errors\">\n  <TryCatch.Try>\n    <Sequence>\n",
  "wrapAfter": "\n    </Sequence>\n  </TryCatch.Try>\n  <TryCatch.Catches>\n    <Catch x:TypeArguments=\"s:Exception\">\n      <ActivityAction x:TypeArguments=\"s:Exception\">\n        <ActivityAction.Argument>\n          <DelegateInArgument x:TypeArguments=\"s:Exception\" Name=\"exception\" />\n        </ActivityAction.Argument>\n        <ui:LogMessage Level=\"Error\" Message=\"[exception.Message]\" />\n      </ActivityAction>\n    </Catch>\n  </TryCatch.Catches>\n</TryCatch>"
}
```

### 4. Delete (`fixType: "delete"`)

Remove content from the file.

**Example**: Remove unused variable

```json
{
  "fixType": "delete",
  "searchPattern": "<Variable x:TypeArguments=\"x:String\" Name=\"strUnused\" />\n"
}
```

### 5. Multi-File (`fixType: "multi-file"`)

Changes span multiple files.

**Example**: Rename workflow file and update references

```json
{
  "fixType": "multi-file",
  "changes": [
    {
      "file": "process_data.xaml",
      "action": "rename",
      "newName": "ProcessData.xaml"
    },
    {
      "file": "Main.xaml",
      "action": "replace",
      "searchPattern": "WorkflowFileName=\"process_data.xaml\"",
      "replacePattern": "WorkflowFileName=\"ProcessData.xaml\""
    }
  ]
}
```

## Auto-Fix Process

### Step 1: Load Fix Suggestions

```javascript
const fixes = JSON.parse(fs.readFileSync('fix-suggestions.json'));
```

### Step 2: Filter Auto-Fixable Issues

```javascript
const autoFixes = fixes.fixes.filter(f => f.autoFixable);
```

### Step 3: Group by File

```javascript
const fixesByFile = autoFixes.reduce((acc, fix) => {
  if (!acc[fix.file]) acc[fix.file] = [];
  acc[fix.file].push(fix);
  return acc;
}, {});
```

### Step 4: Apply Fixes (Bottom-Up)

Apply fixes from bottom of file to top to preserve line numbers.

```javascript
function applyFixes(filePath, fixes) {
  let content = fs.readFileSync(filePath, 'utf-8');
  
  // Sort by line number descending
  const sortedFixes = fixes.sort((a, b) => b.line - a.line);
  
  for (const fix of sortedFixes) {
    content = applyFix(content, fix);
  }
  
  fs.writeFileSync(filePath, content);
}
```

### Step 5: Validate Changes

After applying fixes, re-run the code review to verify:
- All auto-fixed issues are resolved
- No new issues were introduced
- File is still valid XAML/C#

## Auto-Fixable Rules

| Rule ID | Description | Fix Type |
|---------|-------------|----------|
| RULE-001 | Variable naming | replace |
| RULE-002 | Argument naming | replace |
| RULE-004 | Activity DisplayName | replace |
| RULE-023 | Missing SecureString | replace |
| RULE-032 | Missing timeout | insert |
| RULE-055 | Unused variables | delete |
| RULE-060 | Missing namespace | insert |
| RULE-062 | Missing WorkflowViewState | insert |
| RULE-063 | Multiline Activity tag | replace |
| RULE-070 | Missing Argument attribute | insert |
| RULE-071 | Missing using statement | insert |
| RULE-080 | Missing dependency | insert |
| RULE-081 | Outdated dependency | replace |
| RULE-084 | Missing runtime options | insert |

## Manual Fix Rules

These rules require human judgment and cannot be auto-fixed:

| Rule ID | Description | Why Manual |
|---------|-------------|------------|
| RULE-003 | Workflow file naming | Requires file rename + reference updates |
| RULE-010 | Missing Try-Catch | Requires understanding of error handling strategy |
| RULE-011 | Empty catch block | Requires decision on how to handle exception |
| RULE-012 | Generic exception only | Requires knowledge of expected exceptions |
| RULE-013 | Missing retry logic | Requires understanding of transient vs permanent errors |
| RULE-020 | Hardcoded credentials | Requires Orchestrator asset setup |
| RULE-021 | Hardcoded file paths | Requires config structure decision |
| RULE-022 | Sensitive data in logs | Requires review of what should be logged |
| RULE-024 | SQL injection risk | Requires query restructuring |
| RULE-030 | Excessive delays | Requires understanding of why delay exists |
| RULE-031 | Inefficient loop | Requires LINQ knowledge |
| RULE-033 | Selector too specific | Requires UI understanding |
| RULE-040 | Workflow too complex | Requires architectural decision |
| RULE-042 | Duplicate code | Requires refactoring strategy |
| RULE-052 | Missing input validation | Requires validation rules |
| RULE-053 | HTTP without error check | Requires error handling strategy |

## Fix Templates

### Variable Naming Fix Template

```json
{
  "ruleId": "RULE-001",
  "fixType": "replace",
  "searchPattern": "Name=\"{oldName}\"",
  "replacePattern": "Name=\"{newName}\"",
  "additionalReplacements": [
    {"search": "[{oldName}]", "replace": "[{newName}]"},
    {"search": "[{oldName}.", "replace": "[{newName}."},
    {"search": "[{oldName} ", "replace": "[{newName} "},
    {"search": "[{oldName}+", "replace": "[{newName}+"},
    {"search": "[{oldName}&", "replace": "[{newName}&"},
    {"search": "({oldName})", "replace": "({newName})"}
  ]
}
```

### Add Timeout Fix Template

```json
{
  "ruleId": "RULE-032",
  "fixType": "insert",
  "patterns": [
    {
      "match": "<ui:Click ",
      "insert": "Timeout=\"30000\" ",
      "position": "after"
    },
    {
      "match": "<ui:TypeInto ",
      "insert": "Timeout=\"30000\" ",
      "position": "after"
    },
    {
      "match": "<ui:GetText ",
      "insert": "Timeout=\"30000\" ",
      "position": "after"
    }
  ]
}
```

### Add Missing Namespace Fix Template

```json
{
  "ruleId": "RULE-060",
  "fixType": "insert",
  "insertBefore": ">",
  "insertContent": " xmlns:{prefix}=\"{namespace}\"",
  "targetElement": "<Activity"
}
```

## Batch Fix Commands

### Fix All Auto-Fixable Issues

```bash
# Pseudo-command for AI to execute
uipath-review --fix-all --auto-only ./ProjectFolder
```

### Fix Specific Rule

```bash
# Fix only naming convention issues
uipath-review --fix --rule RULE-001 ./ProjectFolder
```

### Fix by Severity

```bash
# Fix all critical and high severity auto-fixable issues
uipath-review --fix --severity critical,high ./ProjectFolder
```

### Dry Run

```bash
# Show what would be fixed without making changes
uipath-review --fix --dry-run ./ProjectFolder
```

## AI Implementation Guide

When the AI applies fixes:

### 1. Read the Fix Suggestion

```
Fix ID: FIX-001
Rule: RULE-001 (Variable Naming)
File: Main.xaml
Auto-fixable: Yes
```

### 2. Read the Target File

Use the Read tool to get the current file content.

### 3. Apply the Fix

Use StrReplace tool with:
- `old_string`: The current code pattern
- `new_string`: The fixed code pattern
- `replace_all`: true if fix should apply to all occurrences

### 4. Apply Additional Replacements

For each additional replacement in the fix:
- Use StrReplace with `replace_all: true`

### 5. Verify the Fix

- Read the file again
- Check that the fix was applied correctly
- Run linter to ensure no syntax errors

### 6. Update Fix Status

Mark the fix as applied in the tracking system.

## Example: Applying a Naming Fix

**Fix Suggestion**:
```json
{
  "id": "FIX-001",
  "ruleId": "RULE-001",
  "file": "Main.xaml",
  "searchPattern": "Name=\"filename\"",
  "replacePattern": "Name=\"strFileName\"",
  "additionalReplacements": [
    {"search": "[filename]", "replace": "[strFileName]"}
  ]
}
```

**AI Actions**:

1. Read Main.xaml
2. StrReplace: `Name="filename"` → `Name="strFileName"`
3. StrReplace (replace_all): `[filename]` → `[strFileName]`
4. Read Main.xaml to verify
5. Report: "Fixed RULE-001: Renamed variable 'filename' to 'strFileName'"

## Rollback

If a fix causes issues:

1. Use git to revert changes: `git checkout -- {file}`
2. Mark fix as "failed" in tracking
3. Add to manual review queue
4. Document why auto-fix failed

## Best Practices

1. **Always backup** before applying fixes
2. **Apply one fix at a time** for complex changes
3. **Verify after each fix** with linter/parser
4. **Test the workflow** after all fixes applied
5. **Document manual fixes** that were skipped
6. **Re-run review** to confirm all issues resolved
