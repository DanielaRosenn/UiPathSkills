# Action Center Integration Reference

Complete guide to integrating UiPath Action Center with long-running workflows.

## Table of Contents

1. [Form Schema Design](#form-schema-design)
2. [Task Assignment](#task-assignment)
3. [Task Actions](#task-actions)
4. [Form Layouts](#form-layouts)
5. [Validation](#validation)
6. [Conditional Fields](#conditional-fields)
7. [File Attachments](#file-attachments)
8. [Integration Patterns](#integration-patterns)

---

## Form Schema Design

### Basic Schema Structure

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "Form Title",
  "description": "Form description",
  "properties": {
    "fieldName": {
      "type": "string",
      "title": "Field Label",
      "description": "Help text"
    }
  },
  "required": ["fieldName"]
}
```

### Field Types

#### String Field

```json
{
  "customerName": {
    "type": "string",
    "title": "Customer Name",
    "minLength": 1,
    "maxLength": 100
  }
}
```

#### Number Field

```json
{
  "amount": {
    "type": "number",
    "title": "Amount",
    "minimum": 0,
    "maximum": 1000000
  }
}
```

#### Integer Field

```json
{
  "quantity": {
    "type": "integer",
    "title": "Quantity",
    "minimum": 1,
    "default": 1
  }
}
```

#### Boolean Field

```json
{
  "approved": {
    "type": "boolean",
    "title": "Approve this request?",
    "default": false
  }
}
```

#### Date Field

```json
{
  "dueDate": {
    "type": "string",
    "format": "date",
    "title": "Due Date"
  }
}
```

#### DateTime Field

```json
{
  "scheduledTime": {
    "type": "string",
    "format": "date-time",
    "title": "Scheduled Time"
  }
}
```

#### Email Field

```json
{
  "contactEmail": {
    "type": "string",
    "format": "email",
    "title": "Contact Email"
  }
}
```

#### Dropdown (Enum)

```json
{
  "priority": {
    "type": "string",
    "title": "Priority",
    "enum": ["Low", "Medium", "High", "Critical"],
    "default": "Medium"
  }
}
```

#### Multi-line Text

```json
{
  "comments": {
    "type": "string",
    "title": "Comments",
    "ui:widget": "textarea"
  }
}
```

#### Read-Only Field

```json
{
  "requestId": {
    "type": "string",
    "title": "Request ID",
    "readOnly": true
  }
}
```

---

## Task Assignment

### Assign to User

```xml
<ui:CreateFormTask
  TaskTitle="Review Request"
  FormSchemaPath="FormSchemas\Review.json">
  <ui:CreateFormTask.TaskAssignee>
    <ui:TaskAssignee
      Type="User"
      Value="john.doe@company.com" />
  </ui:CreateFormTask.TaskAssignee>
</ui:CreateFormTask>
```

### Assign to Group

```xml
<ui:CreateFormTask.TaskAssignee>
  <ui:TaskAssignee
    Type="Group"
    Value="Finance Approvers" />
</ui:CreateFormTask.TaskAssignee>
```

### Dynamic Assignment

```xml
<ui:CreateFormTask
  TaskTitle="[taskTitle]"
  FormSchemaPath="FormSchemas\Approval.json">
  <ui:CreateFormTask.TaskAssignee>
    <ui:TaskAssignee
      Type="User"
      Value="[assigneeEmail]" />
  </ui:CreateFormTask.TaskAssignee>
</ui:CreateFormTask>
```

### Assignment Based on Amount

```xml
<If Condition="[amount &gt; 10000]">
  <If.Then>
    <ui:CreateFormTask.TaskAssignee>
      <ui:TaskAssignee Type="Group" Value="Senior Approvers" />
    </ui:CreateFormTask.TaskAssignee>
  </If.Then>
  <If.Else>
    <ui:CreateFormTask.TaskAssignee>
      <ui:TaskAssignee Type="Group" Value="Standard Approvers" />
    </ui:CreateFormTask.TaskAssignee>
  </If.Else>
</If>
```

---

## Task Actions

### Standard Actions

Forms can define custom actions that appear as buttons:

```json
{
  "type": "object",
  "properties": {
    "decision": {
      "type": "object",
      "properties": {
        "approved": {"type": "boolean"}
      }
    }
  },
  "ui:actions": [
    {
      "name": "approve",
      "label": "Approve",
      "style": "primary"
    },
    {
      "name": "reject",
      "label": "Reject",
      "style": "danger"
    },
    {
      "name": "requestInfo",
      "label": "Request More Info",
      "style": "secondary"
    }
  ]
}
```

### Processing Actions in XAML

```xml
<ui:WaitForFormTaskAndResume
  TaskId="[taskId]"
  TaskAction="[action]"
  TaskData="[formData]" />

<Switch Expression="[action]">
  <Case Value="approve">
    <Sequence DisplayName="Handle Approval">
      <ui:LogMessage Message="Request approved" />
      <!-- Approval logic -->
    </Sequence>
  </Case>
  <Case Value="reject">
    <Sequence DisplayName="Handle Rejection">
      <ui:LogMessage Message="Request rejected" />
      <!-- Rejection logic -->
    </Sequence>
  </Case>
  <Case Value="requestInfo">
    <Sequence DisplayName="Handle Info Request">
      <ui:LogMessage Message="More info requested" />
      <!-- Create follow-up task -->
    </Sequence>
  </Case>
</Switch>
```

---

## Form Layouts

### Grouped Fields

```json
{
  "type": "object",
  "properties": {
    "customerInfo": {
      "type": "object",
      "title": "Customer Information",
      "properties": {
        "name": {"type": "string", "title": "Name"},
        "email": {"type": "string", "format": "email", "title": "Email"},
        "phone": {"type": "string", "title": "Phone"}
      }
    },
    "orderInfo": {
      "type": "object",
      "title": "Order Information",
      "properties": {
        "orderId": {"type": "string", "title": "Order ID"},
        "amount": {"type": "number", "title": "Amount"},
        "date": {"type": "string", "format": "date", "title": "Order Date"}
      }
    }
  }
}
```

### Array of Items

```json
{
  "type": "object",
  "properties": {
    "lineItems": {
      "type": "array",
      "title": "Line Items",
      "items": {
        "type": "object",
        "properties": {
          "description": {"type": "string", "title": "Description"},
          "quantity": {"type": "integer", "title": "Qty", "minimum": 1},
          "unitPrice": {"type": "number", "title": "Unit Price"},
          "total": {"type": "number", "title": "Total", "readOnly": true}
        }
      }
    }
  }
}
```

### Two-Column Layout

```json
{
  "type": "object",
  "properties": {
    "firstName": {"type": "string", "title": "First Name"},
    "lastName": {"type": "string", "title": "Last Name"}
  },
  "ui:layout": {
    "firstName": {"ui:column": 1},
    "lastName": {"ui:column": 2}
  }
}
```

---

## Validation

### Required Fields

```json
{
  "type": "object",
  "properties": {
    "approverName": {"type": "string", "title": "Approver Name"},
    "decision": {"type": "boolean", "title": "Decision"}
  },
  "required": ["approverName", "decision"]
}
```

### Pattern Validation

```json
{
  "invoiceNumber": {
    "type": "string",
    "title": "Invoice Number",
    "pattern": "^INV-[0-9]{6}$",
    "description": "Format: INV-000000"
  }
}
```

### Range Validation

```json
{
  "discountPercent": {
    "type": "number",
    "title": "Discount %",
    "minimum": 0,
    "maximum": 50,
    "description": "Maximum 50% discount allowed"
  }
}
```

### String Length

```json
{
  "comments": {
    "type": "string",
    "title": "Comments",
    "minLength": 10,
    "maxLength": 500,
    "description": "Please provide at least 10 characters"
  }
}
```

### Custom Error Messages

```json
{
  "email": {
    "type": "string",
    "format": "email",
    "title": "Email",
    "ui:errorMessages": {
      "format": "Please enter a valid email address"
    }
  }
}
```

---

## Conditional Fields

### Show Field Based on Value

```json
{
  "type": "object",
  "properties": {
    "requiresApproval": {
      "type": "boolean",
      "title": "Requires Additional Approval?"
    },
    "approverEmail": {
      "type": "string",
      "format": "email",
      "title": "Approver Email"
    }
  },
  "dependencies": {
    "requiresApproval": {
      "oneOf": [
        {
          "properties": {
            "requiresApproval": {"enum": [false]}
          }
        },
        {
          "properties": {
            "requiresApproval": {"enum": [true]},
            "approverEmail": {"type": "string"}
          },
          "required": ["approverEmail"]
        }
      ]
    }
  }
}
```

### Conditional Required

```json
{
  "type": "object",
  "properties": {
    "decision": {
      "type": "string",
      "enum": ["Approve", "Reject", "Escalate"]
    },
    "rejectionReason": {
      "type": "string",
      "title": "Rejection Reason"
    }
  },
  "if": {
    "properties": {"decision": {"const": "Reject"}}
  },
  "then": {
    "required": ["rejectionReason"]
  }
}
```

---

## File Attachments

### File Upload Field

```json
{
  "supportingDocs": {
    "type": "array",
    "title": "Supporting Documents",
    "items": {
      "type": "string",
      "format": "data-url"
    },
    "ui:options": {
      "accept": ".pdf,.doc,.docx,.xlsx"
    }
  }
}
```

### Single File Upload

```json
{
  "invoice": {
    "type": "string",
    "format": "data-url",
    "title": "Invoice Document",
    "ui:options": {
      "accept": ".pdf"
    }
  }
}
```

### Processing Attachments in XAML

```xml
<ui:WaitForFormTaskAndResume
  TaskId="[taskId]"
  TaskAction="[action]"
  TaskData="[formDataJson]" />

<!-- Parse form data -->
<Assign>
  <Assign.To>[formData]</Assign.To>
  <Assign.Value>[JObject.Parse(formDataJson)]</Assign.Value>
</Assign>

<!-- Get attachment (base64) -->
<Assign>
  <Assign.To>[attachmentBase64]</Assign.To>
  <Assign.Value>[formData("invoice").ToString()]</Assign.Value>
</Assign>

<!-- Save attachment -->
<InvokeCode Language="CSharp">
  <InvokeCode.Code>
    var base64Data = attachmentBase64.Split(',')[1];
    var bytes = Convert.FromBase64String(base64Data);
    File.WriteAllBytes(outputPath, bytes);
  </InvokeCode.Code>
</InvokeCode>
```

---

## Integration Patterns

### Pattern 1: Simple Approval

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "Approval Request",
  "properties": {
    "requestDetails": {
      "type": "object",
      "title": "Request Details",
      "properties": {
        "requestId": {"type": "string", "readOnly": true},
        "requestor": {"type": "string", "readOnly": true},
        "description": {"type": "string", "readOnly": true},
        "amount": {"type": "number", "readOnly": true}
      }
    },
    "decision": {
      "type": "object",
      "title": "Your Decision",
      "properties": {
        "approved": {"type": "boolean", "title": "Approve?"},
        "comments": {"type": "string", "title": "Comments", "ui:widget": "textarea"}
      },
      "required": ["approved"]
    }
  }
}
```

### Pattern 2: Data Entry Form

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "Invoice Data Entry",
  "properties": {
    "invoiceInfo": {
      "type": "object",
      "title": "Invoice Information",
      "properties": {
        "invoiceNumber": {
          "type": "string",
          "title": "Invoice Number",
          "pattern": "^INV-[0-9]+$"
        },
        "invoiceDate": {
          "type": "string",
          "format": "date",
          "title": "Invoice Date"
        },
        "vendorName": {
          "type": "string",
          "title": "Vendor Name"
        },
        "totalAmount": {
          "type": "number",
          "title": "Total Amount",
          "minimum": 0
        }
      },
      "required": ["invoiceNumber", "invoiceDate", "vendorName", "totalAmount"]
    },
    "lineItems": {
      "type": "array",
      "title": "Line Items",
      "minItems": 1,
      "items": {
        "type": "object",
        "properties": {
          "description": {"type": "string", "title": "Description"},
          "quantity": {"type": "integer", "title": "Qty", "minimum": 1},
          "unitPrice": {"type": "number", "title": "Unit Price"}
        },
        "required": ["description", "quantity", "unitPrice"]
      }
    }
  }
}
```

### Pattern 3: Exception Handling Form

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "Exception Resolution",
  "properties": {
    "exceptionDetails": {
      "type": "object",
      "title": "Exception Details",
      "properties": {
        "errorType": {"type": "string", "readOnly": true},
        "errorMessage": {"type": "string", "readOnly": true},
        "timestamp": {"type": "string", "readOnly": true},
        "affectedRecord": {"type": "string", "readOnly": true}
      }
    },
    "resolution": {
      "type": "object",
      "title": "Resolution",
      "properties": {
        "action": {
          "type": "string",
          "title": "Action",
          "enum": ["Retry", "Skip", "Manual Fix", "Escalate"]
        },
        "correctedData": {
          "type": "string",
          "title": "Corrected Data (if applicable)"
        },
        "notes": {
          "type": "string",
          "title": "Resolution Notes",
          "ui:widget": "textarea"
        }
      },
      "required": ["action", "notes"]
    }
  }
}
```

### Pattern 4: Multi-Step Review

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "Document Review",
  "properties": {
    "documentInfo": {
      "type": "object",
      "title": "Document Information",
      "properties": {
        "documentId": {"type": "string", "readOnly": true},
        "documentType": {"type": "string", "readOnly": true},
        "submittedBy": {"type": "string", "readOnly": true},
        "submittedDate": {"type": "string", "readOnly": true}
      }
    },
    "contentReview": {
      "type": "object",
      "title": "Content Review",
      "properties": {
        "accuracyScore": {
          "type": "integer",
          "title": "Accuracy (1-5)",
          "minimum": 1,
          "maximum": 5
        },
        "completenessScore": {
          "type": "integer",
          "title": "Completeness (1-5)",
          "minimum": 1,
          "maximum": 5
        },
        "contentNotes": {
          "type": "string",
          "title": "Content Notes"
        }
      }
    },
    "complianceReview": {
      "type": "object",
      "title": "Compliance Review",
      "properties": {
        "meetsPolicy": {"type": "boolean", "title": "Meets Policy?"},
        "complianceIssues": {
          "type": "array",
          "title": "Compliance Issues",
          "items": {"type": "string"}
        }
      }
    },
    "finalDecision": {
      "type": "object",
      "title": "Final Decision",
      "properties": {
        "decision": {
          "type": "string",
          "enum": ["Approve", "Reject", "Request Revision"]
        },
        "finalComments": {
          "type": "string",
          "ui:widget": "textarea"
        }
      },
      "required": ["decision"]
    }
  }
}
```
