# Production XAML Template

Complete XAML extracted from the ApprovalCycleWorkflowTemplate for reference.

**Source**: `C:\Users\DanielaRosenstein\OneDrive - Cato Networks\Documents\UiPath\ApprovalCycleWorkflowTemplate\Main.xaml`

> **IMPORTANT: Actual UiPath Project Location**
> 
> The actual UiPath Studio project for SALES02 is located at:
> `C:\Users\DanielaRosenstein\OneDrive - Cato Networks\Documents\UiPath\SALES02_RenewalPriceCommitment`

## project.json

```json
{
  "name": "ApprovalCycleWorkflowTemplate",
  "projectId": "409e3b9a-e946-43d0-a6e4-0bd4f1a12c9f",
  "description": "ApprovalCycle Workflow - Creates HITL approval and sends email and Slack notification",
  "main": "Main.xaml",
  "dependencies": {
    "UiPath.HttpWebhook.IntegrationService.Activities": "[5.0.0-preview]",
    "UiPath.IntegrationService.Activities": "[1.0.2]",
    "UiPath.Mail.Activities": "[1.23.0]",
    "UiPath.MicrosoftOffice365.Activities": "[3.6.2-preview]",
    "UiPath.System.Activities": "[24.10.6]",
    "UiPath.UIAutomation.Activities": "[25.10.25]",
    "UiPath.WebAPI.Activities": "[2.3.2]"
  },
  "schemaVersion": "4.0",
  "studioVersion": "24.10.0.0",
  "projectVersion": "1.0.2",
  "runtimeOptions": {
    "autoDispose": false,
    "netFrameworkLazyLoading": false,
    "isPausable": true,
    "isAttended": false,
    "requiresUserInteraction": false,
    "supportsPersistence": false,
    "workflowSerialization": "DataContract",
    "excludedLoggedData": ["Private:*", "*password*"],
    "executionType": "Workflow",
    "mustRestoreAllDependencies": true
  },
  "designOptions": {
    "projectProfile": "Developement",
    "outputType": "Process"
  },
  "expressionLanguage": "VisualBasic",
  "entryPoints": [
    {
      "filePath": "Main.xaml",
      "uniqueId": "376c7665-07d1-4876-bc98-79ec51cddcfc",
      "input": [],
      "output": []
    }
  ],
  "isTemplate": true,
  "targetFramework": "Windows"
}
```

---

## Complete Activity Tag (Single Line - CRITICAL)

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="Main" this:Main.in_ProcessId="PROC-1001" this:Main.in_NotificationChannel="both" this:Main.in_ApprovalProcessName="Sales Request" this:Main.in_Title="Sales Deal Approval" this:Main.in_ApproverEmail="approver@company.com" this:Main.in_Amount="1" this:Main.in_CallBackURL="https://cloud.uipath.com/.../webhooks/events/{webhookId}" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:njl="clr-namespace:Newtonsoft.Json.Linq;assembly=Newtonsoft.Json" xmlns:s="clr-namespace:System;assembly=System.Private.CoreLib" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:umam="clr-namespace:UiPath.MicrosoftOffice365.Activities.Mail;assembly=UiPath.MicrosoftOffice365.Activities" xmlns:umame="clr-namespace:UiPath.MicrosoftOffice365.Activities.Mail.Enums;assembly=UiPath.MicrosoftOffice365.Activities" xmlns:umamm="clr-namespace:UiPath.MicrosoftOffice365.Activities.Mail.Models;assembly=UiPath.MicrosoftOffice365.Activities" xmlns:usau="clr-namespace:UiPath.Shared.Activities.Utils;assembly=UiPath.MicrosoftOffice365.Activities" xmlns:uwah="clr-namespace:UiPath.Web.Activities.Http;assembly=UiPath.Web.Activities" xmlns:uwahm="clr-namespace:UiPath.Web.Activities.Http.Models;assembly=UiPath.Web.Activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
```

---

## Arguments Declaration

```xml
<x:Members>
  <x:Property Name="in_ProcessId" Type="InArgument(x:String)" />
  <x:Property Name="in_DealName" Type="InArgument(x:String)" />
  <x:Property Name="in_DealAmount" Type="InArgument(x:Decimal)" />
  <x:Property Name="in_CustomerName" Type="InArgument(x:String)" />
  <x:Property Name="in_CpiPercentage" Type="InArgument(x:Decimal)" />
  <x:Property Name="in_AgreementMonths" Type="InArgument(x:Int32)" />
  <x:Property Name="in_BusinessJustification" Type="InArgument(x:String)" />
  <x:Property Name="in_NotificationChannel" Type="InArgument(x:String)" />
  <x:Property Name="out_ApprovalId" Type="OutArgument(x:String)" />
  <x:Property Name="out_ResponseToken" Type="OutArgument(x:String)" />
  <x:Property Name="in_RequestURL" Type="InArgument(x:String)" />
  <x:Property Name="in_APIKey" Type="InArgument(x:String)" />
  <x:Property Name="in_ApprovalProcessName" Type="InArgument(x:String)" />
  <x:Property Name="in_formSchema" Type="InArgument(x:String)" />
  <x:Property Name="in_Title" Type="InArgument(x:String)" />
  <x:Property Name="in_formData" Type="InArgument(x:String)" />
  <x:Property Name="in_ApproverEmail" Type="InArgument(x:String)" />
  <x:Property Name="in_Amount" Type="InArgument(x:String)" />
  <x:Property Name="in_FactSet" Type="InArgument(x:String)" />
  <x:Property Name="in_CallBackURL" Type="InArgument(x:String)" />
</x:Members>
```

---

## Variables Declaration

```xml
<Sequence.Variables>
  <Variable x:TypeArguments="x:String" Name="v_ResponseToken" />
  <Variable x:TypeArguments="x:String" Name="v_ApprovalId" />
  <Variable x:TypeArguments="x:String" Name="OpenFormUrl" />
  <Variable x:TypeArguments="x:String" Name="ApproveUrl" />
  <Variable x:TypeArguments="x:String" Name="RejectUrl" />
  <Variable x:TypeArguments="x:String" Name="HttpResponse" />
  <Variable x:TypeArguments="x:String" Name="EmailBody" />
  <Variable x:TypeArguments="x:Int32" Name="HttpStatusCode" />
  <Variable x:TypeArguments="x:String" Name="RequestBody" />
  <Variable x:TypeArguments="uwahm:HttpResponseSummary" Name="responseContent" />
  <Variable x:TypeArguments="njl:JObject" Name="httpRoseponseJSON" />
</Sequence.Variables>
```

---

## HTTP Request Pattern

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
  RetryStatusCodes="[New List (Of System.Net.HttpStatusCode) From { System.Net.HttpStatusCode.RequestTimeout, System.Net.HttpStatusCode.TooManyRequests, System.Net.HttpStatusCode.InternalServerError, System.Net.HttpStatusCode.BadGateway, System.Net.HttpStatusCode.ServiceUnavailable, System.Net.HttpStatusCode.GatewayTimeout }]" 
  SaveRawRequestResponse="False" 
  SaveResponseAsFile="False" 
  TextPayload="[RequestBody]" 
  TextPayloadContentType="[&quot;application/json&quot;]" 
  TextPayloadEncoding="[&quot;UTF-8&quot;]" 
  TlsProtocol="Automatic" 
  UseJitter="True">
  <uwah:NetHttpRequest.TimeoutInMiliseconds>
    <InArgument x:TypeArguments="s:Nullable(x:Int32)">
      <Literal x:TypeArguments="s:Nullable(x:Int32)" Value="10000" />
    </InArgument>
  </uwah:NetHttpRequest.TimeoutInMiliseconds>
</uwah:NetHttpRequest>
```

---

## JSON Deserialization Pattern

```xml
<ui:DeserializeJson x:TypeArguments="njl:JObject" 
  DisplayName="Deserialize JSON" 
  JsonObject="[httpRoseponseJSON]" 
  JsonString="[responseContent.TextContent.ToString]" />
```

---

## Extract Response Token

```xml
<Assign DisplayName="Extract Response Token (v_ResponseToken)">
  <Assign.To>
    <OutArgument x:TypeArguments="x:String">[v_ResponseToken]</OutArgument>
  </Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String">[httpRoseponseJSON("approvalLines")(0)("responseToken").ToString]</InArgument>
  </Assign.Value>
</Assign>
```

---

## Extract Approval ID

```xml
<Assign DisplayName="Extract Approval ID (v_ApprovalId)">
  <Assign.To>
    <OutArgument x:TypeArguments="x:String">[v_ApprovalId]</OutArgument>
  </Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String">[httpRoseponseJSON("id").ToString]</InArgument>
  </Assign.Value>
</Assign>
```

---

## Build URLs

```xml
<Assign DisplayName="Build Approve URL">
  <Assign.To>
    <OutArgument x:TypeArguments="x:String">[ApproveUrl]</OutArgument>
  </Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String">["https://djun97l419cdy.cloudfront.net/api/v1/approvals/adaptive-card/" &amp; v_ResponseToken]</InArgument>
  </Assign.Value>
</Assign>

<Assign DisplayName="Build Reject URL">
  <Assign.To>
    <OutArgument x:TypeArguments="x:String">[RejectUrl]</OutArgument>
  </Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String">["https://djun97l419cdy.cloudfront.net/api/v1/approvals/adaptive-card/" &amp; v_ResponseToken]</InArgument>
  </Assign.Value>
</Assign>
```

---

## Conditional Email Notification

```xml
<If Condition="[in_NotificationChannel.ToLower() = &quot;email&quot; OrElse in_NotificationChannel.ToLower() = &quot;both&quot;]" DisplayName="If notification channel is email or both, send an email">
  <If.Then>
    <Sequence DisplayName="Send Approval Email">
      <!-- Build and send email -->
    </Sequence>
  </If.Then>
  <If.Else>
    <Sequence DisplayName="Skip Email - Different Notification Channel">
      <ui:LogMessage DisplayName="Log: Email Skipped" Level="Info" Message="[&quot;Email notification skipped. Channel is: &quot; &amp; in_NotificationChannel]" />
    </Sequence>
  </If.Else>
</If>
```

---

## Send Email via Microsoft Graph

```xml
<umam:SendMailConnections 
  Body="[EmailBody]" 
  ConnectionId="3ee6f8bc-b39b-413d-b96b-59244fce5cea" 
  DisplayName="Send Email" 
  InputType="HTML" 
  SaveAsDraft="False" 
  To="[New String(){&quot;approver@company.com&quot;}]" 
  UseConnectionService="True" 
  UseSharedMailbox="False">
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

---

## Form Schema String Pattern

```vb
in_formSchema = """form_schema"":[
  {""name"":""requesterName"",""type"":""readonly"",""label"":""Requester""},
  {""name"":""customerName"",""type"":""readonly"",""label"":""Customer Name""},
  {""name"":""deal_amount"",""type"":""number"",""label"":""Deal Amount"",""required"":true,""sensitive"":true,""min"":1},
  {""name"":""agreement_months"",""type"":""select"",""label"":""Agreement (Months)"",""required"":true,""options"":[
    {""value"":""12"",""label"":""12""},
    {""value"":""24"",""label"":""24""},
    {""value"":""36"",""label"":""36""}
  ]},
  {""name"":""wants_renewal"",""type"":""checkbox"",""label"":""Auto-Renew""},
  {""name"":""start_date"",""type"":""date"",""label"":""Start Date""},
  {""name"":""meeting_time"",""type"":""time"",""label"":""Meeting Time""},
  {""name"":""business_justification"",""type"":""textarea"",""label"":""Business Justification"",""required"":true}
]"
```

---

## Form Data String Pattern

```vb
in_formData = """form_data"":{
  ""requesterName"":""" + in_ApproverEmail + """,
  ""customerName"":""" + in_CustomerName + """,
  ""deal_amount"":75000,
  ""agreement_months"":""24"",
  ""start_date"":""2026-02-15""
},
""metadata"":{
  ""source_id"":""CRM-447"",
  ""cost_center"":""SALES""
}"
```

---

## Request Body String Pattern

```vb
RequestBody = "{
  ""process_id"":""" + in_ProcessId + """,
  ""title"":""" + in_Title + """,
  ""notification_channel"":""" + in_NotificationChannel + """,
  " + in_formSchema + ",
  " + in_formData + ",
  ""approvers"":[{
    ""email"":""" + in_ApproverEmail + """,
    ""notificationChannel"":""both""
  }],
  ""callbackUrl"":""" + in_CallBackURL + """
}"
```

---

## TextExpression.NamespacesForImplementation

```xml
<TextExpression.NamespacesForImplementation>
  <sco:Collection x:TypeArguments="x:String">
    <x:String>GlobalConstantsNamespace</x:String>
    <x:String>GlobalVariablesNamespace</x:String>
    <x:String>System</x:String>
    <x:String>System.Collections.Generic</x:String>
    <x:String>System.Collections.ObjectModel</x:String>
    <x:String>System.Linq</x:String>
    <x:String>UiPath.Core</x:String>
    <x:String>UiPath.Core.Activities</x:String>
    <x:String>System.Activities</x:String>
    <x:String>UiPath.Mail</x:String>
    <x:String>UiPath.Mail.Activities.Business</x:String>
    <x:String>UiPath.Shared.Activities</x:String>
    <x:String>UiPath.Shared.Activities.ConnectionService.Contracts</x:String>
    <x:String>UiPath.Mail.Activities</x:String>
    <x:String>System.Activities.Statements</x:String>
    <x:String>UiPath.Web.Activities.Http.Models</x:String>
    <x:String>System.Runtime.Serialization</x:String>
    <x:String>System.Security</x:String>
    <x:String>System.Net</x:String>
    <x:String>UiPath.Platform.ResourceHandling</x:String>
    <x:String>System.Reflection</x:String>
    <x:String>UiPath.Web.Activities.Http</x:String>
    <x:String>UiPath.Web.Activities</x:String>
    <x:String>UiPath.MicrosoftOffice365.Activities.Mail.Models</x:String>
    <x:String>UiPath.MicrosoftOffice365.Activities.Mail.Enums</x:String>
    <x:String>UiPath.Shared.Activities.Utils</x:String>
    <x:String>UiPath.MicrosoftOffice365.Activities.Mail</x:String>
    <x:String>UiPath.MicrosoftOffice365.Activities</x:String>
    <x:String>Newtonsoft.Json.Linq</x:String>
    <x:String>Newtonsoft.Json</x:String>
    <x:String>System.Dynamic</x:String>
    <x:String>System.ComponentModel</x:String>
    <x:String>UiPath.HttpWebhook.IntegrationService.Activities</x:String>
  </sco:Collection>
</TextExpression.NamespacesForImplementation>
```

---

## TextExpression.ReferencesForImplementation

```xml
<TextExpression.ReferencesForImplementation>
  <sco:Collection x:TypeArguments="AssemblyReference">
    <AssemblyReference>Newtonsoft.Json</AssemblyReference>
    <AssemblyReference>System.Activities</AssemblyReference>
    <AssemblyReference>System.Collections</AssemblyReference>
    <AssemblyReference>System.ComponentModel.TypeConverter</AssemblyReference>
    <AssemblyReference>System.Linq</AssemblyReference>
    <AssemblyReference>System.ObjectModel</AssemblyReference>
    <AssemblyReference>System.Private.CoreLib</AssemblyReference>
    <AssemblyReference>UiPath.Mail</AssemblyReference>
    <AssemblyReference>UiPath.Mail.Activities</AssemblyReference>
    <AssemblyReference>UiPath.System.Activities</AssemblyReference>
    <AssemblyReference>UiPath.Web.Activities</AssemblyReference>
    <AssemblyReference>UiPath.MicrosoftOffice365.Activities</AssemblyReference>
    <AssemblyReference>UiPath.HttpWebhook.IntegrationService.Activities</AssemblyReference>
  </sco:Collection>
</TextExpression.ReferencesForImplementation>
```
