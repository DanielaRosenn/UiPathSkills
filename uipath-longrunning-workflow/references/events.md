# LRW Events Reference

Intermediate events are **points where execution pauses** for a time condition or external message, then resumes.

---

## Resume after Delay

**BPMN**: Timer Intermediate Catch Event.  
**Use when**: Process must pause for a **duration or until a date/time**, then resume automatically (no human or external trigger).

**Persistence**: Yes — robot suspends here.  
**Activity**: `Resume After Delay` from UiPath.Persistence.Activities.  
**Properties**: Duration (TimeSpan) or ResumeAt (DateTime); StatusMessage.

**Do not use** the standard `Delay` activity in an LRW main flow — it does not support persistence; after robot restart the remaining time is lost.

**Examples**:
- Wait 2 hours (SLA check, retry window).
- Wait until next business day 9am (ResumeAt).
- Wait 30 days (renewal reminder).

---

## Wait for Message

**BPMN**: Message Intermediate Catch Event.  
**Use when**: Process must pause until an **external event** occurs (email reply, Salesforce update, webhook), then resume with event data.

**Persistence**: Yes.  
**Implementation**: In our template we use **ConnectorPersistenceActivity** (Wait for an Event on HTTP Webhook and Resume) inside a Subprocess TaskNode. Other connectors (M365, Salesforce, etc.) use the corresponding Integration Service "Wait for an Event on {Connector} and Resume" activity.

**Supported sources** (Integration Service): Microsoft 365 (email, calendar, Teams), Google Workspace, Salesforce, Data Service, custom webhooks, etc.

**Pattern**: Send request/email → Wait for Message (or webhook) → parse response → Decision.

---

## Event placeholder

On the new LRW canvas, an **Event Placeholder** is a design-time placeholder until you choose "Resume after Delay" or "Wait for Message". In ProcessDiagram XAML we use the concrete node types (IntermediateEvent.Catch.Message for webhook, or the timer equivalent where supported).
