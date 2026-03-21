# Module: Error Handling, Retries & Exceptions
**Covers:** Try/Catch patterns, exception types, retry logic, Global Exception Handler, logging on failure

---

## Exception Type Hierarchy

```
Exception
├── BusinessRuleException (UiPath.Core)      → data/validation issue — do NOT retry
│   └── "Invalid invoice number format"
├── SelectorNotFoundException               → UI element not found
├── ImageOperationException                 → image/CV operation failed
├── ApplicationException                    → generic system exception — retry
├── InvalidOperationException               → wrong state/sequence
├── ArgumentException                       → bad argument value
├── NullReferenceException                  → null dereference
├── FormatException                         → type conversion failed
├── TimeoutException                        → activity timed out
└── IOException                             → file/network I/O failure
```

**Rule:** Catch the most specific type possible. Never use empty catch blocks.

---

## Standard Try/Catch/Finally Pattern

```xml
<TryCatch sap2010:WorkflowViewState.IdRef="TryCatch_1"
    DisplayName="Try: Process Document">
  <TryCatch.Try>
    <Sequence DisplayName="Main Logic">
      <!-- risky activities here -->
    </Sequence>
  </TryCatch.Try>

  <TryCatch.Catches>
    <!-- 1. Most specific exception first -->
    <Catch x:TypeArguments="ui:BusinessRuleException">
      <ActivityAction x:TypeArguments="ui:BusinessRuleException">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="ui:BusinessRuleException" Name="bizEx"/>
        </ActivityAction.Argument>
        <Sequence DisplayName="Handle Business Exception">
          <ui:LogMessage Level="Warn"
            Message="[&quot;Business rule: &quot; + bizEx.Message + &quot; | Source: &quot; + bizEx.Source]"/>
          <!-- Do NOT rethrow — log and handle gracefully -->
          <Assign>[out_Status = "BusinessException"]</Assign>
          <Assign>[out_ErrorMessage = bizEx.Message]</Assign>
        </Sequence>
      </ActivityAction>
    </Catch>

    <!-- 2. UI-specific exceptions -->
    <Catch x:TypeArguments="ui:SelectorNotFoundException">
      <ActivityAction x:TypeArguments="ui:SelectorNotFoundException">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="ui:SelectorNotFoundException" Name="selectorEx"/>
        </ActivityAction.Argument>
        <Sequence DisplayName="Handle Selector Not Found">
          <ui:TakeScreenshot FileName="[&quot;Logs\Screenshots\selectorFail_&quot; + DateTime.Now.ToString(&quot;HHmmss&quot;) + &quot;.png&quot;]"/>
          <ui:LogMessage Level="Error" Message="[&quot;Selector not found: &quot; + selectorEx.Message]"/>
          <Rethrow/>
        </Sequence>
      </ActivityAction>
    </Catch>

    <!-- 3. Generic system exception — always last -->
    <Catch x:TypeArguments="s:Exception">
      <ActivityAction x:TypeArguments="s:Exception">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="s:Exception" Name="exception"/>
        </ActivityAction.Argument>
        <Sequence DisplayName="Handle System Exception">
          <ui:LogMessage Level="Error"
            Message="[&quot;System exception in &quot; + workflowName + &quot;: &quot; + exception.Message + Environment.NewLine + exception.StackTrace]"/>
          <Rethrow/>
        </Sequence>
      </ActivityAction>
    </Catch>
  </TryCatch.Catches>

  <TryCatch.Finally>
    <!-- Runs ALWAYS — even if exception rethrown -->
    <Sequence DisplayName="Cleanup">
      <!-- Close resources, disconnect DB, release locks -->
    </Sequence>
  </TryCatch.Finally>
</TryCatch>
```

---

## Retry Scope (built-in retry activity)

```xml
<ui:RetryScope sap2010:WorkflowViewState.IdRef="RetryScope_1"
    DisplayName="Retry: Find and Click Element"
    NumberOfRetries="3"
    RetryInterval="[TimeSpan.FromSeconds(5)]">
  <ui:RetryScope.Try>
    <Sequence DisplayName="Attempt">
      <ui:Click DisplayName="Click Submit" Target.Selector="[btnSelector]"/>
    </Sequence>
  </ui:RetryScope.Try>
  <ui:RetryScope.Catch>
    <!-- Condition that determines whether to retry -->
    <!-- If True = retry; if False = stop retrying and throw -->
    <InArgument x:TypeArguments="x:Boolean">
      [Not (exception.Message.Contains("Unauthorized"))]
    </InArgument>
  </ui:RetryScope.Catch>
</ui:RetryScope>
```

---

## Manual Retry Pattern (counter-based)

Use when you need more control than RetryScope:

```xml
<Sequence DisplayName="Manual Retry Logic">
  <Assign DisplayName="Init Retry Counter"><Assign.To>[retryCount]</Assign.To><Assign.Value>[0]</Assign.Value></Assign>
  <Assign DisplayName="Init Success Flag"><Assign.To>[isSuccess]</Assign.To><Assign.Value>[False]</Assign.Value></Assign>

  <While DisplayName="Retry Loop" Condition="[Not isSuccess AndAlso retryCount &lt; maxRetries]">
    <Sequence>
      <TryCatch DisplayName="Try Operation">
        <TryCatch.Try>
          <Sequence DisplayName="Attempt Operation">
            <!-- actual work -->
            <Assign>[isSuccess = True]</Assign>
          </Sequence>
        </TryCatch.Try>
        <TryCatch.Catches>
          <Catch x:TypeArguments="ui:BusinessRuleException">
            <ActivityAction x:TypeArguments="ui:BusinessRuleException">
              <ActivityAction.Argument>
                <DelegateInArgument x:TypeArguments="ui:BusinessRuleException" Name="ex"/>
              </ActivityAction.Argument>
              <!-- BizEx — don't retry, bubble up -->
              <Rethrow/>
            </ActivityAction>
          </Catch>
          <Catch x:TypeArguments="s:Exception">
            <ActivityAction x:TypeArguments="s:Exception">
              <ActivityAction.Argument>
                <DelegateInArgument x:TypeArguments="s:Exception" Name="ex"/>
              </ActivityAction.Argument>
              <Sequence>
                <Assign>[retryCount = retryCount + 1]</Assign>
                <ui:LogMessage Level="Warn" Message="[&quot;Attempt &quot; + retryCount.ToString() + &quot; failed: &quot; + ex.Message]"/>
                <If Condition="[retryCount &gt;= maxRetries]">
                  <If.Then><Rethrow/></If.Then>
                  <If.Else><Delay Duration="[TimeSpan.FromSeconds(waitSeconds)]"/></If.Else>
                </If>
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

## Global Exception Handler

Place in `GlobalExceptionHandler.xaml` — catches any unhandled exception at process level.

```xml
<!-- GlobalExceptionHandler.xaml root -->
<Sequence DisplayName="Global Exception Handler">
  <!-- Arguments auto-provided by framework: -->
  <!-- in_ErrorInfo As ErrorInfo -->
  <!-- out_Restart As Boolean -->

  <ui:LogMessage Level="Fatal"
    Message="[&quot;UNHANDLED EXCEPTION: &quot; + in_ErrorInfo.Exception.Message + Environment.NewLine + &quot;Activity: &quot; + in_ErrorInfo.ActivityInfo?.DisplayName + Environment.NewLine + &quot;Stack: &quot; + in_ErrorInfo.Exception.StackTrace]"/>

  <ui:TakeScreenshot DisplayName="Capture State"
    FileName="[&quot;Logs\GEH_&quot; + DateTime.Now.ToString(&quot;yyyyMMddHHmmss&quot;) + &quot;.png&quot;]"/>

  <!-- Send alert email -->
  <ui:SendSMTPMail DisplayName="Alert Team"
    Server="[smtpServer]" Port="587" EnableSSL="True"
    To="[alertEmail]"
    Subject="[&quot;[ALERT] UiPath Process Failed - &quot; + in_ErrorInfo.Exception.Message]"
    Body="[&quot;Exception: &quot; + in_ErrorInfo.Exception.Message + &quot;&lt;br/&gt;Activity: &quot; + in_ErrorInfo.ActivityInfo?.DisplayName]"
    IsBodyHtml="True"
    Username="[smtpUser]" Password="[smtpPwd]"/>

  <!-- Decide: restart process or stop -->
  <If Condition="[in_ErrorInfo.RetryCount &lt; 1 AndAlso TypeOf in_ErrorInfo.Exception IsNot ui:BusinessRuleException]">
    <If.Then>
      <Assign DisplayName="Request Restart">[out_Restart = True]</Assign>
    </If.Then>
    <If.Else>
      <Assign DisplayName="Stop Process">[out_Restart = False]</Assign>
    </If.Else>
  </If>
</Sequence>
```

Register in `project.json`:
```json
{
  "globalHandlerFilePath": "GlobalExceptionHandler.xaml"
}
```

---

## Throw Custom Exceptions

```xml
<!-- Throw BusinessRuleException -->
<ui:Throw DisplayName="Throw: Invalid Format">
  <ui:Throw.Exception>
    <InArgument x:TypeArguments="ui:BusinessRuleException">
      [New UiPath.Core.Activities.RobotException.BusinessRuleException(
        "Invoice number format invalid: " + invoiceNumber + ". Expected: INV-XXXXXX")]
    </InArgument>
  </ui:Throw.Exception>
</ui:Throw>

<!-- Throw generic Exception -->
<ui:Throw DisplayName="Throw: Config Missing">
  <ui:Throw.Exception>
    <InArgument x:TypeArguments="s:Exception">
      [New Exception("Required config key 'BaseURL' not found in Config.xlsx")]
    </InArgument>
  </ui:Throw.Exception>
</ui:Throw>
```

---

## Logging Standards

```
INFO  — Normal milestones: "Started processing batch X", "Completed 45/100 items"
WARN  — Recoverable issues: "Retry attempt 2/3", "Defaulting to fallback value"
ERROR — Handled failures: "Business rule: invalid amount", "Selector not found, retrying"
FATAL — Global handler only: "Unhandled exception caused process abort"

NEVER log:
- Passwords or credentials (even masked)
- PII data (names, SSNs, emails) unless required by business
- Raw exception StackTrace at INFO level (use ERROR or DEBUG)
```

---

## Error Handling Checklist

- [ ] Every `Try` has a non-empty `Catch`
- [ ] `BusinessRuleException` caught separately — never retried
- [ ] System exceptions rethrown after logging (let REF/GEH handle)
- [ ] `Finally` block present for resource cleanup (DB connections, file handles)
- [ ] Screenshots taken on UI automation failures
- [ ] Log message includes enough context to diagnose without running again
- [ ] `RetryScope` or manual retry only for transient failures
- [ ] `GlobalExceptionHandler.xaml` registered in project settings
