# Workflow Templates Reference

Complete long-running workflow templates for common use cases.

**Note**: For general error handling and logging patterns, see:
- [Error Handling](../../_shared/error-handling.md)
- [Logging Standards](../../_shared/logging-standards.md)
- [Project Configuration](../../_shared/project-configuration.md)

## Table of Contents

1. [Single Approval Workflow](#single-approval-workflow)
2. [Multi-Level Approval Workflow](#multi-level-approval-workflow)
3. [Exception Handling Workflow](#exception-handling-workflow)
4. [Document Review Workflow](#document-review-workflow)
5. [Escalation Workflow](#escalation-workflow)

---

## Single Approval Workflow

Basic approval workflow with one human task.

### Main.xaml

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="Main" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:nj="clr-namespace:Newtonsoft.Json.Linq;assembly=Newtonsoft.Json" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <VisualBasic.Settings>
    <x:Null />
  </VisualBasic.Settings>
  <sap2010:WorkflowViewState.IdRef>ActivityBuilder_1</sap2010:WorkflowViewState.IdRef>
  <Sequence DisplayName="Single Approval Process">
    <Sequence.Variables>
      <Variable x:TypeArguments="x:String" Name="taskId" />
      <Variable x:TypeArguments="x:String" Name="action" />
      <Variable x:TypeArguments="x:String" Name="formDataJson" />
      <Variable x:TypeArguments="nj:JObject" Name="formData" />
      <Variable x:TypeArguments="x:Boolean" Name="isApproved" />
      <Variable x:TypeArguments="x:String" Name="comments" />
    </Sequence.Variables>
    
    <!-- Arguments -->
    <Sequence.Arguments>
      <InArgument x:TypeArguments="x:String" x:Key="in_RequestId" />
      <InArgument x:TypeArguments="x:String" x:Key="in_RequestTitle" />
      <InArgument x:TypeArguments="x:Double" x:Key="in_Amount" />
      <InArgument x:TypeArguments="x:String" x:Key="in_Requestor" />
      <OutArgument x:TypeArguments="x:Boolean" x:Key="out_Approved" />
      <OutArgument x:TypeArguments="x:String" x:Key="out_Comments" />
    </Sequence.Arguments>
    
    <!-- Log Start -->
    <ui:LogMessage
      DisplayName="Log Process Start"
      Level="Info"
      Message="[String.Format(&quot;Starting approval for request {0}&quot;, in_RequestId)]" />
    
    <!-- Create Approval Task -->
    <ui:CreateFormTask
      DisplayName="Create Approval Task"
      TaskTitle="[String.Format(&quot;Approve: {0}&quot;, in_RequestTitle)]"
      TaskPriority="Normal"
      FormSchemaPath="FormSchemas\ApprovalForm.json"
      TaskDataJson="[String.Format(&quot;{{&quot;&quot;requestDetails&quot;&quot;: {{&quot;&quot;requestId&quot;&quot;: &quot;&quot;{0}&quot;&quot;, &quot;&quot;requestor&quot;&quot;: &quot;&quot;{1}&quot;&quot;, &quot;&quot;description&quot;&quot;: &quot;&quot;{2}&quot;&quot;, &quot;&quot;amount&quot;&quot;: {3}}}}}&quot;, in_RequestId, in_Requestor, in_RequestTitle, in_Amount)]"
      Result="[taskId]" />
    
    <!-- Log Task Created -->
    <ui:LogMessage
      DisplayName="Log Task Created"
      Level="Info"
      Message="[String.Format(&quot;Approval task created: {0}&quot;, taskId)]" />
    
    <!-- Wait for Approval -->
    <ui:WaitForFormTaskAndResume
      DisplayName="Wait for Approval"
      TaskId="[taskId]"
      TaskAction="[action]"
      TaskData="[formDataJson]" />
    
    <!-- Parse Response -->
    <Assign DisplayName="Parse Form Data">
      <Assign.To>
        <OutArgument x:TypeArguments="nj:JObject">[formData]</OutArgument>
      </Assign.To>
      <Assign.Value>
        <InArgument x:TypeArguments="nj:JObject">
          [nj:JObject.Parse(formDataJson)]
        </InArgument>
      </Assign.Value>
    </Assign>
    
    <!-- Extract Decision -->
    <Assign DisplayName="Get Approval Status">
      <Assign.To>
        <OutArgument x:TypeArguments="x:Boolean">[isApproved]</OutArgument>
      </Assign.To>
      <Assign.Value>
        <InArgument x:TypeArguments="x:Boolean">
          [formData("decision")("approved").Value(Of Boolean)]
        </InArgument>
      </Assign.Value>
    </Assign>
    
    <Assign DisplayName="Get Comments">
      <Assign.To>
        <OutArgument x:TypeArguments="x:String">[comments]</OutArgument>
      </Assign.To>
      <Assign.Value>
        <InArgument x:TypeArguments="x:String">
          [If(formData("decision")("comments") IsNot Nothing, formData("decision")("comments").ToString(), "")]
        </InArgument>
      </Assign.Value>
    </Assign>
    
    <!-- Set Output -->
    <Assign DisplayName="Set Output - Approved">
      <Assign.To>
        <OutArgument x:TypeArguments="x:Boolean">[out_Approved]</OutArgument>
      </Assign.To>
      <Assign.Value>
        <InArgument x:TypeArguments="x:Boolean">[isApproved]</InArgument>
      </Assign.Value>
    </Assign>
    
    <Assign DisplayName="Set Output - Comments">
      <Assign.To>
        <OutArgument x:TypeArguments="x:String">[out_Comments]</OutArgument>
      </Assign.To>
      <Assign.Value>
        <InArgument x:TypeArguments="x:String">[comments]</InArgument>
      </Assign.Value>
    </Assign>
    
    <!-- Log Result -->
    <ui:LogMessage
      DisplayName="Log Result"
      Level="Info"
      Message="[String.Format(&quot;Request {0}: {1}. Comments: {2}&quot;, in_RequestId, If(isApproved, &quot;Approved&quot;, &quot;Rejected&quot;), comments)]" />
    
  </Sequence>
</Activity>
```

---

## Multi-Level Approval Workflow

Sequential approval through multiple levels.

### Main.xaml

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="Main" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:nj="clr-namespace:Newtonsoft.Json.Linq;assembly=Newtonsoft.Json" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <VisualBasic.Settings>
    <x:Null />
  </VisualBasic.Settings>
  <sap2010:WorkflowViewState.IdRef>ActivityBuilder_1</sap2010:WorkflowViewState.IdRef>
  <Sequence DisplayName="Multi-Level Approval">
    <Sequence.Variables>
      <!-- Level 1 -->
      <Variable x:TypeArguments="x:String" Name="level1TaskId" />
      <Variable x:TypeArguments="x:String" Name="level1Action" />
      <Variable x:TypeArguments="x:String" Name="level1Data" />
      <Variable x:TypeArguments="x:Boolean" Name="level1Approved" />
      <!-- Level 2 -->
      <Variable x:TypeArguments="x:String" Name="level2TaskId" />
      <Variable x:TypeArguments="x:String" Name="level2Action" />
      <Variable x:TypeArguments="x:String" Name="level2Data" />
      <Variable x:TypeArguments="x:Boolean" Name="level2Approved" />
      <!-- Final -->
      <Variable x:TypeArguments="x:Boolean" Name="finalApproved" />
    </Sequence.Variables>
    
    <!-- Level 1: Manager Approval -->
    <Sequence DisplayName="Level 1: Manager Approval">
      <ui:LogMessage Level="Info" Message="Starting Level 1 approval..." />
      
      <ui:CreateFormTask
        DisplayName="Create Manager Task"
        TaskTitle="[String.Format(&quot;Manager Approval: {0}&quot;, in_RequestTitle)]"
        TaskPriority="Normal"
        FormSchemaPath="FormSchemas\ManagerApproval.json"
        TaskDataJson="[in_TaskData]"
        Result="[level1TaskId]">
        <ui:CreateFormTask.TaskAssignee>
          <ui:TaskAssignee Type="User" Value="[in_ManagerEmail]" />
        </ui:CreateFormTask.TaskAssignee>
      </ui:CreateFormTask>
      
      <ui:WaitForFormTaskAndResume
        TaskId="[level1TaskId]"
        TaskAction="[level1Action]"
        TaskData="[level1Data]" />
      
      <Assign>
        <Assign.To>[level1Approved]</Assign.To>
        <Assign.Value>[level1Action = "approve"]</Assign.Value>
      </Assign>
    </Sequence>
    
    <!-- Level 2: Finance Approval (only if Level 1 approved) -->
    <If Condition="[level1Approved]">
      <If.Then>
        <Sequence DisplayName="Level 2: Finance Approval">
          <ui:LogMessage Level="Info" Message="Level 1 approved. Starting Level 2..." />
          
          <ui:CreateFormTask
            DisplayName="Create Finance Task"
            TaskTitle="[String.Format(&quot;Finance Approval: {0}&quot;, in_RequestTitle)]"
            TaskPriority="Normal"
            FormSchemaPath="FormSchemas\FinanceApproval.json"
            TaskDataJson="[level1Data]"
            Result="[level2TaskId]">
            <ui:CreateFormTask.TaskAssignee>
              <ui:TaskAssignee Type="Group" Value="Finance Approvers" />
            </ui:CreateFormTask.TaskAssignee>
          </ui:CreateFormTask>
          
          <ui:WaitForFormTaskAndResume
            TaskId="[level2TaskId]"
            TaskAction="[level2Action]"
            TaskData="[level2Data]" />
          
          <Assign>
            <Assign.To>[level2Approved]</Assign.To>
            <Assign.Value>[level2Action = "approve"]</Assign.Value>
          </Assign>
          
          <Assign>
            <Assign.To>[finalApproved]</Assign.To>
            <Assign.Value>[level2Approved]</Assign.Value>
          </Assign>
        </Sequence>
      </If.Then>
      <If.Else>
        <Sequence DisplayName="Level 1 Rejected">
          <ui:LogMessage Level="Info" Message="Level 1 rejected. Skipping Level 2." />
          <Assign>
            <Assign.To>[finalApproved]</Assign.To>
            <Assign.Value>False</Assign.Value>
          </Assign>
        </Sequence>
      </If.Else>
    </If>
    
    <!-- Final Status -->
    <ui:LogMessage
      Level="Info"
      Message="[String.Format(&quot;Final approval status: {0}&quot;, If(finalApproved, &quot;Approved&quot;, &quot;Rejected&quot;))]" />
    
  </Sequence>
</Activity>
```

---

## Exception Handling Workflow

Workflow for handling process exceptions with human review.

### Main.xaml

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="ExceptionHandler" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:nj="clr-namespace:Newtonsoft.Json.Linq;assembly=Newtonsoft.Json" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <VisualBasic.Settings>
    <x:Null />
  </VisualBasic.Settings>
  <sap2010:WorkflowViewState.IdRef>ActivityBuilder_1</sap2010:WorkflowViewState.IdRef>
  <Sequence DisplayName="Exception Handling Workflow">
    <Sequence.Variables>
      <Variable x:TypeArguments="x:String" Name="taskId" />
      <Variable x:TypeArguments="x:String" Name="action" />
      <Variable x:TypeArguments="x:String" Name="formDataJson" />
      <Variable x:TypeArguments="nj:JObject" Name="formData" />
      <Variable x:TypeArguments="x:String" Name="resolution" />
      <Variable x:TypeArguments="x:String" Name="correctedData" />
      <Variable x:TypeArguments="x:Int32" Name="retryCount" Default="0" />
      <Variable x:TypeArguments="x:Int32" Name="maxRetries" Default="3" />
    </Sequence.Variables>
    
    <!-- Create Exception Task -->
    <ui:CreateFormTask
      DisplayName="Create Exception Task"
      TaskTitle="[String.Format(&quot;Exception: {0}&quot;, in_ErrorType)]"
      TaskPriority="High"
      FormSchemaPath="FormSchemas\ExceptionForm.json"
      TaskDataJson="[String.Format(&quot;{{&quot;&quot;exceptionDetails&quot;&quot;: {{&quot;&quot;errorType&quot;&quot;: &quot;&quot;{0}&quot;&quot;, &quot;&quot;errorMessage&quot;&quot;: &quot;&quot;{1}&quot;&quot;, &quot;&quot;timestamp&quot;&quot;: &quot;&quot;{2}&quot;&quot;, &quot;&quot;affectedRecord&quot;&quot;: &quot;&quot;{3}&quot;&quot;}}}}&quot;, in_ErrorType, in_ErrorMessage, DateTime.Now.ToString(&quot;o&quot;), in_RecordId)]"
      Result="[taskId]">
      <ui:CreateFormTask.TaskAssignee>
        <ui:TaskAssignee Type="Group" Value="Exception Handlers" />
      </ui:CreateFormTask.TaskAssignee>
    </ui:CreateFormTask>
    
    <!-- Wait for Resolution -->
    <ui:WaitForFormTaskAndResume
      TaskId="[taskId]"
      TaskAction="[action]"
      TaskData="[formDataJson]" />
    
    <!-- Parse Response -->
    <Assign>
      <Assign.To>[formData]</Assign.To>
      <Assign.Value>[nj:JObject.Parse(formDataJson)]</Assign.Value>
    </Assign>
    
    <Assign>
      <Assign.To>[resolution]</Assign.To>
      <Assign.Value>[formData("resolution")("action").ToString()]</Assign.Value>
    </Assign>
    
    <!-- Handle Resolution -->
    <Switch Expression="[resolution]">
      
      <!-- Retry -->
      <Case Value="Retry">
        <Sequence DisplayName="Handle Retry">
          <If Condition="[retryCount &lt; maxRetries]">
            <If.Then>
              <Sequence>
                <Assign>
                  <Assign.To>[retryCount]</Assign.To>
                  <Assign.Value>[retryCount + 1]</Assign.Value>
                </Assign>
                <ui:LogMessage
                  Level="Info"
                  Message="[String.Format(&quot;Retrying... Attempt {0} of {1}&quot;, retryCount, maxRetries)]" />
                <!-- Invoke retry logic -->
                <InvokeWorkflowFile
                  WorkflowFileName="Workflows\RetryProcess.xaml">
                  <InvokeWorkflowFile.Arguments>
                    <InArgument x:Key="in_RecordId" x:TypeArguments="x:String">[in_RecordId]</InArgument>
                  </InvokeWorkflowFile.Arguments>
                </InvokeWorkflowFile>
              </Sequence>
            </If.Then>
            <If.Else>
              <ui:LogMessage Level="Error" Message="Max retries exceeded" />
            </If.Else>
          </If>
        </Sequence>
      </Case>
      
      <!-- Skip -->
      <Case Value="Skip">
        <Sequence DisplayName="Handle Skip">
          <ui:LogMessage Level="Warning" Message="Record skipped by user" />
          <ui:AddTransactionComment
            Comment="[String.Format(&quot;Record {0} skipped: {1}&quot;, in_RecordId, formData(&quot;resolution&quot;)(&quot;notes&quot;).ToString())]"
            CommentType="Warning" />
        </Sequence>
      </Case>
      
      <!-- Manual Fix -->
      <Case Value="Manual Fix">
        <Sequence DisplayName="Handle Manual Fix">
          <Assign>
            <Assign.To>[correctedData]</Assign.To>
            <Assign.Value>[formData("resolution")("correctedData").ToString()]</Assign.Value>
          </Assign>
          <ui:LogMessage
            Level="Info"
            Message="[String.Format(&quot;Applying manual fix: {0}&quot;, correctedData)]" />
          <!-- Apply corrected data -->
          <InvokeWorkflowFile
            WorkflowFileName="Workflows\ApplyCorrection.xaml">
            <InvokeWorkflowFile.Arguments>
              <InArgument x:Key="in_RecordId" x:TypeArguments="x:String">[in_RecordId]</InArgument>
              <InArgument x:Key="in_CorrectedData" x:TypeArguments="x:String">[correctedData]</InArgument>
            </InvokeWorkflowFile.Arguments>
          </InvokeWorkflowFile>
        </Sequence>
      </Case>
      
      <!-- Escalate -->
      <Case Value="Escalate">
        <Sequence DisplayName="Handle Escalation">
          <ui:LogMessage Level="Warning" Message="Exception escalated" />
          <InvokeWorkflowFile
            WorkflowFileName="Workflows\EscalateException.xaml">
            <InvokeWorkflowFile.Arguments>
              <InArgument x:Key="in_ExceptionData" x:TypeArguments="x:String">[formDataJson]</InArgument>
            </InvokeWorkflowFile.Arguments>
          </InvokeWorkflowFile>
        </Sequence>
      </Case>
      
    </Switch>
    
  </Sequence>
</Activity>
```

---

## Document Review Workflow

Workflow for document review with multiple reviewers.

### Main.xaml

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="DocumentReview" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <VisualBasic.Settings>
    <x:Null />
  </VisualBasic.Settings>
  <sap2010:WorkflowViewState.IdRef>ActivityBuilder_1</sap2010:WorkflowViewState.IdRef>
  <Sequence DisplayName="Document Review Workflow">
    <Sequence.Variables>
      <Variable x:TypeArguments="x:String" Name="reviewTaskId" />
      <Variable x:TypeArguments="x:String" Name="reviewAction" />
      <Variable x:TypeArguments="x:String" Name="reviewData" />
      <Variable x:TypeArguments="x:String" Name="decision" />
      <Variable x:TypeArguments="scg:List(x:String)" Name="reviewComments" />
    </Sequence.Variables>
    
    <!-- Initialize -->
    <Assign>
      <Assign.To>[reviewComments]</Assign.To>
      <Assign.Value>[New List(Of String)]</Assign.Value>
    </Assign>
    
    <!-- Set Progress -->
    <ui:SetTransactionProgress Progress="0" Status="Starting document review..." />
    
    <!-- Create Review Task -->
    <ui:CreateFormTask
      DisplayName="Create Review Task"
      TaskTitle="[String.Format(&quot;Review: {0}&quot;, in_DocumentTitle)]"
      TaskPriority="Normal"
      FormSchemaPath="FormSchemas\DocumentReview.json"
      TaskDataJson="[String.Format(&quot;{{&quot;&quot;documentInfo&quot;&quot;: {{&quot;&quot;documentId&quot;&quot;: &quot;&quot;{0}&quot;&quot;, &quot;&quot;documentType&quot;&quot;: &quot;&quot;{1}&quot;&quot;, &quot;&quot;submittedBy&quot;&quot;: &quot;&quot;{2}&quot;&quot;, &quot;&quot;submittedDate&quot;&quot;: &quot;&quot;{3}&quot;&quot;}}}}&quot;, in_DocumentId, in_DocumentType, in_SubmittedBy, in_SubmittedDate)]"
      Result="[reviewTaskId]">
      <ui:CreateFormTask.TaskAssignee>
        <ui:TaskAssignee Type="User" Value="[in_ReviewerEmail]" />
      </ui:CreateFormTask.TaskAssignee>
    </ui:CreateFormTask>
    
    <ui:SetTransactionProgress Progress="25" Status="Waiting for review..." />
    
    <!-- Wait for Review -->
    <ui:WaitForFormTaskAndResume
      TaskId="[reviewTaskId]"
      TaskAction="[reviewAction]"
      TaskData="[reviewData]" />
    
    <ui:SetTransactionProgress Progress="75" Status="Processing review..." />
    
    <!-- Process Review -->
    <Assign>
      <Assign.To>[decision]</Assign.To>
      <Assign.Value>[Newtonsoft.Json.Linq.JObject.Parse(reviewData)("finalDecision")("decision").ToString()]</Assign.Value>
    </Assign>
    
    <!-- Handle Decision -->
    <Switch Expression="[decision]">
      
      <Case Value="Approve">
        <Sequence DisplayName="Document Approved">
          <ui:LogMessage Level="Info" Message="Document approved" />
          <ui:AddTransactionComment Comment="Document approved by reviewer" CommentType="Info" />
          <!-- Proceed with approved document -->
          <InvokeWorkflowFile WorkflowFileName="Workflows\ProcessApprovedDocument.xaml" />
        </Sequence>
      </Case>
      
      <Case Value="Reject">
        <Sequence DisplayName="Document Rejected">
          <ui:LogMessage Level="Warning" Message="Document rejected" />
          <ui:AddTransactionComment Comment="Document rejected by reviewer" CommentType="Warning" />
          <!-- Notify submitter -->
          <InvokeWorkflowFile WorkflowFileName="Workflows\NotifyRejection.xaml" />
        </Sequence>
      </Case>
      
      <Case Value="Request Revision">
        <Sequence DisplayName="Revision Requested">
          <ui:LogMessage Level="Info" Message="Revision requested" />
          <ui:AddTransactionComment Comment="Revision requested by reviewer" CommentType="Info" />
          <!-- Create revision task for submitter -->
          <InvokeWorkflowFile WorkflowFileName="Workflows\RequestRevision.xaml" />
        </Sequence>
      </Case>
      
    </Switch>
    
    <ui:SetTransactionProgress Progress="100" Status="Review complete" />
    
  </Sequence>
</Activity>
```

---

## Escalation Workflow

Workflow with automatic escalation after timeout.

### Main.xaml

```xml
<Activity mc:Ignorable="sap sap2010" x:Class="EscalationWorkflow" xmlns="http://schemas.microsoft.com/netfx/2009/xaml/activities" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" xmlns:sap="http://schemas.microsoft.com/netfx/2009/xaml/activities/presentation" xmlns:sap2010="http://schemas.microsoft.com/netfx/2010/xaml/activities/presentation" xmlns:scg="clr-namespace:System.Collections.Generic;assembly=System.Private.CoreLib" xmlns:sco="clr-namespace:System.Collections.ObjectModel;assembly=System.Private.CoreLib" xmlns:this="clr-namespace:" xmlns:ui="http://schemas.uipath.com/workflow/activities" xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <VisualBasic.Settings>
    <x:Null />
  </VisualBasic.Settings>
  <sap2010:WorkflowViewState.IdRef>ActivityBuilder_1</sap2010:WorkflowViewState.IdRef>
  <Sequence DisplayName="Escalation Workflow">
    <Sequence.Variables>
      <Variable x:TypeArguments="x:String" Name="taskId" />
      <Variable x:TypeArguments="x:String" Name="taskStatus" />
      <Variable x:TypeArguments="x:String" Name="taskData" />
      <Variable x:TypeArguments="x:String" Name="action" />
      <Variable x:TypeArguments="x:DateTime" Name="taskCreatedTime" />
      <Variable x:TypeArguments="x:Int32" Name="escalationLevel" Default="0" />
      <Variable x:TypeArguments="x:Boolean" Name="taskCompleted" Default="False" />
    </Sequence.Variables>
    
    <!-- Create Initial Task -->
    <ui:CreateFormTask
      DisplayName="Create Initial Task"
      TaskTitle="[in_TaskTitle]"
      TaskPriority="Normal"
      FormSchemaPath="FormSchemas\ApprovalForm.json"
      TaskDataJson="[in_TaskData]"
      Result="[taskId]">
      <ui:CreateFormTask.TaskAssignee>
        <ui:TaskAssignee Type="User" Value="[in_InitialAssignee]" />
      </ui:CreateFormTask.TaskAssignee>
    </ui:CreateFormTask>
    
    <Assign>
      <Assign.To>[taskCreatedTime]</Assign.To>
      <Assign.Value>[DateTime.Now]</Assign.Value>
    </Assign>
    
    <!-- Monitor and Escalate Loop -->
    <DoWhile Condition="[Not taskCompleted]">
      <Sequence DisplayName="Monitor Task">
        
        <!-- Check Task Status -->
        <ui:GetFormTaskData
          TaskId="[taskId]"
          TaskStatus="[taskStatus]"
          TaskData="[taskData]" />
        
        <If Condition="[taskStatus = &quot;Completed&quot;]">
          <If.Then>
            <Sequence DisplayName="Task Completed">
              <Assign>
                <Assign.To>[taskCompleted]</Assign.To>
                <Assign.Value>True</Assign.Value>
              </Assign>
              <ui:LogMessage Level="Info" Message="Task completed" />
            </Sequence>
          </If.Then>
          <If.Else>
            <Sequence DisplayName="Check Escalation">
              
              <!-- Check if escalation needed -->
              <If Condition="[DateTime.Now.Subtract(taskCreatedTime).TotalHours &gt; 24 And escalationLevel = 0]">
                <If.Then>
                  <Sequence DisplayName="Level 1 Escalation">
                    <ui:LogMessage Level="Warning" Message="Escalating to Level 1" />
                    
                    <!-- Reassign to manager -->
                    <ui:CompleteTask
                      TaskId="[taskId]"
                      Action="escalate"
                      TaskData="{}{&quot;reason&quot;: &quot;Timeout - escalated to manager&quot;}" />
                    
                    <!-- Create new task for manager -->
                    <ui:CreateFormTask
                      TaskTitle="[String.Format(&quot;ESCALATED: {0}&quot;, in_TaskTitle)]"
                      TaskPriority="High"
                      FormSchemaPath="FormSchemas\ApprovalForm.json"
                      TaskDataJson="[taskData]"
                      Result="[taskId]">
                      <ui:CreateFormTask.TaskAssignee>
                        <ui:TaskAssignee Type="User" Value="[in_ManagerEmail]" />
                      </ui:CreateFormTask.TaskAssignee>
                    </ui:CreateFormTask>
                    
                    <Assign>
                      <Assign.To>[escalationLevel]</Assign.To>
                      <Assign.Value>1</Assign.Value>
                    </Assign>
                    <Assign>
                      <Assign.To>[taskCreatedTime]</Assign.To>
                      <Assign.Value>[DateTime.Now]</Assign.Value>
                    </Assign>
                  </Sequence>
                </If.Then>
              </If>
              
              <!-- Level 2 Escalation -->
              <If Condition="[DateTime.Now.Subtract(taskCreatedTime).TotalHours &gt; 24 And escalationLevel = 1]">
                <If.Then>
                  <Sequence DisplayName="Level 2 Escalation">
                    <ui:LogMessage Level="Error" Message="Escalating to Level 2 - Director" />
                    
                    <ui:CompleteTask
                      TaskId="[taskId]"
                      Action="escalate"
                      TaskData="{}{&quot;reason&quot;: &quot;Timeout - escalated to director&quot;}" />
                    
                    <ui:CreateFormTask
                      TaskTitle="[String.Format(&quot;URGENT: {0}&quot;, in_TaskTitle)]"
                      TaskPriority="Critical"
                      FormSchemaPath="FormSchemas\ApprovalForm.json"
                      TaskDataJson="[taskData]"
                      Result="[taskId]">
                      <ui:CreateFormTask.TaskAssignee>
                        <ui:TaskAssignee Type="User" Value="[in_DirectorEmail]" />
                      </ui:CreateFormTask.TaskAssignee>
                    </ui:CreateFormTask>
                    
                    <Assign>
                      <Assign.To>[escalationLevel]</Assign.To>
                      <Assign.Value>2</Assign.Value>
                    </Assign>
                    <Assign>
                      <Assign.To>[taskCreatedTime]</Assign.To>
                      <Assign.Value>[DateTime.Now]</Assign.Value>
                    </Assign>
                  </Sequence>
                </If.Then>
              </If>
              
              <!-- Wait before next check -->
              <Delay Duration="[TimeSpan.FromHours(1)]" />
              
            </Sequence>
          </If.Else>
        </If>
        
      </Sequence>
    </DoWhile>
    
    <!-- Process Final Result -->
    <ui:WaitForFormTaskAndResume
      TaskId="[taskId]"
      TaskAction="[action]"
      TaskData="[taskData]" />
    
    <ui:LogMessage
      Level="Info"
      Message="[String.Format(&quot;Task completed with action: {0}, escalation level: {1}&quot;, action, escalationLevel)]" />
    
  </Sequence>
</Activity>
```
