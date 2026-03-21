# UiPath XAML Syntax Reference

Complete reference for UiPath XAML structure, namespaces, and patterns.

## Table of Contents
1. [File Structure](#file-structure)
2. [Namespace Reference](#namespace-reference)
3. [Workflow Containers](#workflow-containers)
4. [Variables and Arguments](#variables-and-arguments)
5. [Expression Syntax](#expression-syntax)
6. [Common Patterns](#common-patterns)

---

## File Structure

### Complete XAML Template

**CRITICAL**: The entire opening `<Activity>` tag with all namespaces MUST be on a SINGLE LINE. UiPath Studio's XAML parser requires this format. Multi-line formatting will cause parsing errors like `'mc' is an undeclared prefix`.

### Required Dependencies (from SALES02 production reference)
```json
"dependencies": {
  "UiPath.FlowchartBuilder.Activities": "[1.1.2]",
  "UiPath.IntegrationService.Activities": "1.23.0",
  "UiPath.MicrosoftOffice365.Activities": "3.5.0-preview",
  "UiPath.Persistence.Activities": "1.8.1",
  "UiPath.System.Activities": "[25.10.3]",
  "UiPath.UIAutomation.Activities": "[25.10.19]",
  "UiPath.WebAPI.Activities": "[2.3.2]"
}
```

### Critical Activity Namespace Mappings
| Activity | XML Tag | Package |
|---|---|---|
| HTTP Request | `uwah:NetHttpRequest` | `UiPath.WebAPI.Activities` |
| Send Email (Graph/O365) | `umam:SendMailConnections` | `UiPath.MicrosoftOffice365.Activities` |
| Deserialize JSON | `ui:DeserializeJson x:TypeArguments="njl:JObject"` | `UiPath.System.Activities` |
| Get Orchestrator Secret | `ui:GetSecret` | `UiPath.System.Activities` |
| Set Transaction Status | `ui:SetTransactionStatus` | `UiPath.System.Activities` |
| Add Queue Item | `upaq:AddQueueItemAndGetReference` | `UiPath.Persistence.Activities` |
| Invoke Workflow | `ui:InvokeWorkflowFile` | `UiPath.System.Activities` |
| Log Message | `ui:LogMessage` | `UiPath.System.Activities` |

**NEVER use**: `ui:HttpClient`, `ui:SendOutlookMailMessage` — these cause `ErrorActivity: This activity is missing or could not be loaded`

### Full Namespace Set for Approval/HTTP/Email Workflows
```xml
<Activity mc:Ignorable="sap sap2010" x:Class="ApprovalFlow" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:njl="clr-namespace:Newtonsoft.Json.Linq;assembly=Newtonsoft.Json" xmlns:s="clr-namespace:System;assembly=System.Private.CoreLib" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:ss="clr-namespace:System.Security;assembly=System.Private.CoreLib" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:umam="clr-namespace:UiPath.MicrosoftOffice365.Activities.Mail;assembly=UiPath.MicrosoftOffice365.Activities" xmlns:umame="clr-namespace:UiPath.MicrosoftOffice365.Activities.Mail.Enums;assembly=UiPath.MicrosoftOffice365.Activities" xmlns:umamm="clr-namespace:UiPath.MicrosoftOffice365.Activities.Mail.Models;assembly=UiPath.MicrosoftOffice365.Activities" xmlns:upa="clr-namespace:UiPath.Process.Activities;assembly=UiPath.Process.Activities" xmlns:upaq="clr-namespace:UiPath.Persistence.Activities.Queue;assembly=UiPath.Persistence.Activities" xmlns:upas="clr-namespace:UiPath.Process.Activities.Shared;assembly=UiPath.Process.Activities" xmlns:usau="clr-namespace:UiPath.Shared.Activities.Utils;assembly=UiPath.MicrosoftOffice365.Activities" xmlns:uwah="clr-namespace:UiPath.Web.Activities.Http;assembly=UiPath.Web.Activities" xmlns:uwahm="clr-namespace:UiPath.Web.Activities.Http.Models;assembly=UiPath.Web.Activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
```

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="Main" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:mva="clr-namespace:Microsoft.VisualBasic.Activities;assembly=System.Activities" xmlns:s="clr-namespace:System;assembly=System.Private.CoreLib" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:sd="clr-namespace:System.Data;assembly=System.Data" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  
  <!-- Arguments (workflow inputs/outputs) -->
  <x:Members>
    <x:Property Name="in_Argument" Type="InArgument(x:String)" />
    <x:Property Name="out_Result" Type="OutArgument(x:String)" />
  </x:Members>
  
  <!-- VB Settings for expressions -->
  <sap2010:ExpressionActivityEditor.ExpressionActivityEditor>C#</sap2010:ExpressionActivityEditor.ExpressionActivityEditor>
  <sap:VirtualizedContainerService.HintSize>1096,800</sap:VirtualizedContainerService.HintSize>
  <sap2010:WorkflowViewState.IdRef>ActivityBuilder_1</sap2010:WorkflowViewState.IdRef>
  
  <!-- Text Expression imports -->
  <TextExpression.NamespacesForImplementation>
    <sco:Collection x:TypeArguments="x:String">
      <x:String>System</x:String>
      <x:String>System.Collections.Generic</x:String>
      <x:String>System.Data</x:String>
      <x:String>System.Linq</x:String>
      <x:String>System.Text</x:String>
      <x:String>UiPath.Core.Activities</x:String>
    </sco:Collection>
  </TextExpression.NamespacesForImplementation>
  
  <TextExpression.ReferencesForImplementation>
    <sco:Collection x:TypeArguments="AssemblyReference">
      <AssemblyReference>mscorlib</AssemblyReference>
      <AssemblyReference>System</AssemblyReference>
      <AssemblyReference>System.Core</AssemblyReference>
      <AssemblyReference>System.Data</AssemblyReference>
      <AssemblyReference>UiPath.Core.Activities</AssemblyReference>
    </sco:Collection>
  </TextExpression.ReferencesForImplementation>
  
  <!-- Main workflow container -->
  <Sequence DisplayName="Main Sequence">
    <Sequence.Variables>
      <!-- Variable declarations -->
    </Sequence.Variables>
    
    <!-- Activities -->
  </Sequence>
</Activity>
```

---

## Namespace Reference

### Required Namespaces

| Prefix | Namespace | Purpose |
|--------|-----------|---------|
| (default) | `http://schemas.microsoft.com/netfx/2009/xaml/activities` | Core WF activities |
| `x` | `http://schemas.microsoft.com/winfx/2006/xaml` | XAML language |
| `ui` | `http://schemas.uipath.com/workflow/activities` | UiPath activities |
| `s` | `clr-namespace:System;assembly=mscorlib` | System types |
| `sd` | `clr-namespace:System.Data;assembly=System.Data` | DataTable/DataRow |
| `scg` | `clr-namespace:System.Collections.Generic;assembly=mscorlib` | Generic collections |
| `sco` | `clr-namespace:System.Collections.ObjectModel;assembly=mscorlib` | Observable collections |

### Activity Package Namespaces

| Package | Namespace Declaration |
|---------|----------------------|
| UiPath.Excel.Activities | `xmlns:uex="clr-namespace:UiPath.Excel.Activities;assembly=UiPath.Excel.Activities"` |
| UiPath.Mail.Activities | `xmlns:umail="clr-namespace:UiPath.Mail.Activities;assembly=UiPath.Mail.Activities"` |
| UiPath.UIAutomation | `xmlns:uua="clr-namespace:UiPath.UIAutomation.Activities;assembly=UiPath.UIAutomation.Activities"` |
| UiPath.Database.Activities | `xmlns:udb="clr-namespace:UiPath.Database.Activities;assembly=UiPath.Database.Activities"` |
| UiPath.Web.Activities | `xmlns:uweb="clr-namespace:UiPath.Web.Activities;assembly=UiPath.Web.Activities"` |

---

## Workflow Containers

### Sequence
Linear execution, activities run top-to-bottom.

```xml
<Sequence DisplayName="Process Data" sap2010:WorkflowViewState.IdRef="Sequence_1">
  <Sequence.Variables>
    <Variable x:TypeArguments="x:String" Name="strTemp" />
  </Sequence.Variables>
  
  <!-- Activities execute in order -->
  <ui:LogMessage Level="Info" Message="Starting process" />
  <Assign DisplayName="Initialize">
    <Assign.To><OutArgument x:TypeArguments="x:String">[strTemp]</OutArgument></Assign.To>
    <Assign.Value><InArgument x:TypeArguments="x:String">["Initial"]</InArgument></Assign.Value>
  </Assign>
</Sequence>
```

### Flowchart
Visual branching logic with decision nodes.

```xml
<Flowchart DisplayName="Decision Flow" sap2010:WorkflowViewState.IdRef="Flowchart_1">
  <Flowchart.Variables>
    <Variable x:TypeArguments="x:Int32" Name="intValue" Default="0" />
  </Flowchart.Variables>
  
  <Flowchart.StartNode>
    <x:Reference>__ReferenceID0</x:Reference>
  </Flowchart.StartNode>
  
  <!-- Start node -->
  <FlowStep x:Name="__ReferenceID0">
    <Assign DisplayName="Get Value">
      <Assign.To><OutArgument x:TypeArguments="x:Int32">[intValue]</OutArgument></Assign.To>
      <Assign.Value><InArgument x:TypeArguments="x:Int32">[10]</InArgument></Assign.Value>
    </Assign>
    <FlowStep.Next>
      <x:Reference>Decision_1</x:Reference>
    </FlowStep.Next>
  </FlowStep>
  
  <!-- Decision node -->
  <FlowDecision x:Name="Decision_1" Condition="[intValue > 5]" DisplayName="Check Value">
    <FlowDecision.True>
      <FlowStep>
        <ui:LogMessage Level="Info" Message="Value is greater than 5" />
      </FlowStep>
    </FlowDecision.True>
    <FlowDecision.False>
      <FlowStep>
        <ui:LogMessage Level="Info" Message="Value is 5 or less" />
      </FlowStep>
    </FlowDecision.False>
  </FlowDecision>
</Flowchart>
```

### State Machine
State-based workflow with transitions.

```xml
<StateMachine DisplayName="Order Processing" sap2010:WorkflowViewState.IdRef="StateMachine_1">
  <StateMachine.Variables>
    <Variable x:TypeArguments="x:String" Name="strStatus" Default="[&quot;New&quot;]" />
  </StateMachine.Variables>
  
  <StateMachine.InitialState>
    <x:Reference>State_New</x:Reference>
  </StateMachine.InitialState>
  
  <!-- Initial State -->
  <State x:Name="State_New" DisplayName="New Order">
    <State.Entry>
      <ui:LogMessage Level="Info" Message="Order received" />
    </State.Entry>
    
    <State.Transitions>
      <Transition DisplayName="To Processing" To="{x:Reference State_Processing}">
        <Transition.Trigger>
          <Delay Duration="[TimeSpan.FromSeconds(1)]" />
        </Transition.Trigger>
        <Transition.Condition>True</Transition.Condition>
      </Transition>
    </State.Transitions>
  </State>
  
  <!-- Processing State -->
  <State x:Name="State_Processing" DisplayName="Processing">
    <State.Entry>
      <Sequence>
        <ui:LogMessage Level="Info" Message="Processing order" />
      </Sequence>
    </State.Entry>
    
    <State.Transitions>
      <Transition DisplayName="To Complete" To="{x:Reference State_Final}">
        <Transition.Trigger>
          <Delay Duration="[TimeSpan.FromSeconds(1)]" />
        </Transition.Trigger>
      </Transition>
    </State.Transitions>
  </State>
  
  <!-- Final State -->
  <State x:Name="State_Final" DisplayName="Completed" IsFinal="True">
    <State.Entry>
      <ui:LogMessage Level="Info" Message="Order completed" />
    </State.Entry>
  </State>
</StateMachine>
```

---

## Variables and Arguments

### Variable Type Arguments

| .NET Type | TypeArguments | Example Default |
|-----------|---------------|-----------------|
| String | `x:String` | `Default="[&quot;text&quot;]"` or `Default=""` |
| Int32 | `x:Int32` | `Default="0"` |
| Boolean | `x:Boolean` | `Default="False"` |
| DateTime | `s:DateTime` | `Default="[DateTime.Now]"` |
| Double | `x:Double` | `Default="0.0"` |
| Object | `x:Object` | (no default) |
| DataTable | `sd:DataTable` | (no default, initialize with Build Data Table) |
| DataRow | `sd:DataRow` | (no default) |
| Array of String | `x:String[]` | `Default="[New String(){}]"` |
| List of String | `scg:List(x:String)` | `Default="[New List(Of String)]"` |
| Dictionary | `scg:Dictionary(x:String, x:Object)` | `Default="[New Dictionary(Of String, Object)]"` |
| JObject | `nj:JObject` | (requires Newtonsoft.Json namespace) |
| QueueItem | `ui:QueueItem` | (no default) |
| SecureString | `s:Security.SecureString` | (no default) |

### Variable Scope
Variables are scoped to their container and accessible in child activities:

```xml
<Sequence DisplayName="Outer">
  <Sequence.Variables>
    <Variable x:TypeArguments="x:String" Name="outerVar" />
  </Sequence.Variables>
  
  <Sequence DisplayName="Inner">
    <Sequence.Variables>
      <Variable x:TypeArguments="x:String" Name="innerVar" />
    </Sequence.Variables>
    
    <!-- Can access both outerVar and innerVar here -->
    <Assign>
      <Assign.To><OutArgument x:TypeArguments="x:String">[innerVar]</OutArgument></Assign.To>
      <Assign.Value><InArgument x:TypeArguments="x:String">[outerVar + " modified"]</InArgument></Assign.Value>
    </Assign>
  </Sequence>
  
  <!-- Can only access outerVar here, innerVar is out of scope -->
</Sequence>
```

### Argument Types

| Direction | Type | Description |
|-----------|------|-------------|
| In | `InArgument(Type)` | Input only, read-only in workflow |
| Out | `OutArgument(Type)` | Output only, write-only in workflow |
| InOut | `InOutArgument(Type)` | Bidirectional, read-write |

```xml
<x:Members>
  <!-- Input: configuration file path -->
  <x:Property Name="in_ConfigPath" Type="InArgument(x:String)" />
  
  <!-- Input: transaction data -->
  <x:Property Name="in_TransactionData" Type="InArgument(sd:DataRow)" />
  
  <!-- Output: processing result -->
  <x:Property Name="out_TransactionStatus" Type="OutArgument(x:String)" />
  
  <!-- InOut: data table to be enriched -->
  <x:Property Name="io_ResultsTable" Type="InOutArgument(sd:DataTable)" />
</x:Members>
```

---

## Expression Syntax

### VB.NET Expressions (Default)
UiPath uses VB.NET expressions by default. Expressions are wrapped in brackets `[]`.

```xml
<!-- String concatenation -->
<InArgument x:TypeArguments="x:String">["Hello " + strName]</InArgument>

<!-- Comparison -->
<Condition>[intValue > 10 AndAlso strStatus = "Active"]</Condition>

<!-- LINQ query on DataTable -->
<InArgument x:TypeArguments="x:Int32">[dtData.AsEnumerable().Count(Function(r) r("Status").ToString = "Complete")]</InArgument>

<!-- Ternary/conditional -->
<InArgument x:TypeArguments="x:String">[If(boolFlag, "Yes", "No")]</InArgument>

<!-- DateTime -->
<InArgument x:TypeArguments="s:DateTime">[DateTime.Now.AddDays(-7)]</InArgument>

<!-- String methods -->
<InArgument x:TypeArguments="x:String">[strInput.Trim().ToUpper().Replace("OLD", "NEW")]</InArgument>

<!-- Null coalescing -->
<InArgument x:TypeArguments="x:String">[If(strValue, "Default")]</InArgument>

<!-- Array/List initialization -->
<Default>[New String() {"item1", "item2", "item3"}]</Default>
<Default>[New List(Of String) From {"a", "b", "c"}]</Default>
```

### C# Expressions
When using C# expression mode:

```xml
<sap2010:ExpressionActivityEditor.ExpressionActivityEditor>C#</sap2010:ExpressionActivityEditor.ExpressionActivityEditor>

<!-- String interpolation -->
<InArgument x:TypeArguments="x:String">[$"Processing {strItemName} at {DateTime.Now}"]</InArgument>

<!-- Null conditional -->
<InArgument x:TypeArguments="x:String">[strValue?.Trim() ?? "Default"]</InArgument>

<!-- LINQ -->
<InArgument x:TypeArguments="x:Int32">[dtData.AsEnumerable().Where(r => r.Field<string>("Status") == "Active").Count()]</InArgument>
```

---

## Common Patterns

### Reading Arguments in Activities

```xml
<!-- Using InArgument value -->
<ui:LogMessage Level="Info" Message="[in_FilePath]" />

<!-- Setting OutArgument -->
<Assign>
  <Assign.To><OutArgument x:TypeArguments="x:String">[out_Result]</OutArgument></Assign.To>
  <Assign.Value><InArgument x:TypeArguments="x:String">["Success"]</InArgument></Assign.Value>
</Assign>
```

### Passing Arguments to Invoked Workflows

```xml
<ui:InvokeWorkflowFile WorkflowFileName="Workflows\ProcessItem.xaml" DisplayName="Process Item">
  <ui:InvokeWorkflowFile.Arguments>
    <!-- Pass input arguments -->
    <InArgument x:TypeArguments="x:String" x:Key="in_ItemId">[strCurrentItemId]</InArgument>
    <InArgument x:TypeArguments="sd:DataRow" x:Key="in_ItemData">[currentRow]</InArgument>
    
    <!-- Receive output arguments -->
    <OutArgument x:TypeArguments="x:String" x:Key="out_Status">[strProcessingStatus]</OutArgument>
    <OutArgument x:TypeArguments="x:Boolean" x:Key="out_Success">[boolItemSuccess]</OutArgument>
    
    <!-- InOut arguments -->
    <InOutArgument x:TypeArguments="sd:DataTable" x:Key="io_LogTable">[dtProcessingLog]</InOutArgument>
  </ui:InvokeWorkflowFile.Arguments>
</ui:InvokeWorkflowFile>
```

### Annotation Pattern
Adding annotations for documentation:

```xml
<Sequence DisplayName="Main Process" sap2010:WorkflowViewState.IdRef="Sequence_1">
  <sap:WorkflowViewStateService.ViewState>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Boolean x:Key="IsAnnotationDocked">True</x:Boolean>
    </scg:Dictionary>
  </sap:WorkflowViewStateService.ViewState>
  <sap2010:Annotation.AnnotationText>This sequence handles the main business process logic.
Input: Transaction data from queue
Output: Processed results to database</sap2010:Annotation.AnnotationText>
  
  <!-- Activities -->
</Sequence>
```

### Activity Display Properties

```xml
<Assign DisplayName="Calculate Total" 
  sap:VirtualizedContainerService.HintSize="250,60"
  sap2010:WorkflowViewState.IdRef="Assign_1">
  <!-- ... -->
</Assign>
```
