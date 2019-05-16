# 基本使用方法

## DataContext类

`DataContext`是`Light.Data`的核心类, 所有对数据库的操作均通过`DataContext`的方法执行.

例子:

```csharp
var context = new DataContext();
// 查询单个数据
var item = context.Query<TeUser>().Where(x => x.Id == 10).First();
// 查询集合数据
var list = context.Query<TeUser>().Where(x => x.Id > 10).ToList();
// 新增数据
var user = new TeUser() {
    Account = "foo",
    Password = "bar"
}
context.Insert(user);
```

* 建议创建`DataContext`继承类, 并通过依赖注入方式使用. 
* 在事务状态下`DataContext`是非线程安全.
* 方法名含有`Async`的`DataContext`方法均是其对应方法的异步实现.

## DataContext工厂类

由于在事务状态下`DataContext`是非线程安全，如果在需要并发事务的场景，可以使用工厂类生成对应`DataContext`

```csharp
public class TestContext : DataContext
{
    public TestContext(DataContextOptions<TestContext> options) : base(options)
    {

    }
}

public class TestContextFactory : DataContextFactory<TestContext>
{
    public TestContextFactory(DataContextOptions<TestContext> options) : base(options)
    {

    }

    public override TestContext CreateDataContext()
    {
        return new TestContext(options);
    }
}
```

使用`AddDataContextFactory`方法依赖注入工厂类

```csharp
IServiceCollection service = ...;
service.AddDataContextFactory<TestContextFactory, TestContext>(builder =>
{
    builder.UseMssql(connectionString);
}, ServiceLifetime.Singleton);
```

或

```csharp
IServiceCollection service = ...;
var builder = new ConfigurationBuilder()
               .SetBasePath(env.ContentRootPath)
               .AddJsonFile("appsettings.json");
var configuration = builder.Build();
  
 ...
 
service.AddDataContextFactory<MyDataContextFactory, MyDataContext>(
    configuration.GetSection("lightData"), config => {
       config.ConfigName = "mssql";
}, ServiceLifetime.Singleton);
```

## SQL命令输出

在使用`Light.Data`操作数据库时, 如需要获取实际生成的SQL命令, 可以使用`ICommandOutput`接口输出命令.使用SQL命令输出会造成一定的性能损耗. 可以直接使用`CommandOutput`类或自行实现`ICommandOutput`接口.

创建`CommandOutput`

```csharp
 CommandOutput output = new CommandOutput();
 output.Enable = true;
 output.UseConsoleOutput = true;
 output.OutputFullCommand = true;
```

CommandOutput 属性与事件

| 属性 | 说明 |
|:------|:------|
| Enable | 启用命令输出 |
| UseConsoleOutput | 直接把SQL命令输出到控制台 |
| OutputFullCommand	| 输出可以直接在数据库客户端直接执行的命令 |

| 事件 | 说明 |
|:------|:-----|
| OnCommandOutput | SQL命令输出事件, 用于自定义处理方式, 如记入日志 |

直接在`DataContext`设置输出接口

```csharp
DataContext context = new DataContext();
context.SetCommandOutput(output);
```

在`DataContextOptionsBuilder`设置输出接口

```csharp
service.AddDataContext<TestContext>(builder => {
	builder.UseMssql(connectionString);
	builder.SetCommandOutput(output);
}, ServiceLifetime.Transient);
```

执行查询方法

```csharp
var list = context.Query<TeUser>().Where(x => x.Id >= 10).ToList();
```

SQL语句输出

```
action  :QueryDataDefineList
type    :Text
level   :None
region  :0,2147483647
time    :2017-09-05 00:48:49.828
span    :20.7349
trans   :false
success :true
result  :1
params  :
  Input,Int32,@P1=10
command :
    select [Id],[Account],[Password],[NickName],[Gender],[Birthday],[Telephone],[Email],[LevelId],[RegTime],[LastLoginTime],[Status],[HotRate] from [Te_User] where [Id]>=@P1
--------------------
    select [Id],[Account],[Password],[NickName],[Gender],[Birthday],[Telephone],[Email],[LevelId],[RegTime],[LastLoginTime],[Status],[HotRate] from [Te_User] where [Id]>=10
```
