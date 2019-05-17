# 数据处理

## 新增数据

自增`ID`会在Insert后自动赋值

```csharp
TeUser user = new TeUser ();
user.Account = "test";
user.Birthday = new DateTime (2001, 10, 20);
user.Email = "test@test.com";
user.Gender = GenderType.Female;
user.LevelId = 1;
user.NickName = "foo";
user.Password = "bar";
user.RegTime = new DateTime (2015, 12, 30, 18, 0, 0);
user.Status = 1;
user.Telephone = "12345678";
user.HotRate = 1.0d;
context.Insert (user);
```

批量新增数据方法

```csharp
BatchInsert<T>(IEnumerable<T> datas)
```

## 更新数据

数据对象做更新操作需要有主键

```csharp
int id = 1;
TeUser user = context.SelectById<TeUser> (id);
user.Status = 2;
context.Update (user);
```

批量更新数据方法

```csharp
BatchUpdate<T>(IEnumerable<T> datas)
```

## 删除数据

数据对象做删除操作需要有主键

```csharp
int id = 1;
TeUser user = context.SelectById<TeUser> (id);
context.Delete (user);
```

批量删除数据方法

```csharp
BatchDelete<T>(IEnumerable<T> datas)
```

## 新增或更新数据

根据主键自动判定需要新增数据还是更新数据, 当使用自增ID做主键时, 数据库ID必须由1开始.

```csharp
TeUser user = new TeUser ();
user.Account = "test";
user.Birthday = new DateTime (2001, 10, 20);
user.Email = "test@test.com";
user.Gender = GenderType.Female;
user.LevelId = 1;
user.NickName = "foo";
user.Password = "bar";
user.RegTime = new DateTime (2015, 12, 30, 18, 0, 0);
user.Status = 1;
user.Telephone = "12345678";
user.HotRate = 1.0d;
// 新增
context.InsertOrUpdate (user);

user.Status = 2;
// 更新
context.InsertOrUpdate (user);
```

## 数据实体类

映射类型可以通过继承`DataTableEntity`基类, 使用基类函数实现增删改操作, 该类定义为实体表. 

```csharp
[DataTable("Te_User_Entity")]
public class TeUserEntity : DataTableEntity
{
    private int id;

    [DataField("Id", IsIdentity = true, IsPrimaryKey = true)]
    public int Id
    {
        get { 
            return this.id; 
        }
        set { 
            this.id = value; 
            base.UpdateDataNotify(nameof(Id));
        }
    }

    private string account;

    [DataField("Account")]
    public string Account
    {
        get { 
            return this.account; 
        }
        set { 
            this.account = value; 
            base.UpdateDataNotify(nameof(Account));
        }
    }
}
```

```csharp
// 创建数据
TeUserEntity item = context.CreateNew<TeUserEntity>();
item.Account = "foo";
// 新增数据
item.Save();
item.Account = "bar";
// 更新数据
item.Save();
// 删除数据
item.Erase();
```

| 方法 | 说明 |
|:------|:------|
| UpdateDataNotify(string fieldName) | 用于字段的`Set`方法中, 在该字段修改后, 进行update操作, 只会更新该字段而不会更新所有字段, 所有字段需要设置  |
| Save() | 保存数据, 自动判定需要新增数据还是更新数据 |
| Erase() | 清除数据 |
| AllowUpdatePrimaryKey(bool allow = true) | 是否允许更新数据实体的主键, 允许后可以赋值新主键到该数据实体, 在更新操作后数据表的该行数据主键变为新主键 |
| Reset() | 重置数据状态 |
