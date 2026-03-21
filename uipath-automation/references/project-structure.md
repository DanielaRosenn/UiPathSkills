# UiPath Project Structure Reference

Complete guide to UiPath project configuration, dependencies, and file organization. Official reference: [About the project.json file](https://docs.uipath.com/studio/standalone/2024.10/user-guide/about-the-projectjson-file) (adjust Studio version in the URL if needed).

## Table of Contents
1. [Project Types](#project-types)
2. [Project.json Configuration](#projectjson-configuration)
3. [File Organization](#file-organization)
4. [Dependencies](#dependencies)
5. [Settings & Config](#settings--config)
6. [REFramework Structure](#reframework-structure)

---

## Project Types

### Process (Standard Automation)
Most common project type for attended/unattended automation.

```json
{
  "name": "MyAutomation",
  "projectType": "Workflow",
  "main": "Main.xaml",
  "targetFramework": "Windows"
}
```

### Library (Reusable Components)
Shared activities and workflows published to Orchestrator/feed.

```json
{
  "name": "MyLibrary",
  "projectType": "Library",
  "main": "Main.xaml",
  "targetFramework": "Windows",
  "libraryOptions": {
    "includeOriginalXaml": false,
    "privateWorkflows": []
  }
}
```

### Test Automation
Testing framework project.

```json
{
  "name": "MyTests",
  "projectType": "TestAutomation",
  "main": "Main.xaml",
  "targetFramework": "Windows"
}
```

---

## Project.json Configuration

### Complete Project.json Template

```json
{
  "name": "ProcessName",
  "description": "Automation process description",
  "main": "Main.xaml",
  "projectVersion": "1.0.0",
  "projectType": "Workflow",
  "targetFramework": "Windows",
  "expressionLanguage": "VisualBasic",
  "entryPoints": [
    {
      "filePath": "Main.xaml",
      "uniqueId": "00000000-0000-0000-0000-000000000000",
      "input": [],
      "output": []
    }
  ],
  "dependencies": {
    "UiPath.Excel.Activities": "[2.22.0]",
    "UiPath.Mail.Activities": "[1.21.0]",
    "UiPath.System.Activities": "[23.10.0]",
    "UiPath.UIAutomation.Activities": "[23.10.0]"
  },
  "webServices": [],
  "schemaVersion": "4.0",
  "studioVersion": "23.10.0.0",
  "projectId": "00000000-0000-0000-0000-000000000000",
  "runtimeOptions": {
    "autoDispose": false,
    "netFrameworkLazyLoading": false,
    "isPausable": true,
    "isAttended": false,
    "requiresUserInteraction": true,
    "supportsPersistence": false,
    "workflowSerialization": "NewtonsoftJson",
    "excludedLoggedData": ["Private:*", "*password*"],
    "executionType": "Workflow",
    "readyForPictureInPicture": false,
    "startsInPictureInPicture": false,
    "isTestProject": false
  },
  "designOptions": {
    "projectProfile": "Developement",
    "outputType": "Process",
    "libraryOptions": {
      "includeOriginalXaml": false,
      "privateWorkflows": []
    },
    "fileInfoCollection": []
  },
  "toolVersion": "1.0.0.0"
}
```

### Key Configuration Options

#### Target Framework
```json
{
  "targetFramework": "Windows"   // Windows-Legacy, Portable
}
```

| Framework | Description | .NET Version |
|-----------|-------------|--------------|
| Windows | Modern Windows projects | .NET 6+ |
| Windows-Legacy | Legacy compatibility | .NET Framework 4.6.1 |
| Portable | Cross-platform | .NET 6+ |

#### Expression Language
```json
{
  "expressionLanguage": "VisualBasic"  // or "CSharp"
}
```

#### Entry Points
Define multiple entry points for modular execution:
```json
{
  "entryPoints": [
    {
      "filePath": "Main.xaml",
      "uniqueId": "guid-here",
      "input": [
        {
          "name": "in_ConfigPath",
          "type": "System.String",
          "required": true
        }
      ],
      "output": [
        {
          "name": "out_Status",
          "type": "System.String"
        }
      ]
    },
    {
      "filePath": "Dispatcher.xaml",
      "uniqueId": "another-guid",
      "input": [],
      "output": []
    }
  ]
}
```

#### Runtime Options
```json
{
  "runtimeOptions": {
    "autoDispose": false,
    "isPausable": true,
    "isAttended": false,
    "requiresUserInteraction": true,
    "supportsPersistence": false,
    "excludedLoggedData": [
      "Private:*",
      "*password*",
      "*secret*"
    ]
  }
}
```

---

## File Organization

### Standard Project Structure
```
ProjectName/
├── project.json                 # Project configuration
├── Main.xaml                    # Entry point workflow
├── .local/                      # Local settings (git-ignored)
│   └── settings.json
├── .screenshots/                # UI element screenshots
├── .objects/                    # Object repository
│   └── .metadata
├── Data/                        # Data files
│   ├── Config.xlsx              # Configuration
│   └── Templates/               # Document templates
├── Workflows/                   # Modular workflows
│   ├── Common/                  # Shared utilities
│   │   ├── InitAllSettings.xaml
│   │   ├── KillAllProcesses.xaml
│   │   └── TakeScreenshot.xaml
│   ├── Framework/               # Framework workflows
│   │   ├── InitAllApplications.xaml
│   │   ├── CloseAllApplications.xaml
│   │   └── GetTransactionData.xaml
│   └── Process/                 # Business logic
│       ├── ProcessTransaction.xaml
│       └── ValidateData.xaml
├── Tests/                       # Test workflows
│   └── TestProcessTransaction.xaml
└── Exceptions/                  # Custom exception definitions
    └── BusinessExceptions.xaml
```

### REFramework Project Structure
```
REFramework/
├── project.json
├── Main.xaml                    # State machine entry
├── Data/
│   ├── Config.xlsx              # Configuration workbook
│   └── Input/                   # Input data folder
├── Framework/
│   ├── InitAllSettings.xaml     # Load config
│   ├── InitAllApplications.xaml # Open apps
│   ├── CloseAllApplications.xaml
│   ├── KillAllProcesses.xaml
│   ├── GetTransactionData.xaml  # Get queue item
│   ├── Process.xaml             # Process wrapper
│   ├── SetTransactionStatus.xaml
│   └── TakeScreenshot.xaml
├── ExceptionScreenshots/        # Error screenshots
└── Documentation/
    └── PDD.docx                 # Process Definition Document
```

---

## Dependencies

### Common Package Dependencies

```json
{
  "dependencies": {
    "UiPath.System.Activities": "[23.10.0]",
    "UiPath.UIAutomation.Activities": "[23.10.0]",
    "UiPath.Excel.Activities": "[2.22.0]",
    "UiPath.Mail.Activities": "[1.21.0]",
    "UiPath.Database.Activities": "[1.8.0]",
    "UiPath.WebAPI.Activities": "[1.17.0]",
    "UiPath.Credentials.Activities": "[2.0.0]",
    "UiPath.PDF.Activities": "[3.14.0]",
    "UiPath.Word.Activities": "[1.17.0]",
    "UiPath.Presentations.Activities": "[1.12.0]"
  }
}
```

### Version Notation

| Notation | Meaning |
|----------|---------|
| `[1.0.0]` | Exact version 1.0.0 |
| `[1.0.0, )` | Version 1.0.0 or higher |
| `[1.0.0, 2.0.0]` | Between 1.0.0 and 2.0.0 inclusive |
| `(1.0.0, 2.0.0)` | Between 1.0.0 and 2.0.0 exclusive |

### Package Categories

**Core Activities:**
- `UiPath.System.Activities` - Core workflow activities
- `UiPath.UIAutomation.Activities` - UI automation

**Application Integration:**
- `UiPath.Excel.Activities` - Excel automation
- `UiPath.Mail.Activities` - Email (Outlook, IMAP, SMTP)
- `UiPath.Database.Activities` - Database connectivity
- `UiPath.WebAPI.Activities` - HTTP/REST calls

**Document Processing:**
- `UiPath.PDF.Activities` - PDF handling
- `UiPath.Word.Activities` - Word documents
- `UiPath.Presentations.Activities` - PowerPoint

**Modern Experience:**
- `UiPath.UIAutomationNext.Activities` - Modern UI automation
- `UiPath.Excel.WorkBook.Activities` - Modern Excel

**AI/ML:**
- `UiPath.DocumentUnderstanding.ML.Activities`
- `UiPath.IntelligentOCR.Activities`

---

## Settings & Config

### Config.xlsx Structure
Standard configuration workbook format:

**Settings Sheet:**
| Name | Value | Description |
|------|-------|-------------|
| logF_BusinessProcessName | MyProcess | Process name for logging |
| OrchestratorQueueName | ProcessingQueue | Queue name |
| MaxRetryNumber | 3 | Transaction retry limit |
| logF_ApplicationName | SAP,Excel | Applications used |

**Constants Sheet:**
| Name | Value | Description |
|------|-------|-------------|
| MaxWaitTime | 30 | Timeout in seconds |
| EmailRecipient | team@company.com | Notification recipient |
| OutputFolder | C:\Output | Output directory |

**Assets Sheet:**
| Name | Asset | Description |
|------|-------|-------------|
| AppCredential | AppCredential | Orchestrator credential name |
| APIKey | APIKeyAsset | Orchestrator asset name |

### Loading Configuration
```xml
<ui:ReadRange DisplayName="Read Settings" SheetName="Settings" Range="" 
  AddHeaders="True" DataTable="[dtSettings]" WorkbookPath="Data\Config.xlsx" />

<!-- Convert to Dictionary -->
<ui:InvokeCode Language="VisualBasic" DisplayName="Create Config Dictionary">
  <ui:InvokeCode.Code>
    dictConfig = dtSettings.AsEnumerable().ToDictionary(
      Function(row) row("Name").ToString,
      Function(row) row("Value")
    )
  </ui:InvokeCode.Code>
  <ui:InvokeCode.Arguments>
    <InArgument x:TypeArguments="sd:DataTable" x:Key="dtSettings">[dtSettings]</InArgument>
    <OutArgument x:TypeArguments="scg:Dictionary(x:String, x:Object)" x:Key="dictConfig">[dictConfig]</OutArgument>
  </ui:InvokeCode.Arguments>
</ui:InvokeCode>
```

---

## REFramework Structure

### State Machine Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      Main.xaml                               │
│  ┌─────────┐    ┌────────────┐    ┌────────────────────┐    │
│  │  Init   │───▶│ Get Trans  │───▶│ Process Transaction│    │
│  │ State   │    │   Data     │    │       State        │    │
│  └────┬────┘    └─────┬──────┘    └──────────┬─────────┘    │
│       │               │                       │              │
│       ▼               ▼                       ▼              │
│  ┌─────────┐    No Trans        Success/Business Exception   │
│  │  End    │◀────────────────────────────────────────────────│
│  │ Process │                                                 │
│  └─────────┘◀──── System Exception (retry limit reached)     │
└─────────────────────────────────────────────────────────────┘
```

### State Workflows

**Init State:**
```xml
<!-- InitAllSettings.xaml -->
<x:Members>
  <x:Property Name="in_ConfigPath" Type="InArgument(x:String)" />
  <x:Property Name="out_Config" Type="OutArgument(scg:Dictionary(x:String, x:Object))" />
</x:Members>
```

**Get Transaction Data:**
```xml
<!-- GetTransactionData.xaml -->
<x:Members>
  <x:Property Name="in_Config" Type="InArgument(scg:Dictionary(x:String, x:Object))" />
  <x:Property Name="io_TransactionNumber" Type="InOutArgument(x:Int32)" />
  <x:Property Name="out_TransactionItem" Type="OutArgument(ui:QueueItem)" />
</x:Members>
```

**Process Transaction:**
```xml
<!-- Process.xaml -->
<x:Members>
  <x:Property Name="in_TransactionItem" Type="InArgument(ui:QueueItem)" />
  <x:Property Name="in_Config" Type="InArgument(scg:Dictionary(x:String, x:Object))" />
</x:Members>
```

### Config Dictionary Usage
```xml
<!-- Accessing config values -->
<Assign DisplayName="Get Queue Name">
  <Assign.To><OutArgument x:TypeArguments="x:String">[strQueueName]</OutArgument></Assign.To>
  <Assign.Value><InArgument x:TypeArguments="x:String">[dictConfig("OrchestratorQueueName").ToString]</InArgument></Assign.Value>
</Assign>

<!-- With type conversion -->
<Assign DisplayName="Get Max Retry">
  <Assign.To><OutArgument x:TypeArguments="x:Int32">[intMaxRetry]</OutArgument></Assign.To>
  <Assign.Value><InArgument x:TypeArguments="x:Int32">[Convert.ToInt32(dictConfig("MaxRetryNumber"))]</InArgument></Assign.Value>
</Assign>
```

### Exception Handling Strategy

| Exception Type | Handling | Retry? |
|---------------|----------|--------|
| `BusinessRuleException` | Log, skip transaction | No |
| `System.Exception` | Log, retry | Yes (up to limit) |
| `ApplicationException` | Restart apps, retry | Yes |

```xml
<!-- Throwing Business Rule Exception -->
<Throw DisplayName="Invalid Data">
  <Throw.Exception>
    <InArgument x:TypeArguments="s:Exception">
      [New UiPath.Core.BusinessRuleException("Missing required field: CustomerID")]
    </InArgument>
  </Throw.Exception>
</Throw>
```
