# Module: System & File Activities
**Package:** UiPath.System.Activities | **Covers:** File, Folder, Path, Dialogs, Process, Zip, Clipboard, Environment

---

## File Activities

```xml
<!-- Copy File -->
<ui:CopyFile DisplayName="Copy to Archive" From="[sourcePath]" To="[destPath]" Overwrite="True"/>

<!-- Move File -->
<ui:MoveFile DisplayName="Move Processed File" From="[filePath]"
  To="[Path.Combine(archiveFolder, Path.GetFileName(filePath))]" Overwrite="True"/>

<!-- Delete File -->
<ui:DeleteFile DisplayName="Delete Temp File" Path="[tempPath]"/>

<!-- File Exists -->
<ui:PathExists DisplayName="Check File Exists" Path="[filePath]" Exists="[fileExists]" ItemType="File"/>

<!-- Read Text File -->
<ui:ReadTextFile DisplayName="Read Config" FileName="[configPath]"
  Encoding="System.Text.Encoding.UTF8" Content="[fileContent]"/>

<!-- Write Text File -->
<ui:WriteTextFile DisplayName="Write Log" FileName="[logPath]"
  Text="[logLine]" Encoding="System.Text.Encoding.UTF8" Append="True"/>

<!-- Get File Info -->
<ui:GetFileInfo DisplayName="Get File Details" Path="[filePath]" FileInfo="[fi]"/>
<!-- fi.Name, fi.Length, fi.LastWriteTime, fi.Extension, fi.DirectoryName -->
```

---

## Folder Activities

```xml
<!-- Create Folder -->
<ui:CreateDirectory DisplayName="Create Output Folder" Path="[outputFolder]"/>

<!-- Get Files in Folder -->
<ui:GetFiles DisplayName="Get PDF Files" Path="[inputFolder]"
  Filter="&quot;*.pdf&quot;" IncludeSubfolders="False" Files="[fileList]"/>

<!-- Get Subfolders -->
<ui:GetFolders DisplayName="Get Subfolders" Path="[rootFolder]" Folders="[subfolders]"/>

<!-- Delete Folder -->
<ui:DeleteDirectory DisplayName="Delete Temp Folder" Path="[tempFolder]" Recursive="True"/>
```

---

## Path Expressions (VB.NET)

```vb
Path.Combine(folder, fileName)               ' safe path joining
Path.GetFileName(fullPath)                   ' "report.xlsx"
Path.GetFileNameWithoutExtension(fullPath)   ' "report"
Path.GetExtension(fullPath)                  ' ".xlsx"
Path.GetDirectoryName(fullPath)              ' "C:\Reports"
Path.GetFullPath(relativePath)               ' resolve relative
System.IO.File.Exists(path)                  ' boolean (use in expressions)
System.IO.Directory.Exists(path)             ' boolean

' Special folders
Environment.GetFolderPath(Environment.SpecialFolder.Desktop)
Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments)
Environment.GetFolderPath(Environment.SpecialFolder.LocalApplicationData)
Environment.GetEnvironmentVariable("TEMP")
Environment.MachineName
Environment.UserName
```

---

## Zip / Unzip

```xml
<!-- Compress -->
<ui:Compress DisplayName="Zip Output Folder" FromPath="[outputFolder]"
  ZipFileName="[zipPath]" Password="[zipPwd]"/>

<!-- Extract -->
<ui:Decompress DisplayName="Unzip Archive" ZipFileName="[zipPath]"
  ToPath="[extractPath]" Password="[zipPwd]" Overwrite="True"/>
```

---

## Process Control

```xml
<!-- Start external program -->
<ui:StartProcess DisplayName="Open PDF" FileName="[pdfPath]" WaitForExit="False"/>

<!-- Kill process by name -->
<ui:KillProcess DisplayName="Close Chrome" ProcessName="&quot;chrome&quot;"/>
```

---

## Dialogs & Clipboard

```xml
<!-- File picker dialog -->
<ui:SelectFile DisplayName="Pick Input File"
  Filter="&quot;Excel (*.xlsx)|*.xlsx|All (*.*)|*.*&quot;" FilePath="[selected]"/>

<!-- Folder picker -->
<ui:SelectFolder DisplayName="Pick Output Folder" FolderPath="[selected]"/>

<!-- Clipboard -->
<ui:SetToClipboard DisplayName="Copy to Clipboard" Value="[text]"/>
<ui:GetFromClipboard DisplayName="Read Clipboard" Text="[clipText]"/>
```

---

## Common Patterns

### Archive processed file with datestamp
```vb
archivePath = Path.Combine(archiveFolder, DateTime.Now.ToString("yyyyMMdd") & "_" & Path.GetFileName(processedFile))
```
Then `CreateDirectory` → `MoveFile`.

### Batch process all files in folder
```xml
<ui:GetFiles Path="[inputFolder]" Filter="&quot;*.xlsx&quot;" Files="[files]"/>
<ForEach x:TypeArguments="x:String" Values="[files]">
  <ActivityAction x:TypeArguments="x:String">
    <ActivityAction.Argument>
      <DelegateInArgument x:TypeArguments="x:String" Name="filePath"/>
    </ActivityAction.Argument>
    <ui:InvokeWorkflowFile WorkflowFileName="&quot;ProcessFile.xaml&quot;">
      <ui:InvokeWorkflowFile.Arguments>
        <InArgument x:TypeArguments="x:String" x:Key="in_FilePath">[filePath]</InArgument>
      </ui:InvokeWorkflowFile.Arguments>
    </ui:InvokeWorkflowFile>
  </ActivityAction>
</ForEach>
```
