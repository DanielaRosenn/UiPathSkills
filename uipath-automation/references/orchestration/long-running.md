# Module: Long Running Workflows (LRW)
**Source:** UiPath Studio 2025.10 | **Package:** UiPath.Persistence.Activities
**Covers:** BPMN canvas, Suspend/Resume, serialization rules, Action Center integration, approval flows

## Canonical skill (Studio desktop — use this for `ProcessDiagram`)

This file holds **concepts and activity snippets** inside **`uipath-automation`**. For **production LRW codegen** (main entry XAML, node graph rules, dependency contract, HITL platform wiring), load the sibling skill:

| Resource | Path (same `.claude/skills` tree — works with Cursor junction) |
|----------|------------------------------------------------------------------|
| **Skill entry** | [uipath-longrunning-workflow/SKILL.md](../../../uipath-longrunning-workflow/SKILL.md) |
| **HITL API detail** | [hitl-platform-api.md](../../../uipath-longrunning-workflow/references/hitl-platform-api.md) |
| **Shared HITL / integration** | [hitl-platform.md](../../../_shared/hitl-platform.md) |

**Rule:** Root LRW file **`x:Class="LongRunningWorkflow"`**, FlowchartBuilder + Persistence + Form libraries in `project.json` — all spelled out in the linked skill.

---

## When to Use LRW

- Process involves **human approval steps** (manager sign-off, exception review)
- Process may be **suspended for hours/days** (waiting for email reply, external event)
- Spans **multiple robot sessions** (suspend → different robot resumes)
- Integrates with **Action Center** for HITL (Human-In-The-Loop)

Do NOT use LRW for:
- Short processes completing in < 5 minutes (use standard Sequence)
- Pure unattended batch processing (use REFramework)

---

## CRITICAL: Serialization Rules

When a workflow suspends, ALL variables in scope are serialized to Orchestrator storage. This is the #1 source of LRW bugs.

### ✅ Serializable types (safe across suspend points)
```
String, Int32, Int64, Double, Boolean, DateTime, Decimal
Guid, Enum, Char
Array(Of T) where T is serializable
List(Of T) where T is serializable
Dictionary(Of String, T) where T is serializable
Custom classes marked [Serializable]
```

### ❌ Non-serializable types (NEVER use across a Suspend activity)
```
DataTable               → convert to JSON string, re-hydrate after resume
DataRow                 → extract needed values to String/Dictionary before suspend
IEnumerable (open)     → convert to List(Of T) first
MailMessage            → extract Subject, Body, etc. to String variables
UiElement / IWebElement → always re-find after resume
DatabaseConnection     → close before suspend, re-open after
Any COM object         → never cross suspend points
```

### The golden rule
**Extract all data you need into serializable types BEFORE the Suspend activity.**

```xml
<Sequence DisplayName="Before Suspend: Serialize DataTable">
  <!-- Convert DataTable to JSON string -->
  <ui:InvokeCode DisplayName="Serialize DT to JSON">
    <ui:InvokeCode.Parameters>
      <ui:InvokeCodeParameter Direction="In"  Name="dt"   Type="[GetType(DataTable)]" Value="[dtOrders]"/>
      <ui:InvokeCodeParameter Direction="Out" Name="json" Type="[GetType(String)]"    Value="[serializedOrders]"/>
    </ui:InvokeCode.Parameters>
    <ui:InvokeCode.Code><![CDATA[
json = Newtonsoft.Json.JsonConvert.SerializeObject(dt)
    ]]></ui:InvokeCode.Code>
  </ui:InvokeCode>

  <!-- Now suspend — only String variables in scope -->
  <ui:SuspendWorkflow DisplayName="Wait for Approval"
    BookmarkName="[&quot;ApprovalBookmark_&quot; + workflowId]"/>

  <!-- After resume: deserialize back -->
  <ui:InvokeCode DisplayName="Deserialize JSON to DT">
    <ui:InvokeCode.Parameters>
      <ui:InvokeCodeParameter Direction="In"  Name="json" Type="[GetType(String)]"    Value="[serializedOrders]"/>
      <ui:InvokeCodeParameter Direction="Out" Name="dt"   Type="[GetType(DataTable)]" Value="[dtOrders]"/>
    </ui:InvokeCode.Parameters>
    <ui:InvokeCode.Code><![CDATA[
dt = Newtonsoft.Json.JsonConvert.DeserializeObject(Of DataTable)(json)
    ]]></ui:InvokeCode.Code>
  </ui:InvokeCode>
</Sequence>
```

---

## Core LRW Activities

### Suspend Workflow (pause until external signal)
```xml
<ui:SuspendWorkflow sap2010:WorkflowViewState.IdRef="Suspend_1"
    DisplayName="Wait for Manager Approval"
    BookmarkName="[&quot;Approval_&quot; + requestId]"
    WakeUpEvent="[wakeupPayload]"
    RemainingTime="[TimeSpan.FromDays(3)]"/>
<!-- wakeupPayload contains data sent by the resuming party -->
<!-- RemainingTime = how long to wait before auto-timeout -->
```

### Wait for Queue Item and Resume
```xml
<ui:AddQueueItemAndGetReference sap2010:WorkflowViewState.IdRef="AddAndWait_1"
    DisplayName="Submit to Approval Queue and Wait"
    QueueName="&quot;ApprovalQueue&quot;"
    Reference="[requestId]"
    ItemInformation="[approvalData]"
    QueueItem="[queueItemRef]"/>

<ui:WaitForQueueItemAndResume sap2010:WorkflowViewState.IdRef="WaitQueue_1"
    DisplayName="Wait for Approval Decision"
    QueueItem="[queueItemRef]"
    Timeout="[TimeSpan.FromDays(5)]"
    Result="[approvalResult]"/>
<!-- approvalResult.SpecificContent("Decision") = "Approved" or "Rejected" -->
```

### Create External Task (Action Center)
```xml
<ui:CreateExternalTask sap2010:WorkflowViewState.IdRef="CreateTask_1"
    DisplayName="Create Approval Task in Action Center"
    TaskTitle="[&quot;Invoice Approval: &quot; + invoiceNumber]"
    TaskPriority="Medium"
    TaskCatalogName="&quot;Invoice Approvals&quot;"
    TaskObject="[taskRef]">
  <ui:CreateExternalTask.TaskDataCollection>
    <InArgument x:TypeArguments="scg:Dictionary(x:String,x:Object)">
      [New Dictionary(Of String, Object) From {
        {"InvoiceID",  invoiceId},
        {"Amount",     amount.ToString()},
        {"Vendor",     vendorName},
        {"DueDate",    dueDate.ToString("yyyy-MM-dd")},
        {"SubmittedBy", robotName}
      }]
    </InArgument>
  </ui:CreateExternalTask.TaskDataCollection>
</ui:CreateExternalTask>
```

### Wait for External Task and Resume
```xml
<ui:WaitForExternalTaskAndResume sap2010:WorkflowViewState.IdRef="WaitTask_1"
    DisplayName="Wait for Human Decision"
    TaskObject="[taskRef]"
    TaskAction="[actionTaken]"
    TaskObject_Output="[completedTask]"
    Timeout="[TimeSpan.FromDays(7)]"/>
<!-- actionTaken: "Approved", "Rejected", "Escalated" — set by human in Action Center -->
<!-- completedTask.SpecificContent: read any data the human filled in -->
```

---

## Standard Approval Flow Pattern

```xml
<Sequence DisplayName="Invoice Approval Workflow">

  <!-- 1. Prepare — serialize all data -->
  <Assign DisplayName="Serialize Invoice Data">
    <Assign.To>[serializedInvoice]</Assign.To>
    <Assign.Value>[Newtonsoft.Json.JsonConvert.SerializeObject(invoiceData)]</Assign.Value>
  </Assign>

  <!-- 2. Create Action Center task -->
  <ui:CreateExternalTask DisplayName="Request Manager Approval"
      TaskTitle="[&quot;Approve Invoice &quot; + invoiceNumber]"
      TaskCatalogName="&quot;InvoiceApprovals&quot;"
      TaskObject="[taskRef]">
    <ui:CreateExternalTask.TaskDataCollection>
      <InArgument x:TypeArguments="scg:Dictionary(x:String,x:Object)">
        [New Dictionary(Of String, Object) From {
          {"InvoiceNumber", invoiceNumber},
          {"Amount", invoiceAmount.ToString("C")},
          {"Vendor", vendorName}
        }]
      </InArgument>
    </ui:CreateExternalTask.TaskDataCollection>
  </ui:CreateExternalTask>

  <!-- 3. Suspend — wait for human -->
  <ui:WaitForExternalTaskAndResume DisplayName="Wait for Decision"
      TaskObject="[taskRef]"
      TaskAction="[humanDecision]"
      TaskObject_Output="[completedTask]"
      Timeout="[TimeSpan.FromDays(3)]"/>

  <!-- 4. Resume — process decision -->
  <If DisplayName="Route on Decision" Condition="[humanDecision = &quot;Approved&quot;]">
    <If.Then>
      <ui:InvokeWorkflowFile DisplayName="Process Approved Invoice"
          WorkflowFileName="&quot;ProcessApprovedInvoice.xaml&quot;"/>
    </If.Then>
    <If.Else>
      <ui:InvokeWorkflowFile DisplayName="Handle Rejection"
          WorkflowFileName="&quot;NotifyRejection.xaml&quot;"/>
    </If.Else>
  </If>

</Sequence>
```

---

## No Persist Scope

Wrap activities that should NOT be persisted (e.g., during a UI session, holding a lock):

```xml
<ui:NoPersistScope sap2010:WorkflowViewState.IdRef="NoPersist_1"
    DisplayName="No Persist: UI Interaction">
  <!-- Orchestrator will NOT suspend during this block -->
  <!-- Use for: UI sessions, DB transactions, file locks -->
  <Sequence DisplayName="Critical Section">
    <ui:TypeInto DisplayName="Enter Sensitive Data" Private="True"
      Target.Selector="[pwdSelector]" Text="[password]"/>
    <ui:Click DisplayName="Submit"/>
  </Sequence>
</ui:NoPersistScope>
```

---

## LRW Design Checklist

- [ ] No DataTable, DataRow, MailMessage, or COM objects cross a Suspend point
- [ ] All DataTables serialized to JSON before suspend, deserialized after
- [ ] `NoPersistScope` wraps any UI sessions that span a potential suspend
- [ ] Timeout set on every `WaitFor*` activity
- [ ] Timeout handling: what happens if human never acts?
- [ ] Bookmark names are unique per workflow instance (include a GUID or ID)
- [ ] Action Center task catalog exists in Orchestrator before deployment
- [ ] Test serialization round-trip: serialize → persist → deserialize → validate
