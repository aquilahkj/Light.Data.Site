# 数据库配置

## 配置文件
`Light.Data`支持使用json格式的配置文件对数据库类型和数据库连接字符串进行配置, 配置文件地址可通过在程序初始化时进行设置, 只可以设置一个.

例子

```csharp
DataContextConfiguration.SetConfigFilePath("Config/lightdata.json");
```

默认配置文件为当前目录下的`lightdata.json`, 如文件不存在则自动忽略. 该配置为全局配置.

`Light.Data`配置数据结构如下

```json
{
  "lightData": {
    "connections": [
      {
        "name": "mssql",
        "connectionString": "Data Source=127.0.0.1;User ID=sa;Password=***;Initial Catalog=LightData_Test;",
        "providerName": "Light.Data.Mssql.MssqlProvider, Light.Data.Mssql"
      },
      {
        "name": "mssql_2012",
        "connectionString": "Data Source=192.168.0.1;User ID=sa;Password=***;Initial Catalog=LightData_Test;",
        "providerName": "Light.Data.Mssql.MssqlProvider, Light.Data.Mssql",
        "configParams": {
          "version" : "11.0"
        }
      }
    ]
  }
}

```

`lightData`节点是映射配置的根节点, 可加入到任意json的配置文件(如`appsettings.json`)中. 

`connections`节点是指定数据库连接配置的集合.

connections - 字段定义

| 字段名 | 值类型 | 可空 | 说明 |
|:------|:------|:------|:------|
| name | string | false | 连接配置唯一名称 |
| connectionString | string | false | 连接字符串 |
| providerName | string | false | 数据库处理类的类名 |
| configParams | array(configParam) | true | 额外的配置参数 |

providerName - 支持的数据库

| 数据库 | Provider | 说明 |
|:------|:------|:------|
| SqlServer | Light.Data.Mssql.MssqlProvider | 需安装`Light.Data.Mssql`类库 |
| Mysql | Light.Data.Mysql.MysqlProvider | 需安装`Light.Data.Mysql`类库 |
| Postgre | Light.Data.Postgre.PostgreProvider | 需安装`Light.Data.Postgre`类库 |

configParam 参数设置

| 参数名 | 数据格式 | 说明 |
|:------|:------|:------|
| timeout | int | 执行超时时间, 单位毫秒, 默认60000 |
| batchInsertCount | int | 批量新增数据每组语句块处理数, 默认10 |
| batchUpdateCount | int | 批量更新数据每组语句块处理数, 默认10 |
| batchDeleteCount | int | 批量删除数据每组语句块处理数, 默认10 |
| strictMode | bool | 对生成SQL中表名和字段名做严格定义, 如MSSQL的表名和字段加上[ ], 避免表名和字段因使用了保留关键字名而导致错误, 默认为true |
| version | string | 数据库版本 |


SqlServer version 功能列表

| Version | 数据库 | 说明 |
|:------|:------|:------|
| 10.0 | SqlServer 2008 | version>=10.0 优化批量新增数据的SQL语句, 默认 |
| 11.0 | SqlServer 2012 | version>=11.0 支持offset分页查询 |

## 全局配置

使用配置名创建核心类`DataContext`, 如使用`DataContext`无参构造函数创建默认使用第一个配置.

```csharp
DataContext context = new DataContext("mssql");
```

继承`DataContext`创建子类

```csharp
public class MyDataContext : DataContext
{
    public MyDataContext() : base("mssql")
    {

    }
}
```

## 依赖注入配置

使用`Microsoft.Extensions.DependencyInjection`实现依赖注入. 

继承`DataContext`创建子类.

使用`DataContextOptions<T>`作为构造函数的参数

```csharp
public class MyDataContext : DataContext
{
    public MyDataContext(DataContextOptions<MyDataContext> options) : base(options)
    {

    }
}
```

###  连接字符串进行配置

对`IServiceCollection`使用静态扩展函数`AddDataContext`进行配置.

在`DataContextOptionsBuilder<TContext>`中使用`UseMssql`|`UseMysql`|`UsePostgre`配置数据库类型.

```csharp
IServiceCollection service = ...;
service.AddDataContext<MyDataContext>(builder => {
    builder.UseMssql(connectionString);
    builder.SetTimeout(2000);
    builder.SetVersion("11.0");
}, ServiceLifetime.Transient);
```

### 指定配置节进行配置

获取配置文件中的`lightData`节点数据, 并转为`IConfiguration`对象.

对`IServiceCollection`使用静态扩展函数`AddDataContext`进行配置.

并可以通过设置`ConfigName`, 在`DataContextOptionsConfigurator<TContext>`选定配置中的连接配置.

```csharp
IServiceCollection service = ...;
var builder = new ConfigurationBuilder()
               .SetBasePath(env.ContentRootPath)
               .AddJsonFile("appsettings.json");
var configuration = builder.Build();
service.AddDataContext<MyDataContext>(configuration.GetSection("lightData"), config => {
       config.ConfigName = "mssql";
}, ServiceLifetime.Transient);
```

### 全局配置文件配置

对`IServiceCollection`使用静态扩展函数`AddDataContext`进行配置, 并使用全局配置`DataContextConfiguration.Global`, 这样就会选择了在`DataContextConfiguration.SetConfigFilePath`方法设定的配置文件.

并可以通过`DataContextOptionsConfigurator<TContext>`选定配置中的连接配置.

```csharp
IServiceCollection service = ...;
service.AddDataContext<MyDataContext>(DataContextConfiguration.Global, config => {
       config.ConfigName = "mssql";
}, ServiceLifetime.Transient);
```

### 依赖注入调用

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

DataContextOptionsBuilder<TContext> 方法

| 方法 | 说明 |
|:------|:------|
| UseMssql(string connectionString) | 设置`SqlServer`连接字符串, 需安装Light.Data.Mssql类库 |
| UseMysql(string connectionString) | 设置`Mysql`连接字符串, 需安装Light.Data.Mysql类库 |
| UsePostgre(string connectionString) | 设置`Postgre`连接字符串, 需安装Light.Data.Postgre类库 |
| SetTimeout(int timeout) | 设置执行超时时间, 单位毫秒, 默认60000 |
| SetBatchInsertCount(int timeout) | 设置批量新增数据每组语句块处理数, 默认10 |
| SetBatchUpdateCount(int timeout) | 设置批量更新数据每组语句块处理数, 默认10 |
| SetBatchDeleteCount(int timeout) | 设置批量删除数据每组语句块处理数, 默认10 |
| SetVersion(string version) | 设置数据库版本 |
| SetStrictMode(bool strictMode) | 对生成SQL中表名和字段名做严格定义, 如MSSQL的表名和字段加上[ ], 避免表名和字段因使用了保留关键字名而导致错误, 默认为true |
| SetCommandOutput(ICommandOutput output) | 设置`ICommandOutput`实例用于记录SQL |


