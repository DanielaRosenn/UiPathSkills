# Production Long-Running Template Guide

Best practices and patterns for production-ready long-running workflows based on UiPath standards.

**Last Updated**: 2026-02-18

---

## Common Failure Points

### What Typically Goes Wrong

1. **Missing namespaces** - XAML won't compile
2. **No error handling** - Process crashes on any error
3. **No configuration** - Hardcoded values everywhere
4. **No initialization** - Settings not loaded
5. **No queue integration** - Manual triggering only
6. **No timeout handling** - Tasks hang indefinitely
7. **No rejection logic** - Process doesn't handle "No" responses
8. **Missing dependencies** - Activities not available
9. **Incorrect serialization** - State lost at persistence points
10. **No logging** - Can't debug issues

---

## Production Template Structure

### Required Files

```
ProjectName/
├── project.json                    # With supportsPersistence: true
├── Main.xaml                       # Entry point with queue trigger
├── Config/
│   └── Config.xlsx                 # Settings, constants, assets
├── Framework/
│   ├── InitAllSettings.xaml        # Load configuration
│   ├── InitAllApplications.xaml    # Initialize connections
│   └── CloseAllApplications.xaml   # Cleanup
├── Workflows/
│   ├── GetTransactionData.xaml     # Get next item from queue
│   ├── ProcessTransaction.xaml     # Main business logic
│   ├── CreateApprovalTask.xaml     # Create Action Center task
│   ├── WaitForApproval.xaml        # Wait and resume
│   └── HandleRejection.xaml        # Rejection logic
├── FormSchemas/
│   └── ApprovalForm.json           # Action Center form
└── README.md
```

---

## Essential Patterns

### 1. Proper project.json

**Use EXACT versions from SALES02 production reference. Wrong versions cause missing activity errors.**

```json
{
  "name": "ProjectName",
  "description": "Long-running approval workflow",
  "main": "Main-Queue.xaml",
  "projectId": "GENERATE-VALID-GUID",
  "projectVersion": "1.0.0",
  "projectType": "Workflow",
  "targetFramework": "Windows",
  "expressionLanguage": "VisualBasic",
  "schemaVersion": "4.0",
  "studioVersion": "25.10.6.0",
  "entryPoints": [
    {
      "filePath": "Main-Queue.xaml",
      "uniqueId": "GENERATE-VALID-GUID",
      "input": [],
      "output": []
    }
  ],
  "dependencies": {
    "UiPath.FlowchartBuilder.Activities": "[1.1.2]",
    "UiPath.IntegrationService.Activities": "1.23.0",
    "UiPath.MicrosoftOffice365.Activities": "3.5.0-preview",
    "UiPath.Persistence.Activities": "1.8.1",
    "UiPath.System.Activities": "[25.10.3]",
    "UiPath.UIAutomation.Activities": "[25.10.19]",
    "UiPath.WebAPI.Activities": "[2.3.2]"
  },
  "runtimeOptions": {
    "autoDispose": false,
    "netFrameworkLazyLoading": false,
    "isPausable": true,
    "isAttended": false,
    "requiresUserInteraction": false,
    "supportsPersistence": true,
    "workflowSerialization": "NewtonsoftJson",
    "excludedLoggedData": ["Private:*", "*password*"],
    "executionType": "Workflow",
    "mustRestoreAllDependencies": true
  },
  "designOptions": {
    "projectProfile": "Developement",
    "outputType": "Process",
    "libraryOptions": { "privateWorkflows": [] },
    "processOptions": { "ignoredFiles": [] }
  }
}
```

### Critical Activity Tag Reference (from SALES02)

| Activity | CORRECT tag | WRONG tag (causes ErrorActivity) |
|---|---|---|
| HTTP Request | `uwah:NetHttpRequest` | `ui:HttpClient` |
| Send Email | `umam:SendMailConnections` | `ui:SendOutlookMailMessage` |
| Parse JSON | `ui:DeserializeJson x:TypeArguments="njl:JObject"` | `ui:DeserializeJson` alone |
| Get Secret | `ui:GetSecret` | N/A |
| Queue Item | `upaq:AddQueueItemAndGetReference` | `ui:AddQueueItem` |
| Set Queue Status | `ui:SetTransactionStatus` | N/A |
| Main Diagram | `upa:ProcessDiagram` (REQUIRES TEMPLATE - package not available via NuGet) | `<Sequence>` works but not true long-running |

### 2. Main.xaml Structure

**CRITICAL**: The entire opening `<Activity>` tag must be on a single line for UiPath Studio to parse correctly.

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="Main" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <VisualBasic.Settings>
    <x:Null />
  </VisualBasic.Settings>
  <sap2010:WorkflowViewState.IdRef>ActivityBuilder_1</sap2010:WorkflowViewState.IdRef>
  <Sequence DisplayName="Main - Long Running">
    <Sequence.Variables>
      <!-- Config -->
      <Variable x:TypeArguments="scg:Dictionary(x:String, x:Object)" Name="in_Config" />
      <!-- Transaction -->
      <Variable x:TypeArguments="x:String" Name="TransactionID" />
      <Variable x:TypeArguments="x:Object" Name="TransactionData" />
      <!-- Approval -->
      <Variable x:TypeArguments="x:String" Name="TaskId" />
      <Variable x:TypeArguments="x:String" Name="TaskAction" />
      <Variable x:TypeArguments="x:String" Name="TaskData" />
      <!-- Status -->
      <Variable x:TypeArguments="x:Boolean" Name="IsApproved" />
      <Variable x:TypeArguments="x:String" Name="ErrorMessage" />
    </Sequence.Variables>
    
    <!-- INITIALIZATION -->
    <TryCatch DisplayName="Initialize">
      <TryCatch.Try>
        <Sequence>
          <ui:LogMessage Level="Info" Message="[&quot;Initializing...&quot;]" />
          <ui:InvokeWorkflowFile WorkflowFileName="Framework\InitAllSettings.xaml">
            <ui:InvokeWorkflowFile.Arguments>
              <OutArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)" x:Key="out_Config">[in_Config]</OutArgument>
            </ui:InvokeWorkflowFile.Arguments>
          </ui:InvokeWorkflowFile>
        </Sequence>
      </TryCatch.Try>
      <TryCatch.Catches>
        <Catch x:TypeArguments="s:Exception">
          <ActivityAction x:TypeArguments="s:Exception">
            <ActivityAction.Argument>
              <DelegateInArgument x:TypeArguments="s:Exception" Name="exception" />
            </ActivityAction.Argument>
            <ui:LogMessage Level="Fatal" Message="[&quot;Initialization failed: &quot; + exception.Message]" />
          </ActivityAction>
        </Catch>
      </TryCatch.Catches>
    </TryCatch>
    
    <!-- GET TRANSACTION DATA -->
    <ui:InvokeWorkflowFile WorkflowFileName="Workflows\GetTransactionData.xaml">
      <ui:InvokeWorkflowFile.Arguments>
        <InArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)" x:Key="in_Config">[in_Config]</InArgument>
        <OutArgument x:TypeArguments="x:String" x:Key="out_TransactionID">[TransactionID]</OutArgument>
        <OutArgument x:TypeArguments="x:Object" x:Key="out_TransactionData">[TransactionData]</OutArgument>
      </ui:InvokeWorkflowFile.Arguments>
    </ui:InvokeWorkflowFile>
    
    <!-- PROCESS TRANSACTION -->
    <If Condition="[TransactionID IsNot Nothing]">
      <If.Then>
        <TryCatch DisplayName="Process with Approval">
          <TryCatch.Try>
            <Sequence>
              <!-- Process business logic -->
              <ui:InvokeWorkflowFile WorkflowFileName="Workflows\ProcessTransaction.xaml">
                <ui:InvokeWorkflowFile.Arguments>
                  <InArgument x:TypeArguments="x:Object" x:Key="in_TransactionData">[TransactionData]</InArgument>
                </ui:InvokeWorkflowFile.Arguments>
              </ui:InvokeWorkflowFile>
              
              <!-- Create approval task -->
              <ui:InvokeWorkflowFile WorkflowFileName="Workflows\CreateApprovalTask.xaml">
                <ui:InvokeWorkflowFile.Arguments>
                  <InArgument x:TypeArguments="x:String" x:Key="in_TransactionID">[TransactionID]</InArgument>
                  <InArgument x:TypeArguments="x:Object" x:Key="in_TransactionData">[TransactionData]</InArgument>
                  <OutArgument x:TypeArguments="x:String" x:Key="out_TaskId">[TaskId]</OutArgument>
                </ui:InvokeWorkflowFile.Arguments>
              </ui:InvokeWorkflowFile>
              
              <!-- Wait for approval -->
              <ui:InvokeWorkflowFile WorkflowFileName="Workflows\WaitForApproval.xaml">
                <ui:InvokeWorkflowFile.Arguments>
                  <InArgument x:TypeArguments="x:String" x:Key="in_TaskId">[TaskId]</InArgument>
                  <OutArgument x:TypeArguments="x:String" x:Key="out_Action">[TaskAction]</OutArgument>
                  <OutArgument x:TypeArguments="x:String" x:Key="out_Data">[TaskData]</OutArgument>
                  <OutArgument x:TypeArguments="x:Boolean" x:Key="out_IsApproved">[IsApproved]</OutArgument>
                </ui:InvokeWorkflowFile.Arguments>
              </ui:InvokeWorkflowFile>
              
              <!-- Handle result -->
              <If Condition="[IsApproved]">
                <If.Then>
                  <ui:LogMessage Level="Info" Message="[&quot;Transaction approved: &quot; + TransactionID]" />
                </If.Then>
                <If.Else>
                  <ui:InvokeWorkflowFile WorkflowFileName="Workflows\HandleRejection.xaml">
                    <ui:InvokeWorkflowFile.Arguments>
                      <InArgument x:TypeArguments="x:String" x:Key="in_TransactionID">[TransactionID]</InArgument>
                      <InArgument x:TypeArguments="x:String" x:Key="in_TaskData">[TaskData]</InArgument>
                    </ui:InvokeWorkflowFile.Arguments>
                  </ui:InvokeWorkflowFile>
                </If.Else>
              </If>
            </Sequence>
          </TryCatch.Try>
          <TryCatch.Catches>
            <Catch x:TypeArguments="s:Exception">
              <ActivityAction x:TypeArguments="s:Exception">
                <ActivityAction.Argument>
                  <DelegateInArgument x:TypeArguments="s:Exception" Name="exception" />
                </ActivityAction.Argument>
                <Sequence>
                  <ui:LogMessage Level="Error" Message="[&quot;Error: &quot; + exception.Message]" />
                  <Assign>
                    <Assign.To>
                      <OutArgument x:TypeArguments="x:String">[ErrorMessage]</OutArgument>
                    </Assign.To>
                    <Assign.Value>
                      <InArgument x:TypeArguments="x:String">[exception.Message]</InArgument>
                    </Assign.Value>
                  </Assign>
                </Sequence>
              </ActivityAction>
            </Catch>
          </TryCatch.Catches>
        </TryCatch>
      </If.Then>
      <If.Else>
        <ui:LogMessage Level="Info" Message="[&quot;No transaction to process&quot;]" />
      </If.Else>
    </If>
    
    <!-- CLEANUP -->
    <ui:InvokeWorkflowFile WorkflowFileName="Framework\CloseAllApplications.xaml" />
  </Sequence>
</Activity>
```

### 3. Config.xlsx Structure

**Settings Sheet**:
| Name | Value | Description |
|------|-------|-------------|
| LogLevel | Info | Logging level |
| MaxRetries | 3 | Retry attempts |
| TimeoutMinutes | 1440 | Task timeout (24 hours) |
| OrchestratorQueueName | ApprovalQueue | Queue name |

**Constants Sheet**:
| Name | Value | Description |
|------|-------|-------------|
| ProcessName | RenewalApproval | Process identifier |
| Version | 1.0.0 | Current version |

**Assets Sheet**:
| Name | Type | Description |
|------|------|-------------|
| Salesforce_Credential | Credential | SF login |
| Outlook_Credential | Credential | Outlook login |

### 4. InitAllSettings.xaml

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="InitAllSettings" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <x:Members>
    <x:Property Name="out_Config" Type="OutArgument(scg:Dictionary(x:String, x:Object))" />
  </x:Members>
  <VisualBasic.Settings>
    <x:Null />
  </VisualBasic.Settings>
  <sap2010:WorkflowViewState.IdRef>ActivityBuilder_1</sap2010:WorkflowViewState.IdRef>
  <Sequence DisplayName="Init All Settings">
    <Sequence.Variables>
      <Variable x:TypeArguments="sd:DataTable" Name="dtSettings" />
      <Variable x:TypeArguments="sd:DataTable" Name="dtConstants" />
      <Variable x:TypeArguments="scg:Dictionary(x:String, x:Object)" Name="dictConfig" />
    </Sequence.Variables>
    
    <!-- Initialize dictionary -->
    <Assign>
      <Assign.To>
        <OutArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)">[dictConfig]</OutArgument>
      </Assign.To>
      <Assign.Value>
        <InArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)">
          [New Dictionary(Of String, Object)]
        </InArgument>
      </Assign.Value>
    </Assign>
    
    <!-- Read Config.xlsx -->
    <ui:ExcelApplicationScope>
      <ui:ExcelReadRange SheetName="Settings" Range="A1:C100" Result="[dtSettings]" />
      <ui:ExcelReadRange SheetName="Constants" Range="A1:C100" Result="[dtConstants]" />
    </ui:ExcelApplicationScope>
    
    <!-- Load settings into dictionary -->
    <ui:ForEachRow DataTable="[dtSettings]">
      <ActivityAction x:TypeArguments="sd:DataRow">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="sd:DataRow" Name="row" />
        </ActivityAction.Argument>
        <If Condition="[row(&quot;Name&quot;) IsNot Nothing AndAlso row(&quot;Value&quot;) IsNot Nothing]">
          <If.Then>
            <Invoke Method="Add">
              <Invoke.TargetObject>
                <InArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)">[dictConfig]</InArgument>
              </Invoke.TargetObject>
              <InArgument x:TypeArguments="x:String">[row(&quot;Name&quot;).ToString]</InArgument>
              <InArgument x:TypeArguments="x:Object">[row(&quot;Value&quot;)]</InArgument>
            </Invoke>
          </If.Then>
        </If>
      </ActivityAction>
    </ui:ForEachRow>
    
    <!-- Return config -->
    <Assign>
      <Assign.To>
        <OutArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)">[out_Config]</OutArgument>
      </Assign.To>
      <Assign.Value>
        <InArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)">[dictConfig]</InArgument>
      </Assign.Value>
    </Assign>
    
    <ui:LogMessage Level="Info" Message="[&quot;Configuration loaded: &quot; + dictConfig.Count.ToString + &quot; settings&quot;]" />
  </Sequence>
</Activity>
```

### 5. CreateApprovalTask.xaml

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="CreateApprovalTask" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <x:Members>
    <x:Property Name="in_TransactionID" Type="InArgument(x:String)" />
    <x:Property Name="in_TransactionData" Type="InArgument(x:Object)" />
    <x:Property Name="in_Config" Type="InArgument(scg:Dictionary(x:String, x:Object))" />
    <x:Property Name="out_TaskId" Type="OutArgument(x:String)" />
  </x:Members>
  <VisualBasic.Settings>
    <x:Null />
  </VisualBasic.Settings>
  <sap2010:WorkflowViewState.IdRef>ActivityBuilder_1</sap2010:WorkflowViewState.IdRef>
  <Sequence DisplayName="Create Approval Task">
    <Sequence.Variables>
      <Variable x:TypeArguments="x:String" Name="taskTitle" />
      <Variable x:TypeArguments="x:String" Name="taskDataJson" />
    </Sequence.Variables>
    
    <!-- Build task title -->
    <Assign>
      <Assign.To>
        <OutArgument x:TypeArguments="x:String">[taskTitle]</OutArgument>
      </Assign.To>
      <Assign.Value>
        <InArgument x:TypeArguments="x:String">
          [String.Format("Approve: {0}", in_TransactionID)]
        </InArgument>
      </Assign.Value>
    </Assign>
    
    <!-- Build task data JSON -->
    <Assign>
      <Assign.To>
        <OutArgument x:TypeArguments="x:String">[taskDataJson]</OutArgument>
      </Assign.To>
      <Assign.Value>
        <InArgument x:TypeArguments="x:String">
          [Newtonsoft.Json.JsonConvert.SerializeObject(in_TransactionData)]
        </InArgument>
      </Assign.Value>
    </Assign>
    
    <!-- Create form task -->
    <ui:CreateFormTask
      DisplayName="Create Form Task"
      TaskTitle="[taskTitle]"
      TaskPriority="Normal"
      FormSchemaPath="FormSchemas\ApprovalForm.json"
      TaskDataJson="[taskDataJson]"
      Result="[out_TaskId]"
      ContinueOnError="False">
      <ui:CreateFormTask.Timeout>
        <InArgument x:TypeArguments="x:TimeSpan">
          [TimeSpan.FromMinutes(CDbl(in_Config(&quot;TimeoutMinutes&quot;)))]
        </InArgument>
      </ui:CreateFormTask.Timeout>
    </ui:CreateFormTask>
    
    <ui:LogMessage Level="Info" Message="[&quot;Task created: &quot; + out_TaskId]" />
  </Sequence>
</Activity>
```

### 6. WaitForApproval.xaml

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="WaitForApproval" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:nj="clr-namespace:Newtonsoft.Json.Linq;assembly=Newtonsoft.Json" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <x:Members>
    <x:Property Name="in_TaskId" Type="InArgument(x:String)" />
    <x:Property Name="out_Action" Type="OutArgument(x:String)" />
    <x:Property Name="out_Data" Type="OutArgument(x:String)" />
    <x:Property Name="out_IsApproved" Type="OutArgument(x:Boolean)" />
  </x:Members>
  <VisualBasic.Settings>
    <x:Null />
  </VisualBasic.Settings>
  <sap2010:WorkflowViewState.IdRef>ActivityBuilder_1</sap2010:WorkflowViewState.IdRef>
  <Sequence DisplayName="Wait For Approval">
    <Sequence.Variables>
      <Variable x:TypeArguments="nj:JObject" Name="formData" />
    </Sequence.Variables>
    
    <ui:LogMessage Level="Info" Message="[&quot;Waiting for task: &quot; + in_TaskId]" />
    
    <!-- Wait for form task -->
    <ui:WaitForFormTaskAndResume
      DisplayName="Wait For Form Task And Resume"
      TaskId="[in_TaskId]"
      TaskAction="[out_Action]"
      TaskData="[out_Data]" />
    
    <ui:LogMessage Level="Info" Message="[&quot;Task completed with action: &quot; + out_Action]" />
    
    <!-- Parse response -->
    <TryCatch DisplayName="Parse Response">
      <TryCatch.Try>
        <Sequence>
          <Assign>
            <Assign.To>
              <OutArgument x:TypeArguments="nj:JObject">[formData]</OutArgument>
            </Assign.To>
            <Assign.Value>
              <InArgument x:TypeArguments="nj:JObject">
                [nj:JObject.Parse(out_Data)]
              </InArgument>
            </Assign.Value>
          </Assign>
          
          <Assign>
            <Assign.To>
              <OutArgument x:TypeArguments="x:Boolean">[out_IsApproved]</OutArgument>
            </Assign.To>
            <Assign.Value>
              <InArgument x:TypeArguments="x:Boolean">
                [formData("approved").Value(Of Boolean)]
              </InArgument>
            </Assign.Value>
          </Assign>
        </Sequence>
      </TryCatch.Try>
      <TryCatch.Catches>
        <Catch x:TypeArguments="s:Exception">
          <ActivityAction x:TypeArguments="s:Exception">
            <ActivityAction.Argument>
              <DelegateInArgument x:TypeArguments="s:Exception" Name="exception" />
            </ActivityAction.Argument>
            <Sequence>
              <ui:LogMessage Level="Error" Message="[&quot;Failed to parse response: &quot; + exception.Message]" />
              <Assign>
                <Assign.To>
                  <OutArgument x:TypeArguments="x:Boolean">[out_IsApproved]</OutArgument>
                </Assign.To>
                <Assign.Value>
                  <InArgument x:TypeArguments="x:Boolean">False</InArgument>
                </Assign.Value>
              </Assign>
            </Sequence>
          </ActivityAction>
        </Catch>
      </TryCatch.Catches>
    </TryCatch>
  </Sequence>
</Activity>
```

### 7. Form Schema with Proper Fields

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "Approval Form",
  "properties": {
    "transactionId": {
      "type": "string",
      "title": "Transaction ID",
      "readOnly": true
    },
    "requestor": {
      "type": "string",
      "title": "Requestor",
      "readOnly": true
    },
    "amount": {
      "type": "number",
      "title": "Amount",
      "readOnly": true
    },
    "description": {
      "type": "string",
      "title": "Description",
      "readOnly": true
    },
    "approved": {
      "type": "boolean",
      "title": "Approve this request?",
      "default": false
    },
    "comments": {
      "type": "string",
      "title": "Comments",
      "description": "Optional comments or reason for decision"
    }
  },
  "required": ["approved"]
}
```

---

## Key Improvements Needed

### 1. Add Config.xlsx Support
- Settings management
- Environment-specific values
- Asset references

### 2. Add Framework Workflows
- InitAllSettings.xaml
- InitAllApplications.xaml
- CloseAllApplications.xaml

### 3. Add Business Logic Workflows
- GetTransactionData.xaml (queue integration)
- ProcessTransaction.xaml (actual work)
- HandleRejection.xaml (rejection logic)

### 4. Improve XAML Structure
- Complete xmlns declarations
- Proper variable scoping
- Try-Catch blocks everywhere
- Timeout handling

### 5. Better Form Schemas
- Read-only fields for context
- Proper field types
- Validation rules
- Default values

---

## Next Steps

To improve the generator, I need to know:
1. What specifically failed in SALES03?
2. What patterns from the Azure template are most important?
3. Should I add Config.xlsx generation?
4. Should I add Framework workflows?
5. Should I add queue integration?

Please share the key patterns from the Azure template or describe what failed, and I'll update the generator accordingly.
