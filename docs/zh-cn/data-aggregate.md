# 数据汇总

## 汇总统计数据

使用`IQuery<T>.Aggregate<K>(lambda)`方法进行汇总统计数据, 参数是`lambda`表达式, 使用创建new object方式定义数据的统计字段与汇总函数, 支持匿名类输出. `Aggregate`方法返回`IAggregate<K>`模型.

`IAggregate`模型通过Builder模式添加查询要素, 最后通过枚举方式或`ToList()`方法输出统计结果.

`IAggregate<K>` 主要汇总方法

| 方法 | 说明 |
|:------|:------|
| Having(Expression<Func<K, bool>> expression) | 设置`IAggregate`的过滤条件, 如`Having(x => x.Count > 1)` |
| HavingWithAnd(Expression<Func<K, bool>> expression) | 使用`And`方式连接当前过滤条件和`IAggregate`中过滤条件 |
| HavingWithOr(Expression<Func<K, bool>> expression) | 使用`Or`方式连接当前过滤条件和`IAggregate`中过滤条件 |
| HavingReset() | 把`IAggregate`中过滤条件重置 |
| OrderBy<TKey>(Expression<Func<K, TKey>> expression) | 设置`IAggregate`的排序字段, 以正序排序, 如`OrderBy(x => x.Id)` |
| OrderByDescending<TKey> (Expression<Func<K, TKey>> expression) | 设置`IAggregate`的排序字段, 以倒序排序, 如`OrderByDescending(x => x.Id)` |
| OrderByConcat<TKey>(Expression<Func<K, TKey>> expression) | 把`IAggregate`中排序连接当前字段正序排序 |
| OrderByDescendingConcat<TKey>(Expression<Func<K, TKey>> expression) | 把`IAggregate`中排序连接当前字段倒序排序 |
| OrderByReset() | 把`IAggregate`中排序重置 |
| OrderByRandom() | 把`IAggregate`中排序置为随机排序 |
| Take(int count) | 设定`IAggregate`输出结果的数量 |
| Skip(int count) | 设定`IAggregate`输出结果需要跳过的数量 |
| Range(int from, int to) | 设定`IQuery`查询结果从`参数1`位到`参数2`位 |
| PageSize(int page, int size) | 设定`IQuery`输出结果的分页结果, page:从1开始页数,size:每页数量 |
| RangeReset() | 把`IAggregate`中输出结果范围 |
| ToList() | 以`List`的方式输出数据 |
| ToArray() | 以`Array`的方式输出数据 |
| First() | 输出首个数据结果对象, 如无数据则为null |

## 汇总函数

使用静态类 `Function`

| 函数 | 说明 |
|:------|:------|
| Count() | 总数量, 返回int类型结果 |
| LongCount() | 总数量, 返回long类型结果 |
| CountCondition(condition) | 符合条件的总数量, 返回int类型结果 |
| LongCountCondition(condition) | 符合条件的总数量, 返回long类型结果 |
| Count(field) | 指定字段总数量, 不包含空数据, 返回int类型结果 |
| LongCount(field) | 指定字段总数量, 不包含空数据, 返回long类型结果 |
| DistinctCount(field) | 指定字段去重复总数量, 不包含空数据, 返回int类型结果 |
| DistinctLongCount(field) | 指定字段去重复总数量, 不包含空数据, 返回long类型结果 |
| Sum(field) | 指定字段数值累加总和, 返回汇总字段类型结果 |
| LongSum(field) |指定字段数值累加总和, 返回long类型结果 |
| DistinctSum(field) | 指定字段去重复后累加总和, 返回汇总字段类型结果 |
| DistinctLongSum(field) | 指定字段去重复后累加总和, 返回long类型结果 |
| Avg(field) | 指定字段平均值, 返回double类型结果 |
| DistinctAvg(field) | 指定字段去重复后平均值,返回double类型结果 |
| Max(field) | 指定字段的最大值 |
| Min(field) | 指定字段的最小最 |


### 总数量汇总

```csharp
// 普通汇总
var list = context.Query<TeUser> ()
                  .Where (x => x.Id >= 5)
                  .Aggregate (x => new LevelIdAgg () {
                      LevelId = x.LevelId,
                      Data = Function.Count ()
                   })
                  .ToList ();
// 使用匿名类汇总
var list = context.Query<TeUser> ()
                  .Where (x => x.Id >= 5)
                  .Aggregate (x => new {
                      LevelId = x.LevelId,
                      Data = Function.Count ()
                   })
                  .ToList ();
// 条件汇总
var list = context.Query<TeUser> ()
                  .Where (x => x.Id >= 5)
                  .Aggregate (x => new {
                      LevelId = x.LevelId,
                      Valid = Function.CountCondition (x.Status == 1),
                      Invalid = Function.CountCondition (x.Status != 1)
                   })
                  .ToList ();
```

### 指定字段计数汇总

```csharp
// 指定字段总数量
var list = context.Query<TeUser> ()
                  .Aggregate (x => new LevelIdAgg () {
                      LevelId = x.LevelId,
                      Data = Function.Count (x.Area)
                   })
		  .ToList ();
// 符合条件的总数量
var list = context.Query<TeUser> ()
                  .Where (x => x.Id >= 5)
                  .Aggregate (x => new {
                      LevelId = x.LevelId,
                      Valid = Function.Count (x.Status = 1 ? x.Area : null),
                      Invalid = Function.Count (x.Status != 1 ? x.Area : null)
                   })
                  .ToList ();
```

## 汇总数据过滤(Having)

使用`Having(lambda)`方法加入过滤条件, 对汇总数据过滤, 查询参数为`Lambda`表达式.

```csharp
var list = context.Query<TeUser> ()
                  .Aggregate (x => new LevelIdAgg () {
                      LevelId = x.LevelId,
                      Data = Function.Sum (x.LoginTimes)
                   })
                  .Having (y => y.Data > 15)
                  .ToList ();
```

## 汇总数据排序(OrderBy)

使用`IAggregate<K>.OrderBy(lambda)`方法加入排序规则, 查询参数为`Lambda`表达式.

```csharp
// 汇总字段排序
var list = context.Query<TeUser> ()
                  .Aggregate (x => new LevelIdAgg () {
                      LevelId = x.LevelId,
                      Data = Function.Count ()
                   })
                  .OrderBy (x => x.LevelId)
                  .ToList ();
// 汇总结果排序
var list = context.Query<TeUser> ()
                  .Aggregate (x => new LevelIdAgg () {
                      LevelId = x.LevelId,
                      Data = Function.Count ()
                   })
                  .OrderBy (x => x.Data)
                  .ToList ();
```

## 汇总字段扩展

详见 [字段扩展](field-extension.md)

## 指定字段统计

### 日期类统计

```csharp
// 日期
var list = context.Query<TeUser> ()
                  .Aggregate (x => new RegDateAgg () {
                      RegDate = x.RegTime.Date,
                      Data = Function.Count ()
                   })
                  .ToList ();
// 日期格式化
var list = context.Query<TeUser> ()
                  .Aggregate (x => new RegDateFormatAgg () {
                      RegDateFormat = x.RegTime.ToString("yyyy-MM-dd"),
                      Data = Function.Count ()
                   })
                  .ToList ();	
// 获取年份
var list = context.Query<TeUser> ()
                  .Aggregate (x => new NumDataAgg () {
                      Name = x.RegTime.Year,
                      Data = Function.Count ()
                   })
                  .ToList ();	
```

### 字符串类统计

```csharp
// 截取字符串统计
var list = context.Query<TeUser> ()
                  .Aggregate (x => new StringDataAgg () {
                      Name = x.Account.Substring (0, 5),
                      Data = Function.Count ()
                   })
                  .ToList ();
// 字符串长度统计
var list = context.Query<TeUser> ()
                  .Aggregate (x => new NumDataAgg () {
                      Name = x.Account.Length,
                      Data = Function.Count ()
                   })
                  .ToList ();
```

## 汇总结果插入

将汇总数据插入指定的数据表, 会生成`insert into t1(x1,x2...)select a1,count(1) from t1 group by a1`命令直接批量插入数据.

使用`SelectInsert<K>(lambda)`选择指定字段举行插入, 参数是`lambda`表达式, 使用创建new object方式定义数据的插入字段与选择字段, 左侧为插入字段名, 右侧选择字段, 返回结果为成功新增行数. 

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