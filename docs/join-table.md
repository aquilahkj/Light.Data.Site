# Join Table

## Join Method

Using `IQuery<T>.Join<T1>(on_lambda,T1_where_lambda)` method to join tables, `on_lambda` is a relate `Lambda` expression of Table `T` and Table `T1`, Such as `T.Id==T1.Id`, `where_lambda` is query `Lambda` expression of  Table `T1`. `Join` method return `IJoinTable<T,T1>` model.  

Using the Builder pattern add aggregate elements to `IJoinTable` model, or continue add join object, Maximum support for 10 table joins, And support join with IQuery, ISelect and IAggregate. finally use `Select` method to select specified fields and use enumerator or `ToList()` method to the output join query result.

IJoinTable primary query method:

| Method | Introduce |
|:------|:------|
| Where(Expression<Func<T, T1, bool>> expression) | Set the `IJoinTable` query conditions, Such as `Where((x, y) => x.Id > 1 && y.Status == 1)` |
| WhereWithAnd(Expression<Func<T, T1, bool>> expression) | Using `And` way to joint the current query condition and `IJoinTable` query condition |
| WhereWithOr(Expression<Func<T, T1, bool>> expression) | Using `Or` way to joint the current query condition and `IJoinTable` query condition |
| WhereReset() | Reset `IJoinTable` query condition |
| OrderBy<TKey>(Expression<Func<T, T1, TKey>> expression) | Set up `IJoinTable` sort field to positive sequence sorting, Such as `OrderBy((x, y) => x.Id)` |
| OrderByDescending<TKey> (Expression<Func<T, T1, TKey>> expression) | Set up `IJoinTable` sort field to reverse sequence sorting, Such as  |
| OrderByConcat<TKey> (Expression<Func<T, T1, TKey>> expression) | joint the `IJoinTable` sequence sorting  and positive sequence sorting of specified field |
| OrderByDescendingConcat<TKey> (Expression<Func<T, T1, TKey>> expression) | joint the `IJoinTable` sequence sorting and reverse sequence sorting of specified field |
| OrderByReset() | Reset `IJoinTable` sequence sorting |
| OrderByRandom() | Set up `IJoinTable` random sequence sorting |
| Take(int count) | Set up `IJoinTable` output data quantity |
| Skip(int count) | Set up `IJoinTable` output data skip index |
| Range(int from, int to) | Set up `IJoinTable` output data range from `param 1` to `param 2` |
| PageSize(int page, int size) | Set up `IJoinTable` output data paging range, page: from 1 start ,size: quantity each page |
| RangeReset() | Reset `IJoinTable`output data range |
| SetDistinct(bool distinct) | Set whether to use `Distinct` way output data |
| Select<K> (Expression<Func<T, T1, K>> expression)| Select specified field for output |
| Count()	| Return the length of the query data, and the type is int |
| LongCount() | Return the length of the query data, and the type is long |

Three join modes are supported `Inner Join`, `Left Join`和`RightJoin` 

```csharp
// inner join
var join = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id);
        
// left join
var join = context.Query<TeUser> ()
                  .LeftJoin<TeUserExtend>((x,y) => x.Id == y.Id);
        
// right join
var join = context.Query<TeUser> ()
                  .RightJoin<TeUserExtend>((x,y) => x.Id == y.Id);
```

## Join Type

### Basic Table Join

```csharp
// join table
var join = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id);

// join table + table query
var join = context.Query<TeUser> ()
                  .Join<TeUserExtend>(y => y.ExtendData != null, (x, y) => x.Id == y.Id);

// join table + together query
var join = context.Query<TeUser> ()
                  .Join<TeUserExtend>(context.Query<TeUserExtend>()
                  .Where(y => y.ExtendData != null), (x,y) => x.Id == y.Id);
```

### Aggregate Model Join

```csharp
// entity table join aggregate model
var join = context.Query<TeUser>()
                  .Join(context.Query<TeUserSub>()
                               .Aggregate(x => new {
                                   MId = x.MId,
                                   Count = Function.Count(),
                                }), (x, y) => x.Id == y.MId
                   );
        
//aggregate model join entity table
var join = context.Query<TeMainTable>()
                  .Aggregate(x => new {
                      MId = x.MId,
                      Count = Function.Count(),
                   })
                  .Join<TeSubTable>((x, y) => x.MId == y.Id);
```

## Condition Query (Where)

Using `Where(lambda)` method to add query conditions, the parameters is `Lambda` expression.

```csharp
var list = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id)
                  .Where((x,y) => x.Id > 1 && y.Status == 1 );

var list = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id)
                  .Where((x,y) => x.Id > 1)
                  .WhereWithAnd((x,y) => y.Status == 1 );
```

## Sorting (OrderBy)

Using `OrderBy(lambda)` method to add sorting rules, the parameters is `Lambda` expression.

```csharp
var join = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id)
                  .OrderBy((x,y) => x.Id);

var join = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id)
                  .OrderByDescending((x,y) => y.Date)

var join = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id)
                  .OrderBy((x,y) => x.Id)
                  .OrderByDescendingConcat((x,y) => y.Date);
```

## Select Specified Field (Select)

Using `Select(lambda)` method select specified fields, the parameter is `Lambda` expression, use create a new object way to define output object type, and support anonymous class output. `Select` method return `ISelectJoin<K>` model for output data.

```csharp
var list = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id)
                  .Select ((x,y) => new TeUserJoin () {
                      Id = x.Id,
                      Account = x.Account,
                      LevelId = x.LevelId,
                      RegTime = x.RegTime,
                      ExtendData = y.ExtendData
                   })
                  .ToList ();
// anonymous class
var list = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id)
                  .Select ((x,y) => new {
                      x.Id,
                      x.Account,
                      x.LevelId,
                      x.RegTime,
                      y.ExtendData
                   })
                  .ToList ();

var list = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id)
                  .Select ((x,y) => new {
                      Main = x,
                      ExtendEntity = y
                   })
                  .ToList ();
```

When all fields of a join model output no data (that is, null), If the type of the corresponding field is `int`,`decimal`,`DateTime` ..., The framework assigns default values to this field, But if the corresponding field type is type of join model directly, Such as 

```csharp
var list = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id)
                  .Select ((x,y) => new {
                      x.Id,
                      x.Account,
                      x.LevelId,
                      x.RegTime,
                      ExtendEntity = y
                   })
                  .ToList ();
```

The corresponding `ExtendEntity` field will generate a default `TeUserExtend` object, But if `JoinSetting` set to `NoDataSetEntityNull`, the filed will set null.

## Join Setting

In the join method using `JoinSetting`, to make a join setting for the join model. The `JoinSetting` is Enum.

| Enum | Introduce |
|:------|:------|
| JoinSetting.QueryDistinct | Set distinct for join model |
| JoinSetting.NoDataSetEntityNull | When all fields of the join model output no data, the join model will be set null |

```csharp
var list = context.Query<TeUser> ()
                  .Join<TeExtend>((x,y) => x.Id == y.Id, JoinSetting.NoDataSetEntityNull | JoinSetting.QueryDistinct)
                  .Select ((x,y) => new {
                      x.Id,
                      x.Account,
                      x.LevelId,
                      x.RegTime,
                      ExtendEntity = y
                   })
                   .ToList ();
```

## Query Result Insert

Insert query data to specified data table, It will generates `insert into t1(x1,x2,x3...)select t2.y1,t2.y2,t2.y3,t3.z1 from t2 join t3` command to bulk insert data directly.

Using `SelectInsert<K>(lambda)`method to insert specified fields data, the `K` is the type of inserted table, use create a new object way to define the insert fields and select fields, the left side is the insert fields, the right side is select fields, the result is the count of success insert rows.

```csharp
var result = context.Query<TeUser> ()
                    .Join<TeUserExtend>((x,y) => x.Id == y.Id)
                    .SelectInsert ((x,y) => new TeUserJoin () {
                        Id = x.Id,
                        Account = x.Account,
                        LevelId = x.LevelId,
                        RegTime = x.RegTime,
                        ExtendData = y.ExtendData 
                    });
```