# Module: Web API Activities
**Package:** UiPath.Web.Activities (WebAPI) | **Covers:** HTTP Request, JSON/XML, SOAP, Deserialize, OAuth

---

## HTTP Request (Modern — v2.0+)

```xml
<ui:HttpRequest sap2010:WorkflowViewState.IdRef="HttpReq_1"
    DisplayName="POST Create Invoice"
    Endpoint="[baseUrl + &quot;/api/v2/invoices&quot;]"
    Method="POST"
    Body="[requestJson]"
    BodyFormat="application/json"
    Response="[responseBody]"
    StatusCode="[statusCode]"
    AcceptFormat="application/json"
    Timeout="[TimeSpan.FromSeconds(30)]">
  <ui:HttpRequest.Headers>
    <scg:Dictionary x:TypeArguments="x:String, x:String">
      <x:String x:Key="Authorization">[&quot;Bearer &quot; + accessToken]</x:String>
      <x:String x:Key="X-Request-ID">[Guid.NewGuid().ToString()]</x:String>
    </scg:Dictionary>
  </ui:HttpRequest.Headers>
</ui:HttpRequest>
```

HTTP methods: `GET` `POST` `PUT` `PATCH` `DELETE` `HEAD` `OPTIONS`

### Authentication patterns

**Bearer token:**
```vb
"Bearer " + accessToken
```

**Basic auth (build manually):**
```vb
"Basic " + Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(username + ":" + password))
```

**OAuth2 token fetch (via HTTP Request):**
```xml
<ui:HttpRequest DisplayName="Get OAuth Token"
    Endpoint="[tokenUrl]" Method="POST"
    Body="[&quot;grant_type=client_credentials&amp;client_id=&quot; + clientId + &quot;&amp;client_secret=&quot; + clientSecret]"
    BodyFormat="application/x-www-form-urlencoded"
    Response="[tokenResponse]" StatusCode="[tokenStatus]"/>
<ui:DeserializeJson x:TypeArguments="scg:Dictionary(x:String,x:Object)"
    Input="[tokenResponse]" JsonObject="[tokenData]"/>
<Assign DisplayName="Extract Token">
  <Assign.To>[accessToken]</Assign.To>
  <Assign.Value>[tokenData("access_token").ToString()]</Assign.Value>
</Assign>
```

---

## JSON Operations

### Serialize to JSON (Invoke Code)
```vb
' In Invoke Code:
Dim payload As New Dictionary(Of String, Object) From {
    {"invoiceId", invoiceId},
    {"amount", amount},
    {"date", DateTime.Now.ToString("yyyy-MM-dd")},
    {"items", New Object() {"item1", "item2"}}
}
requestJson = Newtonsoft.Json.JsonConvert.SerializeObject(payload)
```

### Deserialize JSON response
```xml
<!-- To Dictionary -->
<ui:DeserializeJson x:TypeArguments="scg:Dictionary(x:String, x:Object)"
    sap2010:WorkflowViewState.IdRef="DeserJSON_1"
    DisplayName="Parse Response"
    Input="[responseBody]"
    JsonObject="[parsedResponse]"/>

<!-- To List -->
<ui:DeserializeJson x:TypeArguments="scg:List(scg:Dictionary(x:String, x:Object))"
    DisplayName="Parse Array Response"
    Input="[responseBody]"
    JsonObject="[itemsList]"/>
```

### Access nested JSON (Invoke Code)
```vb
' Parse deeply nested response
Dim root = Newtonsoft.Json.Linq.JObject.Parse(responseBody)
Dim invoiceId As String = root("data")("invoice")("id").ToString()
Dim items = root("data")("items").ToObject(Of List(Of Dictionary(Of String, Object)))()
totalAmount = CDbl(root("data")("totals")("amount"))
```

---

## XML Operations

### Deserialize XML
```xml
<ui:DeserializeXml sap2010:WorkflowViewState.IdRef="DeserXML_1"
    DisplayName="Parse XML Response"
    Input="[xmlString]"
    XmlDocument="[xmlDoc]"/>
```

### Execute XPath query
```xml
<ui:ExecuteXPath sap2010:WorkflowViewState.IdRef="XPath_1"
    DisplayName="Get Invoice ID from XML"
    XmlDocument="[xmlDoc]"
    XPath="&quot;//Invoice/InvoiceID/text()&quot;"
    Result="[invoiceIdNode]"/>
```

### Parse XML in Invoke Code
```vb
Dim doc As New System.Xml.XmlDocument()
doc.LoadXml(xmlString)
Dim node As System.Xml.XmlNode = doc.SelectSingleNode("//Invoice/Amount")
amount = CDbl(node.InnerText)

' Get all nodes
Dim nodes = doc.SelectNodes("//LineItem")
For Each node As System.Xml.XmlNode In nodes
    ' process each node
Next
```

---

## SOAP Request
```xml
<ui:SoapRequest sap2010:WorkflowViewState.IdRef="Soap_1"
    DisplayName="Call SOAP Service"
    ServiceDescription="[wsdlUrl]"
    Endpoint="[soapEndpoint]"
    MethodName="&quot;GetInvoiceStatus&quot;"
    Response="[soapResponse]">
  <ui:SoapRequest.Parameters>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Object x:Key="InvoiceID">[invoiceId]</x:Object>
    </scg:Dictionary>
  </ui:SoapRequest.Parameters>
</ui:SoapRequest>
```

---

## REST API Pattern with Retry and Status Check

```xml
<Sequence DisplayName="Robust API Call">
  <Assign>[retryCount = 0]</Assign>
  <Assign>[isSuccess = False]</Assign>

  <While Condition="[Not isSuccess AndAlso retryCount &lt; 3]">
    <Sequence>
      <TryCatch DisplayName="Try API Call">
        <TryCatch.Try>
          <Sequence>
            <ui:HttpRequest DisplayName="Call API"
                Endpoint="[apiUrl]" Method="GET"
                Response="[responseBody]" StatusCode="[httpStatus]"
                Timeout="[TimeSpan.FromSeconds(30)]">
              <ui:HttpRequest.Headers>
                <scg:Dictionary x:TypeArguments="x:String, x:String">
                  <x:String x:Key="Authorization">[&quot;Bearer &quot; + token]</x:String>
                </scg:Dictionary>
              </ui:HttpRequest.Headers>
            </ui:HttpRequest>
            <If Condition="[httpStatus = 200]">
              <If.Then>
                <Assign>[isSuccess = True]</Assign>
              </If.Then>
              <If.Else>
                <If Condition="[httpStatus = 429 OrElse httpStatus &gt;= 500]">
                  <!-- Rate limit or server error — retry -->
                  <If.Then>
                    <Sequence>
                      <Assign>[retryCount = retryCount + 1]</Assign>
                      <Delay Duration="[TimeSpan.FromSeconds(5 * retryCount)]"/>
                    </Sequence>
                  </If.Then>
                  <If.Else>
                    <!-- 4xx client error — don't retry -->
                    <Throw>
                      <Throw.Exception>
                        <InArgument x:TypeArguments="ui:BusinessRuleException">
                          [New UiPath.Core.Activities.RobotException.BusinessRuleException("API error " + httpStatus.ToString() + ": " + responseBody)]
                        </InArgument>
                      </Throw.Exception>
                    </Throw>
                  </If.Else>
                </If>
              </If.Else>
            </If>
          </Sequence>
        </TryCatch.Try>
        <TryCatch.Catches>
          <Catch x:TypeArguments="s:Exception">
            <ActivityAction x:TypeArguments="s:Exception">
              <ActivityAction.Argument>
                <DelegateInArgument x:TypeArguments="s:Exception" Name="ex"/>
              </ActivityAction.Argument>
              <Sequence>
                <Assign>[retryCount = retryCount + 1]</Assign>
                <ui:LogMessage Level="Warn" Message="[&quot;API attempt &quot; + retryCount.ToString() + &quot; failed: &quot; + ex.Message]"/>
                <If Condition="[retryCount &gt;= 3]"><If.Then><Rethrow/></If.Then></If>
                <Delay Duration="[TimeSpan.FromSeconds(5)]"/>
              </Sequence>
            </ActivityAction>
          </Catch>
        </TryCatch.Catches>
      </TryCatch>
    </Sequence>
  </While>
</Sequence>
```

---

# Module: Debugging in Studio

## Breakpoints
- Click left margin of activity → set breakpoint (red dot)
- `F9` → toggle breakpoint on selected activity
- Conditional breakpoints: right-click dot → Edit → add condition expression

## Debug Run Modes
- `F5` → Run with Debug (full execution, stops at breakpoints)
- `F6` → Step Into (enter child workflows/activities)
- `F7` → Step Over (execute without entering scope)
- `F8` → Step Out (exit current scope)
- `F11` → Continue to next breakpoint
- `Ctrl+Shift+F5` → Restart debug session

## Debug Panels

### Watch Panel (manual variable monitoring)
- Add expressions: `myVariable`, `myList.Count`, `dt.Rows.Count`
- Updates live at each debug step

### Locals Panel
- Shows all in-scope variables automatically
- Expanding DataTables shows row/column structure

### Immediate Panel
- Type and evaluate any VB.NET expression at current debug position
- Modify variable values mid-execution: `myInt = 42`

### Call Stack Panel
- Shows nested Invoke Workflow File chain
- Click any frame to jump to that context

## Logging Strategy for Debugging
```vb
' Development: use Verbose level (won't appear in production Orchestrator logs)
LogMessage(Level.Trace, "Loop iteration " & counter & ": " & currentItem)

' After debugging, promote to Info for key milestones only:
LogMessage(Level.Info, "Processing batch " & batchId & " — " & count & " items")
```

## Common Debug Scenarios

### "Selector not found" intermittent failure
1. Set breakpoint just before the failing activity
2. Run debug — when paused, manually use UI Explorer on the live app
3. Compare live selector with stored selector — identify what changed
4. Check `WaitForReady` setting — try `Interactive` instead of `Complete`

### "DataTable is Nothing" crash
1. Watch the DataTable variable after each Read Range
2. Check sheet name and range — empty sheet returns Nothing
3. Add null check: `If IsNothing(dtData) OrElse dtData.Rows.Count = 0`

### Infinite loop
1. Set breakpoint inside the loop body
2. Watch loop condition variable
3. Check that the exit condition is actually being set

### Slow performance
1. Use `Verbose` log level during debug run
2. Look at timestamps in Output panel to identify slow activities
3. Common culprits: `WaitForReady=Complete`, missing `SimulateClick`, no connection pooling
