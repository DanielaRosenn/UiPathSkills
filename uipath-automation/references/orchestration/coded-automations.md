# Module: Coded Automations (C#)
**Studio 2025.10 | Windows + Cross-Platform projects**

---

## What Coded Automations Are

C# classes that implement the `CodedWorkflow` base class, giving full .NET power within the UiPath ecosystem. They can invoke XAML workflows and be invoked by XAML workflows — fully interoperable.

**Use coded automations when**:
- Logic is too complex for visual workflow (LINQ, recursion, complex algorithms)
- You want IDE features: IntelliSense, refactoring, unit testing with C# test frameworks
- Building reusable library components
- Writing coded agents (agentic automation)
- Calling external .NET libraries directly

---

## Coded Workflow (`.cs` file)

### Basic Structure
```csharp
using System;
using System.Activities;
using UiPath.CodedWorkflows;
using UiPath.Core;
using UiPath.Core.Activities;

namespace MyProject.Processes
{
    public class ProcessInvoice : CodedWorkflow
    {
        // Entry point — called when workflow is invoked
        [Workflow]
        public string Execute(string invoiceNumber, double amount, string customerName)
        {
            // Use services (replaces serviceContainer, deprecated in 2025.10)
            var log = services.GetService<ILogService>();
            
            Log($"Processing invoice: {invoiceNumber}");
            
            // Call XAML workflow from coded workflow
            var result = RunWorkflow("Processes\\ValidateInvoice.xaml",
                new Dictionary<string, object>
                {
                    { "in_InvoiceNumber", invoiceNumber },
                    { "in_Amount", amount }
                });
            
            bool isValid = (bool)result["out_IsValid"];
            
            if (!isValid)
                throw new BusinessRuleException($"Invoice {invoiceNumber} failed validation");
            
            return $"Invoice {invoiceNumber} processed successfully";
        }
    }
}
```

### ICodedWorkflowServices (2025.10 — replaces serviceContainer)
```csharp
// Access UiPath services via DI:
var orchestratorClient = services.GetService<IOrchestratorClient>();
var logService = services.GetService<ILogService>();
var credentialService = services.GetService<ICredentialService>();

// Get an Orchestrator Asset:
var assetValue = orchestratorClient.Assets.GetStringAsset("MyAssetName");

// Get credential:
var cred = credentialService.GetCredential("System1_Credential");
string username = cred.Username;
System.Security.SecureString password = cred.Password;
```

### RunWorkflow — Calling XAML from Coded
```csharp
// Returns Dictionary<string, object> of out/io arguments
var outputs = RunWorkflow("Processes\\GetCustomerData.xaml",
    new Dictionary<string, object>
    {
        { "in_CustomerId", customerId },
        { "in_Config", config }
    });

var customerData = outputs["out_CustomerData"] as DataTable;
```

### [Workflow] Attribute
- Marks the method as the entry point when invoked from XAML
- Only one `[Workflow]` method per class
- Method parameters become input/output arguments in XAML

---

## Coded Test Case
```csharp
using UiPath.CodedWorkflows;
using UiPath.Testing;

public class ValidateInvoiceTests : CodedWorkflow
{
    [TestCase]
    public void Test_ValidInvoice_ShouldPass()
    {
        // Arrange
        var invoiceNumber = "INV-001";
        var amount = 500.00;
        
        // Act
        var result = RunWorkflow("Processes\\ValidateInvoice.xaml",
            new Dictionary<string, object>
            {
                { "in_InvoiceNumber", invoiceNumber },
                { "in_Amount", amount }
            });
        
        // Assert
        var isValid = (bool)result["out_IsValid"];
        Testing.Verify.IsTrue(isValid, "Valid invoice should pass validation");
    }
    
    [TestCase]
    public void Test_NegativeAmount_ShouldFail()
    {
        // Expects BusinessRuleException to be thrown
        Testing.Verify.ShouldFail(() =>
            RunWorkflow("Processes\\ValidateInvoice.xaml",
                new Dictionary<string, object>
                {
                    { "in_InvoiceNumber", "INV-002" },
                    { "in_Amount", -100 }
                })
        );
    }
}
```

---

## Coded Agent

A coded automation that acts as an AI agent, using LLMs to decide actions.

```csharp
using UiPath.CodedWorkflows;
using UiPath.Agents;

public class InvoiceAgent : CodedWorkflow
{
    [Workflow]
    public AgentResult Execute(string userRequest, Dictionary<string, object> context)
    {
        // Agent loop: plan → act → observe → repeat
        var agent = new Agent()
            .WithSystemPrompt("You are an invoice processing assistant...")
            .WithTool("GetInvoiceStatus", GetInvoiceStatus)
            .WithTool("ApproveInvoice", ApproveInvoice)
            .WithTool("EscalateToManager", EscalateToManager);
        
        return agent.Run(userRequest, context);
    }
    
    private string GetInvoiceStatus(string invoiceNumber)
    {
        // Call XAML or external system
        var result = RunWorkflow("Processes\\GetInvoiceStatus.xaml",
            new Dictionary<string, object> { { "in_InvoiceNumber", invoiceNumber } });
        return result["out_Status"].ToString();
    }
    
    private bool ApproveInvoice(string invoiceNumber, string approvalNote)
    {
        RunWorkflow("Processes\\ApproveInvoice.xaml",
            new Dictionary<string, object>
            {
                { "in_InvoiceNumber", invoiceNumber },
                { "in_Note", approvalNote }
            });
        return true;
    }
}
```

---

## Calling Coded Workflow from XAML

1. Create `.cs` file with `[Workflow]` method
2. In XAML: add `Invoke Workflow File` activity
3. Set path to `.cs` file
4. Arguments automatically mapped from `[Workflow]` method parameters

---

## Using Activities in Coded Workflows

Access UiPath activities programmatically via the `services` container:

```csharp
// UI Automation (requires UiPath.UIAutomation.Activities)
using UiPath.UIAutomationNext.API;

var uiAuto = services.GetService<IUIAutomation>();

// Click an element:
uiAuto.Click(Target.FromUrl("https://app.example.com")
                   .Find(Element.WithName("Submit Button")));

// Type into a field:
uiAuto.TypeInto(Target.FromUrl("https://app.example.com")
                       .Find(Element.WithName("Username")), 
                username);

// Get text:
string value = uiAuto.GetText(Target.FromUrl("https://app.example.com")
                                    .Find(Element.WithId("result-label")));
```

---

## File Structure

```
MyProject/
├── Processes/
│   ├── Main.xaml
│   ├── ProcessInvoice.cs        ← coded workflow
│   ├── InvoiceAgent.cs          ← coded agent
│   └── ValidateInvoice.xaml     ← XAML workflow (can be mixed)
├── Tests/
│   └── ValidateInvoiceTests.cs  ← coded test cases
└── project.json
```

---

## Best Practices

- **One `[Workflow]` per class** — keep classes focused
- **Namespace matches folder**: `MyProject.Processes` for files in `Processes/`
- **Avoid `serviceContainer`** (deprecated 2025.10) — use `services.GetService<T>()`
- **Exception handling**: use `try/catch` — `BusinessRuleException` for data errors
- **Config access**: pass `Dictionary<string, object>` as argument, don't re-read Config file
- **Mixed XAML + coded**: use `RunWorkflow()` to cross-invoke — both are first-class
- **Prefer coded for**: complex algorithms, unit-testable logic, multi-method components
- **Prefer XAML for**: UI automation steps, simple sequential processes, non-developer users

---

## Deprecation Note (2025.10)

`serviceContainer` → replaced by `ICodedWorkflowServices` via `services` property.
Update any existing coded automations using `serviceContainer.Resolve<T>()` to `services.GetService<T>()`.
