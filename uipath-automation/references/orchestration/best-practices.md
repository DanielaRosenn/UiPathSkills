# Module: Best Practices & Workflow Analyzer Rules
**Source:** UiPath Studio 2025.10 docs | **Covers:** Naming, structure, ST-* rules, security, project organization

---

## Naming Conventions (ST-NMG-*)

### Arguments
| Prefix | Direction | Example |
|---|---|---|
| `in_` | Input | `in_FilePath`, `in_Config`, `in_TransactionItem` |
| `out_` | Output | `out_Result`, `out_Status`, `out_ErrorMessage` |
| `io_` | In/Out | `io_RetryCount`, `io_DataTable` |

Use PascalCase after prefix: `in_CustomerName`, NOT `in_customer_name`

### Variables
- camelCase: `filePath`, `transactionId`, `isProcessed`
- Boolean prefix: `is`, `has`, `can` — `isValid`, `hasAttachments`
- DataTable prefix: `dt` — `dtOrders`, `dtResults`
- List prefix: `list` — `listFilePaths`, `listEmails`
- Counter: descriptive — `retryCount`, `processedCount`

### Workflow files
- Verb + Noun: `GetTransactionData.xaml`, `ProcessInvoice.xaml`, `SendNotification.xaml`
- Exception: `Main.xaml` (always named Main)
- CamelCase or PascalCase: `InitAllSettings.xaml`

### Activity Display Names
- Descriptive action: `Click Submit Button`, `Get Invoice Number from ERP`, `Validate Email Format`
- Never leave defaults: not `Click 1`, `Assign`, `Sequence`

---

## Project Structure

```
MyProject/
├── Main.xaml                          ← entry point, orchestrator only
├── project.json
├── GlobalExceptionHandler.xaml        ← registered in project.json
├── Data/
│   └── Config.xlsx                    ← runtime config, never hardcode
├── Workflows/
│   ├── Framework/
│   │   ├── InitAllSettings.xaml
│   │   ├── InitAllApplications.xaml
│   │   ├── GetTransactionData.xaml
│   │   └── CloseAllApplications.xaml
│   ├── Process/
│   │   ├── Process.xaml               ← business logic entry
│   │   └── [sub-workflows]
│   └── Utilities/
│       ├── TakeScreenshot.xaml
│       └── BuildLogMessage.xaml
├── ObjectRepository/
│   └── ObjectRepository.uimap
├── Logs/
│   └── Screenshots/                   ← runtime generated
└── Tests/
    └── [test cases]
```

---

## Workflow Analyzer Rules (ST-* codes)

### Naming Rules
| Rule | Description | Severity |
|---|---|---|
| ST-NMG-001 | Variable naming convention (camelCase) | Warning |
| ST-NMG-002 | Argument naming convention (prefix required) | Error |
| ST-NMG-003 | Activity display name must be set | Warning |
| ST-NMG-004 | Workflow file naming (verb + noun) | Info |
| ST-NMG-007 | Default workflow name (don't use "Sequence1") | Warning |

### Design Rules
| Rule | Description | Severity |
|---|---|---|
| ST-DBP-001 | Unused variables | Warning |
| ST-DBP-002 | Unused arguments | Warning |
| ST-DBP-003 | Empty catch blocks | Error |
| ST-DBP-004 | Too many activities in workflow (>30 recommended) | Warning |
| ST-DBP-006 | Workflow too nested (max 4 levels deep) | Warning |
| ST-DBP-007 | Hardcoded values (URLs, paths, credentials) | Warning |
| ST-DBP-020 | Missing GlobalExceptionHandler | Warning |

### Security Rules
| Rule | Description | Severity |
|---|---|---|
| ST-SEC-001 | Hardcoded credentials | Error |
| ST-SEC-002 | Use SecureString for passwords | Warning |
| ST-SEC-010 | Private=True on activities handling sensitive data | Info |

### Maintainability Rules
| Rule | Description | Severity |
|---|---|---|
| ST-MNT-001 | Workflow has no description/annotation | Info |
| ST-MNT-002 | Variable description missing | Info |
| ST-RDY-001 | Missing await on async activities | Error |

---

## Security Best Practices

### Never hardcode credentials
```xml
<!-- WRONG: -->
<ui:TypeInto Text="&quot;MyPassword123&quot;"/>

<!-- RIGHT: get from Orchestrator Assets -->
<ui:GetCredential AssetName="&quot;AppCredential&quot;" Username="[username]" SecurePassword="[securePass]"/>
<ui:TypeInto Text="[New System.Net.NetworkCredential(String.Empty, securePass).Password]"/>
```

### Mark sensitive activities as Private
```xml
<ui:TypeInto DisplayName="Enter Password" Private="True"
    Target.Selector="[pwdSelector]" Text="[password]"/>
<!-- Private=True prevents value logging at Verbose level -->
```

### Never log sensitive data
```vb
' WRONG:
LogMessage("Processing user " + ssn + " with card " + cardNumber)

' RIGHT:
LogMessage("Processing user ID " + userId.Substring(0,4) + "***")
```

---

## Performance Best Practices

### Selectors
- Avoid `idx` attributes — positional, breaks on layout change
- Prefer `id`, `name`, `automationid` — stable attributes
- Use `*` wildcards for dynamic title parts: `title='Invoice *'`

### DataTable operations
- Filter in the query (SQL WHERE clause) rather than loading all rows then filtering in UiPath
- Use `Filter Data Table` activity instead of looping and checking conditions
- For large tables: use `Invoke Code` with LINQ for complex filtering

### Workflow size
- Keep workflows under 30 activities — extract sub-workflows if larger
- Keep nesting depth under 4 levels
- Avoid Flowcharts for linear processes — Sequences are faster

### UI Automation
- Use `SimulateType`/`SimulateClick` for background execution (no window focus needed)
- Set `WaitForReady="Interactive"` instead of `Complete` when full page load isn't needed
- Use `WaitElementAppear` instead of fixed `Delay`

---

## Dependency Management

```json
// project.json — keep dependencies explicit and pinned
{
  "dependencies": {
    "UiPath.Excel.Activities": "[2.23.1, )",
    "UiPath.Mail.Activities": "[1.20.0, )",
    "UiPath.UIAutomation.Activities": "[25.10.0, )",
    "UiPath.System.Activities": "[25.10.0, )"
  }
}
```

- Use `[MinVersion, )` for minimum-bound ranges in production
- Lock exact versions for regulated environments: `"[25.10.0]"`
- Never use `*` (any version) in production projects
