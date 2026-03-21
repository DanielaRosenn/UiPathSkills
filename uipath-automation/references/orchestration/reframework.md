# Module: REFramework (Robotic Enterprise Framework)
**Template: UiPath.REFramework | Studio 2025.10**
**GitHub: github.com/UiPath/ReFrameWork**

---

## What REFramework Is

A State Machine-based project template enforcing enterprise best practices:
- Structured error handling with automatic retry
- Config-driven settings (no hardcoded values)
- Comprehensive logging
- Application recovery on failure
- Dispatcher/Performer pattern for queue processing

**Use REFramework when**: process is transaction-based, high volume, unattended, or requires robust error handling.

---

## State Machine Structure

```
Main.xaml (State Machine)
├── Initialization State
│   ├── Entry: InitAllSettings.xaml → Config (Dictionary)
│   │         InitAllApplications.xaml
│   ├── Transition → Get Transaction Data (if Init OK)
│   └── Transition → End Process (if System Exception)
│
├── Get Transaction Data State
│   ├── Entry: GetTransactionData.xaml → TransactionItem
│   ├── Transition → Process Transaction (if item found)
│   └── Transition → End Process (if no more items / Nothing)
│
├── Process Transaction State
│   ├── Entry: Process.xaml (your business logic)
│   │         SetTransactionStatus.xaml
│   ├── Transition → Get Transaction Data (if success / business exception)
│   ├── Transition → Get Transaction Data (if retry limit not reached)
│   └── Transition → End Process (if max system exceptions exceeded)
│
└── End Process State
    └── Entry: CloseAllApplications.xaml
              KillAllProcesses.xaml
```

---

## Config.xlsx Structure

Located at `Data/Config.xlsx`

### Settings Sheet (runtime-configurable)
| Name | Value | Description |
|---|---|---|
| OrchestratorQueueName | InvoiceQueue | Orchestrator queue name |
| System1_URL | https://app.example.com | App URL (read from Orchestrator Asset if env differs) |
| MaxRetryNumber | 3 | System exception retry count |
| MaxConsecutiveSystemExceptions | 3 | Stop process after N consecutive failures |

### Constants Sheet (hardcoded, non-configurable)
| Name | Value | Description |
|---|---|---|
| logF_BusinessProcessName | InvoiceProcessor | Process name for logs |
| logF_TransactionID | TransactionId | Log field name |
| REFramework_Version | 2.0 | Framework version |

### Accessing Config Values
```vb
' String value:
Config("System1_URL").ToString

' Integer value:
CInt(Config("MaxRetryNumber"))

' Boolean value:
CBool(Config("UseHeaderRow"))

' Asset value (added by InitAllSettings from Orchestrator):
Config("System1_Credential")   ← NetworkCredential object
```

---

## Key Workflow Files

### InitAllSettings.xaml
- Reads `Config.xlsx` (Settings + Constants sheets) → Dictionary
- Gets Orchestrator Assets and merges into Config
- Sets environment-specific overrides
- **What to add**: any new Config keys, additional Assets to load

### InitAllApplications.xaml
- Opens all applications the process needs
- Logs in using credentials from Config
- **Pattern**:
```
Get Credential (AssetName = Config("System1_Credential").ToString)
  → Username (String), Password (SecureString)
Navigate To / Open Application
Type Into (Username field, Username)
TypeSecureText (Password field, Password)
Click (Login button)
Wait For Element (Dashboard element)
Log Message (Info): "Logged into System1 successfully"
```

### GetTransactionData.xaml

**With Orchestrator Queues** (default):
```
Get Transaction Item (QueueName = Config("OrchestratorQueueName").ToString,
                      Filter = [optional OData filter]) → out_TransactionItem (QueueItem)
' If queue empty: Get Transaction Item returns Nothing
' Signal end: out_TransactionItem = Nothing
```

**Without Queues (linear / tabular data)**:
```
' On first call (in_TransactionNumber = 1):
'   Read Excel → dt_Transactions (store as module-level variable)
' On each call:
If in_TransactionNumber > dt_Transactions.Rows.Count Then
  out_TransactionItem = Nothing   ← signals end of data
Else
  out_TransactionItem = dt_Transactions.Rows(in_TransactionNumber - 1)
  ' Change out_TransactionItem type to DataRow (update all 3 files)
```

**Changing Transaction Type from QueueItem to DataRow**:
1. `GetTransactionData.xaml`: change `out_TransactionItem` type to DataRow
2. `Process.xaml`: change `in_TransactionItem` type to DataRow
3. `SetTransactionStatus.xaml`: change `in_TransactionItem` type to DataRow

### Process.xaml (your business logic lives here)
```
' in_TransactionItem: QueueItem (or DataRow if customized)
' in_Config: Dictionary(Of String, Object)

' Extract data from transaction:
Assign: invoiceNumber = in_TransactionItem.SpecificContent("InvoiceNumber").ToString
Assign: amount = CDbl(in_TransactionItem.SpecificContent("Amount"))
Assign: customerName = in_TransactionItem.SpecificContent("CustomerName").ToString

' Your business logic:
Invoke Workflow: Processes/ValidateInvoice.xaml (in_Amount = amount)
Invoke Workflow: Processes/ProcessPayment.xaml (in_Invoice = invoiceNumber)

' Any unhandled exception here:
'   BusinessRuleException → transaction marked Failed (Business Exception), no retry
'   Any other exception → transaction retried (System Exception)
```

### SetTransactionStatus.xaml
- Called after every transaction attempt
- Sets Orchestrator queue item status: Success, BusinessException, ApplicationException
- Logs transaction completion

### CloseAllApplications.xaml / KillAllProcesses.xaml
- Mirror of InitAllApplications.xaml (symmetric)
- Close everything opened in Init
- KillAllProcesses: force-kill processes if graceful close fails

---

## Exception Types

### BusinessRuleException
- Inherits from Exception
- **Meaning**: the data itself is invalid — retrying won't help
- **Queue effect**: marks item as `Failed` with `BusinessException` status (no retry)
- **When to throw**:
```vb
If amount <= 0 Then
  Throw New BusinessRuleException("Invoice amount must be positive: " + amount.ToString)
End If
If customerName = "" Then
  Throw New BusinessRuleException("Customer name is required")
End If
```

### System Exception (any non-BusinessRuleException)
- **Meaning**: transient system error — retry may succeed
- **Queue effect**: marks item as `ApplicationException`, increments retry counter
- **Automatic retry**: up to `MaxRetryNumber` times
- Examples: selector not found, timeout, network error, application crashed

---

## Dispatcher / Performer Pattern

### Dispatcher Project
- Reads source data (Excel, DB, email, API)
- Adds items to Orchestrator Queue
```
Read Range → dt_Invoices
For Each Row in dt_Invoices
  Add Queue Item (QueueName = Config("OrchestratorQueueName").ToString,
                  ItemInformation = {
                    "InvoiceNumber": row("InvoiceNumber").ToString,
                    "Amount": row("Amount").ToString,
                    "CustomerName": row("CustomerName").ToString
                  })
Log Message: dt_Invoices.Rows.Count.ToString + " items added to queue"
```

### Performer Project
- Reads from queue via REFramework
- `GetTransactionData.xaml` uses `Get Transaction Item`
- `Process.xaml` contains processing logic

---

## Customization Checklist

When starting a REFramework project:
- [ ] Update `Config.xlsx` Settings sheet with process-specific keys
- [ ] Update `Config.xlsx` Constants sheet (`logF_BusinessProcessName`)
- [ ] Remove `MaxRetryNumber` row if using Orchestrator queue retry settings
- [ ] Add Assets to load in `InitAllSettings.xaml`
- [ ] Add login steps to `InitAllApplications.xaml`
- [ ] Mirror cleanup in `CloseAllApplications.xaml`
- [ ] Set `out_TransactionItem` type (QueueItem vs DataRow vs custom)
- [ ] Add business logic to `Process.xaml`
- [ ] Set `TransactionId`/`TransactionField1` for logging in `GetTransactionData.xaml`
- [ ] Add `BusinessRuleException` throws for all data validation failures
- [ ] Test all 4 states independently

---

## When NOT to Use REFramework

- Simple linear processes with no transactions → use basic Process template
- Attended automation → basic Process template (user triggers)
- Very simple single-run scripts → basic template
- Processes with < 10 transactions that never retry → overhead not worth it

**Use REFramework for**: any unattended process that processes multiple items, requires retry logic, or runs for extended periods.
