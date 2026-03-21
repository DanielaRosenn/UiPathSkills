# State Serialization Reference

Rules and patterns for ensuring workflow state can be persisted and resumed correctly.

## Table of Contents

1. [Serialization Requirements](#serialization-requirements)
2. [Serializable Types](#serializable-types)
3. [Non-Serializable Types](#non-serializable-types)
4. [Custom Type Serialization](#custom-type-serialization)
5. [Analyzer Rules](#analyzer-rules)
6. [Common Patterns](#common-patterns)
7. [Troubleshooting](#troubleshooting)

---

## Serialization Requirements

When a workflow reaches a persistence point (Create Form Task, Wait For Form Task And Resume, Persist), all variables in scope must be serializable. This allows the workflow state to be saved to the database and restored later.

### Key Rules

1. All variables in scope at persistence points must be serializable
2. Arguments passed to workflows with persistence must be serializable
3. Avoid holding references to external resources (connections, handles)
4. Close and dispose resources before persistence points

---

## Serializable Types

### Primitive Types

All primitive types are serializable:

| Type | Example |
|------|---------|
| String | `"Hello World"` |
| Int32 | `42` |
| Int64 | `9223372036854775807` |
| Double | `3.14159` |
| Decimal | `99.99m` |
| Boolean | `True` / `False` |
| DateTime | `DateTime.Now` |
| TimeSpan | `TimeSpan.FromHours(1)` |
| Guid | `Guid.NewGuid()` |

### Collection Types

```xml
<!-- List of strings -->
<Variable x:TypeArguments="scg:List(x:String)" Name="stringList" />

<!-- Dictionary -->
<Variable x:TypeArguments="scg:Dictionary(x:String, x:Object)" Name="dataDict" />

<!-- Array -->
<Variable x:TypeArguments="x:String[]" Name="stringArray" />
```

### DataTable

DataTable is fully serializable:

```xml
<Variable x:TypeArguments="sd:DataTable" Name="invoiceData" />
```

### JSON Types

Newtonsoft.Json types are serializable:

```xml
<!-- JObject -->
<Variable x:TypeArguments="Newtonsoft.Json.Linq.JObject" Name="jsonData" />

<!-- JArray -->
<Variable x:TypeArguments="Newtonsoft.Json.Linq.JArray" Name="jsonArray" />

<!-- JToken -->
<Variable x:TypeArguments="Newtonsoft.Json.Linq.JToken" Name="jsonToken" />
```

### UiPath Types

Many UiPath types are serializable:

| Type | Serializable |
|------|--------------|
| QueueItem | Yes |
| Asset | Yes |
| Folder | Yes |
| TransactionItem | Yes |
| MailMessage | Yes |

---

## Non-Serializable Types

### Browser/Application Instances

```xml
<!-- NOT SERIALIZABLE - Do not use across persistence points -->
<Variable x:TypeArguments="ui:Browser" Name="browser" />
<Variable x:TypeArguments="ui:Application" Name="app" />
```

### Connection Objects

```xml
<!-- NOT SERIALIZABLE -->
<Variable x:TypeArguments="sd:SqlConnection" Name="dbConnection" />
<Variable x:TypeArguments="System.Net.Http.HttpClient" Name="httpClient" />
```

### File/Stream Handles

```xml
<!-- NOT SERIALIZABLE -->
<Variable x:TypeArguments="s:IO.FileStream" Name="fileStream" />
<Variable x:TypeArguments="s:IO.StreamReader" Name="reader" />
```

### UI Elements

```xml
<!-- NOT SERIALIZABLE -->
<Variable x:TypeArguments="ui:UiElement" Name="element" />
<Variable x:TypeArguments="ui:Window" Name="window" />
```

---

## Custom Type Serialization

### Creating Serializable Custom Types

```csharp
using System;

[Serializable]
public class InvoiceData
{
    public string InvoiceId { get; set; }
    public decimal Amount { get; set; }
    public DateTime InvoiceDate { get; set; }
    public string VendorName { get; set; }
    public List<LineItem> LineItems { get; set; }
}

[Serializable]
public class LineItem
{
    public string Description { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}
```

### Using Custom Types in XAML

```xml
<Variable x:TypeArguments="local:InvoiceData" Name="invoice" />
```

### JSON Serialization Alternative

Instead of custom types, use JSON:

```xml
<Variable x:TypeArguments="x:String" Name="invoiceJson" />

<!-- Serialize to JSON -->
<Assign DisplayName="Serialize Invoice">
  <Assign.To>
    <OutArgument x:TypeArguments="x:String">[invoiceJson]</OutArgument>
  </Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String">
      [Newtonsoft.Json.JsonConvert.SerializeObject(invoiceData)]
    </InArgument>
  </Assign.Value>
</Assign>

<!-- Deserialize from JSON -->
<Assign DisplayName="Deserialize Invoice">
  <Assign.To>
    <OutArgument x:TypeArguments="local:InvoiceData">[invoiceData]</OutArgument>
  </Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="local:InvoiceData">
      [Newtonsoft.Json.JsonConvert.DeserializeObject(Of InvoiceData)(invoiceJson)]
    </InArgument>
  </Assign.Value>
</Assign>
```

---

## Analyzer Rules

### Enable Serialization Analyzers

In project settings, enable these rules:

| Rule ID | Name | Description |
|---------|------|-------------|
| ST-USG-034 | Variables should be serializable | Warns on non-serializable variables |
| ST-USG-035 | Arguments should be serializable | Warns on non-serializable arguments |

### Configuration in project.json

```json
{
  "analyzerSettings": {
    "rules": {
      "ST-USG-034": {
        "isEnabled": true,
        "action": "Warning"
      },
      "ST-USG-035": {
        "isEnabled": true,
        "action": "Warning"
      }
    }
  }
}
```

---

## Common Patterns

### Pattern 1: Close Resources Before Persistence

```xml
<Sequence DisplayName="Safe Persistence Pattern">
  <!-- Open browser -->
  <ui:OpenBrowser Url="https://example.com" Browser="[browser]" />
  
  <!-- Do work -->
  <ui:TypeInto Target="[inputField]" Text="[data]" />
  <ui:Click Target="[submitButton]" />
  
  <!-- Extract data to serializable variable -->
  <ui:GetText Target="[resultElement]" Result="[resultText]" />
  
  <!-- Close browser BEFORE persistence -->
  <ui:CloseTab Browser="[browser]" />
  
  <!-- Now safe to persist -->
  <ui:CreateFormTask
    TaskTitle="Review Results"
    TaskDataJson="[resultText]"
    Result="[taskId]" />
  
  <ui:WaitForFormTaskAndResume
    TaskId="[taskId]"
    TaskAction="[action]"
    TaskData="[formData]" />
</Sequence>
```

### Pattern 2: Scope Variables Appropriately

```xml
<Sequence DisplayName="Scoped Variables">
  <!-- Outer scope - serializable only -->
  <Sequence.Variables>
    <Variable x:TypeArguments="x:String" Name="processedData" />
    <Variable x:TypeArguments="x:String" Name="taskId" />
  </Sequence.Variables>
  
  <!-- Inner scope for non-serializable -->
  <Sequence DisplayName="Browser Work">
    <Sequence.Variables>
      <!-- Non-serializable in inner scope -->
      <Variable x:TypeArguments="ui:Browser" Name="browser" />
    </Sequence.Variables>
    
    <ui:OpenBrowser Url="https://example.com" Browser="[browser]" />
    <ui:GetText Target="[element]" Result="[processedData]" />
    <ui:CloseTab Browser="[browser]" />
  </Sequence>
  
  <!-- Persistence point - browser out of scope -->
  <ui:CreateFormTask
    TaskTitle="Review"
    TaskDataJson="[processedData]"
    Result="[taskId]" />
</Sequence>
```

### Pattern 3: Convert to Serializable Before Persistence

```xml
<Sequence DisplayName="Convert Before Persist">
  <Sequence.Variables>
    <Variable x:TypeArguments="sd:DataTable" Name="dataTable" />
    <Variable x:TypeArguments="x:String" Name="dataJson" />
  </Sequence.Variables>
  
  <!-- Work with DataTable -->
  <ui:ReadRange WorkbookPath="data.xlsx" DataTable="[dataTable]" />
  
  <!-- Convert to JSON for persistence -->
  <Assign DisplayName="Convert to JSON">
    <Assign.To>
      <OutArgument x:TypeArguments="x:String">[dataJson]</OutArgument>
    </Assign.To>
    <Assign.Value>
      <InArgument x:TypeArguments="x:String">
        [Newtonsoft.Json.JsonConvert.SerializeObject(dataTable)]
      </InArgument>
    </Assign.Value>
  </Assign>
  
  <!-- Persist with JSON -->
  <ui:CreateFormTask
    TaskTitle="Review Data"
    TaskDataJson="[dataJson]"
    Result="[taskId]" />
  
  <ui:WaitForFormTaskAndResume
    TaskId="[taskId]"
    TaskAction="[action]"
    TaskData="[formData]" />
  
  <!-- Convert back after resume -->
  <Assign DisplayName="Convert from JSON">
    <Assign.To>
      <OutArgument x:TypeArguments="sd:DataTable">[dataTable]</OutArgument>
    </Assign.To>
    <Assign.Value>
      <InArgument x:TypeArguments="sd:DataTable">
        [Newtonsoft.Json.JsonConvert.DeserializeObject(Of DataTable)(dataJson)]
      </InArgument>
    </Assign.Value>
  </Assign>
</Sequence>
```

### Pattern 4: State Object Pattern

```xml
<Sequence DisplayName="State Object Pattern">
  <Sequence.Variables>
    <!-- Single state object for all data -->
    <Variable x:TypeArguments="Newtonsoft.Json.Linq.JObject" Name="workflowState" />
  </Sequence.Variables>
  
  <!-- Initialize state -->
  <Assign DisplayName="Initialize State">
    <Assign.To>
      <OutArgument x:TypeArguments="Newtonsoft.Json.Linq.JObject">[workflowState]</OutArgument>
    </Assign.To>
    <Assign.Value>
      <InArgument x:TypeArguments="Newtonsoft.Json.Linq.JObject">
        [Newtonsoft.Json.Linq.JObject.Parse("{}")]
      </InArgument>
    </Assign.Value>
  </Assign>
  
  <!-- Add data to state -->
  <InvokeMethod MethodName="Add" TargetObject="[workflowState]">
    <InArgument x:TypeArguments="x:String">invoiceId</InArgument>
    <InArgument x:TypeArguments="Newtonsoft.Json.Linq.JToken">[invoiceId]</InArgument>
  </InvokeMethod>
  
  <!-- Persistence point -->
  <ui:CreateFormTask
    TaskTitle="Review"
    TaskDataJson="[workflowState.ToString()]"
    Result="[taskId]" />
</Sequence>
```

---

## Troubleshooting

### Error: Type is not serializable

**Symptom**: Workflow fails at persistence point with serialization error.

**Solution**:
1. Identify the non-serializable variable
2. Either remove it from scope or convert to serializable type
3. Use JSON serialization as intermediate format

### Error: Object reference not set

**Symptom**: After resume, variables are null.

**Solution**:
1. Ensure variables are initialized before persistence
2. Check that default values are set
3. Re-initialize connections after resume

### Error: Circular reference detected

**Symptom**: JSON serialization fails with circular reference.

**Solution**:
```csharp
// Use reference handling
var settings = new JsonSerializerSettings
{
    ReferenceLoopHandling = ReferenceLoopHandling.Ignore
};
var json = JsonConvert.SerializeObject(obj, settings);
```

### Best Practices Summary

1. **Minimize scope** - Keep non-serializable variables in inner scopes
2. **Close resources** - Dispose connections before persistence
3. **Use JSON** - Convert complex objects to JSON strings
4. **Enable analyzers** - Catch issues at design time
5. **Test persistence** - Verify workflow can resume correctly
6. **Document state** - Comment which variables persist
