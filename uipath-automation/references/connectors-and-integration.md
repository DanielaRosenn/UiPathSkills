# Connectors, Integration Service, and related activities

This file orients Studio workflow design toward **official** UiPath integration patterns. Deep product setup lives in Automation Cloud docs; here we focus on **activity packs** and **when to use which skill**.

## Integration Service (cloud connectors)

- **Concept:** Prebuilt connectors (Salesforce, Microsoft 365, ServiceNow, etc.) are exposed to Studio after connections are configured in **UiPath Integration Service**.
- **Activities:** Prefer **Integration Service-based activities** from the activities documentation when the connector exists—see [Integration Service-based activities](https://docs.uipath.com/activities/other/latest/workflow/integration-service-based-activities).
- **Package:** Typically `UiPath.IntegrationService.Activities` (pin version per project policy).

**In Studio:** Add the activity → complete connection/credential wiring in the property panel as required by the connector.

## Data Service

Two documentation tracks exist:

1. **Integration Service–based** Data Service activities (newer pattern in the activities doc tree).
2. **Classic** Data Service activities: [Data Service activities](https://docs.uipath.com/activities/other/latest/workflow/data-service-activities).

Use the pack and entity model that matches your tenant and project template.

## HTTP / REST (no connector)

- For generic REST calls, use **UiPath.WebAPI.Activities** patterns (e.g. `NetHttpRequest`) as shown in the main `SKILL.md`—not deprecated HTTP client patterns.
- Combine with **Orchestrator assets** for URLs and API keys.

## Mail

- **Microsoft 365 / Graph** via Integration Service: often **Send Mail** / connection-based activities (see Microsoft 365 connector and activities doc).
- Classic Outlook activities may still exist in legacy projects; prefer **connection-based** patterns for cloud alignment.

## RPA-specific extensions (packages)

| Area | Typical package family | Notes |
|------|-------------------------|--------|
| UI (modern) | `UiPath.UIAutomation.Activities` | Use **Use Application/Browser** (MDE) where possible |
| Excel | `UiPath.Excel.Activities` | Modern Excel scope patterns |
| Persistence / queues | `UiPath.Persistence.Activities` | Long-running, wait/resume |
| Document Understanding | `UiPath.DocumentUnderstanding.*` | See **uipath-document-understanding** skill |
| SAP | `UiPath.SAP.Activities` | SAP GUI scripting |

Always verify **package version** against `project.json` and Studio compatibility.

## Related skills (do not duplicate)

- **uipath-integration-service** – connectors, triggers, auth patterns.
- **uipath-data-service** – entities, CRUD, queries.
- **uipath-document-understanding** – taxonomy, extraction, validation station.
- **uipath-persistence-activities** – HITL, suspend/resume, forms.

The **uipath-automation** skill provides **XAML structure, dependencies, and Studio-oriented defaults**; connector-specific configuration belongs in those skills or in official Integration Service documentation.
