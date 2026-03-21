# Persistence Activities Reference

These are the **inner activities** from `UiPath.Persistence.Activities` that power the suspension/resume behavior inside Human Approval, Wait for Trigger, and other LRW nodes.

---

## Pairing Rules Summary

| Create activity | Must pair with | Suspension? |
|-----------------|----------------|-------------|
| Create App Task | Wait For App Task And Resume | Yes |
| Create Form Task | Wait For Form Task And Resume | Yes |
| Create External Task | Wait For External Task And Resume | Yes |
| Add Queue Item And Get Reference | Wait For Queue Item And Resume | Yes |
| Start Job And Get Reference | Wait For Job And Resume | Yes |
| (none) | Resume After Delay | Yes |
| Event Trigger (IS) | Wait for an Event and Resume | Yes |

Every suspension activity must be in a workflow with `supportsPersistence: true`.

---

## Create App Task

**Use inside**: Human Approval node  
**Purpose**: Creates a task in Action Center backed by a **UiPath App** (recommended for new projects).

| Property | Type | Description |
|----------|------|-------------|
| App Name | String | Name of the UiPath App (must be published to Orchestrator) |
| Title | String | Task title shown in Action Center |
| Priority | TaskPriority | Low / Normal / High / Critical |
| Assignees | String[] | Email(s) of users assigned to the task |
| Data | Dictionary<String,Object> | Key-value pairs passed to the App as input |
| External Reference | String | Optional reference ID linking task to your business entity |
| Created App Task (output) | UserActionData | The task object — pass to Wait for App Task And Resume |

**Pairing rule**: Always follow with `Wait for App Task And Resume`, passing the same `UserActionData` object.

---

## Wait for App Task And Resume

**Use inside**: Human Approval node, after Create App Task  
**Purpose**: **Suspends the workflow** until the assigned user completes the App Task in Action Center. Robot is freed.

| Property | Type | Description |
|----------|------|-------------|
| Created App Task (input) | UserActionData | The task from Create App Task |
| Action taken (output) | String | Button the user clicked (e.g., "Approved", "Rejected", custom label) |
| Status Message | String | Shown in Orchestrator while job is suspended |
| TimeoutMS | Int32 | Max ms to wait (0 = indefinite) |

---

## Configure Task Timer

**Use inside**: Human Approval node, between Create App Task and Wait For App Task And Resume  
**Purpose**: Sets a **deadline** on the task and defines escalation rules if the deadline is missed.

| Property | Type | Description |
|----------|------|-------------|
| Task Object (input) | UserActionData | The task from Create App Task |
| Expiration Duration | TimeSpan | How long before the timer fires |
| Timer Action | Enum | Reassign / Send Reminder / Complete |
| Reassign To | String | Email to reassign to if timer fires |

---

## Create Form Task

**Use inside**: Human Approval node (legacy approach — forms built in Studio's form designer)  
**Purpose**: Creates an Action Center task using the **built-in form designer** (older approach, still works).

| Property | Type | Description |
|----------|------|-------------|
| Task Title | String | Task title in Action Center |
| Catalog Name | String | Name of the form catalog in Orchestrator |
| Assignee Name | String | Username or email of assignee |
| Form Data | Dictionary<String,Argument> | Data passed to form — In/Out/InOut arguments |
| Result (output) | FormTaskData | Pass to Wait For Form Task And Resume |

---

## Wait For Form Task And Resume

**Suspends**: Yes  
**Use with**: Create Form Task

| Property | Type | Description |
|----------|------|-------------|
| Task Object (input) | FormTaskData | From Create Form Task |
| Task Action (output) | String | Action taken by user |
| Task Object (output) | FormTaskData | Updated task with output form data |
| Status Message | String | Message shown in Orchestrator |
| TimeoutMS | Int32 | Max wait time |

**Note**: Any `SecureString` variables are cleared on resume (not serializable). Re-fetch from Orchestrator assets after resuming.

---

## Resume After Delay

**Suspends**: Yes  
**Purpose**: Suspends workflow for a specified duration, then resumes automatically. This is the correct way to add delays in an LRW (standard `Delay` activity doesn't support persistence).

| Property | Type | Description |
|----------|------|-------------|
| Duration | TimeSpan | How long to wait. E.g., `TimeSpan.FromHours(2)` |
| ResumeAt | DateTime | Alternative — resume at a specific datetime |
| Status Message | String | Shown in Orchestrator |

**Either Duration OR ResumeAt — not both.**

---

## Add Queue Item And Get Reference

**Use inside**: Wait for Trigger node or Sequence  
**Purpose**: Adds an item to an Orchestrator queue AND returns a reference object so you can wait for it to be processed.

| Property | Type | Description |
|----------|------|-------------|
| Queue Name | String | Target Orchestrator queue |
| Item Information | Dictionary<String,Argument> | Data to put in the queue item |
| Deadline | DateTime | When item should be processed by |
| Priority | QueueItemPriority | Low / Normal / High |
| Queue Item Object (output) | QueueItemData | Reference — pass to Wait for Queue Item And Resume |

**Must pair with**: `Wait for Queue Item And Resume`

---

## Wait For Queue Item And Resume

**Suspends**: Yes  
**Purpose**: Suspends workflow until the referenced queue item is **successfully processed** by another robot.

| Property | Type | Description |
|----------|------|-------------|
| Queue Item Object (input) | QueueItemData | From Add Queue Item And Get Reference |
| Item Information (output) | Dictionary<String,Argument> | Output args set by the processing robot |
| Status Message | String | Shown in Orchestrator |

---

## Start Job And Get Reference

**Use inside**: Wait for Trigger node  
**Purpose**: Starts an Orchestrator job (another process) and returns a reference to wait for its completion.

| Property | Type | Description |
|----------|------|-------------|
| Process Name | String | Display name of the process in Orchestrator |
| Job Arguments | Dictionary<String,Argument> | Arguments passed to the invoked job |
| Job Object (output) | JobData | Reference — pass to Wait For Job And Resume |
| Entry Point | String | Optional — specific entry point workflow file |
| Folder Path | String | Orchestrator folder if different from current |

---

## Wait For Job And Resume

**Suspends**: Yes  
**Purpose**: Suspends workflow until the referenced Orchestrator job **completes**.

| Property | Type | Description |
|----------|------|-------------|
| Job Object (input) | JobData | From Start Job And Get Reference |
| Output Arguments (output) | Dictionary<String,Object> | Arguments returned by the job |
| Status Message | String | Shown in Orchestrator |

---

## Create External Task / Wait For External Task And Resume

**Purpose**: Creates an Action Center task that will be completed by an **external third-party system** (not by a human in Action Center UI, and not by UiPath Apps). External system calls back to Orchestrator to complete the task.

Create External Task outputs `ExternalTaskData`; Wait For External Task And Resume suspends until callback.

---

## HTTP Webhook (Integration Service) — our production pattern

For HITL Platform and custom webhooks we use **ConnectorPersistenceActivity** (`Wait for an Event on HTTP Webhook and Resume`) inside a **Subprocess** TaskNode. See the main SKILL for the full webhook subprocess pattern:

1. TaskNode (ApprovalFlow) sends approval request to HITL API
2. TaskNode (Subprocess) contains embedded ProcessDiagram with:
   - EventNode (Start)
   - EventNode with ConnectorPersistenceActivity (waits for webhook callback)
   - TaskNode (Get Trigger Event Output)
   - TaskNode (Deserialize webhook payload)
   - EndNode
3. DecisionNode checks decision variable

---

## Other Activities

- **Assign Tasks** / **Forward Task**: Reassign or forward existing task.
- **Complete Task**: Programmatically complete a task without user action.
- **Get Form Tasks** / **Get App Tasks**: Query Action Center.
- **Get Task Data**: Get form values from a specific task.
- **Add Task Comment**: Add comment visible in Action Center.
- **Update Task Labels**: Add/remove labels for categorization.
