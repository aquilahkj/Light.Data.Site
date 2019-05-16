# 数据表处理

## 数据表别名

`DataContext`可以临时为映射类型数据设置数据表别名, 用于特殊场景, 如对日志表进行水平分表. 该别名仅作用于该`DataContext`的操作.

| 方法 | 说明 |
|:------|:------|
| GetTableName<T>() | 获取指定类型`T`的对应数据表名 |
| SetAliasTableName<T>(string name) | 对指定类型`T`设定别名 |
| ResetAliasTableName<T>() | 清除指定类型T的别名 |
| ClearAliasTableName() | 清除所有在此DataContext中的别名 |

设置别名并查询

```csharp
DataContext context = new DataContext();
// 获取对应表名Te_Log
var tableName = context.GetTableName<TeLog>();
// 别名Te_Log_201709
var aliasName = tableName + "_201709";
// 设置别名
context.SetAliasTableName<TeLog>(aliasName);
// 查询表Te_Log_201709
var list = context.Query<TeLog>().ToList();
```

## 截断数据表

```csharp
context.TruncateTable<TeUser> (); 
```

注意：该操作直接使用`truncate table`命令截断数据表.