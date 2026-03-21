# Module: Orchestrator Activities
**Package:** UiPath.System.Activities (Orchestrator section)
**Covers:** Queue items, assets, credentials, storage buckets, job control, logging

---

## Queue Activities

### Add Queue Item
```xml
<ui:AddQueueItem DisplayName="Add to Invoice Queue"
    QueueName="&quot;InvoiceProcessing&quot;"
    ItemInformation="[transactionData]"
    Priority="Normal"
    Reference="[invoiceNumber]"
    Deadline="[DateTime.Now.AddHours(4)]"
    QueueItemOutput="[queueRef]"/>
<!-- transactionData = Dictionary(Of String, Object) -->
```

### Bulk Add Queue Items (DataTable → Queue)
```xml
<ui:BulkAddQueueItems DisplayName="Bulk Load Queue"
    QueueName="&quot;InvoiceProcessing&quot;"
    DataTable="[dtItems]"
    NumberOfSuccessfulTransactions="[successCount]"/>
<!-- DataTable column names become SpecificContent keys -->
<!-- Special columns: Reference, Deadline, Priority → set those queue fields -->
```

### Get Transaction Item (lock + dequeue)
```xml
<ui:GetQueueItem DisplayName="Get Next Transaction"
    QueueName="&quot;InvoiceProcessing&quot;"
    TransactionItem="[txItem]"/>
<!-- Returns Nothing when queue empty → check IsNothing(txItem) to end process -->
```

### Set Transaction Status
```xml
<!-- Success -->
<ui:SetTransactionStatus DisplayName="Mark Successful"
    TransactionItem="[txItem]" Status="Successful"
    Output="[&quot;Processed: &quot; + invoiceId]"/>

<!-- Business Exception (no auto-retry) -->
<ui:SetTransactionStatus DisplayName="Mark Business Exception"
    TransactionItem="[txItem]" Status="Failed"
    ErrorType="BusinessException"
    ExceptionReason="[bizExMessage]"/>

<!-- System Exception (auto-retried up to MaxRetryNumber) -->
<ui:SetTransactionStatus DisplayName="Mark System Exception"
    TransactionItem="[txItem]" Status="Failed"
    ErrorType="ApplicationException"
    ExceptionReason="[exception.Message]"/>
```

### Reading Queue Item Data
```vb
txItem.SpecificContent("InvoiceID").ToString()
CInt(txItem.SpecificContent("Amount"))
CDate(txItem.SpecificContent("DueDate"))
txItem.Reference                    ' the Reference string
txItem.RetryNo                      ' how many times retried (Int32)
txItem.Id                           ' Orchestrator GUID
```

---

## Asset Activities

### Get Asset
```xml
<ui:GetRobotAsset DisplayName="Get API Base URL"
    AssetName="&quot;ExternalAPI_BaseURL&quot;"
    RobotAsset="[assetObj]"/>
<!-- assetObj.StringValue, assetObj.IntegerValue, assetObj.BooleanValue -->
```

### Get Credential (username + password)
```xml
<ui:GetCredential DisplayName="Get SAP Credentials"
    AssetName="&quot;SAP_Credentials&quot;"
    Username="[username]"
    SecurePassword="[securePass]"/>
<!-- Convert SecureString: New Net.NetworkCredential(String.Empty, securePass).Password -->
```

### Set Asset (update value at runtime)
```xml
<ui:SetAsset DisplayName="Update Last Run Time"
    AssetName="&quot;LastRunTimestamp&quot;"
    AssetStringValue="[DateTime.Now.ToString(&quot;yyyy-MM-dd HH:mm:ss&quot;)]"/>
```

---

## Storage Bucket Activities

```xml
<!-- Upload -->
<ui:UploadStorageBucketFile DisplayName="Upload Report"
    BucketName="&quot;ReportsBucket&quot;"
    BlobFileName="[&quot;Reports/&quot; + reportName]"
    LocalFilePath="[localPath]"/>

<!-- Download -->
<ui:DownloadStorageBucketFile DisplayName="Download Template"
    BucketName="&quot;TemplatesBucket&quot;"
    BlobFileName="&quot;templates/invoice.xlsx&quot;"
    LocalFilePath="[localDestPath]" Overwrite="True"/>

<!-- List -->
<ui:ListStorageBucketFiles DisplayName="List Reports"
    BucketName="&quot;ReportsBucket&quot;"
    Directory="&quot;Reports/202501&quot;"
    Files="[bucketFiles]"/>
```

---

## Job Control

```xml
<!-- Trigger another process -->
<ui:StartJob DisplayName="Start Report Generator"
    ProcessName="&quot;GenerateMonthlyReport&quot;"
    Strategy="JobsCount" JobsCount="1"
    InputArguments="[&quot;{'in_Month': '&quot; + currentMonth + &quot;'}&quot;]"
    JobId="[startedJobId]"/>

<!-- Wait for completion -->
<ui:WaitForJob DisplayName="Wait for Report Job"
    JobId="[startedJobId]"
    Timeout="[TimeSpan.FromMinutes(30)]"
    Job="[completedJob]"/>
<!-- completedJob.State: Successful, Faulted, Stopped -->
```

---

## Logging (Orchestrator + Robot logs)

```xml
<ui:LogMessage DisplayName="Log Transaction Start" Level="Info"
    Message="[&quot;[TxID=&quot; + txId + &quot;] Processing started. Attempt &quot; + retryNo.ToString()]"/>
```

**Log Levels & When to Use:**
| Level | Use For |
|---|---|
| `Info` | Normal milestones: start, end, key steps completed |
| `Warn` | Recoverable: retry used, fallback applied, partial result |
| `Error` | Handled failure: business exception, selector not found |
| `Fatal` | Global handler only — unrecoverable crash |
| `Trace` | Dev debugging only — never in production |

**Structured log pattern (searchable in Orchestrator):**
```vb
"[BatchID=" & batchId & "] [InvoiceID=" & invoiceId & "] [Status=Success] Processed " & amount.ToString("C")
```

---

## Config Dictionary Pattern

Standard way to load Config.xlsx into a reusable dictionary (REF and non-REF projects):

```vb
' Invoke Code to build config dictionary from DataTable:
Dim dict As New Dictionary(Of String, Object)
For Each row As DataRow In dtSettings.Rows
    dict(row("Name").ToString()) = row("Value")
Next
For Each row As DataRow In dtConstants.Rows
    dict(row("Name").ToString()) = row("Value")
Next
out_Config = dict
```

Access anywhere: `in_Config("QueueName").ToString()`, `CInt(in_Config("MaxRetries"))`
