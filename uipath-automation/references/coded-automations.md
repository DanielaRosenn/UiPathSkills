# UiPath Coded Automations Reference

Complete guide to C# coded workflows and coded agents in UiPath. Official Studio topics: [Coded workflow](https://docs.uipath.com/studio/standalone/2024.10/user-guide/coded-workflow), [Code source file](https://docs.uipath.com/studio/standalone/2024.10/user-guide/code-source-file).

**Additional patterns (orchestration module, Studio 2025.10–oriented):** [orchestration/coded-automations.md](orchestration/coded-automations.md).

## Table of Contents
1. [Overview](#overview)
2. [Project Setup](#project-setup)
3. [Coded Workflow Syntax](#coded-workflow-syntax)
4. [Activity APIs](#activity-apis)
5. [Coded Agents with LangGraph](#coded-agents-with-langgraph)
6. [Best Practices](#best-practices)

---

## Overview

Coded automations allow writing UiPath workflows in C# instead of XAML, providing:
- Full IDE support (IntelliSense, refactoring)
- Native C# language features
- Better version control (readable diffs)
- Complex logic handling
- Integration with coded agents/AI

---

## Project Setup

### Project.json for Coded Automation

```json
{
  "name": "CodedAutomation",
  "projectVersion": "1.0.0",
  "description": "Coded automation project",
  "main": "Main.cs",
  "projectType": "Workflow",
  "targetFramework": "Windows",
  "expressionLanguage": "CSharp",
  "entryPoints": [
    {
      "filePath": "Main.cs",
      "uniqueId": "00000000-0000-0000-0000-000000000000",
      "input": [],
      "output": []
    }
  ],
  "dependencies": {
    "UiPath.System.Activities": "[23.10.0]",
    "UiPath.UIAutomation.Activities": "[23.10.0]",
    "UiPath.CodedWorkflows": "[23.10.0]"
  }
}
```

### File Structure
```
CodedProject/
├── project.json
├── Main.cs                      # Entry point
├── Workflows/
│   ├── ProcessData.cs
│   └── ValidationWorkflow.cs
├── Services/
│   ├── DataService.cs
│   └── ApiClient.cs
└── Models/
    └── TransactionData.cs
```

---

## Coded Workflow Syntax

### Basic Coded Workflow

```csharp
using System;
using System.Data;
using UiPath.CodedWorkflows;
using UiPath.Core.Activities;

namespace MyAutomation
{
    public class Main : CodedWorkflow
    {
        [Workflow]
        public void Execute()
        {
            // Log message
            Log("Starting automation", LogLevel.Info);
            
            // Assign variable
            var status = "Processing";
            
            // Call another workflow
            var result = RunWorkflow<ProcessDataWorkflow, string>(
                workflow => workflow.Process("input data")
            );
            
            Log($"Completed with result: {result}", LogLevel.Info);
        }
    }
}
```

### Workflow with Arguments

```csharp
using System;
using UiPath.CodedWorkflows;

namespace MyAutomation
{
    public class ProcessTransaction : CodedWorkflow
    {
        [Workflow]
        public TransactionResult Execute(
            [In] string transactionId,
            [In] DataRow transactionData,
            [Out] out string processedStatus)
        {
            Log($"Processing transaction: {transactionId}", LogLevel.Info);
            
            // Process logic here
            processedStatus = "Success";
            
            return new TransactionResult
            {
                TransactionId = transactionId,
                Status = "Completed",
                ProcessedAt = DateTime.Now
            };
        }
    }
    
    public class TransactionResult
    {
        public string TransactionId { get; set; }
        public string Status { get; set; }
        public DateTime ProcessedAt { get; set; }
    }
}
```

### Using Activities in Coded Workflows

```csharp
using System;
using System.Data;
using UiPath.CodedWorkflows;
using UiPath.Excel;
using UiPath.Mail;
using UiPath.UIAutomation;

namespace MyAutomation
{
    public class DataProcessing : CodedWorkflow
    {
        [Workflow]
        public void Execute()
        {
            // Excel operations
            using (var excel = new ExcelFile(@"C:\Data\Input.xlsx"))
            {
                var data = excel.ReadRange("Sheet1", "A1:D100");
                
                foreach (DataRow row in data.Rows)
                {
                    ProcessRow(row);
                }
                
                excel.WriteRange("Results", "A1", processedData);
            }
            
            // Send email
            var mailService = container.Resolve<IMailService>();
            mailService.SendOutlookMail(
                to: "recipient@company.com",
                subject: "Processing Complete",
                body: "All transactions processed successfully."
            );
        }
        
        private void ProcessRow(DataRow row)
        {
            var id = row["ID"].ToString();
            var value = Convert.ToDecimal(row["Amount"]);
            // Processing logic
        }
    }
}
```

### Control Flow in Coded Workflows

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using UiPath.CodedWorkflows;

namespace MyAutomation
{
    public class ControlFlowExamples : CodedWorkflow
    {
        [Workflow]
        public void Execute()
        {
            // If-Else
            var amount = 1500m;
            if (amount > 1000)
            {
                Log("High value transaction - requires approval", LogLevel.Warn);
                RequestApproval(amount);
            }
            else
            {
                ProcessDirectly(amount);
            }
            
            // Switch
            var status = GetTransactionStatus();
            switch (status)
            {
                case "New":
                    InitializeTransaction();
                    break;
                case "Pending":
                    ContinueProcessing();
                    break;
                case "Complete":
                    FinalizeTransaction();
                    break;
                default:
                    Log($"Unknown status: {status}", LogLevel.Error);
                    break;
            }
            
            // For Each
            var items = GetPendingItems();
            foreach (var item in items)
            {
                try
                {
                    ProcessItem(item);
                }
                catch (Exception ex)
                {
                    Log($"Error processing {item.Id}: {ex.Message}", LogLevel.Error);
                }
            }
            
            // Parallel processing
            var tasks = new List<Task>();
            foreach (var batch in GetBatches())
            {
                tasks.Add(Task.Run(() => ProcessBatch(batch)));
            }
            Task.WaitAll(tasks.ToArray());
        }
        
        // Helper methods
        private string GetTransactionStatus() => "New";
        private void RequestApproval(decimal amount) { }
        private void ProcessDirectly(decimal amount) { }
        private void InitializeTransaction() { }
        private void ContinueProcessing() { }
        private void FinalizeTransaction() { }
        private List<TransactionItem> GetPendingItems() => new List<TransactionItem>();
        private void ProcessItem(TransactionItem item) { }
        private List<List<object>> GetBatches() => new List<List<object>>();
        private void ProcessBatch(List<object> batch) { }
    }
    
    public class TransactionItem
    {
        public string Id { get; set; }
    }
}
```

### Error Handling

```csharp
using System;
using UiPath.CodedWorkflows;
using UiPath.Core;

namespace MyAutomation
{
    public class ErrorHandling : CodedWorkflow
    {
        [Workflow]
        public void Execute()
        {
            var retryCount = 0;
            var maxRetries = 3;
            var success = false;
            
            while (!success && retryCount < maxRetries)
            {
                try
                {
                    // Operation that might fail
                    PerformOperation();
                    success = true;
                }
                catch (BusinessRuleException bre)
                {
                    // Business exception - don't retry
                    Log($"Business rule violation: {bre.Message}", LogLevel.Warn);
                    throw; // Re-throw to caller
                }
                catch (Exception ex)
                {
                    retryCount++;
                    Log($"Attempt {retryCount} failed: {ex.Message}", LogLevel.Error);
                    
                    if (retryCount >= maxRetries)
                    {
                        Log("Max retries reached, failing transaction", LogLevel.Error);
                        throw new ApplicationException(
                            $"Operation failed after {maxRetries} attempts", ex);
                    }
                    
                    // Wait before retry
                    System.Threading.Thread.Sleep(TimeSpan.FromSeconds(5));
                }
            }
        }
        
        private void PerformOperation()
        {
            // Simulated operation
        }
    }
}
```

---

## Activity APIs

### Logging

```csharp
// Simple log
Log("Message here", LogLevel.Info);
Log("Warning message", LogLevel.Warn);
Log("Error occurred", LogLevel.Error);

// With fields
Log("Processing item", LogLevel.Info, new Dictionary<string, object>
{
    { "ItemId", itemId },
    { "Amount", amount }
});
```

### UI Automation

```csharp
using UiPath.UIAutomation;

// Click element
uiAutomation.Click(
    target: Target.FromSelector("<webctrl tag='BUTTON' id='submit'/>")
);

// Type into field
uiAutomation.TypeInto(
    target: Target.FromSelector("<webctrl tag='INPUT' id='username'/>"),
    text: "john.doe@company.com"
);

// Get text
var statusText = uiAutomation.GetText(
    target: Target.FromSelector("<webctrl tag='SPAN' class='status'/>")
);

// Check element exists
var exists = uiAutomation.ElementExists(
    target: Target.FromSelector("<webctrl tag='DIV' class='error'/>"),
    timeout: TimeSpan.FromSeconds(3)
);
```

### Excel Operations

```csharp
using UiPath.Excel;

// Read data
using (var workbook = new ExcelFile(@"C:\Data\Input.xlsx"))
{
    var data = workbook.ReadRange("Sheet1", "A1:E100", hasHeaders: true);
    
    // Process data
    foreach (DataRow row in data.Rows)
    {
        var name = row["Name"].ToString();
        var value = Convert.ToDecimal(row["Amount"]);
    }
    
    // Write data
    workbook.WriteRange("Output", "A1", resultTable);
    
    // Save
    workbook.Save();
}
```

### HTTP/REST API

```csharp
using UiPath.Web;
using System.Net.Http;
using Newtonsoft.Json;

// GET request
var response = await httpClient.GetAsync(
    endpoint: "https://api.example.com/data",
    headers: new Dictionary<string, string>
    {
        { "Authorization", $"Bearer {token}" }
    }
);

var data = JsonConvert.DeserializeObject<ApiResponse>(response.Content);

// POST request
var postResponse = await httpClient.PostAsync(
    endpoint: "https://api.example.com/records",
    body: JsonConvert.SerializeObject(newRecord),
    contentType: "application/json",
    headers: new Dictionary<string, string>
    {
        { "Authorization", $"Bearer {token}" }
    }
);
```

### Orchestrator Integration

```csharp
using UiPath.Orchestrator;

// Get queue item
var queueItem = orchestrator.GetTransactionItem("ProcessingQueue");

if (queueItem != null)
{
    // Process item
    var specificContent = queueItem.SpecificContent;
    var customerId = specificContent["CustomerId"].ToString();
    
    // Set status
    orchestrator.SetTransactionStatus(
        queueItem: queueItem,
        status: TransactionStatus.Successful
    );
}

// Add queue item
orchestrator.AddQueueItem(
    queueName: "ProcessingQueue",
    specificContent: new Dictionary<string, object>
    {
        { "CustomerId", "CUST001" },
        { "OrderId", "ORD12345" }
    },
    reference: "CUST001-ORD12345",
    priority: QueueItemPriority.High
);

// Get asset
var apiKey = orchestrator.GetAsset<string>("APIKey");
var credential = orchestrator.GetCredential("SystemCredential");
```

---

## Coded Agents with LangGraph

### Agent Project Structure
```
CodedAgent/
├── project.json
├── Agent.cs                     # Main agent definition
├── Tools/
│   ├── SearchTool.cs
│   ├── DatabaseTool.cs
│   └── ApiTool.cs
├── Nodes/
│   ├── ClassifierNode.cs
│   ├── ProcessorNode.cs
│   └── OutputNode.cs
└── Models/
    └── AgentState.cs
```

### Basic Agent Definition

```csharp
using System;
using System.Threading.Tasks;
using UiPath.CodedWorkflows;
using UiPath.CodedWorkflows.LangGraph;

namespace MyCodedAgent
{
    public class SupportAgent : CodedAgent
    {
        [Agent]
        public async Task<AgentResponse> Execute(string userQuery)
        {
            // Define agent state
            var state = new AgentState
            {
                Query = userQuery,
                Messages = new List<Message>()
            };
            
            // Build graph
            var graph = new StateGraph<AgentState>()
                .AddNode("classifier", ClassifyQuery)
                .AddNode("process", ProcessQuery)
                .AddNode("respond", GenerateResponse)
                .AddConditionalEdges("classifier", RouteBasedOnCategory)
                .AddEdge("process", "respond")
                .SetEntryPoint("classifier")
                .SetFinishPoint("respond");
            
            // Run agent
            var result = await graph.InvokeAsync(state);
            
            return new AgentResponse
            {
                Response = result.FinalResponse,
                Category = result.Category
            };
        }
        
        private AgentState ClassifyQuery(AgentState state)
        {
            // Use LLM to classify the query
            var classification = llmService.Classify(state.Query, 
                categories: new[] { "technical", "billing", "general" });
            
            state.Category = classification;
            return state;
        }
        
        private string RouteBasedOnCategory(AgentState state)
        {
            return state.Category switch
            {
                "technical" => "process_technical",
                "billing" => "process_billing",
                _ => "process"
            };
        }
        
        private AgentState ProcessQuery(AgentState state)
        {
            // Tool execution based on category
            if (state.Category == "technical")
            {
                var searchResults = tools.SearchKnowledgeBase(state.Query);
                state.Context = searchResults;
            }
            
            return state;
        }
        
        private AgentState GenerateResponse(AgentState state)
        {
            var response = llmService.Generate(
                prompt: $"Based on the context: {state.Context}\n\nAnswer: {state.Query}",
                systemPrompt: "You are a helpful support agent."
            );
            
            state.FinalResponse = response;
            return state;
        }
    }
    
    public class AgentState
    {
        public string Query { get; set; }
        public string Category { get; set; }
        public string Context { get; set; }
        public string FinalResponse { get; set; }
        public List<Message> Messages { get; set; }
    }
    
    public class AgentResponse
    {
        public string Response { get; set; }
        public string Category { get; set; }
    }
}
```

### Defining Tools

```csharp
using System;
using UiPath.CodedWorkflows;
using UiPath.CodedWorkflows.LangGraph;

namespace MyCodedAgent.Tools
{
    public class SearchTool : AgentTool
    {
        [Tool(Description = "Search the knowledge base for relevant articles")]
        public SearchResult Search(
            [Parameter(Description = "Search query")] string query,
            [Parameter(Description = "Maximum results to return")] int maxResults = 5)
        {
            // Implement search logic
            var results = knowledgeBaseService.Search(query, maxResults);
            
            return new SearchResult
            {
                Articles = results,
                TotalCount = results.Count
            };
        }
    }
    
    public class DatabaseTool : AgentTool
    {
        [Tool(Description = "Query customer data from database")]
        public CustomerData GetCustomerInfo(
            [Parameter(Description = "Customer ID")] string customerId)
        {
            var customer = databaseService.GetCustomer(customerId);
            
            return new CustomerData
            {
                Id = customer.Id,
                Name = customer.Name,
                AccountStatus = customer.Status,
                OpenTickets = customer.TicketCount
            };
        }
    }
    
    public class ApiTool : AgentTool
    {
        [Tool(Description = "Create a support ticket")]
        public TicketResult CreateTicket(
            [Parameter(Description = "Customer ID")] string customerId,
            [Parameter(Description = "Ticket subject")] string subject,
            [Parameter(Description = "Ticket description")] string description,
            [Parameter(Description = "Priority level")] string priority = "normal")
        {
            var ticket = ticketService.Create(new TicketRequest
            {
                CustomerId = customerId,
                Subject = subject,
                Description = description,
                Priority = priority
            });
            
            return new TicketResult
            {
                TicketId = ticket.Id,
                Status = "Created"
            };
        }
    }
}
```

### Multi-Agent System

```csharp
using System;
using System.Threading.Tasks;
using UiPath.CodedWorkflows;
using UiPath.CodedWorkflows.LangGraph;

namespace MyCodedAgent
{
    public class SupervisorAgent : CodedAgent
    {
        private readonly TechnicalAgent _technicalAgent;
        private readonly BillingAgent _billingAgent;
        private readonly GeneralAgent _generalAgent;
        
        [Agent]
        public async Task<string> Execute(string userQuery)
        {
            // Router decides which agent handles the query
            var category = await ClassifyQuery(userQuery);
            
            return category switch
            {
                "technical" => await _technicalAgent.HandleQuery(userQuery),
                "billing" => await _billingAgent.HandleQuery(userQuery),
                _ => await _generalAgent.HandleQuery(userQuery)
            };
        }
        
        private async Task<string> ClassifyQuery(string query)
        {
            var result = await llm.ClassifyAsync(query, 
                new[] { "technical", "billing", "general" });
            return result.Category;
        }
    }
}
```

---

## Best Practices

### Code Organization
1. **Separate concerns** - UI automation, business logic, data access in different classes
2. **Use dependency injection** - Make code testable
3. **Keep workflows focused** - Single responsibility principle
4. **Extract reusable logic** - Create utility classes

### Error Handling
1. **Distinguish exception types** - Business vs System exceptions
2. **Implement retry logic** - For transient failures
3. **Log context** - Include relevant data in error logs
4. **Clean up resources** - Use `using` statements

### Performance
1. **Batch operations** - Minimize UI interactions
2. **Parallel processing** - Use Task.WhenAll for independent operations
3. **Cache data** - Avoid repeated lookups
4. **Async/await** - Don't block threads unnecessarily

### Testing
```csharp
using Xunit;
using Moq;

public class ProcessTransactionTests
{
    [Fact]
    public void Process_ValidTransaction_ReturnsSuccess()
    {
        // Arrange
        var mockService = new Mock<ITransactionService>();
        var workflow = new ProcessTransaction(mockService.Object);
        
        // Act
        var result = workflow.Execute("TXN001", testData, out var status);
        
        // Assert
        Assert.Equal("Success", status);
        Assert.NotNull(result);
    }
}
```
