---
name: uipath-automation
tags: [uipath-dev, xaml, rpa, workflow, studio, master]
description: >
  Master UiPath Studio orchestration skill: classify the task, load progressive modules under references/orchestration/, then generate XAML, project.json, expressions, and dependencies. Covers all major activity areas (UI automation, Excel/Office, Web API, database, system files, Orchestrator, Invoke Code, REFramework notes, long-running, coded workflows, error handling, deploy/publish, debugging, expressions, best practices, code review). Triggers on UiPath Studio, workflow, XAML, sequence, flowchart, state machine, REFramework, LRW, selector, queue, asset, invoke code, REST, RPA, publish, robot, or any Studio developer task. Delegates verticals (Integration Service setup, DU training, deep persistence) to sibling skills when appropriate.
---

# UiPath Automation Development (Studio) — Master orchestration

Generate production-ready **UiPath Studio** artifacts: XAML workflows, `project.json`, package dependencies, Invoke Code snippets, and—where appropriate—coded workflow entry points. Align generated projects with [official UiPath documentation](references/official-documentation-index.md) for the Studio version and activity packs in use.

**Combined body of knowledge:** The **master pack** lives under `references/orchestration/` (not a separate Cursor skill). The **single table of contents** is **[references/UNIFIED-INDEX.md](references/UNIFIED-INDEX.md)**.

### Canonical content (read each topic once)

| Topic | Primary reference | Notes |
|-------|-------------------|--------|
| Control-flow and activity **XAML catalog** (Assign, If, For Each, Try-Catch, …) | [references/activities.md](references/activities.md) | Do not duplicate long templates in chat; open this file. |
| Try-Catch, retries, GEH, exception strategy | [references/orchestration/error-handling.md](references/orchestration/error-handling.md) | Deep patterns here, not repeated in SKILL.md. |
| UI automation (modern scope, waits, selectors practice) | [references/orchestration/activities/ui-automation.md](references/orchestration/activities/ui-automation.md) | Narrative + Studio-oriented rules. |
| Excel / Office / DataTable | [references/orchestration/activities/excel-office.md](references/orchestration/activities/excel-office.md) | Modern `Use Excel File`, ranges, loops. |
| HTTP / REST / JSON | [references/orchestration/activities/web-api.md](references/orchestration/activities/web-api.md) | Aligns with WebAPI activities. |
| Invoke Code (snippet depth) | [references/orchestration/invoke-code.md](references/orchestration/invoke-code.md) + [references/invoke-code-and-expressions.md](references/invoke-code-and-expressions.md) | SKILL.md keeps only a short summary. |
| Coded workflows (C#) | [references/orchestration/coded-automations.md](references/orchestration/coded-automations.md); long guide: [references/coded-automations.md](references/coded-automations.md) | Orchestration module first for patterns. |
| **This SKILL.md** | — | Canvas/planning rules, **single-line** `<Activity>` root, escaping, variables/arguments, **minimal** modern Excel/browser examples for quick alignment. |
| **Org overlay (SALES02 / HITL)** | [references/org-overlays/sales02-hitl-patterns.md](references/org-overlays/sales02-hitl-patterns.md) | Optional NetHttpRequest, Graph mail, Adaptive Cards, LRW `ProcessDiagram`; use when the user needs that stack. |

**Orchestration README** (topic → file): [references/orchestration/README.md](references/orchestration/README.md).

## Master orchestration (read modules before heavy codegen)

Default entry for Studio work. Beyond a one-line answer: **classify** → **read** the matching module(s) from the table above → **implement**.

1. **Route** with [UNIFIED-INDEX.md](references/UNIFIED-INDEX.md) or [orchestration/README.md](references/orchestration/README.md).
2. **Load** `references/orchestration/**/*.md` as needed (often **multiple** files, e.g. Excel + web-api + `xaml-generator.md`).
3. **Standards:** Valid XAML and namespaces; expression language matches the project; naming and error handling by default; state **Windows** vs **Windows - Legacy** vs cross-platform when it matters.
4. **Out of scope:** Platform admin, Mining hubs, Apps, Studio Web, Test Manager depth, DU training — docs or vertical skills (`uipath-document-understanding`, etc.).
5. **Canvas vs ad hoc:** If a canvas/SDD exists, follow **CRITICAL: Canvas** first; otherwise use modules for **how** to build snippets or small projects.

---

## CRITICAL: Canvas Context Required

**BEFORE generating ANY UiPath code, you MUST:**

0. **Do not generate any XAML or project files until planning is complete.** If the user has or references a canvas/SDD/flow, planning comes first.
1. **Locate the canvas/flow file** - Search for `solution-flow.json`, `*canvas*.json`, or `*flow*.json`
2. **Run the uipath-workflow-planning skill** - Read and follow it to extract all arguments, data flows, and visibility rules; produce workflow-plan.md or equivalent.
3. **Validate the execution plan** - Ensure all fields from canvas are mapped to workflow arguments. Only then proceed to generate code.

```
┌─────────────────────────────────────────────────────────────────┐
│  MANDATORY WORKFLOW                                              │
│                                                                  │
│  1. FIND CANVAS ──> 2. RUN PLANNING ──> 3. GENERATE CODE        │
│     solution-flow.json    uipath-workflow-planning     uipath-automation│
│                           skill                 skill            │
└─────────────────────────────────────────────────────────────────┘
```

### Canvas Location Search

```bash
# Search patterns (in order of priority)
**/solution-flow*.json
**/*canvas*.json
**/*flow*.json
**/docs/*.json
```

### Required Canvas Data Extraction

For EACH workflow to generate, extract from canvas:
- **drillDown.inputArguments** - All input parameters with types and sources
- **drillDown.outputArguments** - All output parameters with types
- **drillDown.fields** - Form fields with visibility rules
- **drillDown.visibilityLevel** - LIMITED, FULL, FULL + CHAIN EDITOR, etc.
- **drillDown.hiddenFields** - Fields NOT to include in this workflow

### Pre-Generation Checklist

- [ ] Canvas file located and read
- [ ] All node drillDown data extracted
- [ ] Data flow matrix created (source → target)
- [ ] Visibility rules identified per workflow
- [ ] Argument types validated against canvas
- [ ] workflow-plan.md (or equivalent plan) generated and in context

**DO NOT PROCEED if canvas data is missing or incomplete. Do not generate XAML until the plan exists.**

### When there is no canvas / SDD

If the user explicitly asks for a **standalone snippet** (e.g. one Sequence, one Invoke Code example, or a small `project.json` fragment) and **no** solution-flow or spec exists, you may generate **only** what they asked for and state any **assumptions** (arguments, file paths, package versions). Do not block on a fictional canvas.

---

## Official documentation (verify in product)

- **Index of Studio & activities links:** [references/official-documentation-index.md](references/official-documentation-index.md) (project types, workflow types, Invoke Code, Integration Service, analyzer rules).
- **Invoke Code, expressions, imports:** [references/invoke-code-and-expressions.md](references/invoke-code-and-expressions.md)
- **Connectors & Integration Service orientation:** [references/connectors-and-integration.md](references/connectors-and-integration.md)

Prefer **documented** activity names and compatibility (**Windows** vs **Windows - Legacy** vs cross-platform) over guesswork.

---

## Quick Reference

### Project Types (Studio)
- **Process / Background process**: Standard server or unattended automation (see Studio *About automation projects*).
- **Library**: Reusable components published for other projects.
- **Test automation / coded test**: Testing project types with Testing activities.
- **Orchestration process**, **REFramework**, **Attended automation** templates: use when the user asks for those patterns (see [Project templates](https://docs.uipath.com/studio/standalone/2024.10/user-guide/project-templates)).
- **Coded automation**: C#-first workflows (`CodedWorkflow`); pair with [references/coded-automations.md](references/coded-automations.md).

### Workflow Types (file kinds in Studio)
| Type | Use Case | Container / notes |
|------|----------|-------------------|
| Sequence | Linear steps | `<Sequence>` |
| Flowchart | Branching, connectors | `<Flowchart>` |
| State machine | States and transitions | `<StateMachine>` |
| Forms | Form designer workflows | See Studio *Forms* workflow type |
| Global Exception Handler | Top-level error handler | Dedicated workflow file |
| Coded workflow | C# body | `.cs` + `project.json` `expressionLanguage` / entry points |
| Long-running / BPMN | Interrupting events, persistence | Often `ProcessDiagram` + dedicated packages (see long-running skill) |

### From request to artifacts (single path)

- **With canvas / SDD:** Complete **CRITICAL: Canvas** (find flow JSON → `uipath-workflow-planning` → validated plan). Then: project type → workflow type → `project.json` → Main + modules → error handling → validate against canvas.
- **Without canvas:** User-specified snippet or small project; load orchestration modules for patterns; no fictional canvas.
- **Always:** Pull detailed activity XML from [activities.md](references/activities.md) or `references/orchestration/` instead of inventing duplicate templates in isolation.

### Regression smoke expectations (`tests.yaml`)

Automated skill tests ([tests.yaml](tests.yaml)) look for **substring** matches in model output. When generating answers that mirror those prompts, prefer including these **type / token** hints so smoke checks stay green (see test `id`):

- **basic-sequence:** `UseExcelFile`, `ReadRange`, `ForEach`, `LogMessage`, `<Sequence`, `DataTable` (aliases allowed via `expected_element_groups` in YAML).
- **browser-automation-modern:** `UseApplicationBrowser`, `TypeInto`, `Click`, `BrowserType=`, `Url=`, `Selector=`.
- **error-handling-trycatch:** `TryCatch`, `Catch`, `BusinessRuleException`, `Exception`, `LogMessage`, `Level="Error"`.
- **project-json-generation:** `projectVersion` or `schemaVersion`, `dependencies`, project name string, `UiPath`, `"targetFramework"`, `"Windows"`.
- **queue-operations:** `GetQueueItem`, `SetTransactionStatus`, `QueueItem`, `QueueName=`, `Status=`, `TransactionItem=`.
- **datatable-operations:** `BuildDataTable`, `WriteRange`, `DataTable`, plus filtering (e.g. `FilterDataTable` or equivalent).

For **release gates**, run the skill tester with a **capable model** (e.g. Claude Sonnet on Bedrock) and optionally `--temperature 0`; see [skill-tester/SKILL.md](../skill-tester/SKILL.md).

## Project Structure

```
ProjectName/
├── project.json              # Project metadata (required)
├── Main.xaml                 # Entry point (required)
├── Workflows/                # Additional workflows
│   ├── ProcessTransaction.xaml
│   └── InitAllSettings.xaml
├── Data/                     # Config and data files
│   └── Config.xlsx
└── .screenshots/             # UI element screenshots
```

## XAML Generation Essentials

### Namespace Declarations
Every XAML file requires these namespaces at the root.

**CRITICAL**: The entire opening `<Activity>` tag with all namespaces MUST be on a SINGLE LINE. UiPath Studio's XAML parser requires this format. Multi-line formatting will cause parsing errors.

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="Main" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:s="clr-namespace:System;assembly=System.Private.CoreLib" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
```

**Key Requirements:**
1. All namespaces on ONE LINE (no line breaks within the Activity tag)
2. `xmlns:mc` must be declared before or on same line as `mc:Ignorable`
3. Include `xmlns:this="clr-namespace:"` for argument default values
4. Include `VisualBasic.Settings` and `TextExpression.NamespacesForImplementation` after `x:Members`

### String Escaping in XAML

**CRITICAL**: When embedding HTML or JSON in XAML string values:
- **NEVER use multiline strings** with `xml:space="preserve"` for HTML/JSON content
- **Use XML entities** for HTML tags: `&lt;` for `<`, `&gt;` for `>`
- **Keep JSON on single lines** - No line breaks in JSON content
- For nested JSON in Action.Http bodies, follow **XAML String Escaping Rules** below (`\""` pattern, not doubled backslashes)

See also [references/xaml-syntax.md](references/xaml-syntax.md) for structure; escaping for HTML/JSON in attributes is summarized in this SKILL under **XAML String Escaping Rules**.

**Common Error:** `'Content-Type' is an unexpected token` - This occurs when HTML attributes are embedded in multiline XAML strings without proper escaping.

### Variable Declaration Pattern
Variables are declared within the workflow container:

```xml
<Sequence.Variables>
  <Variable x:TypeArguments="x:String" Name="strInput" Default="" />
  <Variable x:TypeArguments="x:Int32" Name="intCounter" Default="0" />
  <Variable x:TypeArguments="sd:DataTable" Name="dtResults" />
  <Variable x:TypeArguments="x:Boolean" Name="boolSuccess" Default="False" />
</Sequence.Variables>
```

### Argument Declaration Pattern
Arguments enable workflow input/output:

```xml
<x:Members>
  <x:Property Name="in_ConfigPath" Type="InArgument(x:String)" />
  <x:Property Name="in_TransactionItem" Type="InArgument(ui:QueueItem)" />
  <x:Property Name="out_Result" Type="OutArgument(x:String)" />
  <x:Property Name="io_DataTable" Type="InOutArgument(sd:DataTable)" />
</x:Members>
```

## Control flow and common activities

**Do not duplicate** full templates here. Use **[references/activities.md](references/activities.md)** for Assign, If/Else, Switch, For Each, **Try-Catch**, **Invoke Workflow**, and related XAML. Use **[references/orchestration/error-handling.md](references/orchestration/error-handling.md)** for multi-catch patterns, retries, and GEH.

**Try-Catch:** When the user asks for *business* vs *system* handling, include a catch for **`BusinessRuleException`** (business rules) and a catch for **`Exception`** or narrower system types—see activities catalog and error-handling module.

**DataTable:** Building and filtering often uses **`BuildDataTable`**, **`AddDataRow`** (or row construction), and **`FilterDataTable`**—see [activities.md](references/activities.md) Data Manipulation / Excel sections.

## Invoke Code (summary)

Use **`Invoke Code`** from **UiPath.System.Activities** for inline **VB.NET** or **C#** ([Invoke Code](https://docs.uipath.com/activities/other/latest/workflow/invoke-code)). Set **Language** to **VBNet** or **CSharp**; bind **Edit Arguments**; add **Imports** as needed. Prefer **Invoke Workflow**, **libraries**, or **coded workflows** for larger logic.

Details and examples: **[references/orchestration/invoke-code.md](references/orchestration/invoke-code.md)** and **[references/invoke-code-and-expressions.md](references/invoke-code-and-expressions.md)**. Author in Studio (drag activity → Edit Code) rather than hand-serializing XAML unless matching an export from the same Studio version.

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

## Modern UI Automation (Use Application/Browser)

### Use Application/Browser Pattern (Recommended)

The Modern Design Experience uses `UseApplicationBrowser` as the container for all UI activities:

```xml
<ui:UseApplicationBrowser 
  DisplayName="Use Browser"
  ApplicationName="MyWebApp"
  BrowserType="Chrome"
  Url="https://example.com"
  Open="IfNotOpen"
  Close="Never">
  
  <!-- All UI activities go inside -->
  <ui:TypeInto 
    DisplayName="Enter Username"
    Text="[username]">
    <ui:TypeInto.Target>
      <ui:Target Selector="&lt;webctrl tag='INPUT' id='username' /&gt;" />
    </ui:TypeInto.Target>
  </ui:TypeInto>
  
  <ui:TypeInto 
    DisplayName="Enter Password"
    Text="[password]"
    IsPassword="True">
    <ui:TypeInto.Target>
      <ui:Target Selector="&lt;webctrl tag='INPUT' id='password' type='password' /&gt;" />
    </ui:TypeInto.Target>
  </ui:TypeInto>
  
  <ui:Click DisplayName="Click Login">
    <ui:Click.Target>
      <ui:Target Selector="&lt;webctrl tag='BUTTON' innertext='Login' /&gt;" />
    </ui:Click.Target>
  </ui:Click>
</ui:UseApplicationBrowser>
```

### Modern vs Classic Activities

| Modern Activity | Classic Equivalent | Key Difference |
|-----------------|-------------------|----------------|
| `UseApplicationBrowser` | `Open Browser` + `Attach Browser` | Unified container |
| `ui:TypeInto` (with Target) | `ui:TypeInto` (with Selector) | Target object model |
| `ui:Click` (with Target) | `ui:Click` (with Selector) | Unified targeting |
| `ui:GetText` | `ui:GetText` | Same functionality |
| `ui:GetAttribute` | `ui:GetAttribute` | Same functionality |

### Target Property Pattern

Modern activities use the `Target` property instead of inline selectors:

```xml
<ui:Click DisplayName="Click Submit">
  <ui:Click.Target>
    <ui:Target 
      Selector="&lt;webctrl tag='BUTTON' class='submit-btn' /&gt;"
      TimeoutMS="30000"
      WaitForReady="Interactive" />
  </ui:Click.Target>
</ui:Click>
```

### Object Repository Integration

```xml
<ui:Click 
  DisplayName="Click Login Button"
  ObjectRepository.TargetElement="{ui:ObjectRepositoryTarget 
    ApplicationName='MyWebApp' 
    ScreenName='LoginPage' 
    ElementName='LoginButton'}" />
```

### Computer Vision Activities

For applications where selectors don't work (Citrix, images, etc.):

```xml
<cv:CVScope DisplayName="Computer Vision Scope">
  <cv:CVScope.CVScreenScope>
    <cv:CVScreenScope />
  </cv:CVScope.CVScreenScope>
  
  <cv:CVClick 
    DisplayName="CV Click Login"
    Text="Login"
    Accuracy="0.8" />
  
  <cv:CVTypeInto 
    DisplayName="CV Type Username"
    Text="[username]"
    AnchorText="Username:" />
  
  <cv:CVGetText 
    DisplayName="CV Get Status"
    AnchorText="Status:">
    <cv:CVGetText.Result>
      <OutArgument x:TypeArguments="x:String">[statusText]</OutArgument>
    </cv:CVGetText.Result>
  </cv:CVGetText>
</cv:CVScope>
```

Required package: `UiPath.CV.Activities`

### Native Citrix Automation

```xml
<citrix:CitrixScope 
  DisplayName="Citrix Session"
  ServerName="citrix.company.com"
  ApplicationName="SAP">
  
  <ui:TypeInto 
    DisplayName="Enter Transaction"
    Text="VA01">
    <ui:TypeInto.Target>
      <ui:Target Selector="&lt;citrix ... /&gt;" />
    </ui:TypeInto.Target>
  </ui:TypeInto>
</citrix:CitrixScope>
```

### SAP GUI Automation

```xml
<sap:SAPScope 
  DisplayName="SAP Connection"
  ConnectionString="[sapConnection]">
  
  <sap:SAPLogin 
    DisplayName="SAP Login"
    Client="100"
    User="[sapUser]"
    Password="[sapPassword]" />
  
  <sap:SAPStartTransaction 
    DisplayName="Start VA01"
    TransactionCode="VA01" />
  
  <sap:SAPSetText 
    DisplayName="Enter Order Type"
    Id="wnd[0]/usr/ctxtVBAK-AUART"
    Text="OR" />
  
  <sap:SAPClick 
    DisplayName="Click Enter"
    Id="wnd[0]/tbar[0]/btn[0]" />
</sap:SAPScope>
```

Required package: `UiPath.SAP.Activities`

### Excel Modern Activities

```xml
<excel:UseExcelFile 
  DisplayName="Use Excel"
  FilePath="[excelPath]"
  CreateIfNotExists="True">
  
  <excel:ReadRange 
    DisplayName="Read Data"
    SheetName="Sheet1"
    Range="A1">
    <excel:ReadRange.DataTable>
      <OutArgument x:TypeArguments="sd:DataTable">[dataTable]</OutArgument>
    </excel:ReadRange.DataTable>
  </excel:ReadRange>
  
  <excel:WriteRange 
    DisplayName="Write Results"
    SheetName="Results"
    Range="A1"
    DataTable="[outputTable]" />
  
  <excel:ForEachExcelRow 
    DisplayName="Process Rows"
    SheetName="Sheet1">
    <ActivityAction x:TypeArguments="excel:ExcelRow">
      <ActivityAction.Argument>
        <DelegateInArgument x:TypeArguments="excel:ExcelRow" Name="row" />
      </ActivityAction.Argument>
      <Sequence DisplayName="Process Row">
        <ui:LogMessage Level="Info" Message="[row(&quot;Name&quot;).ToString]" />
      </Sequence>
    </ActivityAction>
  </excel:ForEachExcelRow>
</excel:UseExcelFile>
```

### Browser Automation Best Practices

```xml
<!-- Wait for element before interaction -->
<ui:ElementExists 
  DisplayName="Wait for Page Load"
  Timeout="30000">
  <ui:ElementExists.Target>
    <ui:Target Selector="&lt;webctrl id='main-content' /&gt;" />
  </ui:ElementExists.Target>
  <ui:ElementExists.Exists>
    <OutArgument x:TypeArguments="x:Boolean">[pageLoaded]</OutArgument>
  </ui:ElementExists.Exists>
</ui:ElementExists>

<!-- Use keyboard shortcuts -->
<ui:KeyboardShortcut 
  DisplayName="Press Enter"
  KeyboardShortcut="enter" />

<!-- Take screenshot on error -->
<ui:TakeScreenshot 
  DisplayName="Capture Error"
  OutputPath="[screenshotPath]" />

<!-- Handle popups -->
<ui:CheckAppState 
  DisplayName="Check for Popup"
  Timeout="3000">
  <ui:CheckAppState.Target>
    <ui:Target Selector="&lt;webctrl class='popup-dialog' /&gt;" />
  </ui:CheckAppState.Target>
  <ui:CheckAppState.PassedWhen>
    <ui:AppearanceRule Enabled="True" />
  </ui:CheckAppState.PassedWhen>
</ui:CheckAppState>
```

## Detailed References

Start with **[references/UNIFIED-INDEX.md](references/UNIFIED-INDEX.md)** (full map). Common shortcuts: **[orchestration/README.md](references/orchestration/README.md)** (router), **[official-documentation-index.md](references/official-documentation-index.md)** (docs.uipath.com), **[activities.md](references/activities.md)** (XAML catalog), **[xaml-syntax.md](references/xaml-syntax.md)**, **[selectors.md](references/selectors.md)**, **[project-structure.md](references/project-structure.md)**, **[invoke-code-and-expressions.md](references/invoke-code-and-expressions.md)**, **[connectors-and-integration.md](references/connectors-and-integration.md)**, **[coded-automations.md](references/coded-automations.md)**.

## Best Practices

### Naming Conventions
| Element | Convention | Example |
|---------|------------|---------|
| Variables | camelCase with type prefix | `strFileName`, `intCount`, `dtResults` |
| Arguments | Direction_PascalCase | `in_FilePath`, `out_Status`, `io_DataTable` |
| Workflows | PascalCase | `ProcessTransaction.xaml` |
| Activities | Descriptive DisplayName | `"Read Input Excel"` |

### Error Handling Strategy

See **[references/orchestration/error-handling.md](references/orchestration/error-handling.md)** for full patterns. Summary: Try-Catch around critical sections; RetryScope for transient failures; contextual logging; Global Exception Handler where appropriate.

### XAML String Escaping Rules
1. **NEVER use multiline strings** for HTML/JSON content
2. **Use XML entities** for HTML tags (`&lt;`, `&gt;`, `&amp;`)
3. **Keep JSON on single lines** - No line breaks
4. **Use `\""` (single backslash) for nested JSON quotes** in Action.Http body - NOT `\\""` (double backslash)
5. **Avoid `xml:space="preserve"`** on strings with HTML/JSON
6. **ALWAYS use HTTPS URLs** for Adaptive Card Action.Http - HTTP is blocked by Outlook
7. **Avoid `<` and `<=` in string literals** inside InvokeCode - Use text alternatives like "less than" or "24 months or less" instead of `<= 24`

### Performance Guidelines
- Minimize UI interactions (batch operations when possible)
- Use background activities for non-UI work
- Implement parallel processing with Parallel For Each
- Cache frequently accessed data in variables
