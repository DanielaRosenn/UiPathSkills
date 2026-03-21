# Module: Excel & Office Activities
**Packages: UiPath.Excel.Activities | UiPath.Mail.Activities | Studio 2025.10**

---

## Excel Activities (Modern)

### Use Excel File (Container — Modern, preferred)
```
Use Excel File
  WorkbookPath = in_FilePath   ← always from argument/Config
  [Excel activities go inside]
  ReadRow / ReadColumn / ReadCell / Read Range
  Write Cell / Write Range / Append Range
  For Each Excel Row (loop over sheet)
```
**Note (2025.10)**: In Windows projects, you can add Excel files via Data Manager → Resources, and use without explicit scope — but explicit `Use Excel File` is clearer and recommended.

### Read Range (Workbook)
| Property | Description |
|---|---|
| SheetName | Tab name (String) |
| Range | e.g. "A1:D100" or "" for entire sheet |
| DataTable | Output DataTable |
| AddHeaders | First row = column headers (default True) |
| PreserveFormat | Keep cell formatting |

```vb
' After Read Range:
' dt_CustomerData now contains all rows
' Access cell: dt_CustomerData.Rows(0)("ColumnName").ToString
```

### Write Range (Workbook)
| Property | Description |
|---|---|
| SheetName | Target sheet |
| StartingCell | Top-left cell "A1" |
| DataTable | DataTable to write |
| AddHeaders | Write column names as first row |

### Append Range
- Appends DataTable rows below existing data
- No need to find last row manually

### For Each Excel Row (Modern)
```
For Each Excel Row (CurrentRow)
  in: Use Excel File scope
  TypeArgument: ExcelRow (auto)
  
  ' Access columns:
  Assign: customerName = CurrentRow("Customer Name").ToString
  Assign: amount = CDbl(CurrentRow("Amount"))
  Assign: date = CDate(CurrentRow("Date"))
```

### Read Cell / Write Cell
```
Read Cell:   SheetName="Sheet1", CellName="B2" → out_Value (String)
Write Cell:  SheetName="Sheet1", CellName="B2", Value="Updated"
```

### Filter Data Table (not Excel-specific but commonly used with Excel)
```
Filter Data Table
  Input: dt_AllData
  Output: dt_FilteredData
  Filters: Column "Status" = "Active"
           Column "Amount" > 1000
```

### Sort Data Table
```
Sort Data Table
  Input/Output: dt_Data
  Sort By: Column "Date", Descending=True
```

---

## DataTable Operations

### Build Data Table Activity
- Creates a DataTable with defined columns and types at design time
- Use when you need to construct output tables row by row

### Add Data Row
```
Add Data Row
  DataTable = dt_Results
  ArrayRow = {customerName, amount, status}
  ' Columns must match DataTable schema
```

### Get Row Item / Set Row Item (inside For Each Row loop)
```
Get Row Item: Row=CurrentRow, ColumnName="Status" → value
Set Row Item: Row=CurrentRow, ColumnName="Status", Value="Processed"
```

### Remove Data Row
```
Remove Data Row: Row=CurrentRow  ← use inside For Each Row (index loop)
```

### DataTable to/from String (for LRW serialization)
```vb
' Serialize (before Suspend in LRW):
Invoke Code:
  out_JsonString = Newtonsoft.Json.JsonConvert.SerializeObject(in_dt_Data)

' Deserialize (after Resume):
Invoke Code:
  out_dt_Data = Newtonsoft.Json.JsonConvert.DeserializeObject(Of DataTable)(in_JsonString)
```

---

## Common DataTable Patterns

### LINQ on DataTable
```vb
' Filter rows where Status = "Active":
Dim activeRows = (From row In dt_Data.AsEnumerable()
                  Where row.Field(Of String)("Status") = "Active"
                  Select row).CopyToDataTable()

' Get distinct values:
Dim regions = (From row In dt_Data.AsEnumerable()
               Select row.Field(Of String)("Region")).Distinct().ToList()

' Sum a column:
Dim total = (From row In dt_Data.AsEnumerable()
             Select CDbl(row.Field(Of String)("Amount"))).Sum()

' Count matching rows:
Dim count = (From row In dt_Data.AsEnumerable()
             Where row.Field(Of String)("Status") = "Error").Count()
```

### Check if DataTable is Empty
```vb
If dt_Data Is Nothing OrElse dt_Data.Rows.Count = 0 Then
  Log Message: "No data to process"
End If
```

### Merge DataTables
```vb
' Merge dt_Source into dt_Target (must have same schema):
dt_Target.Merge(dt_Source)
```

---

## Mail Activities (UiPath.Mail.Activities)

### Use Desktop Outlook App (Modern, preferred for Outlook)
```
Use Desktop Outlook App
  Account = "user@company.com"
  
  Get Mail Messages → List(Of MailMessage) messages
  Move Mail Message
  Reply To Mail Message
  Send Mail Message
  Save Mail Message
  Delete Mail Message
```

### Get Mail Messages (Outlook/IMAP)
| Property | Description |
|---|---|
| Account | Email account (String) |
| MailFolder | Folder name: "Inbox", "Sent", custom |
| Filter | OWA filter string or IMAP filter |
| Top | Number of messages to retrieve (Int32) |
| OnlyUnreadMessages | Boolean |
| MarkAsRead | Mark retrieved messages as read |
| Output | List(Of MailMessage) |

### Send Mail Message (SMTP)
| Property | Description |
|---|---|
| To | Recipient(s) — semicolon separated |
| CC / BCC | Optional |
| Subject | Email subject |
| Body | HTML or plain text |
| IsBodyHtml | True for HTML |
| Attachments | List(Of String) — file paths |
| From | Sender address |
| Port / Server | SMTP settings (from Config/Asset) |

### Pattern: Process Inbox Emails
```
Use Desktop Outlook App (Account = Config("EmailAccount").ToString)
  Get Mail Messages (MailFolder = "Inbox", OnlyUnreadMessages = True,
                     Top = 50) → messages
  For Each mail In messages (TypeArgument = MailMessage)
    Assign: subject = mail.Subject
    Assign: body = mail.Body
    Assign: sender = mail.From.Address
    [process mail]
    Move Mail Message (Source = "Inbox", Destination = "Processed")
```

### Reply/Forward Pattern
```
Reply To Mail Message
  MailMessage = currentMail
  Body = "Thank you for your request. " + responseText
  IsBodyHtml = True
  CC = "manager@company.com"
```

### Save Attachments
```
For Each attachment In mail.Attachments
  Save Attachment (MailMessage=mail, FileName=attachment.Name, 
                   FolderPath=outputPath)
```

---

## File / Folder Activities (System)

These are in UiPath.System.Activities, commonly used alongside Excel/Mail:

### File Operations
```
Copy File:    Source, Destination, Overwrite=True
Move File:    Source, Destination
Delete File:  Path
File Exists:  Path → Boolean
Get Files:    Folder, Filter="*.xlsx", IncludeSubFolders → String[]
```

### Path Manipulation (VB.NET expressions)
```vb
' Get file name without extension:
System.IO.Path.GetFileNameWithoutExtension(filePath)

' Combine paths:
System.IO.Path.Combine(folderPath, fileName)

' Get directory:
System.IO.Path.GetDirectoryName(filePath)

' File exists check (alternative to activity):
System.IO.File.Exists(filePath)
```

### Directory Operations
```
Create Directory:  Path (creates all intermediate folders)
Directory Exists:  Path → Boolean
Delete Directory:  Path, Recursive=True
Get Subfolders:    FolderPath → String[]
```

---

## Best Practices

- **Never hardcode file paths** — use Config("ReportFolderPath").ToString
- **Always use `Use Excel File` scope** — never open Excel as UI automation (fragile)
- **Read then modify** — never modify a DataTable while iterating it with For Each
- **Null-check DataTable** before use: `If dt_Data IsNot Nothing AndAlso dt_Data.Rows.Count > 0`
- **Match column names** exactly (case-sensitive in some activities)
- **Use Append Range** instead of writing to specific cells when adding rows
- **SMTP credentials** always from Orchestrator Assets/Get Credential — never hardcoded
- **Attachment file paths** — verify file exists before adding to attachment list
