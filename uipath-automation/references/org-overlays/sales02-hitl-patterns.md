# Org overlay: HITL platform and SALES02 production patterns

Optional **organization-specific** reference: human-in-the-loop HTTP integration, Adaptive Card email, Microsoft Graph mail, NetHttpRequest, Orchestrator secrets, queue/LRW ProcessDiagram patterns, and sample `project.json` dependencies. Skip this file if you use Action Center only or different integrations.

## HITL Platform Integration (For Approval Workflows)

**IMPORTANT**: For human-in-the-loop approval workflows, use the custom HITL Platform instead of UiPath Action Center.

See the **Long-Running Workflow Builder** skill for complete HITL Platform integration patterns.

### Quick Reference

| Feature | HITL Platform | Action Center |
|---------|---------------|---------------|
| Task Creation | HTTP POST to `/api/v1/approvals` | `CreateFormTask` |
| Wait for Response | Webhook callback | `WaitForFormTaskAndResume` |
| Notifications | Adaptive Cards + Slack | Action Center UI only |
| Multi-channel | Email, Slack, Web | Web only |

### HITL Platform API Call Pattern

```xml
<uwah:NetHttpRequest
  Method="POST"
  RequestUrl="[HITL_API_URL + &quot;/api/v1/approvals&quot;]"
  Headers="[new Dictionary(Of String, String) From { { &quot;Content-Type&quot;, &quot;application/json&quot; }, { &quot;x-api-key&quot;, HITL_API_Key } }]"
  TextPayload="[RequestBody]"
  Result="[responseContent]"
  RetryCount="3" />
```

### Template Reference

Store organization-specific templates in **your repo or shared drive** and reference them by path variable or documentation—not hardcoded machine paths. Typical layout:

- **Approval / long-running template** (example): clone or submodule your org’s BPMN or ProcessDiagram template repository.
- **HITL platform services**: integrate via HTTP/API as configured in your environment (base URL from Orchestrator asset, environment variable, or config file).

---

## Critical Activity Patterns (from production reference SALES02)

### Required Dependencies (project.json)
These are the exact packages required for approval/HTTP/email workflows:
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

### Required Namespaces for Approval/HTTP Workflows
Add these to the `<Activity>` tag (single line) and to `TextExpression.NamespacesForImplementation`:
```
xmlns:njl="clr-namespace:Newtonsoft.Json.Linq;assembly=Newtonsoft.Json"
xmlns:uwah="clr-namespace:UiPath.Web.Activities.Http;assembly=UiPath.Web.Activities"
xmlns:uwahm="clr-namespace:UiPath.Web.Activities.Http.Models;assembly=UiPath.Web.Activities"
xmlns:umam="clr-namespace:UiPath.MicrosoftOffice365.Activities.Mail;assembly=UiPath.MicrosoftOffice365.Activities"
xmlns:umame="clr-namespace:UiPath.MicrosoftOffice365.Activities.Mail.Enums;assembly=UiPath.MicrosoftOffice365.Activities"
xmlns:umamm="clr-namespace:UiPath.MicrosoftOffice365.Activities.Mail.Models;assembly=UiPath.MicrosoftOffice365.Activities"
xmlns:usau="clr-namespace:UiPath.Shared.Activities.Utils;assembly=UiPath.MicrosoftOffice365.Activities"
xmlns:ss="clr-namespace:System.Security;assembly=System.Private.CoreLib"
```

### HTTP Request (WebAPI — NOT ui:HttpClient)
**ALWAYS use `uwah:NetHttpRequest` from UiPath.WebAPI.Activities:**
```xml
<uwah:NetHttpRequest
  AuthenticationType="None"
  ContinueOnError="True"
  Cookies="[New System.Collections.Generic.Dictionary(Of String, String) From {}]"
  DisableSslVerification="False"
  DisplayName="HTTP Request"
  EnableCookies="True"
  FollowRedirects="True"
  FormData="[New System.Collections.Generic.Dictionary(Of String, String) From {}]"
  Headers="[new System.Collections.Generic.Dictionary(Of System.String, System.String) From { { &quot;Content-Type&quot;, &quot;application/json&quot; },{ &quot;x-api-key&quot;, in_APIKey } }]"
  Method="POST"
  Parameters="[New System.Collections.Generic.Dictionary(Of String, String) From {}]"
  RequestBodyType="Text"
  RequestUrl="[in_RequestURL]"
  Result="[responseContent]"
  RetryCount="3"
  RetryPolicyType="Basic"
  SaveRawRequestResponse="False"
  SaveResponseAsFile="False"
  TextPayload="[RequestBody]"
  TextPayloadContentType="[&quot;application/json&quot;]"
  TextPayloadEncoding="[&quot;UTF-8&quot;]"
  TlsProtocol="Automatic"
  sap2010:WorkflowViewState.IdRef="NetHttpRequest_1">
  <uwah:NetHttpRequest.TimeoutInMiliseconds>
    <InArgument x:TypeArguments="s:Nullable(x:Int32)">
      <Literal x:TypeArguments="s:Nullable(x:Int32)" Value="10000" />
    </InArgument>
  </uwah:NetHttpRequest.TimeoutInMiliseconds>
</uwah:NetHttpRequest>
```
- Response is in `responseContent.TextContent.ToString` (type: `uwahm:HttpResponseSummary`)
- Parse with: `<ui:DeserializeJson x:TypeArguments="njl:JObject" JsonString="[responseContent.TextContent.ToString]" JsonObject="[httpResponseJSON]" />`
- Access fields: `httpResponseJSON("fieldName").ToString`

### Get Orchestrator Secret
```xml
<ui:GetSecret AssetName="ApprovalApp_APIKey" DisplayName="Get Secret"
  FolderPath="[in_OrchestratorFolderPath]"
  Secret="[APISecret]"
  sap2010:WorkflowViewState.IdRef="GetSecret_1" />
```
Then convert SecureString to plain string:
```xml
<Assign DisplayName="Assign apikey">
  <Assign.To><OutArgument x:TypeArguments="x:String">[in_APIKey]</OutArgument></Assign.To>
  <Assign.Value><InArgument x:TypeArguments="x:String">[New System.Net.NetworkCredential(String.Empty, APISecret).Password]</InArgument></Assign.Value>
</Assign>
```

### Send Email via Microsoft Graph (Office365 Integration Service)
**ALWAYS use `umam:SendMailConnections` — NOT `ui:SendOutlookMailMessage`:**
```xml
<umam:SendMailConnections
  Body="[EmailBody]"
  ConnectionId="YOUR-CONNECTION-ID"
  DisplayName="Send Email"
  InputType="HTML"
  SaveAsDraft="False"
  To="[New String(){in_ApproverEmail}]"
  UseConnectionService="True"
  UseSharedMailbox="False"
  sap2010:WorkflowViewState.IdRef="SendMailConnections_1">
  <umam:SendMailConnections.AttachmentsBackup>
    <usau:BackupSlot x:TypeArguments="umame:AttachmentInputMode" StoredValue="Existing">
      <usau:BackupSlot.BackupValues>
        <scg:Dictionary x:TypeArguments="umame:AttachmentInputMode, scg:List(x:Object)" />
      </usau:BackupSlot.BackupValues>
    </usau:BackupSlot>
  </umam:SendMailConnections.AttachmentsBackup>
  <umam:SendMailConnections.InputTypeBackup>
    <usau:BackupSlot x:TypeArguments="umame:BodyInputType" StoredValue="HTML">
      <usau:BackupSlot.BackupValues>
        <scg:Dictionary x:TypeArguments="umame:BodyInputType, scg:List(x:Object)" />
      </usau:BackupSlot.BackupValues>
    </usau:BackupSlot>
  </umam:SendMailConnections.InputTypeBackup>
  <umam:SendMailConnections.MailboxArg>
    <umamm:MailboxArgument SharedMailbox="{x:Null}" UseSharedMailbox="False">
      <umamm:MailboxArgument.Backup>
        <usau:BackupSlot x:TypeArguments="umame:MailboxSelectionMode" StoredValue="NoMailbox">
          <usau:BackupSlot.BackupValues>
            <scg:List x:TypeArguments="x:Object" x:Key="NoMailbox" Capacity="1"><x:Null /></scg:List>
            <scg:List x:TypeArguments="x:Object" x:Key="UseISMailbox" Capacity="1"><x:Null /></scg:List>
          </usau:BackupSlot.BackupValues>
        </usau:BackupSlot>
      </umamm:MailboxArgument.Backup>
    </umamm:MailboxArgument>
  </umam:SendMailConnections.MailboxArg>
</umam:SendMailConnections>
```

### Adaptive Card Email Body Pattern

**CRITICAL**: Adaptive Card emails require specific escaping rules:

1. **Action.Http URLs MUST use HTTPS** - HTTP URLs will fail with "Target URL scheme 'http' is not allowed"
2. **Use `\""` (single backslash)** for nested JSON quotes in Action.Http body - NOT `\\""` (double backslash)
3. **Use XML entities** for HTML tags: `&lt;` for `<`, `&gt;` for `>`

**Production pattern:** `https://<your-hitl-api-host>/api/v1/approvals/adaptive-card/{token}` (must be **HTTPS**; replace host with your deployment).

```xml
<Assign DisplayName="Build Email Body HTML">
  <Assign.To><OutArgument x:TypeArguments="x:String">[EmailBody]</OutArgument></Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String">["&lt;html&gt;&lt;head&gt;&lt;meta http-equiv=""Content-Type"" content=""text/html; charset=utf-8""&gt;&lt;script type=""application/adaptivecard+json""&gt;{""type"":""AdaptiveCard"",""version"":""1.0"",""originator"":""61fed71d-3b8c-4605-8f35-d95b70ab0803"",""hideOriginalBody"":true,""body"":[{""type"":""TextBlock"",""size"":""Large"",""weight"":""Bolder"",""text"":""" &amp; in_Title &amp; """}," &amp; FactSet &amp; "],""actions"":[{""type"":""Action.Http"",""title"":""Approve"",""method"":""POST"",""url"":""" &amp; ApproveUrl &amp; """,""style"":""positive"",""headers"":[{""name"":""Content-Type"",""value"":""application/json""}],""body"":""{\""action\"":\""approved\"",\""comments\"":\""{{comments.value}}\""}""}]}&lt;/script&gt;&lt;/head&gt;&lt;body&gt;&lt;p&gt;Please open in Outlook to view the approval form.&lt;/p&gt;&lt;/body&gt;&lt;/html&gt;"]</InArgument>
  </Assign.Value>
</Assign>
```

### Approve URL Assignment (MUST use HTTPS)

```xml
<Assign DisplayName="Build Approve URL">
  <Assign.To><OutArgument x:TypeArguments="x:String">[ApproveUrl]</OutArgument></Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String">[in_HitlBaseUrlHttps &amp; "/api/v1/approvals/adaptive-card/" &amp; v_ResponseToken]</InArgument>
  </Assign.Value>
</Assign>
```

**WRONG** (will fail): HTTP base URL from config for Action.Http targets.
**CORRECT**: HTTPS base URL only—e.g. `[in_HitlBaseUrlHttps & "/api/v1/approvals/adaptive-card/" & token]` where `in_HitlBaseUrlHttps` starts with `https://`.

### Build FactSet Variable (Single Line)

```xml
<Assign DisplayName="Build FactSet JSON">
  <Assign.To><OutArgument x:TypeArguments="x:String">[FactSet]</OutArgument></Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String">["{""type"":""FactSet"",""facts"":[{""title"":""Customer"",""value"":""" + in_CustomerName + """},{""title"":""Amount"",""value"":""$" + in_DealAmount.ToString("N0") + """}]}"]</InArgument>
  </Assign.Value>
</Assign>
```

### Queue Operations (Long-Running / Persistence)

**Standard queue processing:** **`GetQueueItem`** (next transaction) → process → **`SetTransactionStatus`**. Use **`AddQueueItem`** / **`AddQueueItemAndGetReference`** when publishing work to a queue.

```xml
<!-- Get next transaction (Orchestrator queue) -->
<ui:GetQueueItem DisplayName="Get Next Transaction" QueueName="[in_QueueName]" TransactionItem="[TransactionItem]" />
```

```xml
<!-- Add item to queue -->
<upaq:AddQueueItemAndGetReference
  DisplayName="Add Queue Item And Get Reference"
  FolderPath="[in_OrchestratorFolderPath]"
  Priority="Normal"
  QueueItemOutput="[QueueItem]"
  QueueName="[in_QueueName]"
  Reference="[QuoteID]"
  sap2010:WorkflowViewState.IdRef="AddQueueItemAndGetReference_1">
  <upaq:AddQueueItemAndGetReference.ItemInformation>
    <InArgument x:TypeArguments="x:String" x:Key="accountName">[accountName]</InArgument>
    <InArgument x:TypeArguments="x:String" x:Key="acv">[acv]</InArgument>
    <InArgument x:TypeArguments="x:String" x:Key="QuoteID">[QuoteID]</InArgument>
  </upaq:AddQueueItemAndGetReference.ItemInformation>
</upaq:AddQueueItemAndGetReference>

<!-- Set transaction status -->
<ui:SetTransactionStatus
  DisplayName="Set Transaction Status"
  ErrorType="Business"
  Status="Successful"
  TransactionItem="[TransactionItem]"
  sap2010:WorkflowViewState.IdRef="SetTransactionStatus_1">
  <ui:SetTransactionStatus.Analytics>
    <scg:Dictionary x:TypeArguments="x:String, InArgument" />
  </ui:SetTransactionStatus.Analytics>
  <ui:SetTransactionStatus.Output>
    <scg:Dictionary x:TypeArguments="x:String, InArgument" />
  </ui:SetTransactionStatus.Output>
</ui:SetTransactionStatus>
```
Required namespaces: `xmlns:upaq="clr-namespace:UiPath.Persistence.Activities.Queue;assembly=UiPath.Persistence.Activities"`

### Main Long-Running Workflow (ProcessDiagram Pattern)

**CRITICAL WARNING**: ProcessDiagram workflows CANNOT be generated from scratch - the `UiPath.Process.Activities` package is not available via NuGet for Windows projects. You MUST use the template from GitHub: `https://github.com/cato-networks-IT/PROCESSNAME_LongRunningAutomationTemplate`

The main entry point for long-running workflows uses `upa:ProcessDiagram` (not `<Sequence>`):
```xml
<upa:ProcessDiagram DisplayName="Long Running Workflow" sap2010:WorkflowViewState.IdRef="ProcessDiagram_1">
  <upa:ProcessDiagram.Variables>
    <Variable x:TypeArguments="scg:Dictionary(x:String, x:Object)" Name="Config" />
    <Variable x:TypeArguments="ui:QueueItem" Name="TransactionItem" />
    <Variable x:TypeArguments="upaq:QueueItemData" Name="QueueItem" />
    <Variable x:TypeArguments="njl:JObject" Name="retrievedQuote" />
    <Variable x:TypeArguments="x:String" Name="QuoteID" />
  </upa:ProcessDiagram.Variables>
  <upa:ProcessDiagram.StartNode>
    <x:Reference>__ReferenceID0</x:Reference>
  </upa:ProcessDiagram.StartNode>
  <upa:EventNode x:Name="__ReferenceID0" DisplayName="Start" sap2010:WorkflowViewState.IdRef="EventNode_1">
    <upa:EventNode.Behavior>
      <upa:StartBehavior>
        <upa:StartBehavior.DesignerMetadata>
          <upas:DesignerMetadata NodeType="StartEvent.Interrupting.None" />
        </upa:StartBehavior.DesignerMetadata>
      </upa:StartBehavior>
    </upa:EventNode.Behavior>
    <upa:EventNode.Next>
      <!-- First task node -->
    </upa:EventNode.Next>
  </upa:EventNode>
</upa:ProcessDiagram>
```
Required namespaces:
```
xmlns:upa="clr-namespace:UiPath.Process.Activities;assembly=UiPath.Process.Activities"
xmlns:upaq="clr-namespace:UiPath.Persistence.Activities.Queue;assembly=UiPath.Persistence.Activities"
xmlns:upas="clr-namespace:UiPath.Process.Activities.Shared;assembly=UiPath.Process.Activities"
```
