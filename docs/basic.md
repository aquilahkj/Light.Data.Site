# Basic Method of Use

## DataContext Class

`DataContext` is the`Light.Data` core class, all the operation of the database by `DataContext` method to execute.

e.g.

```csharp
var context = new DataContext();
// query single data
var item = context.Query<TeUser>().Where(x => x.Id == 10).First();
// query collection datas
var list = context.Query<TeUser>().Where(x => x.Id > 10).ToList();
// create date
var user = new TeUser() {
    Account = "foo",
    Password = "bar"
}
context.Insert(user);
// update data
user.Password = "bar1";
context.Update(user);
// delete data
context.Delete(user);
```

* Recommend create `DataContext` derived classes, and by using Dependency-Injection mode.
* In the Transaction State, `DataContext` is not Thread-Safe
* The methods in the `DataContext` which name contains `Async`, are the asynchronous implementation of the corresponding methods.

## DataContext Factory Class

Because of in the Transaction State, `DataContext` is not Thread-Safe，If concurrent transactions scene in need, you can use the factory class to generate corresponding `DataContext`

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

Using `AddDataContextFactory` method dependency injection Factory Class

```csharp
IServiceCollection service = ...;
service.AddDataContextFactory<TestContextFactory, TestContext>(builder =>
{
    builder.UseMssql(connectionString);
}, ServiceLifetime.Singleton);
```

Or

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

## SQL Command Output

In the use of `Light. Data` database operation, if you need to get the actual generated SQL command, you can use `ICommandOutput` interface output command. Using SQL command output can cause some performance loss. Can be used directly `CommandOutput` class or implement `ICommandOutput` interface.

Create `CommandOutput`

```csharp
 CommandOutput output = new CommandOutput();
 output.Enable = true;
 output.UseConsoleOutput = true;
 output.OutputFullCommand = true;
```

CommandOutput Properties and events

| Property | Introduce |
|:------|:------|
| Enable | Enable command output |
| UseConsoleOutput | Outputs command to console directly |
| OutputFullCommand	| Outputs command that can be executed directly on the database client |

| Event | Introduce |
|:------|:-----|
| OnCommandOutput | SQL command output event, Used to customize processing, such as logging |

Set up output interface in `DataContext` Directly

```csharp
DataContext context = new DataContext();
context.SetCommandOutput(output);
```

Set up output interface in `DataContextOptionsBuilder`

```csharp
service.AddDataContext<TestContext>(builder => {
	builder.UseMssql(connectionString);
	builder.SetCommandOutput(output);
}, ServiceLifetime.Transient);
```

Execute query method

```csharp
var list = context.Query<TeUser>().Where(x => x.Id >= 10).ToList();
```

SQL command output

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
