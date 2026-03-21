# Module: Invoke Code — Deep Reference
**Activity:** `Invoke Code` | **Package:** `UiPath.System.Activities`
**Language:** VB.NET (default) or C# — matches your project's expression language

---

## What Invoke Code Is (and Isn't)

Invoke Code runs a **single synchronous code block** inside a workflow. It's the escape hatch when activities can't express the logic you need. It is NOT a full class — no methods, no constructors, no `using`/`Imports` inside the block itself. Think of it as an inline function body.

**Use it when:**
- Complex string/data manipulation that would take 10+ activities
- LINQ filtering on DataTable that Filter Data Table can't express
- Custom algorithms (scoring, validation, formatting)
- .NET library calls with no corresponding activity
- JSON/XML manipulation beyond what built-in activities cover
- Bulk operations (bulk insert, batch transforms)

**Don't use it when:**
- Simple assignment → use `Assign`
- REST API call → use `HTTP Request`
- Python script → use `Python Scope`
- Logic > 50 lines → use `Coded Workflow` (C# class)
- Same code needed in 2+ workflows → extract to a Library

---

## XAML Syntax (complete, copy-paste ready)

```xml
<ui:InvokeCode sap2010:WorkflowViewState.IdRef="InvokeCode_1"
    DisplayName="[Descriptive name of what the code does]"
    Language="VB">
  <ui:InvokeCode.Parameters>
    <!-- IN parameter: pass a workflow variable INTO the code -->
    <ui:InvokeCodeParameter
        Direction="In"
        Name="inputTable"
        Type="[GetType(System.Data.DataTable)]"
        Value="[dtOrders]"/>

    <!-- IN parameter: primitive type -->
    <ui:InvokeCodeParameter
        Direction="In"
        Name="threshold"
        Type="[GetType(System.Int32)]"
        Value="[maxAmount]"/>

    <!-- IN parameter: string -->
    <ui:InvokeCodeParameter
        Direction="In"
        Name="rawText"
        Type="[GetType(System.String)]"
        Value="[in_RawText]"/>

    <!-- OUT parameter: get a value BACK into a workflow variable -->
    <ui:InvokeCodeParameter
        Direction="Out"
        Name="resultText"
        Type="[GetType(System.String)]"
        Value="[processedResult]"/>

    <!-- OUT parameter: DataTable output -->
    <ui:InvokeCodeParameter
        Direction="Out"
        Name="filteredTable"
        Type="[GetType(System.Data.DataTable)]"
        Value="[dtFiltered]"/>

    <!-- INOUT parameter: read AND write the same variable -->
    <ui:InvokeCodeParameter
        Direction="InOut"
        Name="counter"
        Type="[GetType(System.Int32)]"
        Value="[retryCount]"/>
  </ui:InvokeCode.Parameters>

  <ui:InvokeCode.Code>
    <![CDATA[
' Your VB.NET code here
' Parameters are available directly by their Name (not the workflow variable name)
' inputTable, threshold, rawText are readable
' resultText, filteredTable must be assigned before the block ends
' counter can be read and written

resultText = rawText.Trim().ToUpper()
    ]]>
  </ui:InvokeCode.Code>
</ui:InvokeCode>
```

### Parameter type reference

| VB.NET Type | `GetType(...)` expression |
|---|---|
| String | `GetType(System.String)` |
| Integer | `GetType(System.Int32)` |
| Long | `GetType(System.Int64)` |
| Double | `GetType(System.Double)` |
| Boolean | `GetType(System.Boolean)` |
| DateTime | `GetType(System.DateTime)` |
| DataTable | `GetType(System.Data.DataTable)` |
| DataRow | `GetType(System.Data.DataRow)` |
| List(Of String) | `GetType(System.Collections.Generic.List(Of System.String))` |
| List(Of Object) | `GetType(System.Collections.Generic.List(Of System.Object))` |
| Dictionary(Of String, Object) | `GetType(System.Collections.Generic.Dictionary(Of System.String, System.Object))` |
| Object | `GetType(System.Object)` |
| String() (array) | `GetType(System.String())` |

### C# variant (for C# XAML projects)

```xml
<ui:InvokeCode DisplayName="Process Data (C#)" Language="CS">
  <ui:InvokeCode.Parameters>
    <ui:InvokeCodeParameter Direction="In"  Name="inputText" Type="[GetType(System.String)]" Value="[rawInput]"/>
    <ui:InvokeCodeParameter Direction="Out" Name="result"    Type="[GetType(System.String)]" Value="[output]"/>
  </ui:InvokeCode.Parameters>
  <ui:InvokeCode.Code>
    <![CDATA[
// C# code — no class/method wrapper needed
result = inputText.Trim().ToUpperInvariant();
    ]]>
  </ui:InvokeCode.Code>
</ui:InvokeCode>
```

---

## Imports / Namespaces

Invoke Code doesn't support `Imports` inside the `CDATA` block. Instead, add namespaces in the **Imports panel** in Studio (Project tab → Imports). Common ones to pre-add:

```
System
System.Text
System.Text.RegularExpressions
System.IO
System.Linq
System.Collections.Generic
System.Data
System.Net
System.Net.Http
Newtonsoft.Json
Newtonsoft.Json.Linq
```

Once imported, use the short names: `Regex.IsMatch(...)`, `JObject.Parse(...)`, `StringBuilder`, etc.

---

## VB.NET Code Patterns (production-ready)

### 1. String Processing

```vb
' Clean and normalize a string
Dim sb As New System.Text.StringBuilder()
For Each ch As Char In rawText
    If Char.IsLetterOrDigit(ch) OrElse ch = " "c OrElse ch = "-"c Then
        sb.Append(ch)
    End If
Next
cleanText = sb.ToString().Trim()

' Remove extra whitespace
cleanText = System.Text.RegularExpressions.Regex.Replace(rawText.Trim(), "\s+", " ")

' Extract all numbers from a string
Dim matches = System.Text.RegularExpressions.Regex.Matches(rawText, "\d+(?:\.\d+)?")
Dim numbers As New List(Of Double)
For Each m As System.Text.RegularExpressions.Match In matches
    numbers.Add(CDbl(m.Value))
Next
extractedNumbers = numbers

' Parse key=value pairs from a string like "Name=John; Age=30; City=Tel Aviv"
Dim dict As New Dictionary(Of String, String)
For Each pair As String In rawText.Split(";"c)
    Dim parts = pair.Split("="c)
    If parts.Length = 2 Then
        dict(parts(0).Trim()) = parts(1).Trim()
    End If
Next
parsedDict = dict

' Truncate string to max length with ellipsis
maxLength = 100
result = If(rawText.Length > maxLength, rawText.Substring(0, maxLength - 3) & "...", rawText)

' Build CSV line safely (escape commas and quotes)
Dim fields() As String = {invoiceId, vendorName.Replace("""", """"""), amount.ToString("F2"), status}
csvLine = """" & String.Join(""",""", fields) & """"
```

### 2. DataTable Filtering & Transformation

```vb
' Filter rows by multiple conditions
Dim filtered = (From row As DataRow In inputTable.AsEnumerable()
                Where row.Field(Of String)("Status") = "Active"
                AndAlso CDbl(row.Field(Of Object)("Amount")) > threshold
                AndAlso Not String.IsNullOrEmpty(row.Field(Of String)("Email"))
                Select row).CopyToDataTable()
filteredTable = If(filtered.Rows.Count > 0, filtered, inputTable.Clone())
' Note: CopyToDataTable() throws if source is empty — always guard with Count check

' Sort a DataTable
Dim view As New DataView(inputTable)
view.Sort = "DueDate ASC, Amount DESC"
sortedTable = view.ToTable()

' Deduplicate by a column
Dim seen As New HashSet(Of String)()
Dim deduped As DataTable = inputTable.Clone()
For Each row As DataRow In inputTable.Rows
    Dim key As String = row("InvoiceID").ToString()
    If seen.Add(key) Then  ' Add returns False if already present
        deduped.ImportRow(row)
    End If
Next
dedupedTable = deduped

' Pivot: group by Status and count
Dim grouped As New Dictionary(Of String, Integer)()
For Each row As DataRow In inputTable.Rows
    Dim key As String = If(IsDBNull(row("Status")), "Unknown", row("Status").ToString())
    If grouped.ContainsKey(key) Then
        grouped(key) += 1
    Else
        grouped(key) = 1
    End If
Next
groupCounts = grouped

' Add a computed column to a DataTable
Dim dtCopy As DataTable = inputTable.Copy()
dtCopy.Columns.Add("TaxAmount", GetType(Double))
dtCopy.Columns.Add("IsOverdue", GetType(Boolean))
For Each row As DataRow In dtCopy.Rows
    Dim amt As Double = If(IsDBNull(row("Amount")), 0.0, CDbl(row("Amount")))
    Dim due As DateTime = If(IsDBNull(row("DueDate")), DateTime.MaxValue, CDate(row("DueDate")))
    row("TaxAmount") = Math.Round(amt * 0.17, 2)   ' 17% VAT
    row("IsOverdue") = due < DateTime.Today
Next
enrichedTable = dtCopy

' Aggregate: sum, average, min, max per column
totalAmount = inputTable.AsEnumerable().Sum(Function(r) If(IsDBNull(r("Amount")), 0.0, CDbl(r("Amount"))))
avgAmount   = inputTable.AsEnumerable().Average(Function(r) If(IsDBNull(r("Amount")), 0.0, CDbl(r("Amount"))))
maxAmount   = inputTable.AsEnumerable().Max(Function(r) If(IsDBNull(r("Amount")), 0.0, CDbl(r("Amount"))))
rowCount    = inputTable.Rows.Count

' Merge two DataTables with same schema
Dim combined As DataTable = table1.Copy()
For Each row As DataRow In table2.Rows
    combined.ImportRow(row)
Next
mergedTable = combined

' Convert DataTable to List(Of Dictionary)
Dim rows As New List(Of Dictionary(Of String, Object))()
For Each row As DataRow In inputTable.Rows
    Dim d As New Dictionary(Of String, Object)()
    For Each col As DataColumn In inputTable.Columns
        d(col.ColumnName) = If(IsDBNull(row(col)), Nothing, row(col))
    Next
    rows.Add(d)
Next
rowList = rows
```

### 3. JSON Operations

```vb
' Serialize Dictionary to JSON string
Dim payload As New Dictionary(Of String, Object) From {
    {"invoiceId", invoiceId},
    {"amount",    amount},
    {"dueDate",   dueDate.ToString("yyyy-MM-dd")},
    {"status",    "pending"},
    {"items",     New List(Of String) From {"item1", "item2"}}
}
jsonOutput = Newtonsoft.Json.JsonConvert.SerializeObject(payload, Newtonsoft.Json.Formatting.Indented)

' Serialize DataTable to JSON array
jsonOutput = Newtonsoft.Json.JsonConvert.SerializeObject(inputTable)

' Deserialize JSON string to Dictionary
Dim parsed = Newtonsoft.Json.JsonConvert.DeserializeObject(Of Dictionary(Of String, Object))(jsonInput)
invoiceId  = parsed("invoiceId").ToString()
amount     = CDbl(parsed("amount"))

' Deserialize JSON array to List of Dictionaries
Dim items = Newtonsoft.Json.JsonConvert.DeserializeObject(Of List(Of Dictionary(Of String, Object)))(jsonArrayInput)
For Each item In items
    Dim id As String = item("id").ToString()
    ' process...
Next

' Navigate nested JSON with JObject (handles nulls safely)
Dim root = Newtonsoft.Json.Linq.JObject.Parse(jsonInput)
customerName = root("data")?("customer")?("name")?.ToString() ?? "Unknown"
itemCount    = CInt(root("data")?("items")?.Count() ?? 0)

' Extract all values of a key from a JSON array
Dim jArray = Newtonsoft.Json.Linq.JArray.Parse(jsonArrayInput)
Dim ids As New List(Of String)()
For Each token In jArray
    ids.Add(token("id").ToString())
Next
idList = ids

' Modify JSON and re-serialize
Dim obj = Newtonsoft.Json.Linq.JObject.Parse(jsonInput)
obj("status") = "processed"
obj("processedAt") = DateTime.Now.ToString("yyyy-MM-ddTHH:mm:ss")
jsonOutput = obj.ToString()
```

### 4. Date & Time Processing

```vb
' Parse dates in various formats
Dim formats() As String = {"dd/MM/yyyy", "MM-dd-yyyy", "yyyy-MM-dd", "d MMMM yyyy"}
Dim parsedDate As DateTime
Dim success As Boolean = DateTime.TryParseExact(rawDateString, formats,
    System.Globalization.CultureInfo.InvariantCulture,
    System.Globalization.DateTimeStyles.None, parsedDate)
If success Then
    outputDate = parsedDate
Else
    outputDate = DateTime.MinValue  ' signal failure
    errorMessage = "Could not parse date: " & rawDateString
End If

' Business days calculation (skip weekends)
Dim current As DateTime = startDate
Dim businessDays As Integer = 0
Do While current < endDate
    current = current.AddDays(1)
    If current.DayOfWeek <> DayOfWeek.Saturday AndAlso current.DayOfWeek <> DayOfWeek.Sunday Then
        businessDays += 1
    End If
Loop
dayCount = businessDays

' Get first and last day of current month
firstDay = New DateTime(DateTime.Now.Year, DateTime.Now.Month, 1)
lastDay  = firstDay.AddMonths(1).AddDays(-1)

' Age in days, with overdue flag
daysOld   = CInt((DateTime.Today - documentDate).TotalDays)
isOverdue = daysOld > 30

' Format a date range for report titles
dateRange = String.Format("{0:MMMM d} – {1:MMMM d, yyyy}", periodStart, periodEnd)
```

### 5. File I/O (beyond standard activities)

```vb
' Read all lines into a List
Dim lines As New List(Of String)(System.IO.File.ReadAllLines(filePath, System.Text.Encoding.UTF8))
lineList = lines

' Write a List of strings as lines
System.IO.File.WriteAllLines(outputPath, lineList.ToArray(), System.Text.Encoding.UTF8)

' Append a single line with timestamp
Using sw As New System.IO.StreamWriter(logPath, append:=True, encoding:=System.Text.Encoding.UTF8)
    sw.WriteLine(DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss") & " | " & message)
End Using

' Read CSV manually (when Excel activities can't be used)
Dim dt As New DataTable()
Dim allLines = System.IO.File.ReadAllLines(csvPath, System.Text.Encoding.UTF8)
If allLines.Length > 0 Then
    For Each header As String In allLines(0).Split(","c)
        dt.Columns.Add(header.Trim(""""c).Trim())
    Next
    For i As Integer = 1 To allLines.Length - 1
        Dim values = allLines(i).Split(","c)
        Dim row = dt.NewRow()
        For j As Integer = 0 To Math.Min(values.Length - 1, dt.Columns.Count - 1)
            row(j) = values(j).Trim(""""c).Trim()
        Next
        dt.Rows.Add(row)
    Next
End If
outputTable = dt

' List files matching pattern recursively
Dim files As New List(Of String)()
For Each f As String In System.IO.Directory.GetFiles(folderPath, "*.pdf", System.IO.SearchOption.AllDirectories)
    files.Add(f)
Next
fileList = files

' Read XML file and extract values
Dim xmlDoc As New System.Xml.XmlDocument()
xmlDoc.Load(xmlPath)
Dim nodes = xmlDoc.SelectNodes("//Invoice/InvoiceNumber")
Dim invoiceNumbers As New List(Of String)()
For Each node As System.Xml.XmlNode In nodes
    invoiceNumbers.Add(node.InnerText)
Next
numberList = invoiceNumbers
```

### 6. Collections & Data Structures

```vb
' Build a lookup dictionary from a DataTable (key → row)
Dim lookup As New Dictionary(Of String, DataRow)()
For Each row As DataRow In inputTable.Rows
    Dim key As String = row("InvoiceID").ToString()
    If Not lookup.ContainsKey(key) Then
        lookup(key) = row
    End If
Next
lookupDict = lookup

' Flatten a nested list
Dim flat As New List(Of String)()
For Each group As List(Of String) In nestedList
    flat.AddRange(group)
Next
flatList = flat

' Batch a list into chunks of N
Dim batches As New List(Of List(Of String))()
Dim batchSize As Integer = 50
For i As Integer = 0 To inputList.Count - 1 Step batchSize
    batches.Add(inputList.GetRange(i, Math.Min(batchSize, inputList.Count - i)))
Next
batchedList = batches

' Set operations on two lists
Dim setA As New HashSet(Of String)(listA)
Dim setB As New HashSet(Of String)(listB)

' Items in A not in B (difference)
Dim onlyInA As New List(Of String)(setA)
onlyInA.RemoveAll(Function(x) setB.Contains(x))

' Items in both (intersection)
Dim inBoth As New List(Of String)(setA)
inBoth.RemoveAll(Function(x) Not setB.Contains(x))

differenceList    = onlyInA
intersectionList  = inBoth

' Stack / Queue pattern
Dim stack As New Stack(Of String)()
stack.Push(item)
Dim top As String = stack.Pop()

Dim queue As New Queue(Of String)()
queue.Enqueue(item)
Dim next As String = queue.Dequeue()
```

### 7. HTTP Calls (when HTTP Request activity isn't flexible enough)

```vb
' POST with custom serialization and timeout
Dim client As New System.Net.Http.HttpClient()
client.Timeout = TimeSpan.FromSeconds(30)
client.DefaultRequestHeaders.Add("Authorization", "Bearer " & accessToken)
client.DefaultRequestHeaders.Add("Accept", "application/json")

Dim payload As String = Newtonsoft.Json.JsonConvert.SerializeObject(New Dictionary(Of String, Object) From {
    {"invoiceId", invoiceId},
    {"status",    "processed"}
})
Dim content As New System.Net.Http.StringContent(payload, System.Text.Encoding.UTF8, "application/json")

Dim response = client.PostAsync(apiEndpoint, content).Result
statusCode   = CInt(response.StatusCode)
responseBody = response.Content.ReadAsStringAsync().Result
client.Dispose()

' Handle response
If statusCode >= 200 AndAlso statusCode < 300 Then
    Dim result = Newtonsoft.Json.JsonConvert.DeserializeObject(Of Dictionary(Of String, Object))(responseBody)
    outputId = result("id").ToString()
Else
    errorMessage = "API error " & statusCode.ToString() & ": " & responseBody
End If
```

### 8. Math & Business Logic

```vb
' Currency rounding (banker's rounding vs standard)
roundedAmount = Math.Round(rawAmount, 2, MidpointRounding.AwayFromZero)

' Percentage calculation with zero-division guard
percentComplete = If(totalCount > 0, Math.Round(CDbl(doneCount) / CDbl(totalCount) * 100, 1), 0.0)

' Generate a sequential batch reference
batchRef = String.Format("BATCH-{0}-{1:D6}", DateTime.Now.ToString("yyyyMMdd"), batchCounter)

' Validate Israeli ID number (Luhn algorithm)
Dim digits() As Integer = id.Select(Function(c) Integer.Parse(c.ToString())).ToArray()
Dim sum As Integer = 0
For i As Integer = 0 To digits.Length - 1
    Dim d As Integer = digits(i)
    If i Mod 2 = 1 Then
        d *= 2
        If d > 9 Then d -= 9
    End If
    sum += d
Next
isValid = (sum Mod 10 = 0)

' Compound interest calculation
finalAmount = principal * Math.Pow(1 + (annualRate / 12), months)

' Binning: categorize a value into a bucket
Dim bucket As String
Select Case amount
    Case Is < 1000 : bucket = "Small"
    Case Is < 10000 : bucket = "Medium"
    Case Is < 100000 : bucket = "Large"
    Case Else : bucket = "Enterprise"
End Select
amountCategory = bucket
```

### 9. DataTable Serialization for LRW (cross-suspend)

```vb
' === BEFORE SuspendWorkflow: Serialize ===
' Convert DataTable to JSON string (serializable across suspend points)
jsonSerializedTable = Newtonsoft.Json.JsonConvert.SerializeObject(dtOrders)

' === AFTER Resume: Deserialize ===
' Rebuild DataTable from JSON string
dtOrders = Newtonsoft.Json.JsonConvert.DeserializeObject(Of DataTable)(jsonSerializedTable)
```

### 10. Error-Safe Helpers

```vb
' Safe parse Integer (no exception if invalid)
Dim parsed As Integer
intValue = If(Integer.TryParse(rawString, parsed), parsed, defaultValue)

' Safe parse Double
Dim parsedDbl As Double
dblValue = If(Double.TryParse(rawString.Replace(",", "."),
    System.Globalization.NumberStyles.Any,
    System.Globalization.CultureInfo.InvariantCulture,
    parsedDbl), parsedDbl, 0.0)

' Safe dictionary get (no KeyNotFoundException)
stringValue = If(myDict.ContainsKey(keyName), myDict(keyName).ToString(), fallbackValue)

' Safe DataTable column read (handles DBNull)
Function SafeString(row As DataRow, colName As String, Optional defaultVal As String = "") As String
    Return If(row.Table.Columns.Contains(colName) AndAlso Not IsDBNull(row(colName)),
              row(colName).ToString(), defaultVal)
End Function
' Call: cellValue = SafeString(row, "Amount", "0")
' Note: can't define Functions inside Invoke Code — put this logic inline or use a Library

' Inline DBNull-safe read pattern:
cellValue = If(IsDBNull(row("Amount")), "0", row("Amount").ToString())
```

---

## Invoke Code Checklist

Before finalizing any Invoke Code block:

- [ ] All input variables passed as `Direction="In"` parameters — not accessed directly as workflow variables
- [ ] All output variables declared as `Direction="Out"` — assigned before block ends
- [ ] `CDATA` wrapping used (`<![CDATA[ ... ]]>`) — required for any code with `<`, `>`, `&`
- [ ] Empty DataTable guard on `CopyToDataTable()` — throws if zero rows
- [ ] DBNull guards on all DataRow cell reads
- [ ] No `Imports` inside the code block — add to project Imports panel instead
- [ ] `Private="True"` set if block handles credentials/passwords/PII
- [ ] Display name describes what the code does (not just "Invoke Code")
- [ ] Code is ≤ 50 lines — if longer, consider Coded Workflow
- [ ] Exception handling: wrap risky operations in Try/Catch inside the code if partial failure is possible

---

## Common Mistakes & Fixes

| Mistake | Fix |
|---|---|
| Accessing a workflow variable directly (e.g. `dtOrders`) without passing as parameter | Add an `In` parameter and use that name in code |
| `CopyToDataTable()` throws `InvalidOperationException` | Guard: `If filtered.Any() Then ... CopyToDataTable() Else ... Clone()` |
| `CDATA` not used → XML parse error in Studio | Wrap ALL code in `<![CDATA[ ... ]]>` |
| `KeyNotFoundException` on Dictionary access | Use `If dict.ContainsKey(key) Then ...` |
| `NullReferenceException` on JObject navigation | Use `?` null-conditional: `root("a")?("b")?.ToString()` |
| `Format` exception parsing dates | Use `DateTime.TryParseExact` with multiple format patterns |
| `Object reference not set` on DataRow cell | Use `IsDBNull(row("col"))` check first |
| Code runs but Out variable is `Nothing` in workflow | Make sure you assign the Out parameter inside the code block |
| `System.Net.Http.HttpClient` not found | Add `System.Net.Http` to project references via Manage Packages |
