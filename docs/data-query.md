# Data Query

## Query Method

Using the `Query` method of the `DataContext` generates `IQuery` model, and the `T` is mapping type of data table.

```csharp
IQuery query = context.Query<T>()
```

Using the Builder pattern add query elements to `IQuery` model, finally use enumerator or `ToList()` method to the output query result.

IQuery primary query method:

| Method | Introduce |
|:------|:------|
| Where(Expression<Func<T, bool>> expression) | Set the `IQuery` query conditions, Such as `Where(x => x.Id > 1)` |
| WhereWithAnd(Expression<Func<T, bool>> expression) | Using `And` way to joint the current query condition and `IQuery` query condition |
| WhereWithOr(Expression<Func<T, bool>> expression) | Using `Or` way to joint the current query condition and `IQuery` query condition |
| WhereReset() | Reset `IQuery` query condition |
| OrderBy<TKey>(Expression<Func<T, TKey>> expression) | Set up `IQuery` sort field to positive sequence sorting, Such as `OrderBy(x => x.Id)` |
| OrderByDescending<TKey>(Expression<Func<T, TKey>> expression) | Set up `IQuery` sort field to reverse sequence sorting, Such as `OrderByDescending(x => x.Id)` |
| OrderByConcat<TKey>(Expression<Func<T, TKey>> expression) | joint the `IQuery` sequence sorting  and positive sequence sorting of specified field |
| OrderByDescendingConcat<TKey>(Expression<Func<T, TKey>> expression) | joint the `IQuery` sequence sorting and reverse sequence sorting of specified field |
| OrderByReset() | Reset `IQuery` sequence sorting |
| OrderByRandom() | Set up `IQuery` random sequence sorting |
| Take(int count) | Set up `IQuery` output data quantity |
| Skip(int count) | Set up `IQuery` output data skip index |
| Range(int from, int to) | Set up `IQuery` output data range from `param 1` to `param 2` |
| PageSize(int page, int size) | Set up `IQuery` output data paging range, page: from 1 start ,size: quantity each page |
| RangeReset() | Reset `IQuery`output data range |
| SetDistinct(bool distinct) | Set whether to use `Distinct` way output data |
| ToList() | Using `List<T>` way to output data |
| ToArray() |  Using `T[]` way to output data |
| First() | Output the first data result object, null if there is no data |
| ElementAt(int index) | Output specifies the index position data result object, null if there is no data |
| Exists() | Determine there is any data |
| Count()	| Return the length of the query data, and the type is int |
| LongCount() | Return the length of the query data, and the type is long |


### Full Query

```csharp
List<TeUser> list = context.Query<TeUser> ().ToList ();
```

### Combination Query

```csharp
List<TeUser> list = context.Query<TeUser> ().Where (x => x.Id > 1).OrderBy (x => x.Id).Take(10).ToList ();
```

## Condition Query (Where)

Using `Where(lambda)` method to add query conditions, the parameters is `Lambda` expression.

```csharp
context.Query<T> ().Where(x => x.Id > 1)
```

### General

```csharp
 List<TeUser> list1 = context.Query<TeUser> ().Where (x => x.Id >= 5 && x.Id <= 10).ToList ();
 List<TeUser> list2 = context.Query<TeUser> ().Where (x => x.Id < 5 || x.Id > 10).ToList ();
```

### In

Using `List<T>.Contains` method

```csharp
int [] array = new int [] { 3, 5, 7 };
List<int> list = new List<int> (array);
// id in (3, 5, 7)
List<TeUser> list1 = context.Query<TeUser> ().Where (x => list.Contains (x.Id)).ToList ();
// id not in (3, 5, 7)
List<TeUser> list2 = context.Query<TeUser> ().Where (x => ！list.Contains (x.Id)).ToList ();
```

### Like

Only support type string, use `string.StartsWith`、`string.EndsWith`、`string.Contains` method query, can support the reverse query.

```csharp
// Account like 'foo%'
List<TeUser> list1 = context.Query<TeUser> ().Where (x => x.Account.StartsWith ("foo")).ToList ();
// Account like '%foo'
List<TeUser> list2 = context.Query<TeUser> ().Where (x => x.Account.EndsWith ("foo")).ToList ();
// Account like '%foo%'
List<TeUser> list3 = context.Query<TeUser> ().Where (x => x.Account.Contains ("foo")).ToList ();
// foo like '%'+Account
List<TeUser> list1 = context.Query<TeUser> ().Where (x => "foo".EndsWith (x.Account)).ToList ();
// Account not like 'foo%'
List<TeUser> list1 = context.Query<TeUser> ().Where (x => !x.Account.StartsWith ("foo")).ToList ();
```

### Null

The query field needs to be of a nullable type (such as int?) or a string type

```csharp
// RefereeId is null
List<TeUser> list1 = context.Query<TeUser> ().Where (x => x.RefereeId == null).ToList ();
// RefereeId is not null
List<TeUser> list2 = context.Query<TeUser> ().Where (x => x.RefereeId != null).ToList ();
```

If the field is not a nullable type, can use extensions `ExtendQuery.IsNull()`

```csharp
// Id is null
List<TeUser> list1 = context.Query<TeUser> ().Where (x => ExtendQuery.IsNull (x.Id)).ToList ();
// Id is not null
List<TeUser> list2 = context.Query<TeUser> ().Where (x => !ExtendQuery.IsNull (x.Id)).ToList ();
```

### Boolean

The query field needs to be of type Boolean

```csharp
// true
List<TeUser> list1 = context.Query<TeUser> ().Where (x => x.DeleteFlag).ToList ();
List<TeUser> list1 = context.Query<TeUser> ().Where (x => x.DeleteFlag == true).ToList ();
List<TeUser> list1 = context.Query<TeUser> ().Where (x => x.DeleteFlag != false).ToList ();
// false
List<TeUser> list2 = context.Query<TeUser> ().Where (x => !x.DeleteFlag).ToList ();
List<TeUser> list2 = context.Query<TeUser> ().Where (x => x.DeleteFlag != true).ToList ();
List<TeUser> list2 = context.Query<TeUser> ().Where (x => x.DeleteFlag == false).ToList ();
```
### Exists With Cross Table

Fixed condition query

```csharp
// exist (select 1 from TeUserLevel where Status = 1)
List<TeUser> list1 = context.Query<TeUser> ().Where (x => ExtendQuery.Exists<TeUserLevel> (y => y.Status == 1)).ToList ();
// not exist (select 1 from TeUserLevel where Status = 1)
List<TeUser> list2 = context.Query<TeUser> ().Where (x => !ExtendQuery.Exists<TeUserLevel> (y => y.Status == 1)).ToList ();
```
Association condition query

```csharp
// exist exist (select 1 from TeUserLevel where Id = TeUser.LevelId)
List<TeUser> list1 = context.Query<TeUser> ().Where (x => ExtendQuery.Exists<TeUserLevel> (y => y.Id == x.LevelId)).ToList ();
// not exist (select 1 from TeUserLevel where Id = TeUser.LevelId)
List<TeUser> list2 = context.Query<TeUser> ().Where (x => !ExtendQuery.Exists<TeUserLevel> (y => y.Id == x.LevelId)).ToList ();
```

## Sorting (OrderBy)

Using `OrderBy(lambda)` method to add sorting rules, the parameters is `Lambda` expression.

```csharp
context.Query<T> ().OrderBy(x => x.Id)
```

### Positive Sorting

```csharp
List<TeUser> list = context.Query<TeUser> ().OrderBy (x => x.Id).ToList ();
```

### Reverse Sorting

```csharp
List<TeUser> list = context.Query<TeUser> ().OrderByDescending (x => x.Id).ToList ();
```

### Concat Sorting

Use `Concat` method to joint multiple Sorting

```csharp
List<TeUser> list = context.Query<TeUser> ().OrderBy (x => x.Gender).OrderByDescendingConcat (x => x.Id).ToList ();
```

## Select Specified Field (Select)

Using `Select(lambda)` method specified number of field of new type, support anonymous class output.

```csharp
var list = context.Query<TeUser> ()
                  .Select (x => new TeUserSimple () {
                      Id = x.Id,
                      Account = x.Account,
                      LevelId = x.LevelId,
                      RegTime = x.RegTime
                   })
                  .ToList ();
//匿名类
var list = context.Query<TeUser> ()
                  .Select (x => new {
                      x.Id,
                      x.Account,
                      x.LevelId,
                      x.RegTime
                   })
                  .ToList ();
```

Query the list of single-column fields

```csharp
List<int> list = context.Query<TeUser> ().Select (x => x.Id).ToList ();
```

## Query Batch Update

Using `Update(lambda)` method to batch update, the parameter is `Lambda` expression, use create a new object way to define the data update fields and the update content, the left side is the update fields , the right side is update content, the update content can be the original field, the result is the count of success update rows.

```csharp
var result = context.Query<TeUser> ()
                    .Update (x => new TeUser {
                        LastLoginTime = DateTime.Now,
                        Status = 2
                     });
// the update content can be the original field
var result = context.Query<TeUser2> ()
                    .Update (x => new TeUser2 {
                        LastLoginTime = x.RecordTime,
                        Status = x.Status + 1
                     });
```

## Query Batch Delete

Using `Delete()` to batch delete, the result is the count of success delete rows.

```csharp
var result = context.Query<TeUser> ().Where(x => x.Id > 1)
                    .Delete();
```

## Query Batch Insert

Insert query data from specified data table, you can use all the fields or the specified fields, It will generates `insert into t1(x1,x2,x3...)select y1,y2,y3 from t2` command to bulk insert data directly.

### All Fields Insert

Using `Insert<K>()` method to insert all fields data , the `K` is the type of insert table, query table fields must be one-to-one correspondence with insert table fields, the result is the count of success insert rows.

If the query table has an auto increment field in the same position as the insert table, then the increment field of insert table will continues auto increment.

```csharp
var result = context.Query<TeDataLog> ()
		    .Insert<TeDataLogHistory> ();
```

### Specified Fields Insert

Using `SelectInsert<K>(lambda)` method to insert specified fields data, the `K` is the type of inserted table, use create a new object way to define the insert fields and select fields, the left side is the insert fields, the right side is select fields, the result is the count of success insert rows.

```csharp
var result = context.Query<TeDataLog> ()
                    .SelectInsert (x => new TeDataLogHistory () {
                        Id = x.Id,
                        UserId = x.UserId,
                        ArticleId = x.ArticleId,
                        RecordTime = x.RecordTime,
                        Status = x.Status,
                        Action = x.Action,
                        RequestUrl = x.RequestUrl,
                        CheckId = 3,
                     });
```


## Primary Key/Auto Increment Fields Query

Query data by Auto Increment Id field

```csharp
int id = 1;
TeUser user = context.SelectById<TeUser> (id);
```

Query data by Primary Key, if there are multiple primary keys, put them all in order.

```csharp
int key = 1;
TeUser user = context.SelectByKey<TeUser> (key);
```