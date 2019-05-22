# 数据表连接

## 连接方法

使用`IQuery<T>.Join<T1>(on_lambda, where_lambda)`方法连接数据表, `on_lambda`是表`T`和表`T1`的on关联表达式, 如`T.Id==T1.Id`, `where_lambda`表达式中表T1的where查询. 函数返回`IJoinTable<T,T1>`模型. 

`IJoinTable`模型通过Builder模式添加查询要素, 或继续添加连接对象, 最多支持10个表连接. 支持`IQuery(查询结果)`与`ISelect(字段选择查询)`与`IAggregate(数据汇总)`的连接. 最后通过`Select`方法选择指定的字段并使用枚举方式或`ToList()`方法输出统计结果.

IJoinTable主要查询方法

| 方法 | 说明 |
|:------|:------|
| Where(Expression<Func<T, T1, bool>> expression) | 设置`IJoinTable`的查询条件,如`Where((x, y) => x.Id > 1 && y.Status == 1)` |
| WhereWithAnd(Expression<Func<T, T1, bool>> expression) | 使用`And`方式连接当前查询条件和`IJoinTable`中查询条件 |
| WhereWithOr(Expression<Func<T, T1, bool>> expression) | 使用`Or`方式连接当前查询条件和`IJoinTable`中查询条件 |
| WhereReset() | 把`IJoinTable`中查询条件重置 |
| OrderBy<TKey>(Expression<Func<T, T1, TKey>> expression) | 设置`IJoinTable`的排序字段, 以正序排序, 如`OrderBy((x, y) => x.Id)` |
| OrderByDescending<TKey> (Expression<Func<T, T1, TKey>> expression) | 设置`IJoinTable`的排序字段, 以倒序排序, 如`OrderByDescending((x, y) => x.Id)` |
| OrderByConcat<TKey> (Expression<Func<T, T1, TKey>> expression) | 把`IJoinTable`中排序连接当前字段正序排序 |
| OrderByDescendingConcat<TKey> (Expression<Func<T, T1, TKey>> expression) | 把`IJoinTable`中排序连接当前字段倒序排序 |
| OrderByReset() | 把`IJoinTable`中排序重置 |
| OrderByRandom() | 把`IJoinTable`中排序置为随机排序 |
| Take(int count) | 设定`IJoinTable`输出结果的数量 |
| Skip(int count) | 设定`IJoinTable`输出结果需要跳过的数量 |
| Range(int from, int to) | 设定`IJoinTable`查询结果从`参数1`位到`参数2`位 |
| PageSize(int page, int size) | 设定`IJoinTable`输出结果的分页结果,page:从1开始页数,size:每页数量 |
| RangeReset() | 重置`IJoinTable`中输出结果范围 |
| SetDistinct(bool distinct) | 设定是否使用`Distinct`方式输出结果 |
| Select<K> (Expression<Func<T, T1, K>> expression)| 选择需要输出的字段 |
| Count()  | 返回查询数据的数据长度, 返回类型为int |
| LongCount() | 返回查询数据的数据长度, 返回类型为long|

支持三种连接方式 `Inner Join`, `Left Join`和`RightJoin` 

```csharp
//内连接
var join = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id);
        
//左连接
var join = context.Query<TeUser> ()
                  .LeftJoin<TeUserExtend>((x,y) => x.Id == y.Id);
        
//右连接
var join = context.Query<TeUser> ()
                  .RightJoin<TeUserExtend>((x,y) => x.Id == y.Id);
```

## 连接类型

### 基本表连接

```csharp
//表连接
var join = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id);

//表连接+查询
var join = context.Query<TeUser> ()
                  .Join<TeUserExtend>(y => y.ExtendData != null, (x, y) => x.Id == y.Id);

//查询连接
var join = context.Query<TeUser> ()
                  .Join<TeUserExtend>(context.Query<TeUserExtend>()
                  .Where(y => y.ExtendData != null), (x,y) => x.Id == y.Id);
```

### 统计表连接

```csharp
//实体表连接统计结果
var join = context.Query<TeUser>()
                  .Join(context.Query<TeUserSub>()
                               .Aggregate(x => new {
                                   MId = x.MId,
                                   Count = Function.Count(),
                                }), (x, y) => x.Id == y.MId
                   );
        
//统计结果连接实体表             
var join = context.Query<TeMainTable>()
                  .Aggregate(x => new {
                      MId = x.MId,
                      Count = Function.Count(),
                   })
                  .Join<TeSubTable>((x, y) => x.MId == y.Id);
```

## 条件查询(Where)

使用`IJoinTable<T,T1>.Where(lambda)`方法加入查询条件,查询参数为`lambda`表达式

```csharp
var list = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id)
                  .Where((x,y) => x.Id > 1 && y.Status == 1 );

var list = context.Query<TeUser> ()
                  .Join<TeUserExtend>((x,y) => x.Id == y.Id)
                  .Where((x,y) => x.Id > 1)
                  .WhereWithAnd((x,y) => y.Status == 1 );
```

## 排序(OrderBy)

***

使用`IJoinTable<T,T1>.OrderBy(lambda)`方法加入查询条件, 查询参数为`lambda`表达式

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

## 选择指定字段(Select)

使用`Select(lambda)`方法进行汇总统计数据, 参数是`lambda`表达式, 使用创建new object方式定义输出数据类型, 支持匿名类输出. `Select`方法返回`ISelectJoin<K>`模型.

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
// 匿名类
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

当某个连接表的所有字段输出无数据(即为null), 如果对应字段类型是`int`,`decimal`,`DateTime`等值类型, 框架会赋予该类型的默认值, 但如果对应字段类型直接为某个连接表的对应类型, 如

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

对应的`ExtendEntity`字段则会生成一个`TeUserExtend`的默认对象, 但如果`JoinSetting`设置为`NoDataSetEntityNull`, 则为null

## 连接设置

在连接方法中使用`JoinSetting`, 可以对连接对象进行连接设置. `JoinSetting`是Enum

| Enum | 说明 |
|:------|:------|
| JoinSetting.QueryDistinct | 连接对象去重复的数据 |
| JoinSetting.NoDataSetEntityNull | 指定输出映射对象如果所有字段无数据时为null |

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

## 查询结果插入

将查询数据插入指定的数据表, 会生成`insert into t1(x1,x2,x3...)select t2.y1,t2.y2,t2.y3,t3.z1 from t2 join t3`命令直接批量插入数据.

使用`SelectInsert<K>(lambda)`选择指定字段举行插入, 参数是`lambda`表达式, 使用创建new object方式定义数据的插入字段与选择字段, 左侧为插入字段名, 右侧选择字段, 返回结果为成功新增行数. 

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