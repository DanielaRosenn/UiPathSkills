# Module: Developer Activities
**Package: UiPath.System.Activities + UiPath.Python.Activities + UiPath.Web.Activities + UiPath.Database.Activities**
**Studio 2025.10**

---

## Invoke Code

**Package**: UiPath.System.Activities
**Use when**: you need custom VB.NET or C# logic that can't be expressed with standard activities

### Properties
| Property | Type | Description |
|---|---|---|
| Code | String | The code to execute (VB.NET or C# — not both in same project) |
| Language | Enum | `VBNet` or `CSharp` |
| Arguments | Collection | In/Out parameters passed to/from the code |
| Private | Boolean | Hides values from logs at Verbose level |
| ContinueOnError | Boolean | Continue workflow if this activity throws |

### Argument Direction in Invoke Code
- Arguments collection: each entry has Name, Type, Direction (In/Out/InOut), Value
- **In**: pass data into the code block
- **Out**: receive data from the code block
- **InOut**: read and modify

### VB.NET Example — String manipulation
```vb
' In: in_RawText (String)
' Out: out_CleanText (String)
Dim cleaned As String = System.Text.RegularExpressions.Regex.Replace(in_RawText, "[^a-zA-Z0-9\s]", "")
out_CleanText = cleaned.Trim()
```

### VB.NET Example — List/LINQ operations
```vb
' In: in_dt_Data (DataTable)
' Out: out_TotalAmount (Double)
Dim total As Double = (From row As DataRow In in_dt_Data.AsEnumerable()
                       Where row.Field(Of String)("Status") = "Approved"
                       Select CDbl(row.Field(Of String)("Amount"))).Sum()
out_TotalAmount = total
```

### C# Example — JSON parsing
```csharp
// In: in_JsonString (String)
// Out: out_CustomerName (String)
using Newtonsoft.Json.Linq;
var json = JObject.Parse(in_JsonString);
out_CustomerName = json["customer"]["name"].ToString();
```

### Best Practices
- Add required namespaces in the Imports panel (e.g. `System.Text.RegularExpressions`, `Newtonsoft.Json`)
- Keep code blocks short (< 30 lines) — extract longer logic to a Library coded workflow
- Always use `Private = True` if code handles sensitive data
- Prefer **Coded Automations** (C# class) over Invoke Code for complex logic (see [orchestration/coded-automations.md](../coded-automations.md); project-level guide: [coded-automations.md](../../coded-automations.md))
- Use `out_` prefix arguments for all outputs

### When NOT to Use
- Complex multi-method logic → use Coded Automation instead
- Simple assignments → use Assign activity
- Calling external APIs → use HTTP Request activity

---

## HTTP Request / Web Activities

**Package**: UiPath.Web.Activities

### HTTP Request Activity
| Property | Description |
|---|---|
| URL | Endpoint URL (string variable — never hardcode) |
| Method | GET, POST, PUT, DELETE, PATCH, HEAD |
| Headers | Dictionary<String, String> of request headers |
| Body | Request body (JSON string, form data) |
| BodyFormat | Json, FormUrlEncoded, XWwwFormUrlEncoded |
| Response | Output: response body as String |
| StatusCode | Output: HTTP status code (Int32) |
| Timeout | Milliseconds (get from Config) |
| AcceptFormat | Json, Xml, Any |

### Pattern: REST API Call with Auth
```vb
' 1. Build headers Dictionary
Dim headers As New Dictionary(Of String, String)
headers.Add("Authorization", "Bearer " + in_ApiToken)
headers.Add("Content-Type", "application/json")

' 2. Assign to HTTP Request Headers property
' 3. Set URL from Config: Config("ApiBaseUrl").ToString + "/endpoint"
' 4. Set Body if POST:
Dim body As String = "{""key"": """ + in_Value + """}"
```

### Pattern: Parse JSON Response
```vb
' After HTTP Request, response in: responseBody (String)
Dim json As JObject = JObject.Parse(responseBody)
Dim customerId As String = json("data")("id").ToString()
```

### Deserialize JSON Activity
- Converts JSON string to .NET object
- TypeArgument: specify the target .NET type
- Use with `List(Of T)` for JSON arrays

### Add HTTP Request Headers (activity)
- Adds individual headers to a request
- Use inside HTTP Client scope for multiple requests to same endpoint

---

## Python Activities

**Package**: UiPath.Python.Activities
**Prerequisite**: Python installed on robot machine; .NET Desktop Runtime 6+ for Windows projects

### Activity Chain
```
Python Scope (container)
  ├── Load Python Script   → PythonObject (the loaded script)
  ├── Invoke Python Method → PythonObject (raw result)
  └── Get Python Object    → .NET typed result (String, Int32, List, etc.)
```

### Python Scope Properties
| Property | Description |
|---|---|
| Path | Python installation path (e.g. `"C:\Python312\"`) |
| Version | Python version (Auto, 3.x) |
| Timeout | Max execution time in ms |
| WorkingFolder | Script's working directory |

### Load Python Script
- **Code**: inline Python code as String variable
- **File**: path to `.py` file (recommended for larger scripts)
- Result: PythonObject variable (e.g. `pyScript`)

### Invoke Python Method
- **Instance**: the PythonObject from Load Python Script
- **Name**: method name to call (String)
- **InputParameters**: IEnumerable(Of Object) — list of arguments
- **Result**: PythonObject with raw result

### Get Python Object
- **PythonObject**: the result from Invoke Python Method
- **TypeArgument**: target .NET type (String, Int32, List(Of String), DataTable, etc.)
- **Result**: typed .NET variable

### Run Python Script (simple)
- Runs a script file or inline code without method invocation
- Good for side-effect scripts (write to file, call subprocess, etc.)

### Complete Example Pattern
```
Python Scope (Path = Config("PythonPath").ToString)
  Assign: pyCode = "def process_data(input_list):" + Environment.NewLine + 
                   "    return [x.upper() for x in input_list]"
  Load Python Script (Code = pyCode) → pyScript
  Assign: params = New List(Of Object) From {in_DataList}
  Invoke Python Method (Instance = pyScript, Name = "process_data", 
                        InputParameters = params) → pyResult
  Get Python Object (PythonObject = pyResult, 
                     TypeArgument = List(Of String)) → out_ProcessedList
```

### Best Practices
- Store Python path in Orchestrator Asset or Config — never hardcode
- Use `.py` files (not inline code) for scripts > 10 lines
- Handle PythonObject → .NET conversion always (raw PythonObject is not usable in VB.NET directly)
- Wrap in Try/Catch — Python runtime errors surface as ApplicationException

---

## Database Activities

**Package**: UiPath.Database.Activities

### Connect Activity (preferred — explicit connection management)
```
Connect (ConnectionString = Config("DB_ConnectionString").ToString,
         ProviderName = "System.Data.SqlClient") → dbConnection
  Execute Query → dt_Results
  Execute Non-Query → rowsAffected (Int32)
Disconnect
```

### Connection String Patterns
```
SQL Server:  "Server=myserver;Database=mydb;Integrated Security=True;"
SQL + creds: "Server=myserver;Database=mydb;User Id=user;Password=pass;"
Oracle:      "Data Source=mydb;User Id=user;Password=pass;"
```
**Security**: always store connection string in Orchestrator Asset, never hardcode.

### Execute Query
| Property | Description |
|---|---|
| ExistingDbConnection | Connection from Connect activity |
| Sql | SQL query string |
| Parameters | Dictionary(Of String, Argument) for parameterized queries |
| DataTable | Output DataTable result |

### Parameterized Query (always use — prevents SQL injection)
```vb
' Parameters dictionary:
Dim sqlParams As New Dictionary(Of String, Argument)
sqlParams.Add("@CustomerId", New InArgument(Of String)(in_CustomerId))
sqlParams.Add("@Status", New InArgument(Of String)("Active"))
' SQL: "SELECT * FROM Customers WHERE Id = @CustomerId AND Status = @Status"
```

### Execute Non-Query (INSERT/UPDATE/DELETE)
| Property | Description |
|---|---|
| Sql | INSERT/UPDATE/DELETE statement |
| Parameters | Parameterized inputs (always use) |
| AffectedRecords | Output: number of rows affected (Int32) |

### Insert DataTable
- Bulk insert a DataTable into a DB table
- More efficient than looping Execute Non-Query
- Requires matching column names

### Best Practices
- Always use parameterized queries (never string concatenation for SQL)
- Use Connect/Disconnect explicitly (don't rely on auto-connection)
- Wrap in Try/Catch/Finally — always disconnect in Finally
- Store connection strings in Orchestrator Assets
- Use `Execute Query` for reads, `Execute Non-Query` for writes

---

## Invoke Method / Invoke Function

**Use when**: calling a .NET method on an existing object

### Invoke Method
| Property | Description |
|---|---|
| TargetObject | The object to call the method on |
| MethodName | String name of the method |
| Parameters | InArgument collection (positional) |
| Result | Output of the method call |

### Example — String method
```
Invoke Method:
  TargetObject = myString (String variable)
  MethodName = "Split"
  Parameters: In[0] = New Char() {","c}
  Result = out_Parts (String[])
```

---

## PowerShell / Script Activities

### Run Script (Shell command / PowerShell)
- **FileName**: executable path (e.g. `"powershell.exe"`)
- **Arguments**: script arguments string
- **WorkingDirectory**: execution context path

### Pattern: Run PowerShell Script
```
Run Script:
  FileName = "powershell.exe"
  Arguments = "-ExecutionPolicy Bypass -File """ + scriptPath + """ -Param1 """ + in_Value + """"
```

---

## Cryptography Activities

**Package**: UiPath.Cryptography.Activities
- **Encrypt/Decrypt Text**: symmetric encryption (AES, DES, 3DES)
- **Hash Text**: one-way hashing (MD5, SHA1, SHA256, SHA384, SHA512)
- **Encrypt/Decrypt File**: file-level encryption

### Best Practice: Hashing Passwords for Comparison
```
Hash Text:
  Input = in_Password
  Algorithm = SHA256
  Encoding = UTF8
  Result = hashedPassword (String)
```
