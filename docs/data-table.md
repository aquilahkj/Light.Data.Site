# Data Table

## Data Table Alias

`DataContext` can temporarily set the Data table alias for mapping type Data, For special scenarios, For example, divide the log table horizontally. The alias only applies to the `DataContext` operation.

| Method | Introduce |
|:------|:------|
| GetTableName<T>() | Gets the name of the corresponding data table of the specified type `T` |
| SetAliasTableName<T>(string name) | Sets the alias for the specified type `T` |
| ResetAliasTableName<T>() | Clears the alias for the specified type `T` |
| ClearAliasTableName() | Clear all aliases in this `DataContext` |

Set the alias and query

```csharp
DataContext context = new DataContext();
// gets the table name Te_Log
var tableName = context.GetTableName<TeLog>();
// alias Te_Log_201709
var aliasName = tableName + "_201709";
// set up alias
context.SetAliasTableName<TeLog>(aliasName);
// query table Te_Log_201709
var list = context.Query<TeLog>().ToList();
```

## Truncate Data Table

```csharp
context.TruncateTable<TeUser> (); 
```

Notice: this operation directly using `truncate table` command to truncate the data tables.