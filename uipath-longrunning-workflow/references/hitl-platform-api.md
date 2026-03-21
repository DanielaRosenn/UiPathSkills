# HITL Platform API Reference

Complete API reference for the Human-in-the-Loop Platform.

**See first (shared, all skills):** [hitl-platform.md](../../_shared/hitl-platform.md) — integration overview, payloads, Adaptive Card rules. This file adds endpoint-level detail.

**Paths:** From `spec-kit-ui/.claude/skills/` or `%USERPROFILE%\.cursor\skills` (junction), `_shared` is the sibling folder of `uipath-longrunning-workflow/`.

## Endpoints

### Create Approval

```http
POST /api/v1/approvals
Content-Type: application/json
x-api-key: {API_KEY}

{
  "process_id": "string",
  "title": "string",
  "notification_channel": "email|slack|both",
  "form_schema": [...],
  "form_data": {...},
  "approvers": [...],
  "approvalType": "sequential|parallel|any",
  "callbackUrl": "string",
  "metadata": {...}
}
```

### Get Approval

```http
GET /api/v1/approvals/{approvalId}
x-api-key: {API_KEY}
```

### Cancel Approval

```http
POST /api/v1/approvals/{approvalId}/cancel
x-api-key: {API_KEY}
```

### Respond via Adaptive Card

```http
POST /api/v1/approvals/adaptive-card/{responseToken}
Content-Type: application/json

{
  "action": "approved|rejected",
  "comments": "string",
  "responseData": {...}
}
```

### Respond via Token URL

```http
POST /api/v1/approvals/token/{responseToken}/respond
Content-Type: application/json

{
  "action": "approved|rejected",
  "comments": "string"
}
```

---

## Request/Response Examples

### Create Approval - Full Example

**Request:**
```json
{
  "process_id": "QUOTE-2026-001",
  "title": "Sales Deal Approval - Acme Corp",
  "notification_channel": "both",
  "form_schema": [
    {
      "name": "requesterName",
      "type": "readonly",
      "label": "Requester"
    },
    {
      "name": "customerName",
      "type": "readonly",
      "label": "Customer Name"
    },
    {
      "name": "deal_amount",
      "type": "number",
      "label": "Deal Amount",
      "required": true,
      "sensitive": true,
      "min": 1
    },
    {
      "name": "agreement_months",
      "type": "select",
      "label": "Agreement (Months)",
      "required": true,
      "options": [
        { "value": "12", "label": "12" },
        { "value": "24", "label": "24" },
        { "value": "36", "label": "36" }
      ]
    },
    {
      "name": "wants_renewal",
      "type": "checkbox",
      "label": "Auto-Renew"
    },
    {
      "name": "start_date",
      "type": "date",
      "label": "Start Date"
    },
    {
      "name": "meeting_time",
      "type": "time",
      "label": "Meeting Time"
    },
    {
      "name": "business_justification",
      "type": "textarea",
      "label": "Business Justification",
      "required": true
    }
  ],
  "form_data": {
    "requesterName": "john.doe@company.com",
    "customerName": "Acme Corp",
    "deal_amount": 75000,
    "agreement_months": "24",
    "start_date": "2026-02-15"
  },
  "metadata": {
    "source_id": "CRM-447",
    "cost_center": "SALES",
    "opportunity_id": "OPP-789"
  },
  "approvers": [
    {
      "email": "manager@company.com",
      "name": "John Manager",
      "notificationChannel": "both"
    }
  ],
  "approvalType": "sequential",
  "callbackUrl": "https://cloud.uipath.com/catonetworks/20bf226d-f42c-4668-9cad-1c86ed398cf0/elements_/v1/webhooks/events/MUkoaX93iHW4Y7PTbNuAUORocC0ee0nCzHl_pm0auXzHcBVXXAJh-4QlvisqnMLUslQjKap1g11eSf-KYtCGsA"
}
```

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "processId": "QUOTE-2026-001",
  "title": "Sales Deal Approval - Acme Corp",
  "status": "pending",
  "approvalType": "sequential",
  "notificationChannel": "both",
  "approvalLines": [
    {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "email": "manager@company.com",
      "name": "John Manager",
      "status": "pending",
      "sequenceOrder": 1,
      "responseToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
  ],
  "formSchema": [...],
  "formData": {...},
  "metadata": {...},
  "callbackUrl": "https://cloud.uipath.com/...",
  "createdAt": "2026-02-22T10:00:00.000Z",
  "updatedAt": "2026-02-22T10:00:00.000Z"
}
```

---

## Webhook Callback Payload

When an approver responds, the HITL Platform sends this payload to the callback URL:

```json
{
  "event": "approval.completed",
  "approvalId": "550e8400-e29b-41d4-a716-446655440000",
  "processId": "QUOTE-2026-001",
  "title": "Sales Deal Approval - Acme Corp",
  "status": "approved",
  "action": "approved",
  "respondedBy": {
    "email": "manager@company.com",
    "name": "John Manager"
  },
  "respondedAt": "2026-02-22T10:30:00.000Z",
  "comments": "Approved - strategic customer",
  "formData": {
    "requesterName": "john.doe@company.com",
    "customerName": "Acme Corp",
    "deal_amount": 75000,
    "agreement_months": "24",
    "business_justification": "Strategic partnership opportunity"
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

## Error Responses

### 400 Bad Request

```json
{
  "error": "Bad Request",
  "message": "Missing required field: process_id",
  "statusCode": 400
}
```

### 401 Unauthorized

```json
{
  "error": "Unauthorized",
  "message": "Invalid or missing API key",
  "statusCode": 401
}
```

### 404 Not Found

```json
{
  "error": "Not Found",
  "message": "Approval not found",
  "statusCode": 404
}
```

### 409 Conflict

```json
{
  "error": "Conflict",
  "message": "Approval has already been completed",
  "statusCode": 409
}
```

---

## Rate Limits

| Endpoint | Rate Limit |
|----------|------------|
| POST /api/v1/approvals | 100 requests/minute |
| GET /api/v1/approvals | 200 requests/minute |
| POST /api/v1/approvals/adaptive-card | 500 requests/minute |

---

## Best Practices

1. **Store API keys in Orchestrator Assets** - Never hardcode credentials
2. **Use retry logic** - Network issues can cause transient failures
3. **Log approval IDs** - For debugging and audit trails
4. **Handle webhook failures** - Implement idempotent webhook handlers
5. **Set appropriate timeouts** - 30 seconds recommended for API calls
