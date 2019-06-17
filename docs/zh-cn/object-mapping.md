# 对象映射

## Attribute映射数据定义

`Light.Data`通过对实体模型类的`Attribute`或者配置文件进行配置与数据表的对应关系. 使用核心类`DataContext`对数据表进行`CRUD`的操作.

```csharp
[DataTable("Te_User", IsEntityTable = true)]
public class TeUser
{
    /// <summary>
    /// Id
    /// </summary>
    /// <value></value>
    [DataField("Id", IsIdentity = true, IsPrimaryKey = true)]
    public int Id
    {
        get;
        set;
    }

    /// <summary>
    /// Account
    /// </summary>
    /// <value></value>
    [DataField("Account")]
    public string Account
    {
        get;
        set;
    }

    /// <summary>
    /// Telephone
    /// </summary>
    /// <value></value>
    [DataField("Telephone", IsNullable = true)]
    public string Telephone
    {
        get;
        set;
    }
    ....
}
```

DataTable:指定对应数据表的表名

* IsEntityTable: 指定数据表的是否实体表, 否则为视图. 视图不可以进行增删改操作, 默认为true.

DataField:指定对应数据字段的字段名

* IsIdentity: 字段是否自增, 默认为false
* IsPrimaryKey: 字段是否主键, 默认为false
* IsNullable: 字段是否可为空, 默认为false, 影响数据新增与更新
* DefaultValue: 默认值,可空类型字段在新增数据时, 如果数据为空, 自动使用该默认值. 如数据类型是datetime, 可使用Enum的`DefaultTime`类型生成当前时间


 | Enum类型 | 说明 |
 |:------|:------|
 | `DefaultTime.Today` | 默认值为当天 |
 | `DefaultTime.Now` | 默认值为当前时间 |
 | `DefaultTime.TimeStamp` | 默认值为当前时间, 对更新也同样生效 |

* FunctionControl: 字段读写访问控制功能, 默认为全控制


 | Enum类型 | 说明 |
 |:------|:------|
 | `FunctionControl.Read` | 只读 |
 | `FunctionControl.Create` | 读和新增 |
 | `FunctionControl.Update` | 读和更新 |


## 配置文件映射数据定义
`Light.Data`支持使用json格式的配置文件进行对数据实体的映射配置, 数据模型不需引用`Light.Data`类库进行`Attribute`标记

例如:

```csharp
public class TeUser
{
    /// <summary>
    /// Id
    /// </summary>
    /// <value></value>
    public int Id
    {
        get;
        set;
    }

    /// <summary>
    /// Account
    /// </summary>
    /// <value></value>
    public string Account
    {
       get;
       set;
    }
    
    /// <summary>
    /// Telephone
    /// </summary>
    /// <value></value>
    public string Telephone
    {
       get;
       set;
    }
    ....
}
```

配置文件地址可在程序初始化时进行设置, 可设置多个

```csharp
DataMapperConfiguration.AddConfigFilePath("Config/lightdata.json");
```

默认配置文件为当前目录下的`lightdata.json`, 如文件不存在则自动忽略.

配置数据结构为

```json
{
  "lightDataMapper": {
    "dataTypes": [
      {
        "type": "MyTest.TeUser,MyTest",
        "tableName": "Te_User",
        "isEntityTable": true,
        "dataFields": [
          {
            "fieldName": "Id",
            "isPrimaryKey": true, 
            "isIdentity": true, 
            "name": "Id"
          },
          {
            "fieldName": "Account",
            "name": "Account"
          },
          {
            "fieldName": "Telephone",
            "name": "Telephone"
          },
        ]
      },
    ]
  }
}
```

`lightDataMapper`为映射配置的根节点, 可加入到任意json的配置文件中. 

dataType 字段定义

| 字段名 | 值类型 | 可空 | 说明 |
|:------|:------|:------|:------|
| type | string | false | 实体类的类名全名 |
| tableName | string | false | 对应数据实体类的表名 |
| isEntityTable | bool | true | 指定数据表的是否实体表, 否为视图, 不可以进行增删改操作, 默认为true |
| dataFields | array(dataField) | true | 该实体类下的映射字段集合 |

dataField 字段定义

| 字段名 | 值类型 | 可空 | 说明 |
|:------|:------|:------|:------|
| fieldName | string | false | 实体类的字段名 |
| isPrimaryKey | bool | true | 字段是否自增, 默认为false |
| isIdentity | bool | true | 字段是否自增, 默认为false |
| isNullable | bool  | true | 字段是否可为空, 默认为false, 影响数据新增与更新 |
| name | string | true | 字段映射的数据库字段名, 默认为实体类的字段名 |
| defaultValue | string | true | 默认值, 可空类型字段在新增数据时, 如果数据为空, 自动使用该默认值；如数据类型是datetime, 空值或最小值时可使用`DefaultTime.Now`与`DefaultTime.TimeStamp`表示默认值为当前时间; `DefaultTime.Today`表示默认值为当天; `TimeStamp`在更新数据时同样有效 |
| functionControl | string | true | 字段功能控制，`Read`代表字段只读, 新增和修改数据时不生效, `Create`代表字段只增, `Update`代表字段只改, 默认为全控制 |

## 字段类型

`Light.Data`的映射字段支持以下类型

* 原生数据类型 `bool`, `byte`, `sbyte`, `short`, `ushort `, `int`, `uint`, `long`, `ulong`, `double`, `float`, `decimal`, `DateTime`及其可空类型, 字段类型需要与数据库类型对应, 如数据库不支持该类型, 可使用位数更大的类型, 如在Mssql中`sbyte `=>`smallint`, `ushort`=>`int`, `uint`=>`bigint`, `ulong`=>`decimal(20,0)`
* `Enum`类型及其可空类型, 数据库需对应Enum本身所继承的数字类型
* `string`类型, 需在数据库中使用字符串类型, 如`char`, `varchar`, `text`等
* `byte[]`类型, 需在数据库中使用二进制类型, 如`binary`, `varbinary`等, 该类型只支持是否为空的条件查询
* 自定义对象类型, 需在数据库中使用字符串类型, 如`char`, `varchar`, `text`等, `Light.Data`会自动把对象序列化为json格式存取, 该类型只支持是否为空和整体相等的条件查询

## 主表与子表关联

如数据库有主表和子表需要做关联查询, 可以把子表的映射以单个对象(一对一)或集合(一对多)作为主表映射类的一个属性, 组成组合对象. 组合对象只能用于查询

```csharp
[DataTable("Te_User", IsEntityTable = true)]
public class TeUser
{
    [DataField("Id", IsIdentity = true, IsPrimaryKey = true)]
    public int Id
    {
        get;
        set;
    }

    [DataField("Account")]
    public string Account
    {
        get;
        set;
    }
    
    [DataField("Telephone", IsNullable = true)]
    public string Telephone
    {
        get;
        set;
    }
}
```

子表映射

```csharp
[DataTable("Te_UserExtend", IsEntityTable = true)]
public class TeUserExtend
{
    [DataField("Id", IsIdentity = true, IsPrimaryKey = true)]
    public int Id
    {
        get;
        set;
    }

    [DataField("MainId")]
    public int MainId
    {
        get;
        set;
    }
    
    [DataField("Data", IsNullable = true)]
    public string Data
    {
        get;
        set;
    }
}
```

继承`TeUser`新增一个类

```csharp
public class TeUserAndExtend : TeUser
{
    [RelationField("Id", "MainId")]
    public TeUserExtend Extend
    {
        get;
        set;
    }
}
```

在继承类`TeUserAndExtend`中添加一个类型是`TeUserExtend`的公共属性`Extend`, 并加上Attribute`RelationField`, 在查询`TeUserAndExtend`时, 会把关联的`TeUserExtend`数据也一并查出. 

`RelationField`的参数1是主表的关联字段属性名, 参数2是子表的关联字段属性名, 如有主表和子表有多对关联字段, 可以设置多组Attribute, 如

```csharp
[RelationField("MKey1", "SKey1")]
[RelationField("MKey2", "SKey2")]
public TeUserExtend Extend
{
    get;
    set;
}
```

如主表与子表是一对多关系, 属性类型需要使用`ICollection<T>`或`LCollection<T>`, `T`为子表类型, 如

```csharp
public class TeUserAndExtend : TeUser
{
    [RelationField("Id", "MainId")]
    public ICollection<TeUserExtend> Extends
    {
        get;
        set;
    }
}
```

关联属性也可以在配置文件中的`dataType`节点中进行配置, 关联字段节点为`relationFields`. 继承类的配置会默认继承基类的`dataFields`和`relationFields`配置, 如

```json
      {
        "type": "MyTest.TeUserAndExtend,MyTest",
        "tableName": "Te_User",
        "isEntityTable": true,
        "relationFields": [
          {
            "fieldName": "Extend",
            "relationPairs": [
              {
                "masterKey": "Id",
                "relateKey": "MainId"
              }
            ]
          }
        ]
      }
```

relationFields 字段定义

| 字段名 | 值类型 | 可空 | 说明 |
|:------|:------|:------|:------|
| fieldName | string | false | 实体类的字段名 |
| relationPairs | array(relationPair) | false | 关联字段组合 |

relationPair 字段定义

| 字段名 | 值类型 | 可空 | 说明 |
|:------|:------|:------|:------|
| masterKey | string | false | 主表的关联字段属性名 |
| masterKey | string | false | 子表的关联字段属性名 |

### 多个子表关联

一个主表类支持多个不同子表类做关联属性

```csharp
public class TeRelateA_BC : TeRelateA
{
    [RelationField("Id", "RelateAId")]
    public TeRelateB RelateB {
        get;
        set;
    }

    [RelationField("Id", "RelateAId")]
    public TeRelateC RelateC {
        get;
        set;
    }
}

public class TeRelateA_B_Collection : TeRelateA
{
    [RelationField("Id", "RelateAId")]
    public TeRelateB RelateB {
        get;
        set;
    }

    [RelationField("Id", "RelateAId")]
    public ICollection<TeRelateCollection> RelateCollection {
        get;
        set;
    }
}
```

### 子表级联

子表类可以作为主表连接另外的子表

```csharp
public class TeRelateA_BC : TeRelateA
{
    [RelationField("Id", "RelateAId")]
    public TeRelateB_C RelateB {
        get;
        set;
    }
}

public class TeRelateB_C : TeRelateB
{
    [RelationField("Id", "RelateBId")]
    public TeRelateC RelateC {
        get;
        set;
    }
}
```

### 两数据表互为主从

当两个表类互为主从时, 需确保双方的关联字段主从相反, 定义保持一致, 否则会出现错误

```csharp
public class TeRelateA_B_A : TeRelateA
{
    [RelationField("Id", "RelateAId")]
    public TeRelateB_A_B RelateB {
        get;
        set;
    }
}

public class TeRelateB_A_B : TeRelateB
{
    [RelationField("RelateAId", "Id")]
    public TeRelateA_B_A RelateA {
        get;
        set;
    }
}
```

## 实体类与配置文件生成模板

可通过T4模版直接生成映射类或json配置文件

[模版地址](https://github.com/aquilahkj/Light.Data2/tree/master/template)

另外需要使用类库`Light.Data.Template.dll`, 并使用对应数据库的ADO.NET类库

[类库地址](https://github.com/aquilahkj/Light.Data2/tree/master/lib) 

使用方法
```csharp
<#@ template debug="true" hostspecific="true" language="C#"  #>
<#@ output extension=".cs" #>
<#@ assembly name="System.Core"#>
<#@ assembly name="System.Data"#>
<#@ assembly name="System.Xml"#>
<#@ assembly name="$(SolutionDir)/lib/MySql.Data.dll"  #>
<#@ assembly name="$(SolutionDir)/lib/Light.Data.Template.dll"  #>
<#@ import namespace="System"#>
<#@ import namespace="System.Data"#>
<#@ import namespace="System.Collections.Generic"#>
<#@ import namespace="System.Diagnostics" #>
<#@ import namespace="System.Linq" #>
<#@ import namespace="System.Data.SqlClient"#>
<#@ import namespace="System.Text"#>
<#@ import namespace="System.Text.RegularExpressions"#>
<#@ import namespace="Light.Data.Template"#>
```
