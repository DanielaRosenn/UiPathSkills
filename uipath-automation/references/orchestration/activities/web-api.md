# Module: Web API Activities
**Package:** UiPath.Web.Activities (WebAPI) | **Covers:** HTTP Request, JSON/XML deserialization, SOAP

---

## HTTP Request (Modern — v2.0+)

```xml
<!-- GET request -->
<ui:HttpRequest sap2010:WorkflowViewState.IdRef="HttpGet_1"
    DisplayName="GET: Fetch Orders"
    Endpoint="[baseUrl + &quot;/api/orders?status=pending&quot;]"
    Method="GET"
    Response="[responseBody]"
    StatusCode="[httpStatus]"
    AcceptFormat="application/json">
  <ui:HttpRequest.Headers>
    <scg:Dictionary x:TypeArguments="x:String, x:String">
      <x:String x:Key="Authorization">[&quot;Bearer &quot; + accessToken]</x:String>
    </scg:Dictionary>
  </ui:HttpRequest.Headers>
</ui:HttpRequest>

<!-- POST request with JSON body -->
<ui:HttpRequest sap2010:WorkflowViewState.IdRef="HttpPost_1"
    DisplayName="POST: Create Record"
    Endpoint="[baseUrl + &quot;/api/records&quot;]"
    Method="POST"
    Body="[jsonPayload]"
    BodyFormat="application/json"
    Response="[responseBody]"
    StatusCode="[httpStatus]"/>

<!-- PUT request -->
<ui:HttpRequest Method="PUT" Endpoint="[baseUrl + &quot;/api/records/&quot; + recordId]"
    Body="[updatePayload]" BodyFormat="application/json"
    Response="[responseBody]" StatusCode="[httpStatus]"/>

<!-- DELETE request -->
<ui:HttpRequest Method="DELETE" Endpoint="[baseUrl + &quot;/api/records/&quot; + recordId]"
    Response="[responseBody]" StatusCode="[httpStatus]"/>
```

**HTTP Methods:** GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS

**Common header patterns:**
```vb
' Bearer token
"Authorization" → "Bearer " + accessToken

' Basic auth
"Authorization" → "Basic " + Convert.ToBase64String(Text.Encoding.ASCII.GetBytes(user + ":" + pass))

' API key
"X-API-Key" → apiKey
"api-key" → apiKey

' Content type (usually set via BodyFormat, but can be explicit)
"Content-Type" → "application/json"
```

---

## JSON Operations

### Deserialize JSON to Dictionary
```xml
<ui:DeserializeJson x:TypeArguments="scg:Dictionary(x:String, x:Object)"
    sap2010:WorkflowViewState.IdRef="DeserJSON_1"
    DisplayName="Parse JSON Response"
    Input="[responseBody]"
    JsonObject="[parsedDict]"/>
<!-- Access: parsedDict("key").ToString() -->
```

### Deserialize JSON Array to List
```xml
<ui:DeserializeJsonArray x:TypeArguments="x:Object"
    sap2010:WorkflowViewState.IdRef="DeserJSONArr_1"
    DisplayName="Parse JSON Array"
    Input="[jsonArrayString]"
    JsonObject="[itemList]"/>
```

### Serialize to JSON (Invoke Code pattern)
```vb
' In Invoke Code:
json = Newtonsoft.Json.JsonConvert.SerializeObject(myObject)

' Serialize Dictionary:
Dim payload As New Dictionary(Of String, Object) From {
    {"invoiceId", invoiceId},
    {"amount", amount},
    {"status", "pending"}
}
json = Newtonsoft.Json.JsonConvert.SerializeObject(payload)
```

### Access nested JSON values (Invoke Code)
```vb
' Parse deeply nested JSON:
Dim root = Newtonsoft.Json.Linq.JObject.Parse(responseBody)
Dim value As String = root("data")("items")(0)("id").ToString()
Dim items = root("data")("items").ToObject(Of List(Of Dictionary(Of String, Object)))()
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

### Execute XPath
```xml
<ui:ExecuteXPath sap2010:WorkflowViewState.IdRef="XPath_1"
    DisplayName="Extract XML Value"
    XmlDocument="[xmlDoc]"
    XPath="&quot;//Invoice/InvoiceNumber/text()&quot;"
    Result="[invoiceNumber]"/>
```

### Get XML Nodes
```xml
<ui:GetXmlNodes sap2010:WorkflowViewState.IdRef="GetNodes_1"
    DisplayName="Get All Invoice Nodes"
    XmlDocument="[xmlDoc]"
    XPath="&quot;//Invoice&quot;"
    XmlNodeList="[invoiceNodes]"/>
```

---

## SOAP Request
```xml
<ui:SOAPRequest sap2010:WorkflowViewState.IdRef="SOAP_1"
    DisplayName="SOAP: Get Invoice Status"
    ServiceUrl="[&quot;https://api.example.com/InvoiceService.asmx&quot;]"
    SOAPVersion="SOAP11"
    MethodName="&quot;GetInvoiceStatus&quot;"
    TargetNamespace="&quot;http://example.com/invoice&quot;"
    Response="[soapResponse]">
  <ui:SOAPRequest.Parameters>
    <scg:Dictionary x:TypeArguments="x:String, x:String">
      <x:String x:Key="InvoiceID">[invoiceId]</x:String>
    </scg:Dictionary>
  </ui:SOAPRequest.Parameters>
</ui:SOAPRequest>
```

---

## REST API Integration Pattern (full workflow)

```xml
<Sequence DisplayName="Fetch and Process API Data">
  <Sequence.Variables>
    <Variable x:TypeArguments="x:String" Name="responseBody"/>
    <Variable x:TypeArguments="x:Int32" Name="httpStatus"/>
    <Variable x:TypeArguments="scg:Dictionary(x:String,x:Object)" Name="parsedData"/>
    <Variable x:TypeArguments="x:String" Name="accessToken"/>
  </Sequence.Variables>

  <!-- Step 1: Get token -->
  <ui:HttpRequest DisplayName="POST: Get OAuth Token"
      Endpoint="[tokenUrl]" Method="POST"
      Body="[&quot;grant_type=client_credentials&amp;client_id=&quot; + clientId + &quot;&amp;client_secret=&quot; + clientSecret]"
      BodyFormat="application/x-www-form-urlencoded"
      Response="[responseBody]" StatusCode="[httpStatus]"/>

  <ui:DeserializeJson x:TypeArguments="scg:Dictionary(x:String,x:Object)"
      Input="[responseBody]" JsonObject="[parsedData]"/>
  <Assign>[accessToken = parsedData("access_token").ToString()]</Assign>

  <!-- Step 2: Call API with token -->
  <TryCatch DisplayName="API Call with Retry">
    <TryCatch.Try>
      <ui:HttpRequest DisplayName="GET: Fetch Data"
          Endpoint="[apiEndpoint]" Method="GET"
          Response="[responseBody]" StatusCode="[httpStatus]">
        <ui:HttpRequest.Headers>
          <scg:Dictionary x:TypeArguments="x:String,x:String">
            <x:String x:Key="Authorization">[&quot;Bearer &quot; + accessToken]</x:String>
          </scg:Dictionary>
        </ui:HttpRequest.Headers>
      </ui:HttpRequest>

      <If Condition="[httpStatus &lt;&gt; 200]">
        <If.Then>
          <Throw>
            <Throw.Exception>
              <InArgument x:TypeArguments="s:Exception">
                [New Exception("API error: HTTP " + httpStatus.ToString() + " - " + responseBody)]
              </InArgument>
            </Throw.Exception>
          </Throw>
        </If.Then>
      </If>
    </TryCatch.Try>
    <TryCatch.Catches>
      <Catch x:TypeArguments="s:Exception">
        <ActivityAction x:TypeArguments="s:Exception">
          <ActivityAction.Argument>
            <DelegateInArgument x:TypeArguments="s:Exception" Name="ex"/>
          </ActivityAction.Argument>
          <ui:LogMessage Level="Error" Message="[&quot;API call failed: &quot; + ex.Message]"/>
        </ActivityAction>
      </Catch>
    </TryCatch.Catches>
  </TryCatch>
</Sequence>
```
