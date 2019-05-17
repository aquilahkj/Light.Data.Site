# Data Process

## Insert Data

The increase `ID` will be auto assigned after Insert.

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

The method of batch insert datas 

```csharp
BatchInsert<T>(IEnumerable<T> datas)
```

## Update Data

Data object needs to have a primary key for update operation

```csharp
int id = 1;
TeUser user = context.SelectById<TeUser> (id);
user.Status = 2;
context.Update (user);
```

The method of batch update datas 

```csharp
BatchUpdate<T>(IEnumerable<T> datas)
```

## Delete Data

Data object needs to have a primary key for delete operation

```csharp
int id = 1;
TeUser user = context.SelectById<TeUser> (id);
context.Delete (user);
```

The method of batch delete datas

```csharp
BatchDelete<T>(IEnumerable<T> datas)
```

## Insert Or Update Data

Auto determine whether insert or updated data is needed based on the primary key, When using an auto increment ID as the primary key, the database ID must start with 1.

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
// Insert
context.InsertOrUpdate (user);

user.Status = 2;
// Update
context.InsertOrUpdate (user);
```

## Data Entity Class

Mapping type can be inherited base class `DataTableEntity`, and use base class methods for CUD operation. This type is defined as an entity table. 

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
//  create data
TeUserEntity item = context.CreateNew<TeUserEntity>();
item.Account = "foo";
// insert data
item.Save();
item.Account = "bar";
// update date
item.Save();
// delete data
item.Erase();
```

| Method | Introduce |
|:------|:------|
| UpdateDataNotify(string fieldName) | Used in the `Set` method of field, after the field is modified, update operation is performed, only the field is updated and not all fields are updated. All fields need to be Set |
| Save() | Save data and automatically determine whether insert or updated data is needed |
| Erase() | Delete datas |
| AllowUpdatePrimaryKey(bool allow = true) | Set whether the primary key of a data entity is allowed to be updated, and then a new primary key can be assigned to the data entity. After the update operation, the data primary key of the row in the data table becomes the new primary key |
| Reset() | Reset data status |
