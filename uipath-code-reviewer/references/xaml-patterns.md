# XAML Patterns Reference

Good vs bad XAML patterns for UiPath code review.

## Variable Declarations

### Bad: No Type Prefix

```xml
<Variable x:TypeArguments="x:String" Name="filename" />
<Variable x:TypeArguments="x:Int32" Name="counter" />
<Variable x:TypeArguments="sd:DataTable" Name="data" />
```

### Good: With Type Prefix

```xml
<Variable x:TypeArguments="x:String" Name="strFileName" />
<Variable x:TypeArguments="x:Int32" Name="intCounter" />
<Variable x:TypeArguments="sd:DataTable" Name="dtData" />
```

---

## Argument Declarations

### Bad: No Direction Prefix

```xml
<x:Property Name="FilePath" Type="InArgument(x:String)" />
<x:Property Name="result" Type="OutArgument(x:String)" />
<x:Property Name="DataTable" Type="InOutArgument(sd:DataTable)" />
```

### Good: With Direction Prefix

```xml
<x:Property Name="in_FilePath" Type="InArgument(x:String)" />
<x:Property Name="out_Result" Type="OutArgument(x:String)" />
<x:Property Name="io_DataTable" Type="InOutArgument(sd:DataTable)" />
```

---

## Activity DisplayNames

### Bad: Generic Names

```xml
<Assign DisplayName="Assign">
<If DisplayName="If">
<Sequence DisplayName="Sequence">
<TryCatch DisplayName="Try Catch">
<ForEach DisplayName="For Each">
```

### Good: Descriptive Names

```xml
<Assign DisplayName="Set Transaction Status to Complete">
<If DisplayName="Check If File Exists">
<Sequence DisplayName="Process Invoice Data">
<TryCatch DisplayName="Handle API Call Errors">
<ForEach DisplayName="Process Each Invoice Row">
```

---

## Error Handling

### Bad: No Error Handling

```xml
<Sequence DisplayName="Process Data">
  <ui:HttpClient Endpoint="[apiUrl]" Response="[response]" />
  <ui:DeserializeJson JsonString="[response]" JsonObject="[joResult]" />
  <ui:ExcelWriteRange DataTable="[dtData]" />
</Sequence>
```

### Bad: Empty Catch Block

```xml
<TryCatch DisplayName="Handle Errors">
  <TryCatch.Try>
    <ui:HttpClient Endpoint="[apiUrl]" Response="[response]" />
  </TryCatch.Try>
  <TryCatch.Catches>
    <Catch x:TypeArguments="s:Exception">
      <ActivityAction x:TypeArguments="s:Exception">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="s:Exception" Name="exception" />
        </ActivityAction.Argument>
        <!-- Empty! Swallowing exception -->
      </ActivityAction>
    </Catch>
  </TryCatch.Catches>
</TryCatch>
```

### Bad: Only Generic Exception

```xml
<TryCatch.Catches>
  <Catch x:TypeArguments="s:Exception">
    <!-- Only catches generic Exception -->
  </Catch>
</TryCatch.Catches>
```

### Good: Proper Error Handling

```xml
<TryCatch DisplayName="Handle API Call Errors">
  <TryCatch.Try>
    <Sequence DisplayName="API Operations">
      <ui:HttpClient Endpoint="[apiUrl]" Response="[response]" StatusCode="[statusCode]" />
      <If Condition="[statusCode &lt;&gt; 200]">
        <If.Then>
          <Throw>
            <Throw.Exception>
              <InArgument x:TypeArguments="s:Exception">[New Exception(&quot;API returned &quot; + statusCode.ToString)]</InArgument>
            </Throw.Exception>
          </Throw>
        </If.Then>
      </If>
      <ui:DeserializeJson JsonString="[response]" JsonObject="[joResult]" />
    </Sequence>
  </TryCatch.Try>
  <TryCatch.Catches>
    <Catch x:TypeArguments="s:TimeoutException">
      <ActivityAction x:TypeArguments="s:TimeoutException">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="s:TimeoutException" Name="timeoutEx" />
        </ActivityAction.Argument>
        <Sequence DisplayName="Handle Timeout">
          <ui:LogMessage Level="Warn" Message="[&quot;API timeout: &quot; + timeoutEx.Message]" />
          <Rethrow />
        </Sequence>
      </ActivityAction>
    </Catch>
    <Catch x:TypeArguments="s:Exception">
      <ActivityAction x:TypeArguments="s:Exception">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="s:Exception" Name="exception" />
        </ActivityAction.Argument>
        <Sequence DisplayName="Handle General Error">
          <ui:LogMessage Level="Error" Message="[&quot;API error: &quot; + exception.Message]" />
          <Rethrow />
        </Sequence>
      </ActivityAction>
    </Catch>
  </TryCatch.Catches>
</TryCatch>
```

---

## Retry Logic

### Bad: No Retry for Transient Operations

```xml
<ui:Click DisplayName="Click Submit">
  <ui:Click.Target>
    <ui:Target Selector="&lt;webctrl tag='BUTTON' id='submit'/&gt;" />
  </ui:Click.Target>
</ui:Click>
```

### Good: With Retry Scope

```xml
<ui:RetryScope DisplayName="Retry Click Submit" NumberOfRetries="3" RetryInterval="00:00:02">
  <ui:RetryScope.ActivityBody>
    <ActivityAction>
      <ui:Click DisplayName="Click Submit" Timeout="10000">
        <ui:Click.Target>
          <ui:Target Selector="&lt;webctrl tag='BUTTON' id='submit'/&gt;" />
        </ui:Click.Target>
      </ui:Click>
    </ActivityAction>
  </ui:RetryScope.ActivityBody>
</ui:RetryScope>
```

---

## Credentials

### Bad: Hardcoded Password

```xml
<Assign DisplayName="Set Password">
  <Assign.To>
    <OutArgument x:TypeArguments="x:String">[strPassword]</OutArgument>
  </Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String">["MySecretPassword123!"]</InArgument>
  </Assign.Value>
</Assign>
```

### Bad: Password as String Variable

```xml
<Variable x:TypeArguments="x:String" Name="strPassword" />
```

### Good: Using Orchestrator Credential

```xml
<Variable x:TypeArguments="s:Security.SecureString" Name="secPassword" />

<ui:GetRobotCredential 
  CredentialAssetName="SystemCredential" 
  DisplayName="Get System Credential"
  Username="[strUsername]" 
  Password="[secPassword]" />
```

### Good: Using GetSecret for API Keys

```xml
<ui:GetSecret 
  AssetName="APIKey" 
  DisplayName="Get API Key"
  FolderPath="Shared"
  Secret="[secAPIKey]" />

<Assign DisplayName="Convert to String">
  <Assign.To>
    <OutArgument x:TypeArguments="x:String">[strAPIKey]</OutArgument>
  </Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String">[New System.Net.NetworkCredential(String.Empty, secAPIKey).Password]</InArgument>
  </Assign.Value>
</Assign>
```

---

## File Paths

### Bad: Hardcoded Absolute Path

```xml
<ui:ReadTextFile 
  FileName="C:\Users\John\Documents\input.txt" 
  Content="[strContent]" />
```

### Good: Using Config or Arguments

```xml
<ui:ReadTextFile 
  FileName="[in_ConfigPath + &quot;\input.txt&quot;]" 
  Content="[strContent]" />
```

### Good: Using Environment Variable

```xml
<ui:ReadTextFile 
  FileName="[Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments) + &quot;\input.txt&quot;]" 
  Content="[strContent]" />
```

---

## Logging

### Bad: No Logging

```xml
<Sequence DisplayName="Process Transaction">
  <ui:HttpClient Endpoint="[apiUrl]" Response="[response]" />
  <ui:ExcelWriteRange DataTable="[dtData]" />
</Sequence>
```

### Bad: Logging Sensitive Data

```xml
<ui:LogMessage Level="Info" Message="[&quot;Password: &quot; + strPassword]" />
<ui:LogMessage Level="Info" Message="[&quot;API Key: &quot; + strAPIKey]" />
```

### Good: Appropriate Logging

```xml
<Sequence DisplayName="Process Transaction">
  <ui:LogMessage Level="Info" Message="[&quot;Starting transaction processing for ID: &quot; + strTransactionId]" />
  
  <ui:HttpClient Endpoint="[apiUrl]" Response="[response]" StatusCode="[statusCode]" />
  <ui:LogMessage Level="Info" Message="[&quot;API call completed with status: &quot; + statusCode.ToString]" />
  
  <ui:ExcelWriteRange DataTable="[dtData]" />
  <ui:LogMessage Level="Info" Message="[&quot;Data written to Excel. Rows: &quot; + dtData.Rows.Count.ToString]" />
  
  <ui:LogMessage Level="Info" Message="[&quot;Transaction &quot; + strTransactionId + &quot; completed successfully&quot;]" />
</Sequence>
```

---

## Delays

### Bad: Hardcoded Delay

```xml
<Delay Duration="[TimeSpan.FromSeconds(10)]" DisplayName="Wait for page to load" />
<ui:Click Target="[selector]" />
```

### Good: Element Wait

```xml
<ui:WaitElementVanish 
  DisplayName="Wait for Loading Spinner"
  Timeout="30000">
  <ui:WaitElementVanish.Target>
    <ui:Target Selector="&lt;webctrl class='loading-spinner'/&gt;" />
  </ui:WaitElementVanish.Target>
</ui:WaitElementVanish>

<ui:Click Target="[selector]" Timeout="10000" />
```

---

## Selectors

### Bad: Too Specific (Volatile)

```xml
<ui:Target Selector="&lt;html title='Dashboard - Session 12345 - John'/&gt;&lt;webctrl idx='3' tableRow='5'/&gt;" />
```

### Bad: Using Index Only

```xml
<ui:Target Selector="&lt;html/&gt;&lt;webctrl idx='7'/&gt;" />
```

### Good: Stable Attributes

```xml
<ui:Target Selector="&lt;html title='Dashboard*'/&gt;&lt;webctrl tag='BUTTON' id='submit' /&gt;" />
```

### Good: Using aaname or innertext

```xml
<ui:Target Selector="&lt;html/&gt;&lt;webctrl tag='A' aaname='View Details' /&gt;" />
```

---

## Input Validation

### Bad: No Validation

```xml
<Sequence DisplayName="Process File">
  <ui:ReadTextFile FileName="[in_FilePath]" Content="[strContent]" />
</Sequence>
```

### Good: With Validation

```xml
<Sequence DisplayName="Process File">
  <!-- Validate input argument -->
  <If Condition="[String.IsNullOrWhiteSpace(in_FilePath)]" DisplayName="Check FilePath Not Empty">
    <If.Then>
      <Throw DisplayName="Throw Missing FilePath">
        <Throw.Exception>
          <InArgument x:TypeArguments="s:Exception">[New ArgumentException(&quot;in_FilePath is required&quot;)]</InArgument>
        </Throw.Exception>
      </Throw>
    </If.Then>
  </If>
  
  <!-- Check file exists -->
  <ui:PathExists Path="[in_FilePath]" PathType="File" Exists="[boolFileExists]" DisplayName="Check File Exists" />
  <If Condition="[Not boolFileExists]" DisplayName="Validate File Exists">
    <If.Then>
      <Throw DisplayName="Throw File Not Found">
        <Throw.Exception>
          <InArgument x:TypeArguments="s:Exception">[New IO.FileNotFoundException(&quot;File not found: &quot; + in_FilePath)]</InArgument>
        </Throw.Exception>
      </Throw>
    </If.Then>
  </If>
  
  <!-- Now safe to read -->
  <ui:ReadTextFile FileName="[in_FilePath]" Content="[strContent]" DisplayName="Read Input File" />
</Sequence>
```

---

## HTTP Requests

### Bad: No Status Check

```xml
<ui:HttpClient Endpoint="[apiUrl]" Response="[response]" />
<ui:DeserializeJson JsonString="[response]" JsonObject="[joResult]" />
```

### Good: With Status Check

```xml
<ui:HttpClient 
  Endpoint="[apiUrl]" 
  Response="[response]" 
  StatusCode="[intStatusCode]"
  DisplayName="Call API" />

<If Condition="[intStatusCode &lt; 200 OrElse intStatusCode &gt;= 300]" DisplayName="Check HTTP Status">
  <If.Then>
    <Sequence DisplayName="Handle HTTP Error">
      <ui:LogMessage Level="Error" Message="[&quot;API returned status &quot; + intStatusCode.ToString + &quot;: &quot; + response]" />
      <Throw DisplayName="Throw HTTP Error">
        <Throw.Exception>
          <InArgument x:TypeArguments="s:Exception">[New Exception(&quot;API call failed with status &quot; + intStatusCode.ToString)]</InArgument>
        </Throw.Exception>
      </Throw>
    </Sequence>
  </If.Then>
</If>

<ui:DeserializeJson JsonString="[response]" JsonObject="[joResult]" DisplayName="Parse Response" />
```

---

## DataTable Operations

### Bad: Loop with Filter Inside

```xml
<ForEach x:TypeArguments="sd:DataRow" DisplayName="Process Rows" Values="[dtData.Rows]">
  <ActivityAction x:TypeArguments="sd:DataRow">
    <ActivityAction.Argument>
      <DelegateInArgument x:TypeArguments="sd:DataRow" Name="row" />
    </ActivityAction.Argument>
    <If Condition="[row(&quot;Status&quot;).ToString = &quot;Active&quot;]">
      <If.Then>
        <!-- Process only active rows -->
      </If.Then>
    </If>
  </ActivityAction>
</ForEach>
```

### Good: Filter First with LINQ

```xml
<Assign DisplayName="Filter Active Rows">
  <Assign.To>
    <OutArgument x:TypeArguments="sd:DataTable">[dtActiveRows]</OutArgument>
  </Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="sd:DataTable">[dtData.AsEnumerable().Where(Function(r) r(&quot;Status&quot;).ToString = &quot;Active&quot;).CopyToDataTable()]</InArgument>
  </Assign.Value>
</Assign>

<ForEach x:TypeArguments="sd:DataRow" DisplayName="Process Active Rows" Values="[dtActiveRows.Rows]">
  <ActivityAction x:TypeArguments="sd:DataRow">
    <ActivityAction.Argument>
      <DelegateInArgument x:TypeArguments="sd:DataRow" Name="row" />
    </ActivityAction.Argument>
    <!-- Process row -->
  </ActivityAction>
</ForEach>
```

---

## Workflow Complexity

### Bad: Everything in One Workflow

```xml
<!-- Main.xaml with 100+ activities, 8 levels of nesting -->
<Sequence DisplayName="Main">
  <If>
    <If.Then>
      <Sequence>
        <If>
          <If.Then>
            <TryCatch>
              <TryCatch.Try>
                <ForEach>
                  <!-- More nesting... -->
                </ForEach>
              </TryCatch.Try>
            </TryCatch>
          </If.Then>
        </If>
      </Sequence>
    </If.Then>
  </If>
</Sequence>
```

### Good: Modular Workflows

```xml
<!-- Main.xaml - orchestration only -->
<Sequence DisplayName="Main">
  <ui:InvokeWorkflowFile 
    WorkflowFileName="Workflows\InitAllSettings.xaml" 
    DisplayName="Initialize Settings">
    <ui:InvokeWorkflowFile.Arguments>
      <OutArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)" x:Key="out_Config">[dictConfig]</OutArgument>
    </ui:InvokeWorkflowFile.Arguments>
  </ui:InvokeWorkflowFile>
  
  <ui:InvokeWorkflowFile 
    WorkflowFileName="Workflows\GetTransactionData.xaml" 
    DisplayName="Get Transaction Data">
    <ui:InvokeWorkflowFile.Arguments>
      <InArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)" x:Key="in_Config">[dictConfig]</InArgument>
      <OutArgument x:TypeArguments="sd:DataTable" x:Key="out_TransactionData">[dtTransactions]</OutArgument>
    </ui:InvokeWorkflowFile.Arguments>
  </ui:InvokeWorkflowFile>
  
  <ui:InvokeWorkflowFile 
    WorkflowFileName="Workflows\ProcessTransactions.xaml" 
    DisplayName="Process Transactions">
    <ui:InvokeWorkflowFile.Arguments>
      <InArgument x:TypeArguments="sd:DataTable" x:Key="in_TransactionData">[dtTransactions]</InArgument>
      <InArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)" x:Key="in_Config">[dictConfig]</InArgument>
    </ui:InvokeWorkflowFile.Arguments>
  </ui:InvokeWorkflowFile>
</Sequence>
```

---

## Annotations

### Bad: No Documentation

```xml
<Sequence DisplayName="Process Invoice">
  <!-- No explanation of what this does -->
</Sequence>
```

### Good: With Annotations

```xml
<Sequence DisplayName="Process Invoice" sap2010:WorkflowViewState.IdRef="Sequence_1">
  <sap2010:Annotation.AnnotationText>
Process a single invoice from the queue.

Input Arguments:
- in_InvoicePath: Full path to the invoice PDF
- in_Config: Configuration dictionary

Output Arguments:
- out_ProcessingStatus: "Success" or "Failed"
- out_ErrorMessage: Error details if failed

Business Rules:
- Invoice amount must be > 0
- Vendor must exist in master data
- Due date cannot be in the past
  </sap2010:Annotation.AnnotationText>
  
  <!-- Workflow activities -->
</Sequence>
```

---

## Queue Operations

### Bad: No Transaction Status

```xml
<ui:GetTransactionItem QueueName="InvoiceQueue" TransactionItem="[queueItem]" />
<!-- Process item -->
<!-- No SetTransactionStatus! -->
```

### Good: Always Set Status

```xml
<ui:GetTransactionItem 
  QueueName="InvoiceQueue" 
  TransactionItem="[queueItem]" 
  DisplayName="Get Next Invoice" />

<TryCatch DisplayName="Process with Status Update">
  <TryCatch.Try>
    <Sequence DisplayName="Process Invoice">
      <!-- Processing logic -->
      
      <ui:SetTransactionStatus 
        TransactionItem="[queueItem]" 
        Status="Successful"
        DisplayName="Set Success Status" />
    </Sequence>
  </TryCatch.Try>
  <TryCatch.Catches>
    <Catch x:TypeArguments="ui:BusinessRuleException">
      <ActivityAction x:TypeArguments="ui:BusinessRuleException">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="ui:BusinessRuleException" Name="breException" />
        </ActivityAction.Argument>
        <ui:SetTransactionStatus 
          TransactionItem="[queueItem]" 
          Status="Failed"
          ErrorType="Business"
          Reason="[breException.Message]"
          DisplayName="Set Business Failure" />
      </ActivityAction>
    </Catch>
    <Catch x:TypeArguments="s:Exception">
      <ActivityAction x:TypeArguments="s:Exception">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="s:Exception" Name="exception" />
        </ActivityAction.Argument>
        <ui:SetTransactionStatus 
          TransactionItem="[queueItem]" 
          Status="Failed"
          ErrorType="Application"
          Reason="[exception.Message]"
          DisplayName="Set Application Failure" />
      </ActivityAction>
    </Catch>
  </TryCatch.Catches>
</TryCatch>
```
