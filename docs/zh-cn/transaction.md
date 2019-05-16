# 事务处理

## 事务状态

`DataContext`有两种状态: `事务状态`和`非事务状态`

`事务状态`是非线程安全, `DataContext`进入事务状态后，进行一系列操作, 需要显式提交, 操作才会完成.

`非事务状态`是线程安全, `DataContext`每一次操作后, 都会自动提交.

## 事务方法

| 方法 | 说明 |
|:------|:------|
| BeginTrans | 开始事务, 每次事务开始前需执行, 执行后`DataContext`转为`事务状态` |
| CommitTrans | 提交事务, 提交后该事务才完成, 事务提交后不能再使用 |
| RollbackTrans | 回滚事务, 另外在执行过程中抛异常时会自动回滚, 事务回滚后不能再使用 |
| ReleaseTrans | 执行完事务后显式释放事务, 转为`非事务状态`, `autoRelease`参数为false |

## 事务说明

### 事务执行

如要进入事务状态, 通过`DataContext`的`BeginTrans`方法生成事务域类`TransactionScope`,`DataContext`由非事务状态转为事务状态, 最后通过`CommitTrans`方法提交事务, 事务只能提交一次.

### 释放事务

通过参数`autoRelease`设置自动或手动释放事务

当`autoRelease`是true时, 事务在`CommitTrans`或`RollbackTrans`后, 自动释放事务, `DataContext`由事务状态转为非事务状态;

当`autoRelease`是false时, 需要显式使用 `ReleaseTrans`方法释放事务, 事务状态转为非事务状态. 当执行`ReleaseTrans`方法时, 但事物还没进行被提交, 事务自动回滚.

### 事务安全级别

通过参数`safeLevel`设置事务安全级别

* Default/None: 数据库默认
* Low: 未提交读
* Normal: 提交读
* High: 可重复读
* Serializable: 序列化

## 使用方式

### Scope模式

标准模式, 如果离开`using`块时还未提交, 事务自动回滚.

```csharp
//提交
using (TransactionScope trans = context.BeginTrans ()) {
       TeUser user1 = context.SelectById<TeUser> (3);
       user1.Account = "foo";
       context.Update(user1);
       TeUser user2 = context.SelectById<TeUser> (4);
       context.Delete(user2);
       trans.CommitTrans ();
}
//回滚
using (TransactionScope trans = context.BeginTrans ()) {
       TeUser user1 = new <TeUser> ();
       user1.Account = "foo";
       context.Insert (user1);
       if(user1.Id > 5){
           trans.RollbackTrans ();
       }
       else{
           trans.CommitTrans ();
       }
}
```

### 手动模式

主要用于把事务操作分拆在不同模块中, 最终确保事务能被释放.

```csharp
var autoRelease = false;
context.BeginTrans (autoRelease);
user1.Account = "foo";
context.Insert (user1);
trans.CommitTrans ();
trans.ReleaseTrans ();
```

## 异常处理

在事务执行中, 操作抛出异常

启用`autoRelease`, 当事务出现异常时, 事务自动回滚, 自动释放事务, 状态转为非事务状态;

不启用`autoRelease`, 当事务出现异常时, 事务自动回滚, 需要显式使用 `ReleaseTrans`方法释放事务, 事务状态转为非事务状态.