# 执行SQL语句

## SQL执行器

当有比较复杂的SQL语句`Light.Data`不能生成时, 可以使用`SqlExecutor`直接执行SQL语句或者存储过程. `SqlExecutor`由`DataContext`创建. 另外可以在`DataContext`中也可以直接执行语句.

`DataContext`中的`ExecuteNonQueryXXXX`, `ExecuteScalarXXXX`, `QueryXXXX`方法同样实现`SqlExecutor`的功能.

| 方法 | 说明 |
|:------|:------|
| CreateSqlStringExecutor | 创建SQL语句的执行器 |
| CreateStoreProcedureExecutor | 创建存储过程的执行器 |

`SqlExecutor`主要方法

| 方法 | 说明 |
|:------|:------|
| ExecuteNonQuery | 执行语句或存储过程并返回受影响的行数 |
| ExecuteScalar | 执行语句或存储过程执行查询并返回由查询返回的结果集中的第一行的第一列 |
| Query<T>| 执行语句或存储过程, 返回`IEnumable<T>`类型的枚举器, 类型`T`的字段需要与语句或存储过程的返回字段一致 |
| QueryList<T> | 执行语句或存储过程, 返回`List<T>`类型的列表, `T`的字段需要与语句或存储过程的返回字段一致 |
| QueryArray<T> | 执行语句或存储过程, 返回`T[]`类型的数组, `T`的字段需要与语句或存储过程的返回字段一致 |


## 执行SQL语句

### 参数数组

使用`DataParameter`数组作参数, 在数组中定义参数名和数据值, 并在语句中以`@参数名`的格式定义参数

```csharp
var sql = "update Te_User set NickName=@P2 where Id=@P1";
var ps = new DataParameter[2];
ps[0] = new DataParameter("P1", 5);
ps[1] = new DataParameter("P2", "abc");
var executor = context.CreateSqlStringExecutor(sql, ps);
var ret = executor.ExecuteNonQuery();
```

### 自定义对象参数

使用自定义对象做参数集合, 取对象中的公共属性名做参数名, 在语句中以`{参数名}`的格式定义参数

```csharp
var sql = "update Te_User set NickName={nickname} where Id={id}";
var executor = context.CreateSqlStringExecutor(sql, new { nickname = "abc", id = 5 });
var ret = executor.ExecuteNonQuery();
```

### 语句用例

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

## 执行存储过程

执行操作方式与执行语句一致, 储存过程的参数可以`Output`.

### 参数数组

使用`DataParameter`数组作参数, 在数组中定义参数名和数据值, 参数名需要与存储过程的参数名一致.

储存过程参数可使用`DataParameterMode.Output`定义参数`Output`方向, 执行完成后, output结果写回在参数的`Value`属性中.

```csharp
var sp = "mysp";
var ps = new DataParameter[2];
ps[0] = new DataParameter("P1", 5);
ps[1] = new DataParameter("P2", 0, DataParameterMode.Output);
var executor = context.CreateStoreProcedureExecutor(sp, ps);
executor.ExecuteNonQuery();

var output = Convert.ToInt32(ps[1].Value);
```

### 自定义对象参数

使用自定义对象做参数集合, 取对象中的公共属性名做参数名, 属性名需要与存储过程的参数名一致.

```csharp
var sp = "mysp";
var executor = context.CreateStoreProcedureExecutor(sp, new { P1 = 5, P2 = 8 });
var list = executor.QueryList<TeUser>();
```

进阶, 存储过程的自定义对象参数可以使用`DataParameterAttribute`定义参数名和参数方向, 含`Output`方向数据会在存储过程执行后回写到对象指定字段中

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
