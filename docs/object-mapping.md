# Object Mapping

## Attribute Mapping Define

`Light.Data` through to the entity model class `Attribute` or configuration files relate data table. Use core class `DataContext` was carried out on the table to `CURD` operation.

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

DataTable: Specifies the table name of the corresponding data table

* IsEntityTable: Specifies whether the data table is an entity table, otherwise it is a view. The view cannot be inserted, deleted or updated. The IsEntityTable default value is true.

DataField: Specifies the field name for the corresponding data field

* IsIdentity: Whether the field is auto incremented, and the default is false
* IsPrimaryKey: whether the field is a primary key, and the default is false
* IsNullable: whether the field can be null, and the default is false, which affects the new and updated data
* DefaultValue: Default Value, nullable type field when new data is added, if the data is null, the DefaultValue will be used automatically.

  If it's a datetime data type, can use Enum `DefaultTime` type generated the current time


 | Enum Type | Introduce |
 |:------|:------|
 | `DefaultTime.Today` | default is today |
 | `DefaultTime.Now` | default is current time |
 | `DefaultTime.TimeStamp` | default is current time, as well as for updates to take effect |
 
* FunctionControl: field Read and write access control function, default for all controls


 | Enum Type | Introduce |
 |:------|:------|
 | `FunctionControl.Read` | read-only, insert and update the data without effect |
 | `FunctionControl.Create` | read and insert |
 | `FunctionControl.Update` | read and update |


## Config Mapping Data Definition
`Light.Data` support using the configuration file of json format for the mapping configuration, so the data model does not need to reference `Light.Data` library for `Attribute` define

e.g.

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

The configuration file path can be set in the program initialization, and more than one can be set.

```csharp
DataMapperConfiguration.AddConfigFilePath("Config/lightdata.json");
```

The default configuration file in the current directory `lightdata.json`, if the file does not exist, it is automatically ignored.

The configuration data structure is as follows

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

`lightData` node is the root node of the mapping configuration, can be added to any json configuration files.

dataType field types

| Field Name | Type | Nullable | Introduce |
|:------|:------|:------|:------|
| type | string | false | The full name of the entity class |
| tableName | string | false | The table name of the corresponding data entity class |
| isEntityTable | bool | true | Specifies whether the data table is an entity table, otherwise it is a view. The view cannot be added, deleted or modified, and the default is true. |
| dataFields | array(dataField) | true | The collection of mapped fields in the entity class |

dataField field defines

| Field Name | Type | Nullable | Introduce |
|:------|:------|:------|:------|
| fieldName | string | false | The field name of the entity class |
| isPrimaryKey | bool | true | Whether the field is auto incremented, and the default is false |
| isIdentity | bool | true | Whether the field is auto incremented, and the default is false |
| isNullable | bool  | true | whether the field can be null, and the default is false, which affects the new and updated data |
| name | string | true | The database field name of the field map, which defaults to the field name of the entity class |
| defaultValue | string | true | Default Value, nullable type field when new data is added, if the data is null, the DefaultValue will be used automatically. |
| functionControl | string | true | field Read and write access control function, default for all controls |

## Field Type

`Light.Data` mapping fields support the following types

* Native data type `bool`, `byte`, `sbyte`, `short`, `ushort `, `int`, `uint`, `long`, `ulong`, `double`, `float`, `decimal`, `DateTime` and its nullable type, The field type needs to correspond to the database type, If the database does not support this type, use a type with a larger bits, Such as in Mssql `sbyte `=>`smallint`, `ushort`=>`int`, `uint`=>`bigint`, `ulong`=>`decimal(20,0)`
* `Enum` types and nullable types, the database needs corresponding numeric types of inherited Enum itself
* `string` type, needs to use the chars type in database, such as `char`, `varchar`, `text` etc. 
* `byte[]` type, needs to use binary types, such as `binary`, `varbinary` etc., This type only supports conditional queries that are null or not
* Custom object types, needs to use the chars type in database, such as `char`, `varchar`, `text`tc. , `Light.Data` will automatically serialize the object to json format for access, and this type only supports conditional queries for null or not, and all equality

## Master Table Associated With Child Table

If the database has a master table and a child table to do association query, the mapping of the child table can be a single object (one-to-one) or a collection (one-to-many) as an property of the master table mapping class. they compose a composite objects. And the composite object can only be used for queries

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

Child table mapping

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

Inheritance `TeUser` and create a class

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

In the derived class `TeUserAndExtend` add a public property `Extend` which the type is
`TeUserExtend`, and add Attribute `RelationFieldAttribute`. In query `TeUserAndExtend`, 
the associated `TeUserExtend` data will be also detect.

The first parameter of `RelationField` is associated field name in master table, the second  parameter of `RelationField` is associated field name in child table. If there are multiple pairs of associated fields in the master and child table, you can set multiple groups of attributes, such as

```csharp
[RelationField("MKey1", "SKey1")]
[RelationField("MKey2", "SKey2")]
public TeUserExtend Extend
{
    get;
    set;
}
```

As the master table with the child table is a one-to-many relationship, property type need to use `ICollection<T>` or `LCollection<T>`, the type `T` is child table type, such as

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

Associated properties can also configure in the configuration file `dataType` node, associated field node is `relationFields`. The derived class configuration will default base class to inherit the `dataFields` and `relationFields` configuration, such as

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

relationFields field defines

| Field Name | Type | Nullable | Introduce |
|:------|:------|:------|:------|
| fieldName | string | false | The field name of the entity class |
| relationPairs | array(relationPair) | false | Association field combination |

relationPair field defines

| Field Name | Type | Nullable | Introduce |
|:------|:------|:------|:------|
| masterKey | string | false | The associated field property name of the master table |
| masterKey | string | false | The associated field property name of the child table |

### Multiple Child Table Associations

A master table class supports multiple different child table classes for association properties

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

### Child Table Cascade

The child table class can  as the master table associate other child tables

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

### Two Tables Are Master And Child To Each Other

When two table classes are master and child to each other, you need to make sure that the associated fields of both parties are master and child to each other, and the definitions are consistent, otherwise an error will occur

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

## Entity Classes And Configuration Files Templates

Mapping classes or json configuration files can be generated directly from the T4 template

[Templates URI](https://github.com/aquilahkj/Light.Data2/tree/master/template)

In addition you need to use the library `Light.Data.Template.dll`, and use ADO.NET library for the corresponding database. 

[Library URI](https://github.com/aquilahkj/Light.Data2/tree/master/lib)

Method of use

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
