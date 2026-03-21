# Module: Database Activities
**Package:** UiPath.Database.Activities | **Covers:** Connect, Execute Query, Execute Non-Query, Stored Procedures, Transactions

---

## Connect to Database

```xml
<ui:DatabaseConnect sap2010:WorkflowViewState.IdRef="DBConnect_1"
    DisplayName="Connect to SQL Server"
    ProviderName="System.Data.SqlClient"
    ConnectionString="[in_Config(&quot;DBConnectionString&quot;).ToString()]"
    DatabaseConnection="[dbConnection]"/>
```

**Connection strings by provider:**

| Database | ProviderName | Connection String Pattern |
|---|---|---|
| SQL Server | `System.Data.SqlClient` | `Server=host;Database=db;User Id=user;Password=pwd;` |
| SQL Server (Windows auth) | `System.Data.SqlClient` | `Server=host;Database=db;Integrated Security=True;` |
| Oracle | `Oracle.ManagedDataAccess.Client` | `Data Source=host/SID;User Id=user;Password=pwd;` |
| MySQL | `MySql.Data.MySqlClient` | `Server=host;Database=db;Uid=user;Pwd=pwd;` |
| PostgreSQL | `Npgsql` | `Host=host;Database=db;Username=user;Password=pwd;` |
| SQLite | `System.Data.SQLite` | `Data Source=C:\path\to\db.sqlite;Version=3;` |
| OLEDB (Excel, Access) | `System.Data.OleDb` | `Provider=Microsoft.ACE.OLEDB.12.0;Data Source=file.xlsx;` |

**NEVER hardcode connection strings.** Store in Orchestrator Assets or Config.xlsx (encrypted field).

---

## Execute Query (SELECT → DataTable)

```xml
<ui:ExecuteQuery sap2010:WorkflowViewState.IdRef="Query_1"
    DisplayName="Get Pending Invoices"
    DatabaseConnection="[dbConnection]"
    Sql="&quot;SELECT InvoiceID, VendorName, Amount, DueDate FROM Invoices WHERE Status = @status AND Amount &gt; @minAmount ORDER BY DueDate&quot;"
    DataTable="[dtInvoices]">
  <ui:ExecuteQuery.Parameters>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Object x:Key="status">[filterStatus]</x:Object>
      <x:Object x:Key="minAmount">[minimumAmount]</x:Object>
    </scg:Dictionary>
  </ui:ExecuteQuery.Parameters>
</ui:ExecuteQuery>
```

**Always use parameterized queries** — never string-concatenate SQL. Prevents SQL injection and handles special characters.

### Stored Procedure (returns results)
```xml
<ui:ExecuteQuery sap2010:WorkflowViewState.IdRef="StoredProc_1"
    DisplayName="Run SP: Get Monthly Summary"
    DatabaseConnection="[dbConnection]"
    CommandType="StoredProcedure"
    Sql="&quot;usp_GetMonthlySummary&quot;"
    DataTable="[dtSummary]">
  <ui:ExecuteQuery.Parameters>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Object x:Key="Year">[reportYear]</x:Object>
      <x:Object x:Key="Month">[reportMonth]</x:Object>
    </scg:Dictionary>
  </ui:ExecuteQuery.Parameters>
</ui:ExecuteQuery>
```

---

## Execute Non-Query (INSERT / UPDATE / DELETE)

```xml
<!-- INSERT -->
<ui:ExecuteNonQuery sap2010:WorkflowViewState.IdRef="Insert_1"
    DisplayName="Insert Processing Log"
    DatabaseConnection="[dbConnection]"
    Sql="&quot;INSERT INTO ProcessingLog (InvoiceID, Status, ProcessedAt, RobotName) VALUES (@id, @status, @ts, @robot)&quot;"
    AffectedRecords="[rowsInserted]">
  <ui:ExecuteNonQuery.Parameters>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Object x:Key="id">[invoiceId]</x:Object>
      <x:Object x:Key="status">[processingStatus]</x:Object>
      <x:Object x:Key="ts">[DateTime.Now]</x:Object>
      <x:Object x:Key="robot">[Environment.MachineName]</x:Object>
    </scg:Dictionary>
  </ui:ExecuteNonQuery.Parameters>
</ui:ExecuteNonQuery>

<!-- UPDATE -->
<ui:ExecuteNonQuery sap2010:WorkflowViewState.IdRef="Update_1"
    DisplayName="Update Invoice Status"
    DatabaseConnection="[dbConnection]"
    Sql="&quot;UPDATE Invoices SET Status = @newStatus, UpdatedAt = @now, ErrorMsg = @err WHERE InvoiceID = @id&quot;"
    AffectedRecords="[rowsUpdated]">
  <ui:ExecuteNonQuery.Parameters>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Object x:Key="newStatus">[statusValue]</x:Object>
      <x:Object x:Key="now">[DateTime.Now]</x:Object>
      <x:Object x:Key="err">[errorMessage]</x:Object>
      <x:Object x:Key="id">[invoiceId]</x:Object>
    </scg:Dictionary>
  </ui:ExecuteNonQuery.Parameters>
</ui:ExecuteNonQuery>

<!-- DELETE -->
<ui:ExecuteNonQuery sap2010:WorkflowViewState.IdRef="Delete_1"
    DisplayName="Delete Old Records"
    DatabaseConnection="[dbConnection]"
    Sql="&quot;DELETE FROM TempStaging WHERE CreatedAt &lt; @cutoff&quot;"
    AffectedRecords="[rowsDeleted]">
  <ui:ExecuteNonQuery.Parameters>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Object x:Key="cutoff">[DateTime.Now.AddDays(-30)]</x:Object>
    </scg:Dictionary>
  </ui:ExecuteNonQuery.Parameters>
</ui:ExecuteNonQuery>
```

---

## Execute Stored Procedure (no result set)

```xml
<ui:ExecuteNonQuery sap2010:WorkflowViewState.IdRef="SP_NoResult_1"
    DisplayName="SP: Archive Processed Records"
    DatabaseConnection="[dbConnection]"
    CommandType="StoredProcedure"
    Sql="&quot;usp_ArchiveProcessed&quot;"
    AffectedRecords="[archivedCount]">
  <ui:ExecuteNonQuery.Parameters>
    <scg:Dictionary x:TypeArguments="x:String, x:Object">
      <x:Object x:Key="BatchID">[currentBatchId]</x:Object>
      <x:Object x:Key="ArchivedBy">[&quot;RPA_Robot&quot;]</x:Object>
    </scg:Dictionary>
  </ui:ExecuteNonQuery.Parameters>
</ui:ExecuteNonQuery>
```

---

## Disconnect

```xml
<ui:DatabaseDisconnect sap2010:WorkflowViewState.IdRef="DBDisconn_1"
    DisplayName="Close DB Connection"
    DatabaseConnection="[dbConnection]"/>
```

**Always disconnect in Finally block:**

```xml
<TryCatch DisplayName="DB Operations with Cleanup">
  <TryCatch.Try>
    <Sequence DisplayName="DB Work">
      <ui:DatabaseConnect DisplayName="Connect" DatabaseConnection="[dbConnection]"
        ProviderName="System.Data.SqlClient"
        ConnectionString="[connStr]"/>
      <ui:ExecuteQuery DisplayName="Query Data" DatabaseConnection="[dbConnection]"
        Sql="&quot;SELECT * FROM Orders&quot;" DataTable="[dtOrders]"/>
      <!-- ... process data ... -->
    </Sequence>
  </TryCatch.Try>
  <TryCatch.Catches>
    <Catch x:TypeArguments="s:Exception">
      <ActivityAction x:TypeArguments="s:Exception">
        <ActivityAction.Argument>
          <DelegateInArgument x:TypeArguments="s:Exception" Name="dbEx"/>
        </ActivityAction.Argument>
        <Sequence>
          <ui:LogMessage Level="Error" Message="[&quot;DB error: &quot; + dbEx.Message]"/>
          <Rethrow/>
        </Sequence>
      </ActivityAction>
    </Catch>
  </TryCatch.Catches>
  <TryCatch.Finally>
    <Sequence DisplayName="Cleanup">
      <If Condition="[dbConnection IsNot Nothing]">
        <If.Then>
          <ui:DatabaseDisconnect DisplayName="Disconnect" DatabaseConnection="[dbConnection]"/>
        </If.Then>
      </If>
    </Sequence>
  </TryCatch.Finally>
</TryCatch>
```

---

## DataTable from DB — Working with Results

```vb
' Check if results returned
If dtOrders.Rows.Count = 0 Then
    LogMessage("No pending orders found")
End If

' Read cell values (always .ToString() + null check)
Dim orderId As String = If(IsDBNull(row("OrderID")), "", row("OrderID").ToString())
Dim amount As Double = If(IsDBNull(row("Amount")), 0.0, CDbl(row("Amount")))
Dim dueDate As DateTime = If(IsDBNull(row("DueDate")), DateTime.MinValue, CDate(row("DueDate")))

' Iterate rows
For Each row As DataRow In dtOrders.Rows
    Dim id = row("OrderID").ToString()
    ' process...
Next
```

---

## Bulk Insert (DataTable → DB)

Use Invoke Code for bulk inserts (no native bulk insert activity):

```vb
' In Invoke Code — bulk insert DataTable to SQL Server
Dim conn As New System.Data.SqlClient.SqlConnection(connectionString)
conn.Open()

Using bulk As New System.Data.SqlClient.SqlBulkCopy(conn)
    bulk.DestinationTableName = "ProcessingResults"
    ' Map columns: DataTable column → DB column
    bulk.ColumnMappings.Add("OrderID", "OrderID")
    bulk.ColumnMappings.Add("Status", "Status")
    bulk.ColumnMappings.Add("Amount", "Amount")
    bulk.BatchSize = 500
    bulk.WriteToServer(dtResults)
End Using

conn.Close()
rowsInserted = dtResults.Rows.Count
```
