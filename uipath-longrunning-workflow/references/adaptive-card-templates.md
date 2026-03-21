# Adaptive Card Templates

Email Adaptive Card templates for approval workflows.

## Overview

Adaptive Cards are used to create interactive approval emails in Outlook. The HITL Platform sends these automatically, but you can also send custom emails from UiPath workflows.

**Design Tool**: https://adaptivecards.io/designer/
**Schema Explorer**: https://adaptivecards.io/explorer/
**Documentation Hub**: https://adaptivecards.microsoft.com/

## CRITICAL: URL Requirements

**Action.Http URLs MUST use HTTPS** - Microsoft Outlook Actionable Messages require HTTPS URLs. HTTP URLs will fail with "Target URL scheme 'http' is not allowed".

**Production URL**: `https://djun97l419cdy.cloudfront.net/api/v1/approvals/adaptive-card/{token}`

Do NOT use the internal HTTP URL from config for adaptive card actions. Always use the HTTPS CloudFront URL.

---

## Complete Element Reference

### Card Structure Elements

| Element | Description | Use Case |
|---------|-------------|----------|
| `AdaptiveCard` | Root container | Required wrapper for all cards |
| `Container` | Groups elements | Logical grouping with optional style |
| `ColumnSet` | Horizontal layout | Side-by-side content |
| `Column` | Column within ColumnSet | Individual column content |
| `FactSet` | Key-value pairs | Read-only data display |
| `ImageSet` | Image gallery | Multiple images |
| `Table` | Tabular data | Structured data (v1.5+) |
| `ActionSet` | Action buttons | Inline action buttons |

### Display Elements

| Element | Description | Properties |
|---------|-------------|------------|
| `TextBlock` | Text display | text, size, weight, color, wrap, spacing |
| `Image` | Single image | url, size, style, altText |
| `Media` | Video/audio | sources, poster |
| `RichTextBlock` | Formatted text | inlines (TextRun) |

### Input Elements (Interactive)

| Element | Description | Key Properties |
|---------|-------------|----------------|
| `Input.Text` | Text input | id, label, placeholder, isMultiline, maxLength |
| `Input.Number` | Number input | id, label, min, max, value |
| `Input.Date` | Date picker | id, label, min, max, value |
| `Input.Time` | Time picker | id, label, min, max, value |
| `Input.Toggle` | Checkbox/switch | id, title, value, valueOn, valueOff |
| `Input.ChoiceSet` | Dropdown/radio | id, choices, style, isMultiSelect, value |

### Action Elements

| Element | Description | Use Case |
|---------|-------------|----------|
| `Action.Http` | HTTP request | Approval/rejection callbacks |
| `Action.OpenUrl` | Open URL | External links |
| `Action.Submit` | Submit form | Bot/Teams scenarios |
| `Action.ShowCard` | Nested card | Expandable content |
| `Action.ToggleVisibility` | Show/hide | Dynamic UI |
| `Action.Execute` | Universal action | Teams/Bot scenarios |

---

## Input.ChoiceSet (Dropdown) - Complete Reference

The `Input.ChoiceSet` is used for dropdown menus and selection lists.

### Basic Dropdown

```json
{
  "type": "Input.ChoiceSet",
  "id": "selectedApprover",
  "style": "compact",
  "value": "default@email.com",
  "choices": [
    { "title": "John Doe (john@company.com)", "value": "john@company.com" },
    { "title": "Jane Smith (jane@company.com)", "value": "jane@company.com" },
    { "title": "Bob Wilson (bob@company.com)", "value": "bob@company.com" }
  ]
}
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | **Required**. Unique identifier for data binding |
| `choices` | array | **Required**. Array of `{title, value}` objects |
| `style` | string | `"compact"` (dropdown) or `"expanded"` (radio buttons) |
| `value` | string | Default selected value |
| `isMultiSelect` | boolean | Allow multiple selections |
| `placeholder` | string | Placeholder text when no selection |
| `isRequired` | boolean | Validation - must select |
| `label` | string | Accessibility label (v1.3+) |
| `errorMessage` | string | Validation error message |

### Expanded Style (Radio Buttons)

```json
{
  "type": "Input.ChoiceSet",
  "id": "approvalDecision",
  "style": "expanded",
  "choices": [
    { "title": "Approve", "value": "approved" },
    { "title": "Reject", "value": "rejected" },
    { "title": "Request More Info", "value": "moreinfo" }
  ]
}
```

### Multi-Select (Checkboxes)

```json
{
  "type": "Input.ChoiceSet",
  "id": "selectedManagers",
  "style": "expanded",
  "isMultiSelect": true,
  "choices": [
    { "title": "Manager Level 1", "value": "mgr1@company.com" },
    { "title": "Manager Level 2", "value": "mgr2@company.com" },
    { "title": "CRO", "value": "cro@company.com" }
  ]
}
```

### Dynamic Choices in XAML

When building choices dynamically in UiPath:

```xml
<!-- The choices JSON must be in format: [{"title":"Display","value":"actual_value"},...] -->
<Assign DisplayName="Build Approver Choices">
  <Assign.To><OutArgument x:TypeArguments="x:String">[ApproverChoicesJSON]</OutArgument></Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String">["[{""title"":""John Doe (john@company.com)"",""value"":""john@company.com""},{""title"":""Jane Smith (jane@company.com)"",""value"":""jane@company.com""}]"]</InArgument>
  </Assign.Value>
</Assign>
```

Then inject into the card:
```
""choices"": "+ApproverChoicesJSON+"
```

**IMPORTANT**: Always provide a fallback for empty choices:
```
""choices"": "+If(String.IsNullOrEmpty(in_ApproverChoicesJSON), "[{""title"":""No options available"",""value"":""""}]", in_ApproverChoicesJSON)+"
```

---

## Input.Number - Complete Reference

```json
{
  "type": "Input.Number",
  "id": "maxCapPercent",
  "label": "Maximum Cap %",
  "min": 0,
  "max": 100,
  "value": 5,
  "placeholder": "Enter percentage"
}
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | **Required**. Unique identifier |
| `min` | number | Minimum allowed value |
| `max` | number | Maximum allowed value |
| `value` | number | Default value |
| `placeholder` | string | Placeholder text |
| `label` | string | Accessibility label (v1.3+) |
| `isRequired` | boolean | Validation requirement |
| `errorMessage` | string | Validation error message |

---

## Input.Toggle - Complete Reference

```json
{
  "type": "Input.Toggle",
  "id": "cpiInclusion",
  "title": "Include CPI Adjustment",
  "value": "true",
  "valueOn": "true",
  "valueOff": "false"
}
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | **Required**. Unique identifier |
| `title` | string | **Required**. Label text |
| `value` | string | Current value (`valueOn` or `valueOff`) |
| `valueOn` | string | Value when toggled on (default: "true") |
| `valueOff` | string | Value when toggled off (default: "false") |
| `label` | string | Accessibility label (v1.3+) |
| `isRequired` | boolean | Validation requirement |

---

## Input.Text - Complete Reference

### Single Line

```json
{
  "type": "Input.Text",
  "id": "approverEmail",
  "label": "Approver Email",
  "placeholder": "Enter email address",
  "style": "email"
}
```

### Multi-Line (Text Area)

```json
{
  "type": "Input.Text",
  "id": "comments",
  "label": "Comments",
  "isMultiline": true,
  "placeholder": "Add your comments...",
  "maxLength": 500
}
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `id` | string | **Required**. Unique identifier |
| `isMultiline` | boolean | Multi-line text area |
| `maxLength` | number | Maximum character count |
| `placeholder` | string | Placeholder text |
| `value` | string | Default value |
| `style` | string | `"text"`, `"tel"`, `"url"`, `"email"`, `"password"` |
| `label` | string | Accessibility label (v1.3+) |
| `isRequired` | boolean | Validation requirement |
| `regex` | string | Validation regex pattern |
| `errorMessage` | string | Validation error message |

---

## Container Styles

Containers can have different visual styles:

```json
{
  "type": "Container",
  "style": "emphasis",
  "items": [
    {
      "type": "TextBlock",
      "text": "This section is emphasized",
      "weight": "Bolder"
    }
  ]
}
```

### Available Styles

| Style | Description |
|-------|-------------|
| `"default"` | Standard background |
| `"emphasis"` | Subtle highlight background |
| `"good"` | Green/success background |
| `"attention"` | Red/warning background |
| `"warning"` | Yellow/caution background |
| `"accent"` | Brand color background |

---

## TextBlock Formatting

```json
{
  "type": "TextBlock",
  "text": "Important Information",
  "size": "Large",
  "weight": "Bolder",
  "color": "Accent",
  "wrap": true,
  "spacing": "Medium"
}
```

### Size Options
`"small"`, `"default"`, `"medium"`, `"large"`, `"extraLarge"`

### Weight Options
`"lighter"`, `"default"`, `"bolder"`

### Color Options
`"default"`, `"dark"`, `"light"`, `"accent"`, `"good"`, `"warning"`, `"attention"`

### Spacing Options
`"none"`, `"small"`, `"default"`, `"medium"`, `"large"`, `"extraLarge"`, `"padding"`

---

## Action.Http - Complete Reference

```json
{
  "type": "Action.Http",
  "title": "Approve",
  "method": "POST",
  "url": "https://api.example.com/approve/{token}",
  "style": "positive",
  "headers": [
    { "name": "Content-Type", "value": "application/json" }
  ],
  "body": "{\"action\":\"approved\",\"comments\":\"{{comments.value}}\"}"
}
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `title` | string | Button text |
| `method` | string | HTTP method (`"GET"`, `"POST"`) |
| `url` | string | **MUST be HTTPS** |
| `style` | string | `"default"`, `"positive"`, `"destructive"` |
| `headers` | array | HTTP headers |
| `body` | string | Request body with template variables |

### Template Variables

Use `{{inputId.value}}` to reference input values:
- `{{comments.value}}` - Value from Input.Text with id="comments"
- `{{maxCapPercent.value}}` - Value from Input.Number with id="maxCapPercent"
- `{{selectedApprover.value}}` - Value from Input.ChoiceSet with id="selectedApprover"

---

## CRITICAL: Working XAML Pattern

The **CORRECT** pattern uses `xml:space="preserve"` with multiline strings.

### Working Email Body Pattern

```xml
<Assign sap2010:Annotation.AnnotationText="https://adaptivecards.io/designer/" DisplayName="Build Email Body HTML">
  <Assign.To>
    <OutArgument x:TypeArguments="x:String">[EmailBody]</OutArgument>
  </Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String" xml:space="preserve">["&lt;html&gt;
&lt;head&gt;
  &lt;meta http-equiv=""Content-Type"" content=""text/html; charset=utf-8""&gt;
  &lt;script type=""application/adaptivecard+json""&gt;
{
  ""type"": ""AdaptiveCard"",
  ""version"": ""1.0"",
  ""originator"": ""61fed71d-3b8c-4605-8f35-d95b70ab0803"",
  ""hideOriginalBody"": true,
  ""body"": [
    {
      ""type"": ""TextBlock"",
      ""size"": ""Large"",
      ""weight"": ""Bolder"",
      ""text"": """+in_Title+"""
    },
    {
      ""type"": ""FactSet"",
      ""facts"": [
        { ""title"": ""Customer"", ""value"": """+in_CustomerName+""" },
        { ""title"": ""Amount"", ""value"": ""$"+in_Amount.ToString("N0")+""" }
      ]
    },
    {
      ""type"": ""Input.ChoiceSet"",
      ""id"": ""selectedApprover"",
      ""style"": ""compact"",
      ""choices"": "+in_ApproverChoicesJSON+"
    },
    {
      ""type"": ""Input.Text"",
      ""id"": ""comments"",
      ""label"": ""Comments"",
      ""isMultiline"": true,
      ""placeholder"": ""Add your comments...""
    }
  ],
  ""actions"": [
    {
      ""type"": ""Action.Http"",
      ""title"": ""Approve"",
      ""method"": ""POST"",
      ""url"": """+ApproveUrl+""",
      ""style"": ""positive"",
      ""headers"": [
        { ""name"": ""Content-Type"", ""value"": ""application/json"" }
      ],
      ""body"": ""{\""action\"":\""approved\"",\""selectedApprover\"":\""{{selectedApprover.value}}\"",\""comments\"":\""{{comments.value}}\""}"" 
    },
    {
      ""type"": ""Action.Http"",
      ""title"": ""Reject"",
      ""method"": ""POST"",
      ""url"": """+ApproveUrl+""",
      ""style"": ""destructive"",
      ""headers"": [
        { ""name"": ""Content-Type"", ""value"": ""application/json"" }
      ],
      ""body"": ""{\""action\"":\""rejected\"",\""comments\"":\""{{comments.value}}\""}"" 
    }
  ]
}
  &lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
  &lt;p&gt;Please open in Outlook to view the approval form.&lt;/p&gt;
&lt;/body&gt;
&lt;/html&gt;"]</InArgument>
  </Assign.Value>
</Assign>
```

---

## CRITICAL: Escaping Rules

| Context | Escape Pattern | Example |
|---------|---------------|---------|
| VB.NET string quote | `""` | `""type"": ""AdaptiveCard""` |
| Action.Http body JSON | `\""` | `""body"": ""{\""action\"":\""approved\""}""` |
| XML HTML tags | `&lt;` `&gt;` | `&lt;html&gt;` |

**WRONG**: `\\""` (double backslash) - This will NOT render the adaptive card
**CORRECT**: `\""` (single backslash) - This is the working pattern

---

## CRITICAL: String Concatenation Pattern for Dynamic Values

When injecting VB.NET expressions into the Adaptive Card JSON, use this exact pattern:

### Pattern: `"""+expression+"""`

This creates proper JSON string values with the expression result.

**CORRECT Examples:**
```
{ ""title"": ""Customer"", ""value"": """+in_CustomerName+""" }
{ ""title"": ""ACV"", ""value"": ""$"+CDec(in_ACV).ToString("N0")+""" }
{ ""title"": ""Quote Owner"", ""value"": """+If(String.IsNullOrEmpty(in_QuoteOwnerName), "N/A", in_QuoteOwnerName)+""" }
```

**WRONG Examples (will break the Adaptive Card):**
```
{ ""title"": ""Customer"", ""value"": ""+in_CustomerName+"" }     // Missing third quote
{ ""title"": ""Customer"", ""value"": """"+in_CustomerName+"""" } // Extra quotes
```

### Why This Pattern Works

In XAML with `xml:space="preserve"`:
- `"""` = Three double-quotes: closes VB string (`"`), adds JSON quote (`""`), starts new VB string
- The expression runs in VB.NET context
- `"""` = Three double-quotes: closes VB string, adds JSON quote, starts new VB string

**Result in JSON:** `"value": "ActualValue"`

### Complete FactSet Example

```xml
{
  ""type"": ""FactSet"",
  ""facts"": [
    { ""title"": ""Customer"", ""value"": """+in_CustomerName+""" },
    { ""title"": ""Quote Number"", ""value"": """+in_QuoteNumber+""" },
    { ""title"": ""Quote Owner"", ""value"": """+If(String.IsNullOrEmpty(in_QuoteOwnerName), "N/A", in_QuoteOwnerName)+""" },
    { ""title"": ""ACV"", ""value"": ""$"+CDec(in_ACV).ToString("N0")+""" },
    { ""title"": ""TCV"", ""value"": ""$"+CDec(in_TCV).ToString("N0")+""" },
    { ""title"": ""Quote Term"", ""value"": """+in_QuoteTerm.ToString+" months"" }
  ]
}
```

### Debugging Tip

If the Adaptive Card doesn't render (shows HTML fallback only), check:
1. Count the quotes around each dynamic value - should be exactly 3 on each side
2. Verify no extra quotes were added during editing
3. Test with hardcoded values first, then add dynamic expressions one at a time

---

## HTML Email Structure (Fallback Body)

The HTML `<body>` section serves as a fallback for email clients that don't support Adaptive Cards. Structure it with clear sections:

### Recommended Section Order

1. **Title** - Main heading
2. **Approval Policy** - Business rules and policy information
3. **Approval Chain** - List of approvers with names and titles
4. **Quote Details** - Data table with deal information

### HTML Body Example

```xml
&lt;body&gt;
  &lt;div style=""font-family: Segoe UI, Arial, sans-serif; max-width: 600px; margin: 0 auto;""&gt;
    &lt;h2 style=""color: #0078D4; margin-bottom: 20px;""&gt;Renewal Price Commitment Summary&lt;/h2&gt;
    
    &lt;h3 style=""color: #0078D4; margin-top: 20px;""&gt;&lt;b&gt;Approval Policy:&lt;/b&gt;&lt;/h3&gt;
    &lt;div style=""padding: 15px; background-color: #f5f5f5; border: 1px solid #ddd; border-radius: 4px;""&gt;
      &lt;p&gt;&lt;b&gt;CRO Level:&lt;/b&gt;&lt;/p&gt;
      &lt;ul&gt;
        &lt;li&gt;Policy rule 1&lt;/li&gt;
        &lt;li&gt;Policy rule 2&lt;/li&gt;
      &lt;/ul&gt;
    &lt;/div&gt;
    
    &lt;h3 style=""color: #0078D4; margin-top: 20px;""&gt;Approval Chain&lt;/h3&gt;
    &lt;p style=""padding: 10px; background-color: #e8f4fd; border: 1px solid #0078D4; border-radius: 4px;""&gt;"+ManagerChainDisplay+"&lt;/p&gt;
    
    &lt;h3 style=""color: #0078D4; margin-top: 20px;""&gt;Quote Details&lt;/h3&gt;
    &lt;table style=""width: 100%; border-collapse: collapse;""&gt;
      &lt;tr style=""background-color: #f5f5f5;""&gt;
        &lt;td style=""padding: 10px; border: 1px solid #ddd; font-weight: bold;""&gt;Customer&lt;/td&gt;
        &lt;td style=""padding: 10px; border: 1px solid #ddd;""&gt;"+in_CustomerName+"&lt;/td&gt;
      &lt;/tr&gt;
    &lt;/table&gt;
  &lt;/div&gt;
&lt;/body&gt;
```

### Building Dynamic Approval Chain Display

Use an InvokeCode activity to build the manager chain display string with HTML line breaks:

```vb
' Build manager chain display string for notification
Dim managerDisplay As New System.Text.StringBuilder()

Try
    If Not String.IsNullOrEmpty(managerChainJSON) AndAlso managerChainJSON <> "[]" Then
        Dim ManagerChainArray = Newtonsoft.Json.Linq.JArray.Parse(managerChainJSON)
        For i As Integer = 0 To ManagerChainArray.Count - 1
            Dim manager = CType(ManagerChainArray(i), Newtonsoft.Json.Linq.JObject)
            Dim level As Integer = If(manager("level") IsNot Nothing, CInt(manager("level")), i + 1)
            Dim name As String = If(manager("name") IsNot Nothing, manager("name").ToString, "")
            Dim title As String = If(manager("title") IsNot Nothing, manager("title").ToString, "")
            If i > 0 Then managerDisplay.Append("<br/>")
            managerDisplay.Append("L" & level.ToString & ": " & name & " (" & title & ")")
        Next
    Else
        managerDisplay.Append("Approval chain will be determined after submission")
    End If
Catch ex As Exception
    managerDisplay.Append("Approval chain will be determined after submission")
End Try

ManagerChainDisplay = managerDisplay.ToString()
```

**Output Example:**
```
L1: John Smith (Area Sales Director)
L2: Jane Doe (VP Sales)
L3: Bob Wilson (CRO)
```

---

## HTML Email Wrapper

Adaptive Cards must be wrapped in HTML for email delivery:

```html
<html>
<head>
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
  <script type="application/adaptivecard+json">
{
  "type": "AdaptiveCard",
  "version": "1.0",
  "originator": "YOUR-ORIGINATOR-GUID",
  "hideOriginalBody": true,
  "body": [...],
  "actions": [...]
}
  </script>
</head>
<body>
  <p>Please open in Outlook to view the approval form.</p>
</body>
</html>
```

---

## Originator GUID

The `originator` field is required for Actionable Messages in Outlook. Register your originator at:
https://outlook.office.com/connectors/oam/publish

**Current Originator**: `61fed71d-3b8c-4605-8f35-d95b70ab0803`

---

## Size Limits

- Maximum card size: **28KB**
- If card exceeds limit, fall back to simplified card with web link

---

## Best Practices

1. **Use xml:space="preserve"** - Allows multiline strings in XAML
2. **Use XML entities for HTML tags** - `&lt;` for `<`, `&gt;` for `>`
3. **Use `""` for JSON quotes** - Double quotes in VB.NET strings
4. **Use `\""` for Action.Http body** - Single backslash escaped quotes in nested JSON (NOT `\\""`)
5. **Build components separately** - FactSet, TextBlock, then concatenate
6. **Use FactSet for read-only data** - Better formatting than TextBlocks
7. **Include fallback body** - For email clients that don't support Adaptive Cards
8. **Test in Outlook** - Cards render differently in different clients
9. **Use template variables** - `{{field.value}}` for dynamic data binding
10. **ALWAYS use HTTPS for Action.Http URLs** - HTTP is blocked by Outlook
11. **Always provide fallback for dynamic choices** - Prevent empty dropdown errors
12. **Use labels for accessibility** - Add `label` property to inputs (v1.3+)

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| Adaptive card not rendering | Wrong escaping `\\""` | Use `\""` (single backslash) |
| "Target URL scheme 'http' is not allowed" | HTTP URL in Action.Http | Use HTTPS CloudFront URL |
| Card shows fallback HTML only | Malformed JSON | Check escaping, validate JSON structure |
| Buttons don't work | Missing headers or wrong body format | Include Content-Type header, use proper escaping |
| Dropdown not showing | Empty choices array | Provide fallback default choices |
| Input values not captured | Wrong template syntax | Use `{{inputId.value}}` format |

---

## Reference: Working Templates

### ApprovalCycleWorkflowTemplate
`C:\Users\DanielaRosenstein\OneDrive - Cato Networks\Documents\UiPath\ApprovalCycleWorkflowTemplate\Main.xaml`

### SALES02 Working Approval Flows (Production Reference)
`C:\UiPath\SALES02_RenewalPriceCommitment\ApprovalFlows\ApprovalFlow_SalesRep.xaml`

> **IMPORTANT**: The production-validated UiPath Studio project is at:
> `C:\UiPath\SALES02_RenewalPriceCommitment`

Always use these templates as the base for new approval workflows.

---

## Complete RevOps Form Example

A complete example with all editable fields:

```json
{
  "type": "AdaptiveCard",
  "version": "1.0",
  "originator": "61fed71d-3b8c-4605-8f35-d95b70ab0803",
  "body": [
    {
      "type": "TextBlock",
      "size": "Large",
      "weight": "Bolder",
      "text": "Renewal Price Commitment Request Preview"
    },
    {
      "type": "FactSet",
      "facts": [
        { "title": "Customer", "value": "Acme Corp" },
        { "title": "Quote Number", "value": "Q-12345" },
        { "title": "ACV", "value": "$150,000" },
        { "title": "TCV", "value": "$450,000" }
      ]
    },
    {
      "type": "TextBlock",
      "text": "Maximum Cap %",
      "weight": "Bolder",
      "spacing": "Medium"
    },
    {
      "type": "Input.Number",
      "id": "maxCapPercent",
      "value": 5,
      "min": 0,
      "max": 100
    },
    {
      "type": "Input.Toggle",
      "id": "cpiInclusion",
      "title": "Include CPI Adjustment",
      "value": "true",
      "valueOn": "true",
      "valueOff": "false"
    },
    {
      "type": "TextBlock",
      "text": "Requested Term (Months)",
      "weight": "Bolder",
      "spacing": "Medium"
    },
    {
      "type": "Input.Number",
      "id": "requestedTerm",
      "value": 36,
      "min": 12,
      "max": 60
    },
    {
      "type": "Input.Toggle",
      "id": "withinPolicy",
      "title": "Within Policy",
      "value": "true",
      "valueOn": "true",
      "valueOff": "false"
    },
    {
      "type": "Input.Toggle",
      "id": "requiresFinance",
      "title": "Requires Finance Review",
      "value": "false",
      "valueOn": "true",
      "valueOff": "false"
    },
    {
      "type": "TextBlock",
      "text": "Select Approver",
      "weight": "Bolder",
      "spacing": "Medium"
    },
    {
      "type": "Input.ChoiceSet",
      "id": "selectedApprover",
      "style": "compact",
      "choices": [
        { "title": "John Doe (john@company.com)", "value": "john@company.com" },
        { "title": "Jane Smith (jane@company.com)", "value": "jane@company.com" }
      ]
    },
    {
      "type": "Input.Text",
      "id": "revOpsComments",
      "label": "RevOps Comments",
      "isMultiline": true,
      "placeholder": "Add your comments..."
    }
  ],
  "actions": [
    {
      "type": "Action.Http",
      "title": "Approve",
      "method": "POST",
      "url": "https://api.example.com/approve/TOKEN",
      "style": "positive",
      "headers": [
        { "name": "Content-Type", "value": "application/json" }
      ],
      "body": "{\"action\":\"approved\",\"responseData\":{\"maxCapPercent\":\"{{maxCapPercent.value}}\",\"cpiInclusion\":\"{{cpiInclusion.value}}\",\"requestedTerm\":\"{{requestedTerm.value}}\",\"withinPolicy\":\"{{withinPolicy.value}}\",\"requiresFinance\":\"{{requiresFinance.value}}\",\"selectedApprover\":\"{{selectedApprover.value}}\",\"revOpsComments\":\"{{revOpsComments.value}}\"}}"
    },
    {
      "type": "Action.Http",
      "title": "Reject",
      "method": "POST",
      "url": "https://api.example.com/approve/TOKEN",
      "style": "destructive",
      "headers": [
        { "name": "Content-Type", "value": "application/json" }
      ],
      "body": "{\"action\":\"rejected\",\"responseData\":{\"revOpsComments\":\"{{revOpsComments.value}}\"}}"
    }
  ]
}
```

Use the [Adaptive Card Designer](https://adaptivecards.io/designer/) to preview and test your cards before implementing in XAML.
