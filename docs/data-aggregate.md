# Data Aggregate

## Aggregate Statistics

Using `IQuery<T>.Aggregate<K>(lambda)` method to aggregate statistics data, the parameter is `Lambda` expression, use create a new object way to define the statistical fields and aggregate functions, and support anonymous class output. `Aggregate` method return `IAggregate<K>` model. 

Using the Builder pattern add aggregate elements to `IAggregate` model, finally use enumerator or `ToList()` method to the output aggregate result.

`IAggregate<K>` primary aggregate method:

| Method | Introduce |
|:------|:------|
| Having(Expression<Func<K, bool>> expression) | Set the `IAggregate` filter conditions, Such as `Having(x => x.Count > 1)` |
| HavingWithAnd(Expression<Func<K, bool>> expression) | Using `And` way to joint the current filter condition and `IAggregate` filter condition |
| HavingWithOr(Expression<Func<K, bool>> expression) | Using `Or` way to joint the current filter condition and `IAggregate` filter condition |
| HavingReset() | Reset `IAggregate` filter condition |
| OrderBy<TKey>(Expression<Func<K, TKey>> expression) | Set up `IAggregate` sort field to positive sequence sorting, Such as `OrderBy(x => x.Count)` |
| OrderByDescending<TKey> (Expression<Func<K, TKey>> expression) | Set up `IAggregate` sort field to reverse sequence sorting, Such as `OrderByDescending(x => x.Count)` |
| OrderByConcat<TKey>(Expression<Func<K, TKey>> expression) | joint the `IAggregate` sequence sorting  and positive sequence sorting of specified field |
| OrderByDescendingConcat<TKey>(Expression<Func<K, TKey>> expression) | joint the `IAggregate` sequence sorting and reverse sequence sorting of specified field |
| OrderByReset() | Reset `IAggregate` sequence sorting |
| OrderByRandom() | Set up `IAggregate` random sequence sorting |
| Take(int count) | Set up `IAggregate` output data quantity |
| Skip(int count) | Set up `IAggregate` output data skip index |
| Range(int from, int to) | Set up `IAggregate` output data range from `param 1` to `param 2` |
| PageSize(int page, int size) | Set up `IAggregate` output data paging range, page: from 1 start ,size: quantity each page |
| RangeReset() | Reset `IAggregate`output data range |
| ToList() | Using `List<T>` way to output data |
| ToArray() | Using `T[]` way to output data |
| First() | Output the first data result object, null if there is no data |

## Aggregate Function

Using Static Class`Function`

| Function | Introduce |
|:------|:------|
| Count() | The total number, result type is int |
| LongCount() | The total number, result type is long |
| CountCondition(condition) | The total number of hit conditions, result type is int |
| LongCountCondition(condition) | The total number of hit conditions, result type is long |
| Count(field) | The total number of specified field which not contain null data, result type is int |
| LongCount(field) | The total number of specified field which not contain null data, result type is long |
| DistinctCount(field) | The total number of specified field which not contain null data and distinct, result type is int |
| DistinctLongCount(field) | The total number of specified field which not contain null data and distinct, result type is long |
| Sum(field) | The sum of the specified field values, result type is field type |
| LongSum(field) | The sum of the specified field values, result type is long |
| DistinctSum(field) | The sum of the specified field values which distinct, result type is field type |
| DistinctLongSum(field) | The sum of the specified field values which distinct, result type is long |
| Avg(field) | The average value of the specified field, result type is double |
| DistinctAvg(field) | The average value of the specified field which distinct, result type is double |
| Max(field) | The maximum value of the specified field |
| Min(field) | The minimum value of the specified field |


### The Total Number Aggregate

```csharp
// general aggregate
var list = context.Query<TeUser> ()
                  .Where (x => x.Id >= 5)
                  .Aggregate (x => new LevelIdAgg () {
                      LevelId = x.LevelId,
                      Data = Function.Count ()
                   })
                  .ToList ();
// use anonymous class aggregate
var list = context.Query<TeUser> ()
                  .Where (x => x.Id >= 5)
                  .Aggregate (x => new {
                      LevelId = x.LevelId,
                      Data = Function.Count ()
                   })
                  .ToList ();
// condition aggregate
var list = context.Query<TeUser> ()
                  .Where (x => x.Id >= 5)
                  .Aggregate (x => new {
                      LevelId = x.LevelId,
                      Valid = Function.CountCondition (x.Status == 1),
                      Invalid = Function.CountCondition (x.Status != 1)
                   })
                  .ToList ();
```

### Specified Field Aggregate

```csharp
// the total number of specified field
var list = context.Query<TeUser> ()
                  .Aggregate (x => new LevelIdAgg () {
                      LevelId = x.LevelId,
                      Data = Function.Count (x.Area)
                   })
		  .ToList ();
// the total number of hit conditions
var list = context.Query<TeUser> ()
                  .Where (x => x.Id >= 5)
                  .Aggregate (x => new {
                      LevelId = x.LevelId,
                      Valid = Function.Count (x.Status = 1 ? x.Area : null),
                      Invalid = Function.Count (x.Status != 1 ? x.Area : null)
                   })
                  .ToList ();
```

## Filter Aggregate Data (Having)

Using `Having(lambda)` method to add filter conditions for filter aggregate result, the parameters is `Lambda` expression.

```csharp
var list = context.Query<TeUser> ()
                  .Aggregate (x => new LevelIdAgg () {
                      LevelId = x.LevelId,
                      Data = Function.Sum (x.LoginTimes)
                   })
                  .Having (y => y.Data > 15)
                  .ToList ();
```

## Sorting Aggregate Data (OrderBy)

Using `Having(lambda)` method to add sorting rules, the parameters is `Lambda` expression.

```csharp
// aggregate field sorting
var list = context.Query<TeUser> ()
                  .Aggregate (x => new LevelIdAgg () {
                      LevelId = x.LevelId,
                      Data = Function.Count ()
                   })
                  .OrderBy (x => x.LevelId)
                  .ToList ();
// aggregate result sorting
var list = context.Query<TeUser> ()
                  .Aggregate (x => new LevelIdAgg () {
                      LevelId = x.LevelId,
                      Data = Function.Count ()
                   })
                  .OrderBy (x => x.Data)
                  .ToList ();
```

## Aggregate Field Extension

See [字段扩展](field-extension.md)

## Specified Field Statistics

### Date Statistics

```csharp
// date
var list = context.Query<TeUser> ()
                  .Aggregate (x => new RegDateAgg () {
                      RegDate = x.RegTime.Date,
                      Data = Function.Count ()
                   })
                  .ToList ();
// date formatter
var list = context.Query<TeUser> ()
                  .Aggregate (x => new RegDateFormatAgg () {
                      RegDateFormat = x.RegTime.ToString("yyyy-MM-dd"),
                      Data = Function.Count ()
                   })
                  .ToList ();	
// get year
var list = context.Query<TeUser> ()
                  .Aggregate (x => new NumDataAgg () {
                      Name = x.RegTime.Year,
                      Data = Function.Count ()
                   })
                  .ToList ();	
```

### String Statistics

```csharp
// substring
var list = context.Query<TeUser> ()
                  .Aggregate (x => new StringDataAgg () {
                      Name = x.Account.Substring (0, 5),
                      Data = Function.Count ()
                   })
                  .ToList ();
// string length
var list = context.Query<TeUser> ()
                  .Aggregate (x => new NumDataAgg () {
                      Name = x.Account.Length,
                      Data = Function.Count ()
                   })
                  .ToList ();
```
## Aggregate Result Insert

Insert aggregate data to specified data table, It will generates `insert into t1(x1,x2...)select a1,count(1) from t1 group by a1` command to bulk insert data directly.

Using `SelectInsert<K>(lambda)`method to insert specified fields data, the `K` is the type of inserted table, use create a new object way to define the insert fields and select fields, the left side is the insert fields, the right side is select fields, the result is the count of success insert rows.

```csharp
var result = context.Query<TeUser> ()
                    .Aggregate (x => new {
                      LevelId = x.LevelId,
                      Data = Function.Count ()
                    })
                    .SelectInsert(x => new TeLevelAgg() {
                        LevelId = x.LevelId,
                        Data = x.Data
                    });
```