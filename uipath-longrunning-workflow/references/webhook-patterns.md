# Webhook Patterns

Patterns for handling webhook callbacks from the HITL Platform.

## Overview

When an approver responds to an approval request, the HITL Platform sends a webhook callback to the configured URL. This document covers patterns for handling these callbacks in UiPath.

## Orchestrator Webhook Setup

### 1. Create Webhook in Orchestrator

1. Navigate to **Orchestrator > Automations > Triggers**
2. Click **Add Trigger > Webhook**
3. Configure:
   - **Name**: `HITL_Approval_Callback`
   - **Process**: Select the callback handler process
   - **Enabled**: Yes
4. Copy the generated webhook URL

### 2. Webhook URL Format

```
https://cloud.uipath.com/{accountName}/{tenantId}/{folderId}/elements_/v1/webhooks/events/{webhookId}
```

Example:
```
https://cloud.uipath.com/catonetworks/20bf226d-f42c-4668-9cad-1c86ed398cf0/elements_/v1/webhooks/events/MUkoaX93iHW4Y7PTbNuAUORocC0ee0nCzHl_pm0auXzHcBVXXAJh-4QlvisqnMLUslQjKap1g11eSf-KYtCGsA
```

---

## Callback Handler Workflow

### ProcessApprovalCallback.xaml

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="ProcessApprovalCallback" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:njl="clr-namespace:Newtonsoft.Json.Linq;assembly=Newtonsoft.Json" xmlns:s="clr-namespace:System;assembly=System.Private.CoreLib" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <x:Members>
    <x:Property Name="in_WebhookPayload" Type="InArgument(x:String)" />
  </x:Members>
  <Sequence DisplayName="Process Approval Callback">
    <Sequence.Variables>
      <Variable x:TypeArguments="njl:JObject" Name="callbackData" />
      <Variable x:TypeArguments="x:String" Name="approvalId" />
      <Variable x:TypeArguments="x:String" Name="processId" />
      <Variable x:TypeArguments="x:String" Name="status" />
      <Variable x:TypeArguments="x:String" Name="action" />
      <Variable x:TypeArguments="x:String" Name="respondedBy" />
      <Variable x:TypeArguments="x:String" Name="comments" />
    </Sequence.Variables>
    
    <!-- Log Start -->
    <ui:LogMessage DisplayName="Log: Callback Received" Level="Info" Message="[&quot;Approval callback received&quot;]" />
    
    <!-- Parse Webhook Payload -->
    <ui:DeserializeJson x:TypeArguments="njl:JObject" DisplayName="Parse Callback Payload" JsonObject="[callbackData]" JsonString="[in_WebhookPayload]" />
    
    <!-- Extract Fields -->
    <Assign DisplayName="Get Approval ID">
      <Assign.To><OutArgument x:TypeArguments="x:String">[approvalId]</OutArgument></Assign.To>
      <Assign.Value><InArgument x:TypeArguments="x:String">[callbackData("approvalId").ToString]</InArgument></Assign.Value>
    </Assign>
    
    <Assign DisplayName="Get Process ID">
      <Assign.To><OutArgument x:TypeArguments="x:String">[processId]</OutArgument></Assign.To>
      <Assign.Value><InArgument x:TypeArguments="x:String">[callbackData("processId").ToString]</InArgument></Assign.Value>
    </Assign>
    
    <Assign DisplayName="Get Status">
      <Assign.To><OutArgument x:TypeArguments="x:String">[status]</OutArgument></Assign.To>
      <Assign.Value><InArgument x:TypeArguments="x:String">[callbackData("status").ToString]</InArgument></Assign.Value>
    </Assign>
    
    <Assign DisplayName="Get Action">
      <Assign.To><OutArgument x:TypeArguments="x:String">[action]</OutArgument></Assign.To>
      <Assign.Value><InArgument x:TypeArguments="x:String">[callbackData("action").ToString]</InArgument></Assign.Value>
    </Assign>
    
    <Assign DisplayName="Get Responded By">
      <Assign.To><OutArgument x:TypeArguments="x:String">[respondedBy]</OutArgument></Assign.To>
      <Assign.Value><InArgument x:TypeArguments="x:String">[callbackData("respondedBy").ToString]</InArgument></Assign.Value>
    </Assign>
    
    <Assign DisplayName="Get Comments">
      <Assign.To><OutArgument x:TypeArguments="x:String">[comments]</OutArgument></Assign.To>
      <Assign.Value><InArgument x:TypeArguments="x:String">[If(callbackData("comments") IsNot Nothing, callbackData("comments").ToString, "")]</InArgument></Assign.Value>
    </Assign>
    
    <!-- Log Extracted Data -->
    <ui:LogMessage DisplayName="Log: Callback Data" Level="Info" Message="[String.Format(&quot;Approval {0} - Status: {1}, Action: {2}, By: {3}&quot;, approvalId, status, action, respondedBy)]" />
    
    <!-- Process Based on Status -->
    <Switch x:TypeArguments="x:String" DisplayName="Switch on Action" Expression="[action.ToLower]">
      <Case Value="approved">
        <Sequence DisplayName="Handle Approved">
          <ui:LogMessage Level="Info" Message="[&quot;Processing APPROVED action for &quot; + processId]" />
          <!-- Add your approved logic here -->
          <!-- Example: Update CRM, send confirmation, trigger next process -->
        </Sequence>
      </Case>
      <Case Value="rejected">
        <Sequence DisplayName="Handle Rejected">
          <ui:LogMessage Level="Info" Message="[&quot;Processing REJECTED action for &quot; + processId]" />
          <!-- Add your rejected logic here -->
          <!-- Example: Send rejection notification, update status -->
        </Sequence>
      </Case>
      <Default>
        <Sequence DisplayName="Handle Unknown">
          <ui:LogMessage Level="Warning" Message="[&quot;Unknown action received: &quot; + action]" />
        </Sequence>
      </Default>
    </Switch>
    
    <!-- Log Complete -->
    <ui:LogMessage DisplayName="Log: Callback Processed" Level="Info" Message="[&quot;Approval callback processed successfully&quot;]" />
    
  </Sequence>
</Activity>
```

---

## Callback Payload Structure

```json
{
  "event": "approval.completed",
  "approvalId": "550e8400-e29b-41d4-a716-446655440000",
  "processId": "QUOTE-2026-001",
  "title": "Sales Deal Approval - Acme Corp",
  "status": "approved",
  "action": "approved",
  "respondedBy": "manager@company.com",
  "respondedAt": "2026-02-22T10:30:00.000Z",
  "comments": "Approved - strategic customer",
  "formData": {
    "customerName": "Acme Corp",
    "deal_amount": 75000,
    "agreement_months": "24"
  },
  "responseData": {
    "cpiPercentage": "5",
    "agreementMonths": "36"
  },
  "metadata": {
    "source_id": "CRM-447",
    "cost_center": "SALES"
  }
}
```

---

## Extracting Nested Data

### Get Form Data Fields

```vb
' Get nested form data
customerName = callbackData("formData")("customerName").ToString
dealAmount = CDec(callbackData("formData")("deal_amount"))

' Get response data (from Adaptive Card inputs)
If callbackData("responseData") IsNot Nothing Then
    cpiPercentage = callbackData("responseData")("cpiPercentage").ToString
    agreementMonths = callbackData("responseData")("agreementMonths").ToString
End If
```

### Handle Optional Fields

```vb
' Safe extraction with null check
comments = If(callbackData("comments") IsNot Nothing, callbackData("comments").ToString, "")

' Check if field exists
If callbackData("responseData") IsNot Nothing AndAlso callbackData("responseData")("cpiPercentage") IsNot Nothing Then
    cpiPercentage = callbackData("responseData")("cpiPercentage").ToString
Else
    cpiPercentage = "0"
End If
```

---

## Idempotency

Webhooks may be delivered multiple times. Implement idempotency:

```xml
<!-- Check if already processed -->
<ui:InvokeWorkflowFile DisplayName="Check If Processed" WorkflowFileName="Workflows\CheckApprovalProcessed.xaml">
  <ui:InvokeWorkflowFile.Arguments>
    <InArgument x:TypeArguments="x:String" x:Key="in_ApprovalId">[approvalId]</InArgument>
    <OutArgument x:TypeArguments="x:Boolean" x:Key="out_AlreadyProcessed">[alreadyProcessed]</OutArgument>
  </ui:InvokeWorkflowFile.Arguments>
</ui:InvokeWorkflowFile>

<If Condition="[alreadyProcessed]">
  <If.Then>
    <Sequence DisplayName="Skip - Already Processed">
      <ui:LogMessage Level="Info" Message="[&quot;Approval &quot; + approvalId + &quot; already processed, skipping&quot;]" />
    </Sequence>
  </If.Then>
  <If.Else>
    <Sequence DisplayName="Process Callback">
      <!-- Process the callback -->
      <!-- Mark as processed -->
    </Sequence>
  </If.Else>
</If>
```

---

## Error Handling

```xml
<TryCatch DisplayName="Handle Callback with Error Handling">
  <TryCatch.Try>
    <Sequence DisplayName="Process Callback">
      <!-- Main processing logic -->
    </Sequence>
  </TryCatch.Try>
  <TryCatch.Catches>
    <Catch x:TypeArguments="s:Exception">
      <ActivityAction x:TypeArguments="s:Exception">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="s:Exception" Name="exception" />
        </ActivityAction.Argument>
        <Sequence DisplayName="Handle Exception">
          <ui:LogMessage Level="Error" Message="[&quot;Error processing callback: &quot; + exception.Message]" />
          <!-- Optionally: Add to retry queue, send alert -->
        </Sequence>
      </ActivityAction>
    </Catch>
  </TryCatch.Catches>
</TryCatch>
```

---

## Correlation with Original Request

To correlate callbacks with original requests, use the `processId`:

```xml
<!-- Look up original request data using processId -->
<ui:InvokeWorkflowFile DisplayName="Get Original Request" WorkflowFileName="Workflows\GetRequestByProcessId.xaml">
  <ui:InvokeWorkflowFile.Arguments>
    <InArgument x:TypeArguments="x:String" x:Key="in_ProcessId">[processId]</InArgument>
    <OutArgument x:TypeArguments="njl:JObject" x:Key="out_OriginalRequest">[originalRequest]</OutArgument>
  </ui:InvokeWorkflowFile.Arguments>
</ui:InvokeWorkflowFile>
```

---

## Best Practices

1. **Log everything** - Webhook payloads, processing steps, outcomes
2. **Implement idempotency** - Handle duplicate deliveries gracefully
3. **Use Try-Catch** - Webhook handlers should not fail silently
4. **Validate payload** - Check required fields before processing
5. **Store callback data** - Keep audit trail of all callbacks
6. **Handle timeouts** - Webhook handlers should complete quickly
