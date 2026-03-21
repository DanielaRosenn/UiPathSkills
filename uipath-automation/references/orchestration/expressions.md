# Module: VB.NET & C# Expressions Reference
**Covers:** String operations, type conversion, date/time, collections, DataTable LINQ, Regex, null handling

---

## VB.NET (default XAML project language)

### String operations
```vb
' Interpolation (Studio 2022.4+)
$"Hello {name}, you have {count} items"

' Classic concatenation
"Prefix_" & variableName & "_" & DateTime.Now.ToString("yyyyMMdd")

' Format
String.Format("{0:yyyy-MM-dd HH:mm}", DateTime.Now)
String.Format("Invoice #{0:D6}", invoiceNumber)       ' zero-padded: 000042

' Check / test
String.IsNullOrEmpty(s)
String.IsNullOrWhiteSpace(s)
s.StartsWith("INV-")
s.EndsWith(".pdf")
s.Contains("ERROR")

' Transform
s.ToUpper()
s.ToLower()
s.Trim()
s.TrimStart("/"c)
s.Replace("old", "new")
s.PadLeft(10, "0"c)                    ' "0000042"
s.PadRight(20)                          ' pad with spaces

' Split / Join
s.Split(","c)                           ' returns String()
s.Split({","}, StringSplitOptions.RemoveEmptyEntries)
String.Join(", ", arrayOrList)

' Substring
s.Substring(0, 10)                      ' first 10 chars
s.Substring(s.Length - 4)              ' last 4 chars
s.Substring(s.IndexOf("@"))            ' from @ onwards
s.IndexOf("@")                          ' position of @

' Remove / insert
s.Remove(0, 4)                          ' remove first 4 chars
```

### Type conversion
```vb
' String → number (throws on invalid)
CInt("42")
CDbl("3.14")
CDate("2024-01-15")
CBool("True")

' String → number (safe — use in Invoke Code)
Dim result As Integer
Integer.TryParse(s, result)            ' result = 0 if failed

' Number → string
myInt.ToString()
myDbl.ToString("F2")                   ' "3.14"
myDbl.ToString("C")                    ' "$3.14" (culture-aware)
myDbl.ToString("N0")                   ' "1,234" (thousands separator)

' Convert (handles DBNull safely)
Convert.ToInt32(row("Amount"))
Convert.ToDouble(row("Price"))
Convert.ToDateTime(row("DueDate"))
Convert.ToString(someObj)              ' null-safe

' Object → typed (when you know the type)
DirectCast(someObj, DataTable)
TryCast(someObj, String)               ' returns Nothing if wrong type
```

### Date / Time
```vb
DateTime.Now                           ' current date + time
DateTime.Today                         ' today at 00:00:00
DateTime.Now.Date                      ' strip time component

' Format
DateTime.Now.ToString("yyyy-MM-dd")
DateTime.Now.ToString("dd/MM/yyyy HH:mm:ss")
DateTime.Now.ToString("yyyyMMdd_HHmmss")   ' for file names

' Arithmetic
DateTime.Now.AddDays(7)
DateTime.Now.AddHours(-2)
DateTime.Now.AddMonths(1)
DateTime.Now.AddSeconds(-30)

' Parse
DateTime.Parse("2024-01-15")
DateTime.ParseExact("15/01/2024", "dd/MM/yyyy", Nothing)

' Difference
(endDate - startDate).TotalDays
(DateTime.Now - startTime).TotalSeconds
(endDate - startDate).TotalMinutes

' Components
DateTime.Now.Year
DateTime.Now.Month
DateTime.Now.DayOfWeek                 ' DayOfWeek enum
DateTime.Now.Hour
```

### Collections
```vb
' List(Of T)
myList.Count
myList.Contains(item)
myList.IndexOf(item)
myList.First()
myList.Last()
myList.FirstOrDefault()                ' Nothing if empty
myList.Where(Function(x) x.Length > 3).ToList()
myList.OrderBy(Function(x) x).ToList()
myList.OrderByDescending(Function(x) CDbl(x)).ToList()
myList.Select(Function(x) x.ToUpper()).ToList()
myList.Distinct().ToList()
myList.Take(10).ToList()
myList.Skip(5).ToList()

' Dictionary(Of String, Object)
myDict("key")
myDict("key").ToString()
myDict.ContainsKey("key")
myDict.Keys
myDict.Values
myDict.Count
New Dictionary(Of String, Object) From {{"k1", v1}, {"k2", v2}}
```

### DataTable operations
```vb
' Basic
dt.Rows.Count
dt.Columns.Count
dt.Columns.Contains("ColName")
dt.Rows(0)("ColName").ToString()

' Filter with Select
dt.Select("Status = 'Active' AND Amount > 1000")    ' returns DataRow()

' LINQ on DataTable (use in Invoke Code)
dt.AsEnumerable() _
  .Where(Function(r) r("Status").ToString() = "Active") _
  .Where(Function(r) CDbl(r("Amount")) > 1000) _
  .OrderBy(Function(r) r("DueDate")) _
  .CopyToDataTable()

' Aggregate
dt.AsEnumerable().Sum(Function(r) CDbl(r("Amount")))
dt.AsEnumerable().Max(Function(r) CDbl(r("Amount")))
dt.AsEnumerable().Min(Function(r) CDate(r("DueDate")))

' Group
Dim grouped = dt.AsEnumerable() _
    .GroupBy(Function(r) r("Status").ToString()) _
    .ToDictionary(Function(g) g.Key, Function(g) g.Count())

' Null / DBNull checks
If IsDBNull(row("Amount")) Then amount = 0 Else amount = CDbl(row("Amount"))
If IsNothing(myObj) OrElse String.IsNullOrEmpty(myObj.ToString()) Then ...
```

### Regex (inline expressions)
```vb
' Test match
System.Text.RegularExpressions.Regex.IsMatch(text, "\d{4}-\d{2}-\d{2}")

' Extract first match
System.Text.RegularExpressions.Regex.Match(text, "INV-(\d+)").Groups(1).Value

' Extract all matches (use in Invoke Code)
Dim matches = System.Text.RegularExpressions.Regex.Matches(text, "\d+")
For Each m As System.Text.RegularExpressions.Match In matches
    values.Add(m.Value)
Next

' Replace
System.Text.RegularExpressions.Regex.Replace(text, "\s+", " ")
System.Text.RegularExpressions.Regex.Replace(text, "[^a-zA-Z0-9]", "")
```

### Null / empty guards
```vb
' Null coalescing (VB style)
If(myString, "default")                 ' if myString is Nothing, use "default"
If(myObj IsNot Nothing, myObj.ToString(), "N/A")

' Safe navigation (must use Invoke Code for complex chains)
Dim val = If(myDict.ContainsKey("x"), myDict("x").ToString(), String.Empty)
```

---

## C# (coded workflows / C# XAML projects)

```csharp
// String interpolation
$"Processing {invoiceId} — attempt {retryCount} of {maxRetries}"

// Null coalescing
string result = value ?? "default";
int count = list?.Count ?? 0;
string name = dict?.ContainsKey("name") == true ? dict["name"].ToString() : "unknown";

// Pattern matching
if (ex is BusinessRuleException bizEx) { Log.Warn(bizEx.Message); }

// LINQ
var filtered = orders
    .Where(o => o.Amount > 1000 && o.Status == "Active")
    .OrderBy(o => o.DueDate)
    .ToList();

var total = orders.Sum(o => o.Amount);
var byStatus = orders.GroupBy(o => o.Status)
    .ToDictionary(g => g.Key, g => g.ToList());

// String
string.IsNullOrEmpty(s);
string.IsNullOrWhiteSpace(s);
string.Join(", ", list);
s.Split(',', StringSplitOptions.RemoveEmptyEntries);
$"{DateTime.Now:yyyy-MM-dd_HHmmss}";

// Type conversion
int.Parse(s);                           // throws on invalid
int.TryParse(s, out int result);        // safe
Convert.ToInt32(obj);                   // handles null → 0
(obj as string) ?? string.Empty;        // safe cast
```
