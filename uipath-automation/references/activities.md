# UiPath Activities Reference

Comprehensive catalog of UiPath activities with XAML syntax. **Routing:** [SKILL.md](../SKILL.md) lists where this file fits versus `references/orchestration/` (read each topic once; this file is the **XAML catalog**).

For **authoritative** activity lists, properties, and compatibility, cross-check [Workflow activities](https://docs.uipath.com/activities/other/latest/workflow/workflow-activities) and pack-specific pages on [docs.uipath.com](https://docs.uipath.com).

## Table of Contents
1. [Control Flow](#control-flow)
2. [Data Manipulation](#data-manipulation)
3. [Excel Activities](#excel-activities)
4. [UI Automation](#ui-automation)
5. [Web Activities](#web-activities)
6. [File & Folder](#file--folder)
7. [Database](#database)
8. [Error Handling](#error-handling)
9. [Orchestrator](#orchestrator)
10. [Mail Activities](#mail-activities)

---

## Control Flow

### Assign
Sets a value to a variable or argument.

```xml
<Assign DisplayName="Set Status">
  <Assign.To>
    <OutArgument x:TypeArguments="x:String">[strStatus]</OutArgument>
  </Assign.To>
  <Assign.Value>
    <InArgument x:TypeArguments="x:String">["Complete"]</InArgument>
  </Assign.Value>
</Assign>
```

### Multiple Assign
Sets multiple values in a single activity.

```xml
<ui:MultipleAssign DisplayName="Initialize Variables">
  <ui:MultipleAssign.AssignOperations>
    <ui:AssignOperation To="[strStatus]" Value="[&quot;Pending&quot;]" />
    <ui:AssignOperation To="[intCount]" Value="[0]" />
    <ui:AssignOperation To="[boolSuccess]" Value="[False]" />
  </ui:MultipleAssign.AssignOperations>
</ui:MultipleAssign>
```

### If
Conditional branching.

```xml
<If Condition="[intValue > 100]" DisplayName="Check Threshold">
  <If.Then>
    <Sequence DisplayName="High Value">
      <ui:LogMessage Level="Warn" Message="Value exceeds threshold" />
    </Sequence>
  </If.Then>
  <If.Else>
    <Sequence DisplayName="Normal Value">
      <ui:LogMessage Level="Info" Message="Value is within range" />
    </Sequence>
  </If.Else>
</If>
```

### Switch
Multi-way branching based on expression value.

```xml
<Switch x:TypeArguments="x:String" Expression="[strAction]" DisplayName="Select Action">
  <Case Key="Create">
    <ui:LogMessage Level="Info" Message="Creating new record" />
  </Case>
  <Case Key="Update">
    <ui:LogMessage Level="Info" Message="Updating record" />
  </Case>
  <Case Key="Delete">
    <ui:LogMessage Level="Info" Message="Deleting record" />
  </Case>
  <Switch.Default>
    <ui:LogMessage Level="Warn" Message="Unknown action" />
  </Switch.Default>
</Switch>
```

### While
Loop while condition is true.

```xml
<While Condition="[intRetryCount &lt; 3 AndAlso Not boolSuccess]" DisplayName="Retry Loop">
  <Sequence DisplayName="Attempt Operation">
    <TryCatch>
      <TryCatch.Try>
        <!-- Operation that might fail -->
        <Assign>
          <Assign.To><OutArgument x:TypeArguments="x:Boolean">[boolSuccess]</OutArgument></Assign.To>
          <Assign.Value><InArgument x:TypeArguments="x:Boolean">[True]</InArgument></Assign.Value>
        </Assign>
      </TryCatch.Try>
      <TryCatch.Catches>
        <Catch x:TypeArguments="s:Exception">
          <ActivityAction x:TypeArguments="s:Exception">
            <Assign>
              <Assign.To><OutArgument x:TypeArguments="x:Int32">[intRetryCount]</OutArgument></Assign.To>
              <Assign.Value><InArgument x:TypeArguments="x:Int32">[intRetryCount + 1]</InArgument></Assign.Value>
            </Assign>
          </ActivityAction>
        </Catch>
      </TryCatch.Catches>
    </TryCatch>
  </Sequence>
</While>
```

### Do While
Execute at least once, then check condition.

```xml
<DoWhile Condition="[Not boolComplete]" DisplayName="Process Until Complete">
  <Sequence>
    <!-- Processing logic -->
    <Assign>
      <Assign.To><OutArgument x:TypeArguments="x:Boolean">[boolComplete]</OutArgument></Assign.To>
      <Assign.Value><InArgument x:TypeArguments="x:Boolean">[CheckCompletion()]</InArgument></Assign.Value>
    </Assign>
  </Sequence>
</DoWhile>
```

### For Each
Iterate over a collection.

```xml
<ForEach x:TypeArguments="x:String" DisplayName="Process Items" Values="[arrItems]">
  <ActivityAction x:TypeArguments="x:String">
    <ActivityAction.Argument>
      <DelegateInArgument x:TypeArguments="x:String" Name="item" />
    </ActivityAction.Argument>
    <Sequence DisplayName="Process Item">
      <ui:LogMessage Level="Info" Message="[&quot;Processing: &quot; + item]" />
    </Sequence>
  </ActivityAction>
</ForEach>
```

### For Each Row in DataTable
Specialized iteration for DataTable rows.

```xml
<ui:ForEachRow DisplayName="Process Data Rows" DataTable="[dtInput]">
  <ui:ForEachRow.Body>
    <ActivityAction x:TypeArguments="sd:DataRow">
      <ActivityAction.Argument>
        <DelegateInArgument x:TypeArguments="sd:DataRow" Name="CurrentRow" />
      </ActivityAction.Argument>
      <Sequence DisplayName="Process Row">
        <Assign DisplayName="Get Cell Value">
          <Assign.To><OutArgument x:TypeArguments="x:String">[strValue]</OutArgument></Assign.To>
          <Assign.Value><InArgument x:TypeArguments="x:String">[CurrentRow("ColumnName").ToString]</InArgument></Assign.Value>
        </Assign>
      </Sequence>
    </ActivityAction>
  </ui:ForEachRow.Body>
</ui:ForEachRow>
```

### Parallel
Execute activities concurrently.

```xml
<Parallel DisplayName="Parallel Processing" CompletionCondition="[boolStopAll]">
  <Sequence DisplayName="Branch 1">
    <ui:LogMessage Level="Info" Message="Branch 1 executing" />
  </Sequence>
  <Sequence DisplayName="Branch 2">
    <ui:LogMessage Level="Info" Message="Branch 2 executing" />
  </Sequence>
</Parallel>
```

### Parallel For Each
Process collection items in parallel.

```xml
<ui:ParallelForEach x:TypeArguments="x:String" DisplayName="Parallel Process" Values="[lstItems]">
  <ui:ParallelForEach.Body>
    <ActivityAction x:TypeArguments="x:String">
      <ActivityAction.Argument>
        <DelegateInArgument x:TypeArguments="x:String" Name="item" />
      </ActivityAction.Argument>
      <!-- Processing for each item -->
      <ui:LogMessage Level="Info" Message="[item]" />
    </ActivityAction>
  </ui:ParallelForEach.Body>
</ui:ParallelForEach>
```

### Delay
Pause execution for specified duration.

```xml
<Delay Duration="[TimeSpan.FromSeconds(5)]" DisplayName="Wait 5 Seconds" />
<Delay Duration="[TimeSpan.FromMilliseconds(500)]" DisplayName="Wait 500ms" />
<Delay Duration="[TimeSpan.FromMinutes(1)]" DisplayName="Wait 1 Minute" />
```

### Invoke Workflow File
Call another XAML workflow.

```xml
<ui:InvokeWorkflowFile WorkflowFileName="Workflows\SubProcess.xaml" DisplayName="Run Sub Process">
  <ui:InvokeWorkflowFile.Arguments>
    <InArgument x:TypeArguments="x:String" x:Key="in_InputParam">[strInput]</InArgument>
    <OutArgument x:TypeArguments="x:String" x:Key="out_Result">[strResult]</OutArgument>
  </ui:InvokeWorkflowFile.Arguments>
</ui:InvokeWorkflowFile>
```

### Invoke Code
Execute inline VB.NET/C# code.

```xml
<ui:InvokeCode Language="VisualBasic" DisplayName="Custom Logic">
  <ui:InvokeCode.Code>
    Dim result As String = ""
    For i As Integer = 0 To inputList.Count - 1
        result += inputList(i) + Environment.NewLine
    Next
    output = result.Trim()
  </ui:InvokeCode.Code>
  <ui:InvokeCode.Arguments>
    <InArgument x:TypeArguments="scg:List(x:String)" x:Key="inputList">[lstInput]</InArgument>
    <OutArgument x:TypeArguments="x:String" x:Key="output">[strOutput]</OutArgument>
  </ui:InvokeCode.Arguments>
</ui:InvokeCode>
```

---

## Data Manipulation

### Build Data Table
Create a DataTable with defined schema.

```xml
<ui:BuildDataTable DisplayName="Create Results Table" DataTable="[dtResults]">
  <ui:BuildDataTable.Columns>
    <ui:DataTableColumn AllowNull="True" AutoIncrement="False" ColumnName="ID" DataType="x:Int32" MaxLength="-1" ReadOnly="False" Unique="False" />
    <ui:DataTableColumn AllowNull="True" AutoIncrement="False" ColumnName="Name" DataType="x:String" MaxLength="-1" ReadOnly="False" Unique="False" />
    <ui:DataTableColumn AllowNull="True" AutoIncrement="False" ColumnName="Status" DataType="x:String" MaxLength="-1" ReadOnly="False" Unique="False" DefaultValue="[&quot;Pending&quot;]" />
    <ui:DataTableColumn AllowNull="True" AutoIncrement="False" ColumnName="ProcessedDate" DataType="s:DateTime" MaxLength="-1" ReadOnly="False" Unique="False" />
  </ui:BuildDataTable.Columns>
</ui:BuildDataTable>
```

### Add Data Row
Add a row to a DataTable.

```xml
<ui:AddDataRow DisplayName="Add Result Row" DataTable="[dtResults]">
  <ui:AddDataRow.ArrayRow>
    <InArgument x:TypeArguments="s:Object[]">[{intId, strName, "Complete", DateTime.Now}]</InArgument>
  </ui:AddDataRow.ArrayRow>
</ui:AddDataRow>
```

### Filter Data Table
Filter rows based on conditions.

```xml
<ui:FilterDataTable DisplayName="Filter Active Records" InputDataTable="[dtAll]" OutputDataTable="[dtFiltered]">
  <ui:FilterDataTable.Filters>
    <ui:FilterRow Column="Status" Condition="Equal" Operand="Active" />
    <ui:FilterRow Column="Amount" Condition="GreaterThan" Operand="100" />
  </ui:FilterDataTable.Filters>
</ui:FilterDataTable>
```

### Sort Data Table
Sort DataTable by columns.

```xml
<ui:SortDataTable DisplayName="Sort by Date" DataTable="[dtData]" OutputDataTable="[dtSorted]">
  <ui:SortDataTable.SortColumns>
    <ui:SortColumn Column="ProcessedDate" SortOrder="Descending" />
    <ui:SortColumn Column="Name" SortOrder="Ascending" />
  </ui:SortDataTable.SortColumns>
</ui:SortDataTable>
```

### Merge Data Table
Merge two DataTables.

```xml
<ui:MergeDataTable Destination="[dtMaster]" Source="[dtNew]" MissingSchemaAction="Add" />
```

### Remove Data Row
Remove a row from DataTable.

```xml
<ui:RemoveDataRow DataTable="[dtData]" Row="[currentRow]" DisplayName="Remove Row" />
```

### Output Data Table
Convert DataTable to string.

```xml
<ui:OutputDataTable DataTable="[dtData]" Text="[strTableOutput]" DisplayName="Table to String" />
```

### Deserialize JSON
Parse JSON string to JObject/JArray.

```xml
<ui:DeserializeJson x:TypeArguments="nj:JObject" JsonString="[strJsonInput]" JsonObject="[joResult]" DisplayName="Parse JSON" />
```

### Serialize JSON
Convert object to JSON string.

```xml
<ui:SerializeJson JsonObject="[objData]" JsonString="[strJsonOutput]" DisplayName="To JSON" />
```

---

## Excel Activities

### Excel Application Scope (Classic)
Open Excel file for processing.

```xml
<ui:ExcelApplicationScope DisplayName="Excel Scope" WorkbookPath="[strFilePath]" Visible="False" CreateNewFile="False">
  <ui:ExcelApplicationScope.Body>
    <ActivityAction x:TypeArguments="ui:WorkbookApplication">
      <ActivityAction.Argument>
        <DelegateInArgument x:TypeArguments="ui:WorkbookApplication" Name="ExcelWorkbook" />
      </ActivityAction.Argument>
      <Sequence DisplayName="Excel Operations">
        <!-- Excel activities here -->
      </Sequence>
    </ActivityAction>
  </ui:ExcelApplicationScope.Body>
</ui:ExcelApplicationScope>
```

### Read Range
Read Excel data to DataTable.

```xml
<ui:ExcelReadRange DisplayName="Read Data" SheetName="Sheet1" Range="A1" AddHeaders="True" DataTable="[dtExcel]" />
```

### Write Range
Write DataTable to Excel.

```xml
<ui:ExcelWriteRange DisplayName="Write Results" SheetName="Results" StartingCell="A1" DataTable="[dtResults]" />
```

### Use Excel File (Modern)
Modern Excel activities scope.

```xml
<ui:UseExcelFile DisplayName="Use Excel" FilePath="[strExcelPath]" ExcelFile="[excelFile]">
  <Sequence DisplayName="Excel Operations">
    <ui:ReadRangeX DisplayName="Read Sheet" SheetName="Data" Range="" 
      DataTable="[dtData]" RangeDetails="[rangeInfo]" ExcelFile="[excelFile]" />
    
    <ui:WriteRangeX DisplayName="Write Results" SheetName="Output" StartingCell="A1" 
      DataTable="[dtOutput]" ExcelFile="[excelFile]" />
  </Sequence>
</ui:UseExcelFile>
```

### For Each Excel Row
Iterate through Excel rows.

```xml
<ui:ForEachExcelRow DisplayName="Process Rows" SheetName="Data" DataTableVariable="[dtSheet]">
  <ActivityAction x:TypeArguments="sd:DataRow">
    <ActivityAction.Argument>
      <DelegateInArgument x:TypeArguments="sd:DataRow" Name="CurrentRow" />
    </ActivityAction.Argument>
    <Sequence>
      <!-- Process CurrentRow -->
    </Sequence>
  </ActivityAction>
</ui:ForEachExcelRow>
```

---

## UI Automation

### Click
Click a UI element.

```xml
<ui:Click DisplayName="Click Submit" ClickType="CLICK_SINGLE" MouseButton="BTN_LEFT">
  <ui:Click.Target>
    <ui:Target Selector="&lt;html app='chrome.exe' title='Application'/&gt;&lt;webctrl tag='BUTTON' id='submit'/&gt;" />
  </ui:Click.Target>
</ui:Click>
```

### Type Into
Type text into a field.

```xml
<ui:TypeInto DisplayName="Enter Username" Text="[strUsername]" DelayBetweenKeys="10" DelayMS="100">
  <ui:TypeInto.Target>
    <ui:Target Selector="&lt;html app='chrome.exe'/&gt;&lt;webctrl tag='INPUT' id='username'/&gt;" />
  </ui:TypeInto.Target>
</ui:TypeInto>
```

### Get Text
Extract text from UI element.

```xml
<ui:GetText DisplayName="Get Status" Value="[strStatusText]">
  <ui:GetText.Target>
    <ui:Target Selector="&lt;html app='chrome.exe'/&gt;&lt;webctrl tag='SPAN' class='status'/&gt;" />
  </ui:GetText.Target>
</ui:GetText>
```

### Get Attribute
Get attribute value from UI element.

```xml
<ui:GetAttribute DisplayName="Get Value" AttributeName="value" Result="[strValue]">
  <ui:GetAttribute.Target>
    <ui:Target Selector="&lt;html app='chrome.exe'/&gt;&lt;webctrl tag='INPUT' id='amount'/&gt;" />
  </ui:GetAttribute.Target>
</ui:GetAttribute>
```

### Element Exists
Check if element exists.

```xml
<ui:ElementExists DisplayName="Check Error Message" Exists="[boolErrorExists]" Timeout="3000">
  <ui:ElementExists.Target>
    <ui:Target Selector="&lt;html app='chrome.exe'/&gt;&lt;webctrl tag='DIV' class='error-message'/&gt;" />
  </ui:ElementExists.Target>
</ui:ElementExists>
```

### Wait Element Vanish
Wait for element to disappear.

```xml
<ui:WaitElementVanish DisplayName="Wait Loading" Timeout="30000">
  <ui:WaitElementVanish.Target>
    <ui:Target Selector="&lt;html app='chrome.exe'/&gt;&lt;webctrl tag='DIV' class='loading-spinner'/&gt;" />
  </ui:WaitElementVanish.Target>
</ui:WaitElementVanish>
```

### Use Application/Browser (Modern)
Modern UI automation scope.

```xml
<ui:UseApplicationBrowser DisplayName="Automate Browser" ApplicationWindow="[browserWindow]" 
  BrowserType="Chrome" Url="https://example.com">
  <Sequence DisplayName="Browser Automation">
    <ui:ClickX DisplayName="Click Login">
      <ui:ClickX.Target>
        <ui:Target Selector="&lt;webctrl tag='BUTTON' innertext='Login'/&gt;" />
      </ui:ClickX.Target>
    </ui:ClickX>
  </Sequence>
</ui:UseApplicationBrowser>
```

### Keyboard Shortcuts
Send keyboard shortcuts.

```xml
<ui:SendHotkey DisplayName="Save" Key="s" ModifierKeys="Ctrl" />
<ui:SendHotkey DisplayName="Select All" Key="a" ModifierKeys="Ctrl" />
<ui:SendHotkey DisplayName="Copy" Key="c" ModifierKeys="Ctrl" />
```

### Set Text
Set text directly (faster than Type Into).

```xml
<ui:SetText DisplayName="Set Value" Text="[strValue]">
  <ui:SetText.Target>
    <ui:Target Selector="&lt;html/&gt;&lt;webctrl tag='INPUT' id='field'/&gt;" />
  </ui:SetText.Target>
</ui:SetText>
```

---

## Web Activities

### HTTP Request
Make HTTP/REST API calls.

**CRITICAL**: Use `uwah:NetHttpRequest` from `UiPath.WebAPI.Activities [2.3.2]`.
`ui:HttpClient` does NOT exist and causes `ErrorActivity: This activity is missing or could not be loaded`.

Required namespace: `xmlns:uwah="clr-namespace:UiPath.Web.Activities.Http;assembly=UiPath.Web.Activities"`
Response type namespace: `xmlns:uwahm="clr-namespace:UiPath.Web.Activities.Http.Models;assembly=UiPath.Web.Activities"`

```xml
<!-- GET request -->
<uwah:NetHttpRequest
  AuthenticationType="None"
  ContinueOnError="True"
  DisableSslVerification="False"
  DisplayName="HTTP Request"
  EnableCookies="True"
  FollowRedirects="True"
  Method="GET"
  RequestBodyType="Text"
  RequestUrl="[&quot;https://api.example.com/data&quot;]"
  Result="[responseContent]"
  RetryCount="3"
  RetryPolicyType="Basic"
  SaveRawRequestResponse="False"
  SaveResponseAsFile="False"
  TlsProtocol="Automatic"
  Cookies="[New System.Collections.Generic.Dictionary(Of String, String) From {}]"
  FormData="[New System.Collections.Generic.Dictionary(Of String, String) From {}]"
  Parameters="[New System.Collections.Generic.Dictionary(Of String, String) From {}]"
  Headers="[new System.Collections.Generic.Dictionary(Of System.String, System.String) From { { &quot;Authorization&quot;, &quot;Bearer &quot; + strToken }, { &quot;Content-Type&quot;, &quot;application/json&quot; } }]">
  <uwah:NetHttpRequest.TimeoutInMiliseconds>
    <InArgument x:TypeArguments="s:Nullable(x:Int32)">
      <Literal x:TypeArguments="s:Nullable(x:Int32)" Value="10000" />
    </InArgument>
  </uwah:NetHttpRequest.TimeoutInMiliseconds>
</uwah:NetHttpRequest>

<!-- POST with body -->
<uwah:NetHttpRequest
  DisplayName="Create Record"
  Method="POST"
  RequestBodyType="Text"
  RequestUrl="[&quot;https://api.example.com/records&quot;]"
  TextPayload="[strJsonBody]"
  TextPayloadContentType="[&quot;application/json&quot;]"
  TextPayloadEncoding="[&quot;UTF-8&quot;]"
  Result="[responseContent]"
  AuthenticationType="None"
  ContinueOnError="True"
  Cookies="[New System.Collections.Generic.Dictionary(Of String, String) From {}]"
  FormData="[New System.Collections.Generic.Dictionary(Of String, String) From {}]"
  Parameters="[New System.Collections.Generic.Dictionary(Of String, String) From {}]"
  Headers="[new System.Collections.Generic.Dictionary(Of System.String, System.String) From { { &quot;Authorization&quot;, &quot;Bearer &quot; + strToken } }]">
  <uwah:NetHttpRequest.TimeoutInMiliseconds>
    <InArgument x:TypeArguments="s:Nullable(x:Int32)">
      <Literal x:TypeArguments="s:Nullable(x:Int32)" Value="10000" />
    </InArgument>
  </uwah:NetHttpRequest.TimeoutInMiliseconds>
</uwah:NetHttpRequest>

<!-- Parse response (responseContent is uwahm:HttpResponseSummary) -->
<ui:DeserializeJson x:TypeArguments="njl:JObject"
  DisplayName="Parse Response"
  JsonString="[responseContent.TextContent.ToString]"
  JsonObject="[responseJSON]" />
<!-- Access fields: responseJSON("fieldName").ToString -->
```

### Open Browser
Open web browser.

```xml
<ui:OpenBrowser DisplayName="Open Website" Url="https://example.com" BrowserType="Chrome" Browser="[browserInstance]">
  <ui:OpenBrowser.Body>
    <ActivityAction x:TypeArguments="x:Object">
      <Sequence DisplayName="Browser Actions">
        <!-- Web automation activities -->
      </Sequence>
    </ActivityAction>
  </ui:OpenBrowser.Body>
</ui:OpenBrowser>
```

### Navigate To
Navigate to URL in existing browser.

```xml
<ui:NavigateTo DisplayName="Go to Page" Url="https://example.com/page" Browser="[browserInstance]" />
```

### Close Browser
Close browser instance.

```xml
<ui:CloseBrowser DisplayName="Close Browser" Browser="[browserInstance]" />
```

---

## File & Folder

### Read Text File
Read file contents.

```xml
<ui:ReadTextFile DisplayName="Read File" FileName="[strFilePath]" Content="[strFileContent]" Encoding="UTF8" />
```

### Write Text File
Write to file.

```xml
<ui:WriteTextFile DisplayName="Write File" FileName="[strOutputPath]" Text="[strContent]" Encoding="UTF8" />
```

### Append Line
Append text to file.

```xml
<ui:AppendLine DisplayName="Add Log Entry" FileName="[strLogPath]" Text="[strLogMessage]" Encoding="UTF8" />
```

### Copy File
Copy file to destination.

```xml
<ui:CopyFile DisplayName="Copy Report" Path="[strSourcePath]" Destination="[strDestPath]" Overwrite="True" />
```

### Move File
Move file to new location.

```xml
<ui:MoveFile DisplayName="Archive File" Path="[strSourcePath]" Destination="[strArchivePath]" />
```

### Delete File
Delete a file.

```xml
<ui:Delete DisplayName="Delete Temp" Path="[strTempFilePath]" />
```

### Path Exists
Check if file or folder exists.

```xml
<ui:PathExists DisplayName="Check File" Path="[strFilePath]" PathType="File" Exists="[boolFileExists]" />
<ui:PathExists DisplayName="Check Folder" Path="[strFolderPath]" PathType="Folder" Exists="[boolFolderExists]" />
```

### Create Folder
Create directory.

```xml
<ui:CreateDirectory DisplayName="Create Output Folder" Path="[strOutputFolder]" />
```

### Get Files
Get list of files in folder.

```xml
<ui:GetFiles DisplayName="Get All PDFs" Directory="[strFolderPath]" SearchPattern="*.pdf" Files="[arrFiles]" />
```

### For Each File in Folder
Iterate through files.

```xml
<ui:ForEachFile DisplayName="Process Files" Directory="[strFolderPath]" SearchPattern="*.xlsx">
  <ui:ForEachFile.Body>
    <ActivityAction x:TypeArguments="s:IO.FileInfo">
      <ActivityAction.Argument>
        <DelegateInArgument x:TypeArguments="s:IO.FileInfo" Name="CurrentFile" />
      </ActivityAction.Argument>
      <Sequence>
        <ui:LogMessage Level="Info" Message="[CurrentFile.FullName]" />
      </Sequence>
    </ActivityAction>
  </ui:ForEachFile.Body>
</ui:ForEachFile>
```

---

## Database

### Connect
Create database connection.

```xml
<ui:DatabaseConnect DisplayName="Connect to DB" 
  ConnectionString="Server=localhost;Database=MyDB;Trusted_Connection=True;" 
  ProviderName="System.Data.SqlClient" 
  DatabaseConnection="[dbConnection]" />
```

### Execute Query
Execute SELECT query.

```xml
<ui:ExecuteQuery DisplayName="Get Records" 
  DatabaseConnection="[dbConnection]" 
  Sql="SELECT * FROM Orders WHERE Status = @status" 
  DataTable="[dtResults]">
  <ui:ExecuteQuery.Parameters>
    <ui:DatabaseParameter Direction="Input" DbType="String" ParameterName="status" Value="[strStatus]" />
  </ui:ExecuteQuery.Parameters>
</ui:ExecuteQuery>
```

### Execute Non Query
Execute INSERT/UPDATE/DELETE.

```xml
<ui:ExecuteNonQuery DisplayName="Update Status" 
  DatabaseConnection="[dbConnection]" 
  Sql="UPDATE Orders SET Status = @newStatus WHERE OrderId = @orderId" 
  AffectedRecords="[intRowsAffected]">
  <ui:ExecuteNonQuery.Parameters>
    <ui:DatabaseParameter Direction="Input" DbType="String" ParameterName="newStatus" Value="[strNewStatus]" />
    <ui:DatabaseParameter Direction="Input" DbType="Int32" ParameterName="orderId" Value="[intOrderId]" />
  </ui:ExecuteNonQuery.Parameters>
</ui:ExecuteNonQuery>
```

### Disconnect
Close database connection.

```xml
<ui:DatabaseDisconnect DisplayName="Close DB" DatabaseConnection="[dbConnection]" />
```

---

## Error Handling

### Try Catch
Handle exceptions.

```xml
<TryCatch DisplayName="Handle Errors">
  <TryCatch.Try>
    <Sequence DisplayName="Main Logic">
      <!-- Activities that might fail -->
    </Sequence>
  </TryCatch.Try>
  <TryCatch.Catches>
    <Catch x:TypeArguments="ui:BusinessRuleException">
      <ActivityAction x:TypeArguments="ui:BusinessRuleException">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="ui:BusinessRuleException" Name="breException" />
        </ActivityAction.Argument>
        <Sequence DisplayName="Handle Business Rule">
          <ui:LogMessage Level="Warn" Message="[breException.Message]" />
        </Sequence>
      </ActivityAction>
    </Catch>
    <Catch x:TypeArguments="s:Exception">
      <ActivityAction x:TypeArguments="s:Exception">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="s:Exception" Name="sysException" />
        </ActivityAction.Argument>
        <Sequence DisplayName="Handle System Exception">
          <ui:LogMessage Level="Error" Message="[sysException.Message]" />
          <Rethrow />
        </Sequence>
      </ActivityAction>
    </Catch>
  </TryCatch.Catches>
  <TryCatch.Finally>
    <Sequence DisplayName="Cleanup">
      <ui:LogMessage Level="Info" Message="Cleanup completed" />
    </Sequence>
  </TryCatch.Finally>
</TryCatch>
```

### Throw
Throw an exception.

```xml
<Throw DisplayName="Throw Error">
  <Throw.Exception>
    <InArgument x:TypeArguments="s:Exception">[New BusinessRuleException("Invalid data format")]</InArgument>
  </Throw.Exception>
</Throw>
```

### Rethrow
Re-throw current exception.

```xml
<Rethrow DisplayName="Rethrow Exception" />
```

### Retry Scope
Retry activities on failure.

```xml
<ui:RetryScope DisplayName="Retry Operation" NumberOfRetries="3" RetryInterval="[TimeSpan.FromSeconds(2)]">
  <ui:RetryScope.ActivityBody>
    <ActivityAction>
      <Sequence DisplayName="Retryable Action">
        <!-- Activities to retry -->
      </Sequence>
    </ActivityAction>
  </ui:RetryScope.ActivityBody>
  <ui:RetryScope.Condition>
    <!-- Optional: Condition to retry on -->
    <ActivityAction x:TypeArguments="x:Boolean">
      <ActivityAction.Argument>
        <DelegateInArgument x:TypeArguments="x:Boolean" Name="retry" />
      </ActivityAction.Argument>
      <Assign>
        <Assign.To><OutArgument x:TypeArguments="x:Boolean">[retry]</OutArgument></Assign.To>
        <Assign.Value><InArgument x:TypeArguments="x:Boolean">[Not boolSuccess]</InArgument></Assign.Value>
      </Assign>
    </ActivityAction>
  </ui:RetryScope.Condition>
</ui:RetryScope>
```

### Terminate Workflow
End workflow with status.

```xml
<TerminateWorkflow DisplayName="Terminate on Error">
  <TerminateWorkflow.Reason>
    <InArgument x:TypeArguments="x:String">["Critical error: " + strErrorMessage]</InArgument>
  </TerminateWorkflow.Reason>
</TerminateWorkflow>
```

---

## Orchestrator

### Get Queue Items
Retrieve queue items.

```xml
<ui:GetQueueItems DisplayName="Get Pending Items" QueueName="ProcessingQueue" 
  FilterStrategy="Filter" Status="New" Reference="[strReferenceFilter]"
  QueueItems="[lstQueueItems]" />
```

### Add Queue Item
Add item to queue.

```xml
<ui:AddQueueItem DisplayName="Add to Queue" QueueName="ProcessingQueue" 
  Reference="[strReference]" Priority="High" DeferDate="[dtDeferDate]">
  <ui:AddQueueItem.ItemInformation>
    <ui:DictionaryArgument>
      <ui:DictionaryArgument.Dictionary>
        <InArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)">[dictItemData]</InArgument>
      </ui:DictionaryArgument.Dictionary>
    </ui:DictionaryArgument>
  </ui:AddQueueItem.ItemInformation>
</ui:AddQueueItem>
```

### Get Transaction Item
Get next queue item for processing.

```xml
<ui:GetTransactionItem DisplayName="Get Transaction" QueueName="ProcessingQueue" 
  TransactionItem="[queueItem]" />
```

### Set Transaction Status
Set queue item status.

```xml
<ui:SetTransactionStatus DisplayName="Set Success" Status="Successful" 
  TransactionItem="[queueItem]" />

<ui:SetTransactionStatus DisplayName="Set Failed" Status="Failed" 
  TransactionItem="[queueItem]" Reason="[strFailureReason]" 
  ErrorType="Business" />
```

### Get Asset
Retrieve Orchestrator asset.

```xml
<ui:GetRobotAsset DisplayName="Get Credential" AssetName="SystemCredential" 
  AssetValue="[strAssetValue]" />
```

### Get Credential
Retrieve Orchestrator credential.

```xml
<ui:GetRobotCredential DisplayName="Get Login" CredentialAssetName="AppCredential" 
  Username="[strUsername]" Password="[secPassword]" />
```

### Log Message
Write to Orchestrator logs.

```xml
<ui:LogMessage DisplayName="Log Info" Level="Info" Message="Processing started" />
<ui:LogMessage DisplayName="Log Warning" Level="Warn" Message="[strWarning]" />
<ui:LogMessage DisplayName="Log Error" Level="Error" Message="[exception.Message]" />
```

---

## Mail Activities

### Get Outlook Mail Messages
Retrieve emails from Outlook.

```xml
<ui:GetOutlookMailMessages DisplayName="Get Inbox" MailFolder="Inbox" 
  Account="[strEmailAccount]" Top="50" Filter="[strFilter]" 
  MarkAsRead="False" Messages="[lstMailMessages]" />
```

### Send Email via Microsoft Graph / Office 365 Integration Service
**CRITICAL**: Use `umam:SendMailConnections` from `UiPath.MicrosoftOffice365.Activities`.
`ui:SendOutlookMailMessage` does NOT exist and causes `ErrorActivity`.

Required namespaces:
- `xmlns:umam="clr-namespace:UiPath.MicrosoftOffice365.Activities.Mail;assembly=UiPath.MicrosoftOffice365.Activities"`
- `xmlns:umame="clr-namespace:UiPath.MicrosoftOffice365.Activities.Mail.Enums;assembly=UiPath.MicrosoftOffice365.Activities"`
- `xmlns:umamm="clr-namespace:UiPath.MicrosoftOffice365.Activities.Mail.Models;assembly=UiPath.MicrosoftOffice365.Activities"`
- `xmlns:usau="clr-namespace:UiPath.Shared.Activities.Utils;assembly=UiPath.MicrosoftOffice365.Activities"`

```xml
<umam:SendMailConnections
  Body="[strEmailBody]"
  ConnectionId="YOUR-CONNECTION-ID"
  DisplayName="Send Email"
  InputType="HTML"
  SaveAsDraft="False"
  To="[New String(){&quot;recipient@example.com&quot;}]"
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

### IMAP Get Mail Messages
Retrieve via IMAP.

```xml
<ui:GetIMAPMailMessages DisplayName="Get IMAP Messages" Server="imap.example.com" 
  Port="993" SecureConnection="SSL" Email="[strEmail]" Password="[secPassword]"
  MailFolder="INBOX" Top="20" Messages="[lstMails]" />
```

### SMTP Send Mail
Send via SMTP.

```xml
<ui:SendMailMessage DisplayName="Send SMTP" Server="smtp.example.com" 
  Port="587" SecureConnection="StartTLS" 
  Email="[strSenderEmail]" Password="[secPassword]"
  To="recipient@example.com" Subject="[strSubject]" Body="[strBody]" />
```

### Save Mail Attachments
Save email attachments.

```xml
<ui:SaveMailAttachments DisplayName="Save Attachments" MailMessage="[mailMessage]" 
  FolderPath="[strAttachmentFolder]" />
```
