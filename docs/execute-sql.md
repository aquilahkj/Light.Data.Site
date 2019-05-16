# Execute SQL

## SQL Executor

When there is a more complex SQL commands `Light.Data` cannot be generated, You can use a `SqlExecutor` directly executing SQL commands or stored procedures. `SqlExecutor` is created by `DataContext`.

THe methods `ExecuteNonQueryXXXX`, `ExecuteScalarXXXX`, `QueryXXXX`, etc in `DataContext` also implement `SqlExecutor` function.

| Method | Introduce |
|:------|:------|
| CreateSqlStringExecutor | Create the executor for the SQL command |
| CreateStoreProcedureExecutor | Create the executor for the stored procedure |

`SqlExecutor` primary method

| Method | Introduce |
|:------|:------|
| ExecuteNonQuery | Executes a command or stored procedure and returns the number of rows affected |
| ExecuteScalar | Executes a command or stored procedure and returns the first column of the first row in the result set returned  |
| Query<T>| Executes a command or stored procedure, return enumerator of `IEnumable<T>` type, All fields of type `T` need to be consistent with the query return fields |
| QueryList<T> | Executes a command or stored procedure, return enumerator of `List<T>` type, All fields of type `T` need to be consistent with the query return fields |
| QueryArray<T> | Executes a command or stored procedure, return enumerator of `T[]` type, All fields of type `T` need to be consistent with the query return fields |


## Execute SQL Command

### Parameter Array

Using `DataParameter` array as parameters, defined parameter names and data values in the array, 
and use `@参数名` format to define the parameters in the command 

```csharp
var sql = "update Te_User set NickName=@P2 where Id=@P1";
var ps = new DataParameter[2];
ps[0] = new DataParameter("P1", 5);
ps[1] = new DataParameter("P2", "abc");
var executor = context.CreateSqlStringExecutor(sql, ps);
var ret = executor.ExecuteNonQuery();
```

### Custom Object Parameter

Using a custom object to a parameter collection, get public property of the object as parameters,
and use `{parameter name}` format to define the parameters in the command.

```csharp
var sql = "update Te_User set NickName={nickname} where Id={id}";
var executor = context.CreateSqlStringExecutor(sql, new { nickname = "abc", id = 5 });
var ret = executor.ExecuteNonQuery();
```

### Command Cases

ExecuteNonQuery

```csharp
var sql = "update Te_User set NickName='abc' where Id=5";
var executor = context.CreateSqlStringExecutor(sql);
var ret = executor.ExecuteNonQuery();
```

ExecuteScalar

```csharp
var sql = "select count(1) from Te_User where Id<=5";
var executor = context.CreateSqlStringExecutor(sql);
var ret = executor. ExecuteScalar();
```

```csharp
var sql = "select count(1) from Te_User where Id<=@P1";
var ps = new DataParameter[1];
ps[0] = new DataParameter("P1", 5);
var executor = context.CreateSqlStringExecutor(sql, ps);
var ret = executor.ExecuteScalar();
```

```csharp
var sql = "select count(1) from Te_User where Id<={id}";
var executor = context.CreateSqlStringExecutor(sql, new { id = 5 });
var ret = executor.ExecuteScalar();
```

Query

```csharp
var sql = "select * from Te_User where Id>5 and Id<8";
var executor = context.CreateSqlStringExecutor(sql);
var list = executor.QueryList<TeUser>();
```

```csharp
var sql = "select * from Te_User where Id>@P1 and Id<=@P2";
var ps = new DataParameter[2];
ps[0] = new DataParameter("P1", 5);
ps[1] = new DataParameter("P2", 8);
var executor = context.CreateSqlStringExecutor(sql, ps);
var list = executor.QueryList<TeUser>();
```

```csharp
var sql = "select * from Te_User where Id>{from_id} and Id<={to_id}";
var executor = context.CreateSqlStringExecutor(sql, new { from_id = 5, to_id = 8 });
var list = executor.QueryList<TeUser>();
```

## Execute Stored Procedure

The execution operation is consistent with the execution command, Parameters of the stored procedure can be `Output`.

### Parameter Array

Using `DataParameter` array as parameters, defined parameter names and data values in the array, 
 The parameter name needs to be the same as the stored procedure's parameter name.


The stored procedure parameters can be defined `Output` direct by using `DataParameterMode.Output`
After completion of execution, The `Output` result will written back into the parameter's `Value` property. 

```csharp
var sp = "mysp";
var ps = new DataParameter[2];
ps[0] = new DataParameter("P1", 5);
ps[1] = new DataParameter("P2", 0, DataParameterMode.Output);
var executor = context.CreateStoreProcedureExecutor(sp, ps);
executor.ExecuteNonQuery();

var output = Convert.ToInt32(ps[1].Value);
```

### Custom Object Parameter

Using a custom object to a parameter collection, get public property of the object as parameters. The property name needs to be the same as the parameter name of the stored procedure.

```csharp
var sp = "mysp";
var executor = context.CreateStoreProcedureExecutor(sp, new { P1 = 5, P2 = 8 });
var list = executor.QueryList<TeUser>();
```

Advanced, The custom object parameters of the stored procedure can use `DataParameterAttribute` defined parameters and direction, The `Output` direction data will be written back to the specified field of the object after the stored procedure execution

```csharp
class TestDataParam
{
    [DataParameter("P1")]
    public int InputData { get; set; }
    [DataParameter("P2", Direction = DataParameterMode.Output)]
    public int OutputData { get; set; }
}
```

```csharp
var sp = "mysp_out";
var ps = new TestDataParam { InputData = 5 };
var executor = context.CreateStoreProcedureExecutor(sp, ps);
var ret = executor.ExecuteNonQuery();
int result = ps.OutputData;
```
