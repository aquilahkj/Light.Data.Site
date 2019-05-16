# Database Configuration

## Configuration File

`Light.Data`support using json format configuration file to configure data type and the database connection string, The configuration file path can be set in the program initialization, can set only one.

e.g.

```csharp
DataContextConfiguration.SetConfigFilePath("Config/lightdata.json");
```

The default configuration file in the current directory `lightdata.json`, if the file does not exist, it automatically ignored. This configuration for global configuration.


`Light.Data` configuration data structure is as follows

```json
{
  "lightData": {
    "connections": [
      {
        "name": "mssql",
        "connectionString": "Data Source=127.0.0.1;User ID=sa;Password=***;Initial Catalog=LightData_Test;",
        "providerName": "Light.Data.Mssql.MssqlProvider"
      },
      {
        "name": "mssql_2012",
        "connectionString": "Data Source=192.168.0.1;User ID=sa;Password=***;Initial Catalog=LightData_Test;",
        "providerName": "Light.Data.Mssql.MssqlProvider",
        "configParams": {
          "version" : "11.0"
        }
      }
    ]
  }
}

```

`lightData` node is the root node of the mapping configuration, can be added to any json configuration files (such as `appsettings.json`).

`connections` node is to specify a collection of database connection configuration.

connections - Field Definition

| Field | Type | Nullable | Introduce |
|:------|:------|:------|:------|
| name | string | false | Connection configuration unique name |
| connectionString | string | false | Connection string |
| providerName | string | false | The class name of the database processing class |
| configParams | array(configParam) | true | Additional configuration parameters |

providerName - Supported databases

| Database | Provider | Introduce |
|:------|:------|:------|
| SqlServer | Light.Data.Mssql.MssqlProvider | Need to use `nuget` install `Light.Data.Mssql` library |
| Mysql | Light.Data.Mysql.MysqlProvider |Need to use `nuget` install `Light.Data.Mysql` library |
| Postgre | Light.Data.Postgre.PostgreProvider | Need to use `nuget` install `Light.Data.Postgre` library |

configParam - Parameter Settings

| ParameterName | Type | Introduce |
|:------|:------|:------|
| timeout | int | Execution timeout, unit milliseconds, defaults 60,000 |
| batchInsertCount | int | The number of batch insert data each group statement block, default 10 |
| batchUpdateCount | int | The number of batch update data each group statement block , default 10 |
| batchDeleteCount | int | The number of batch delete data each group statement block , default 10 |
| strictMode | bool | Make strict definition of table name and field name in generated SQL, such as adding [ ] to table name and field of MSSQL to avoid errors caused by using reserved keyword name, and the default is true |
| version | string | Database version |


SqlServer version - Feature list

| version | Database | Introduce |
|:------|:------|:------|
| 10.0 | SqlServer 2008 | version>=10.0 Optimized batch insert data SQL, Default |
| 11.0 | SqlServer 2012 | version>=11.0 Support use offset to paging query |
| 12.0 | SqlServer 2014 |  |

## Global Configuration

Using a configuration name to create core class `DataContext`, if use no arguments constructor to create `DataContext`, default use the first configuration.

```csharp
DataContext context = new DataContext("mssql");
```

Create a subclass inherits the `DataContext`

```csharp
public class MyDataContext : DataContext
{
    public MyDataContext() : base("mssql")
    {

    }
}
```

## Dependency-Injection Configuration

Using `Microsoft.Extensions.DependencyInjection` to dependency injection.

Create a subclass inherits the `DataContext`.

Using `DataContextOptions<T>` as a constructor parameter

```csharp
public class MyDataContext : DataContext
{
    public MyDataContext(DataContextOptions<MyDataContext> options) : base(options)
    {

    }
}
```

###  The Connection String Configuration

Using static extension function `AddDataContext` configure `IServiceCollection`

Using `UseMssql`|`UseMysql`|`UsePostgre` method configured database type in `DataContextOptionsBuilder`.

```csharp
IServiceCollection service = ...;
service.AddDataContext<MyDataContext>(builder => {
    builder.UseMssql(connectionString);
    builder.SetTimeout(2000);
    builder.SetVersion("11.0");
}, ServiceLifetime.Transient);
```

### Specify Configuration Section Configuration

Get in the configuration file `lightData` node data, and to `IConfiguration` object.

Using static extension function `AddDataContext` configure `IServiceCollection`.

And you can set the `ConfigName`, select connection configuration in `DataContextOptionsConfigurator`

```csharp
IServiceCollection service = ...
var builder = new ConfigurationBuilder()
               .SetBasePath(env.ContentRootPath)
               .AddJsonFile("appsettings.json");
var configuration = builder.Build();
service.AddDataContext<MyDataContext>(configuration.GetSection("lightData"), config => {
       config.ConfigName = "mssql";
}, ServiceLifetime.Transient);
```

### Global Profile Configuration

Using static extension function `AddDataContext` configure `IServiceCollection`.

And use the Global configuration `DataContextConfiguration.Global`, this will chose the configuration file which set in`DataContextConfiguration.SetConfigFilePath` method.

And you can set the `ConfigName`, select connection configuration in `DataContextOptionsConfigurator`

```csharp
IServiceCollection service = ...;
service.AddDataContext<MyDataContext>(DataContextConfiguration.Global, config => {
       config.ConfigName = "mssql";
}, ServiceLifetime.Transient);
```

### Call in Dependency Injection

```csharp
public class MyController : Controller
{
    MyDataContext _context;
    public MyController(MyDataContext context)
    {
        _context = context;
    }
}
```

```csharp
var provider = service.BuildServiceProvider();
var context = provider.GetRequiredService<MyDataContext>();
```

DataContextOptionsBuilder<TContext> method

| Method | Introduce |
|:------|:------|
| UseMssql(string connectionString) | Set up connection string for `SqlServer`, Need to use `nuget` install `Light.Data.Mssql` library |
| UseMysql(string connectionString) | Set up connection string `Mysql`,  Need to use `nuget` install `Light.Data.Mysql` library |
| UsePostgre(string connectionString) | Set up connection string `Postgre`, Need to use `nuget` install `Light.Data.Postgre` library |
| SetTimeout(int timeout) | Set up execution timeout, unit milliseconds, defaults 60,000 |
| SetBatchInsertCount(int timeout) | Set up the number of batch insert data each group statement block, default 10 |
| SetBatchUpdateCount(int timeout) | Set up the number of batch update data each group statement block, default 100 |
| SetBatchDeleteCount(int timeout) | Set up the number of batch delete data each group statement block, default 10 |
| SetVersion(string version) | Database version |
| SetStrictMode(bool strictMode) | Set up Whether make strict definition of table name and field name in generated SQL, such as adding [ ] to table name and field of MSSQL to avoid errors caused by using reserved keyword name, and the default is true |
| SetCommandOutput(ICommandOutput output) | Set ` ICommandOutput ` instance is used to record the SQL |

