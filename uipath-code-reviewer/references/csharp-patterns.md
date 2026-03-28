# C# Patterns Reference

Good vs bad C# patterns for UiPath coded workflow review.

## Argument Declarations

### Bad: Missing Argument Attributes

```csharp
public class ProcessInvoice : CodedWorkflow
{
    [Workflow]
    public void Execute(string filePath, out string result)
    {
        // Missing [Argument] attributes
    }
}
```

### Good: With Argument Attributes

```csharp
public class ProcessInvoice : CodedWorkflow
{
    [Workflow]
    public void Execute(
        [Argument(Direction.In)] string in_FilePath,
        [Argument(Direction.Out)] out string out_Result)
    {
        // Properly attributed arguments
    }
}
```

---

## Naming Conventions

### Bad: Inconsistent Naming

```csharp
public void Execute(
    [Argument(Direction.In)] string filepath,  // lowercase
    [Argument(Direction.In)] int Counter,      // no prefix
    [Argument(Direction.Out)] out string Result)  // no prefix
{
    var data = new DataTable();  // no prefix
    string status = "";          // no prefix
}
```

### Good: Consistent Naming

```csharp
public void Execute(
    [Argument(Direction.In)] string in_FilePath,
    [Argument(Direction.In)] int in_Counter,
    [Argument(Direction.Out)] out string out_Result)
{
    var dtData = new DataTable();
    string strStatus = "";
}
```

---

## Error Handling

### Bad: No Error Handling

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_ApiUrl)
{
    var response = Http.Request(in_ApiUrl, HttpMethod.Get);
    var data = JsonConvert.DeserializeObject<MyData>(response.Content);
    Excel.WriteRange(data.ToDataTable(), "output.xlsx", "Sheet1", "A1");
}
```

### Bad: Empty Catch

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_ApiUrl)
{
    try
    {
        var response = Http.Request(in_ApiUrl, HttpMethod.Get);
    }
    catch (Exception ex)
    {
        // Empty catch - swallowing exception
    }
}
```

### Bad: Catch and Throw New

```csharp
try
{
    ProcessData();
}
catch (Exception ex)
{
    throw new Exception("Error processing data");  // Lost original exception
}
```

### Good: Proper Error Handling

```csharp
[Workflow]
public void Execute(
    [Argument(Direction.In)] string in_ApiUrl,
    [Argument(Direction.Out)] out bool out_Success,
    [Argument(Direction.Out)] out string out_ErrorMessage)
{
    out_Success = false;
    out_ErrorMessage = null;
    
    try
    {
        Log($"Starting API call to {in_ApiUrl}", LogLevel.Info);
        
        var response = Http.Request(in_ApiUrl, HttpMethod.Get);
        
        if (response.StatusCode != 200)
        {
            throw new ApplicationException($"API returned status {response.StatusCode}");
        }
        
        var data = JsonConvert.DeserializeObject<MyData>(response.Content);
        Excel.WriteRange(data.ToDataTable(), "output.xlsx", "Sheet1", "A1");
        
        out_Success = true;
        Log("Processing completed successfully", LogLevel.Info);
    }
    catch (HttpRequestException ex)
    {
        out_ErrorMessage = $"Network error: {ex.Message}";
        Log(out_ErrorMessage, LogLevel.Error);
        throw;
    }
    catch (JsonException ex)
    {
        out_ErrorMessage = $"Invalid response format: {ex.Message}";
        Log(out_ErrorMessage, LogLevel.Error);
        throw new BusinessRuleException("API returned invalid data", ex);
    }
    catch (Exception ex)
    {
        out_ErrorMessage = $"Unexpected error: {ex.Message}";
        Log(out_ErrorMessage, LogLevel.Error);
        throw;
    }
}
```

---

## Resource Management

### Bad: No Disposal

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_ConnectionString)
{
    var connection = new SqlConnection(in_ConnectionString);
    connection.Open();
    
    var command = new SqlCommand("SELECT * FROM Users", connection);
    var reader = command.ExecuteReader();
    
    // Connection, command, reader never disposed!
}
```

### Good: Using Statements

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_ConnectionString)
{
    using (var connection = new SqlConnection(in_ConnectionString))
    {
        connection.Open();
        
        using (var command = new SqlCommand("SELECT * FROM Users", connection))
        using (var reader = command.ExecuteReader())
        {
            while (reader.Read())
            {
                // Process data
            }
        }
    }
}
```

### Good: C# 8+ Using Declaration

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_ConnectionString)
{
    using var connection = new SqlConnection(in_ConnectionString);
    connection.Open();
    
    using var command = new SqlCommand("SELECT * FROM Users", connection);
    using var reader = command.ExecuteReader();
    
    while (reader.Read())
    {
        // Process data
    }
}
```

---

## Logging

### Bad: No Logging

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_TransactionId)
{
    var data = GetTransactionData(in_TransactionId);
    ProcessTransaction(data);
    SaveResults(data);
}
```

### Bad: Logging Sensitive Data

```csharp
[Workflow]
public void Execute(
    [Argument(Direction.In)] string in_Username,
    [Argument(Direction.In)] string in_Password)
{
    Log($"Logging in with username: {in_Username}, password: {in_Password}", LogLevel.Info);
}
```

### Good: Appropriate Logging

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_TransactionId)
{
    Log($"Starting transaction processing: {in_TransactionId}", LogLevel.Info);
    
    try
    {
        var data = GetTransactionData(in_TransactionId);
        Log($"Retrieved transaction data. Amount: {data.Amount:C}", LogLevel.Info);
        
        ProcessTransaction(data);
        Log("Transaction processed successfully", LogLevel.Info);
        
        SaveResults(data);
        Log($"Results saved for transaction: {in_TransactionId}", LogLevel.Info);
    }
    catch (Exception ex)
    {
        Log($"Transaction {in_TransactionId} failed: {ex.Message}", LogLevel.Error);
        throw;
    }
}
```

---

## Input Validation

### Bad: No Validation

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_FilePath)
{
    var content = File.ReadAllText(in_FilePath);  // Will throw if null/empty/not exists
}
```

### Good: With Validation

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_FilePath)
{
    // Validate input
    if (string.IsNullOrWhiteSpace(in_FilePath))
    {
        throw new ArgumentException("in_FilePath is required", nameof(in_FilePath));
    }
    
    if (!File.Exists(in_FilePath))
    {
        throw new FileNotFoundException($"File not found: {in_FilePath}", in_FilePath);
    }
    
    // Safe to proceed
    var content = File.ReadAllText(in_FilePath);
}
```

---

## Credentials

### Bad: Hardcoded Credentials

```csharp
[Workflow]
public void Execute()
{
    var apiKey = "sk-abc123xyz789";  // Hardcoded!
    var password = "SuperSecret123!";  // Hardcoded!
    
    var client = new HttpClient();
    client.DefaultRequestHeaders.Add("Authorization", $"Bearer {apiKey}");
}
```

### Bad: Password as String

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_Password)
{
    // Password should be SecureString
}
```

### Good: Using Orchestrator Assets

```csharp
[Workflow]
public void Execute()
{
    // Get credential from Orchestrator
    var credential = Orchestrator.GetCredential("SystemCredential");
    var username = credential.Username;
    var password = credential.Password;  // SecureString
    
    // Get asset from Orchestrator
    var apiKey = Orchestrator.GetAsset("APIKey");
}
```

---

## LINQ Operations

### Bad: Inefficient Loop

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] DataTable in_Data)
{
    var results = new List<DataRow>();
    
    foreach (DataRow row in in_Data.Rows)
    {
        if (row["Status"].ToString() == "Active")
        {
            if (Convert.ToDecimal(row["Amount"]) > 1000)
            {
                results.Add(row);
            }
        }
    }
}
```

### Good: Using LINQ

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] DataTable in_Data)
{
    var results = in_Data.AsEnumerable()
        .Where(row => row.Field<string>("Status") == "Active")
        .Where(row => row.Field<decimal>("Amount") > 1000)
        .ToList();
}
```

### Good: Complex LINQ Operations

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] DataTable in_Data)
{
    // Filter and transform
    var activeHighValue = in_Data.AsEnumerable()
        .Where(row => row.Field<string>("Status") == "Active")
        .Where(row => row.Field<decimal>("Amount") > 1000)
        .Select(row => new
        {
            Id = row.Field<int>("Id"),
            Amount = row.Field<decimal>("Amount"),
            FormattedAmount = row.Field<decimal>("Amount").ToString("C")
        })
        .OrderByDescending(x => x.Amount)
        .ToList();
    
    // Group and aggregate
    var summaryByStatus = in_Data.AsEnumerable()
        .GroupBy(row => row.Field<string>("Status"))
        .Select(g => new
        {
            Status = g.Key,
            Count = g.Count(),
            TotalAmount = g.Sum(row => row.Field<decimal>("Amount")),
            AverageAmount = g.Average(row => row.Field<decimal>("Amount"))
        })
        .ToList();
}
```

---

## Retry Logic

### Bad: No Retry for Transient Operations

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_ApiUrl)
{
    var response = Http.Request(in_ApiUrl, HttpMethod.Get);  // Will fail on transient errors
}
```

### Good: With Retry Logic

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_ApiUrl)
{
    var response = RetryOperation(
        () => Http.Request(in_ApiUrl, HttpMethod.Get),
        maxRetries: 3,
        delayMs: 2000
    );
}

private T RetryOperation<T>(Func<T> operation, int maxRetries = 3, int delayMs = 2000)
{
    int attempt = 0;
    while (true)
    {
        try
        {
            return operation();
        }
        catch (Exception ex) when (IsTransientError(ex) && attempt < maxRetries)
        {
            attempt++;
            Log($"Attempt {attempt} failed: {ex.Message}. Retrying in {delayMs}ms...", LogLevel.Warn);
            Thread.Sleep(delayMs * attempt);  // Exponential backoff
        }
    }
}

private bool IsTransientError(Exception ex)
{
    return ex is HttpRequestException ||
           ex is TimeoutException ||
           ex is SocketException;
}
```

---

## Method Length

### Bad: Long Method

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_FilePath)
{
    // 200+ lines of code in a single method
    // Multiple responsibilities
    // Hard to test and maintain
}
```

### Good: Single Responsibility Methods

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_FilePath)
{
    ValidateInput(in_FilePath);
    var data = ReadInputFile(in_FilePath);
    var processedData = ProcessData(data);
    SaveResults(processedData);
    SendNotification();
}

private void ValidateInput(string filePath)
{
    if (string.IsNullOrWhiteSpace(filePath))
        throw new ArgumentException("File path is required");
    if (!File.Exists(filePath))
        throw new FileNotFoundException("File not found", filePath);
}

private DataTable ReadInputFile(string filePath)
{
    Log($"Reading file: {filePath}", LogLevel.Info);
    return Excel.ReadRange(filePath, "Sheet1", "A1");
}

private DataTable ProcessData(DataTable data)
{
    Log($"Processing {data.Rows.Count} rows", LogLevel.Info);
    // Processing logic
    return data;
}

private void SaveResults(DataTable data)
{
    Log("Saving results", LogLevel.Info);
    Excel.WriteRange(data, "output.xlsx", "Results", "A1");
}

private void SendNotification()
{
    Log("Sending completion notification", LogLevel.Info);
    // Notification logic
}
```

---

## Async Operations

### Bad: Blocking Async

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_ApiUrl)
{
    var result = CallApiAsync(in_ApiUrl).Result;  // Blocking!
}
```

### Bad: Async Void

```csharp
public async void ProcessDataAsync()  // async void is dangerous
{
    await Task.Delay(1000);
}
```

### Good: Proper Async

```csharp
[Workflow]
public async Task ExecuteAsync([Argument(Direction.In)] string in_ApiUrl)
{
    var result = await CallApiAsync(in_ApiUrl);
    await ProcessResultAsync(result);
}

private async Task<string> CallApiAsync(string url)
{
    using var client = new HttpClient();
    return await client.GetStringAsync(url);
}

private async Task ProcessResultAsync(string result)
{
    await Task.Run(() => {
        // CPU-bound processing
    });
}
```

---

## Magic Numbers

### Bad: Hardcoded Values

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] decimal in_Amount)
{
    if (in_Amount > 10000)  // Magic number
    {
        // Requires approval
    }
    
    Thread.Sleep(5000);  // Magic number
    
    for (int i = 0; i < 3; i++)  // Magic number
    {
        // Retry logic
    }
}
```

### Good: Named Constants

```csharp
private const decimal ApprovalThreshold = 10000m;
private const int DefaultDelayMs = 5000;
private const int MaxRetryAttempts = 3;

[Workflow]
public void Execute([Argument(Direction.In)] decimal in_Amount)
{
    if (in_Amount > ApprovalThreshold)
    {
        // Requires approval
    }
    
    Thread.Sleep(DefaultDelayMs);
    
    for (int i = 0; i < MaxRetryAttempts; i++)
    {
        // Retry logic
    }
}
```

### Good: Configuration-Based

```csharp
[Workflow]
public void Execute(
    [Argument(Direction.In)] decimal in_Amount,
    [Argument(Direction.In)] Dictionary<string, object> in_Config)
{
    var approvalThreshold = Convert.ToDecimal(in_Config["ApprovalThreshold"]);
    var maxRetries = Convert.ToInt32(in_Config["MaxRetries"]);
    
    if (in_Amount > approvalThreshold)
    {
        // Requires approval
    }
}
```

---

## Null Handling

### Bad: No Null Checks

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_Data)
{
    var length = in_Data.Length;  // NullReferenceException if null
    var upper = in_Data.ToUpper();
}
```

### Good: Null Checks

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] string in_Data)
{
    if (in_Data == null)
    {
        throw new ArgumentNullException(nameof(in_Data));
    }
    
    var length = in_Data.Length;
    var upper = in_Data.ToUpper();
}
```

### Good: Null-Conditional Operators

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] MyObject in_Data)
{
    var name = in_Data?.Name ?? "Unknown";
    var count = in_Data?.Items?.Count ?? 0;
    
    in_Data?.Process();
}
```

---

## String Operations

### Bad: String Concatenation in Loop

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] List<string> in_Items)
{
    string result = "";
    foreach (var item in in_Items)
    {
        result += item + ", ";  // Creates new string each iteration
    }
}
```

### Good: StringBuilder

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] List<string> in_Items)
{
    var sb = new StringBuilder();
    foreach (var item in in_Items)
    {
        sb.Append(item).Append(", ");
    }
    var result = sb.ToString().TrimEnd(',', ' ');
}
```

### Good: String.Join

```csharp
[Workflow]
public void Execute([Argument(Direction.In)] List<string> in_Items)
{
    var result = string.Join(", ", in_Items);
}
```

---

## Exception Types

### Bad: Generic Exception

```csharp
throw new Exception("Something went wrong");
```

### Good: Specific Exceptions

```csharp
// For business rule violations
throw new BusinessRuleException("Invoice amount exceeds approval limit");

// For validation errors
throw new ArgumentException("Invalid file format", nameof(in_FilePath));

// For not found scenarios
throw new FileNotFoundException("Config file not found", configPath);

// For application errors
throw new ApplicationException("Failed to connect to external service");
```

### Good: Custom Exceptions

```csharp
public class InvoiceProcessingException : Exception
{
    public string InvoiceId { get; }
    public string Stage { get; }
    
    public InvoiceProcessingException(string invoiceId, string stage, string message)
        : base(message)
    {
        InvoiceId = invoiceId;
        Stage = stage;
    }
    
    public InvoiceProcessingException(string invoiceId, string stage, string message, Exception inner)
        : base(message, inner)
    {
        InvoiceId = invoiceId;
        Stage = stage;
    }
}

// Usage
throw new InvoiceProcessingException(
    invoiceId: "INV-001",
    stage: "Validation",
    message: "Invoice amount is negative"
);
```
