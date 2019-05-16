# 数据查询

## 查询方法

使用`DataContext`的`Query`方法生成`IQuery`模型, `T`为数据表的映射类型.

```csharp
IQuery query = context.Query<T>()
```
`IQuery`模型通过Builder模式添加查询要素, 最后通过枚举方式或`ToList()`方法输出查询结果.

IQuery主要查询用方法:

| 方法 | 说明 |
|:------|:------|
| Where(Expression<Func<T, bool>> expression) | 设置`IQuery`的查询条件, 如`Where(x => x.Id > 1)` |
| WhereWithAnd(Expression<Func<T, bool>> expression) | 使用`And`方式连接当前查询条件和`IQuery`中查询条件 |
| WhereWithOr(Expression<Func<T, bool>> expression) | 使用`Or`方式连接当前查询条件和`IQuery`中查询条件 |
| WhereReset() | 把`IQuery`中查询条件重置 |
| OrderBy<TKey>(Expression<Func<T, TKey>> expression) | 设置`IQuery`的排序字段, 以正序排序, 如`OrderBy(x => x.Id)` |
| OrderByDescending<TKey>(Expression<Func<T, TKey>> expression) | 设置`IQuery`的排序字段, 以倒序排序, 如`OrderByDescending(x => x.Id)` |
| OrderByConcat<TKey>(Expression<Func<T, TKey>> expression) | 把`IQuery`中排序连接当前字段正序排序 |
| OrderByDescendingConcat<TKey>(Expression<Func<T, TKey>> expression) | 把`IQuery`中排序连接当前字段倒序排序 |
| OrderByReset() | 把`IQuery`中排序重置 |
| OrderByRandom() | 把`IQuery`中排序置为随机排序 |
| Take(int count) | 设定`IQuery`输出结果的数量 |
| Skip(int count) | 设定`IQuery`输出结果需要跳过的数量 |
| Range(int from, int to) | 设定`IQuery`查询结果从`参数1`位到`参数2`位 |
| PageSize(int page, int size) | 设定`IQuery`输出结果的分页结果, page:从1开始页数,size:每页数量 |
| RangeReset() | 重置`IQuery`中输出结果范围 |
| SetDistinct(bool distinct) | 设定是否使用`Distinct`方式输出结果 |
| ToList() | 以`List`的方式输出数据 |
| ToArray() | 以`Array`的方式输出数据 |
| First() | 输出首个数据结果对象, 如无数据则为null |
| ElementAt(int index) | 输出指定索引位置数据结果对象, 如无数据则为null |
| Exists() | 判断是否有查询数据 |
| Count()	| 返回查询数据的数据长度, 返回类型为int |
| LongCount() | 返回查询数据的数据长度, 返回类型为long |


### 全查询

```csharp
List<TeUser> list = context.Query<TeUser> ().ToList ();
```

### 组合查询

```csharp
List<TeUser> list = context.Query<TeUser> ().Where (x => x.Id > 1).OrderBy (x => x.Id).Take(10).ToList ();
```

## 条件查询(Where)

使用`Where(lambda)`方法加入查询条件, 参数为`Lambda`表达式

```csharp
context.Query<T> ().Where(x => x.Id > 1)
```

### 普通条件查询

```csharp
 List<TeUser> list1 = context.Query<TeUser> ().Where (x => x.Id >= 5 && x.Id <= 10).ToList ();
 List<TeUser> list2 = context.Query<TeUser> ().Where (x => x.Id < 5 || x.Id > 10).ToList ();
```

### In

使用`List<T>.Contains`方法

```csharp
int [] array = new int [] { 3, 5, 7 };
List<int> list = new List<int> (array);
//in
List<TeUser> list1 = context.Query<TeUser> ().Where (x => list.Contains (x.Id)).ToList ();
//not in
List<TeUser> list2 = context.Query<TeUser> ().Where (x => ！list.Contains (x.Id)).ToList ();
```

### Like

只支持string类型,使用`string.StartsWith`、`string.EndsWith`、`string.Contains`方法查询,可支持反向查.

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

### null查询

查询字段需为可空类型(如int?)或string类型

```csharp
// RefereeId is null
List<TeUser> list1 = context.Query<TeUser> ().Where (x => x.RefereeId == null).ToList ();
// RefereeId is not null
List<TeUser> list2 = context.Query<TeUser> ().Where (x => x.RefereeId != null).ToList ();
```

如非可空类型可用扩展查询方式`ExtendQuery.IsNull()`

```csharp
// id is null
List<TeUser> list1 = context.Query<TeUser> ().Where (x => ExtendQuery.IsNull (x.Id)).ToList ();
// id is not null
List<TeUser> list2 = context.Query<TeUser> ().Where (x => !ExtendQuery.IsNull (x.Id)).ToList ();
```

### 布尔值字段查询

查询字段需为布尔(boolean)类型

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
### 跨表Exists查询

固定条件查询

```csharp
// exist (select 1 from TeUserLevel where Status = 1)
List<TeUser> list1 = context.Query<TeUser> ().Where (x => ExtendQuery.Exists<TeUserLevel> (y => y.Status == 1)).ToList ();
// not exist (select 1 from TeUserLevel where Status = 1)
List<TeUser> list2 = context.Query<TeUser> ().Where (x => !ExtendQuery.Exists<TeUserLevel> (y => y.Status == 1)).ToList ();
```
关联条件查询

```csharp
// exist exist (select 1 from TeUserLevel where Id = TeUser.LevelId)
List<TeUser> list1 = context.Query<TeUser> ().Where (x => ExtendQuery.Exists<TeUserLevel> (y => y.Id == x.LevelId)).ToList ();
// not exist (select 1 from TeUserLevel where Id = TeUser.LevelId)
List<TeUser> list2 = context.Query<TeUser> ().Where (x => !ExtendQuery.Exists<TeUserLevel> (y => y.Id == x.LevelId)).ToList ();
```

## 排序(OrderBy)

使用`OrderBy(lambda)`方法加入排序规则, 参数为Lambda表达式.

```csharp
context.Query<T> ().OrderBy(x => x.Id)
```

### 正向排序

```csharp
List<TeUser> list = context.Query<TeUser> ().OrderBy (x => x.Id).ToList ();
```

### 反向排序

```csharp
List<TeUser> list = context.Query<TeUser> ().OrderByDescending (x => x.Id).ToList ();
```

### 连接排序

使用带`Concat`的方法连接多个排序

```csharp
List<TeUser> list = context.Query<TeUser> ().OrderBy (x => x.Gender).OrderByDescendingConcat (x => x.Id).ToList ();
```

## 选择指定字段 (Select)

使用`Select(lambda)`方法指定若干字段转换新的类型, 支持匿名类输出.

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

查询单列字段列表

```csharp
List<int> list = context.Query<TeUser> ().Select (x => x.Id).ToList ();
```

## 查询批量更新

使用`Update(lambda)`方法对查询数据进行批量更新, 参数是`lambda`表达式, 使用创建new object方式定义数据的更新字段与更新内容, 左侧为更新字段名, 右侧更新内容, 更新内容可为原字段, 返回结果为成功更新行数. 

```csharp
var result = context.Query<TeUser> ()
                    .Update (x => new TeUser {
                        LastLoginTime = DateTime.Now,
                        Status = 2
                     });
// 更新内容为原字段
var result = context.Query<TeUser2> ()
                    .Update (x => new TeUser2 {
                        LastLoginTime = x.RecordTime,
                        Status = x.Status + 1
                     });
```

## 查询批量删除

使用`Delete()`方法对查询数据进行批量删除操作, 返回结果为成功删除行数.

```csharp
var result = context.Query<TeUser> ().Where(x => x.Id > 1)
                    .Delete();
```

## 查询批量插入

将查询数据插入指定数据表, 可以使用全字段或指定字段, 会生成`insert into t1(x1,x2,x3...)select y1,y2,y3 from t2`命令直接批量插入数据.

### 全字段插入

使用`Insert<K>()`全数据插入, `K`是插入表类型, 查询表的字段必须与插入表的字段一一对应, 返回结果为成功插入行数.

如果查询表与插入表有同位字段是自增字段, 则插入表的自增字段数据继续自增.

```csharp
var result = context.Query<TeDataLog> ()
		    .Insert<TeDataLogHistory> ();
```

### 指定字段插入

使用`SelectInsert<K>(lambda)`选择指定字段举行插入, 参数是`lambda`表达式, 使用创建new object方式定义数据的插入字段与选择字段, 左侧为插入字段名, 右侧选择字段, 返回结果为成功新增行数. 


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


## 主键/自增字段查询

通过自增ID查数据

```csharp
int id = 1;
TeUser user = context.SelectById<TeUser> (id);
```

通过主键查数据, 若有多个主键, 则按顺序全部部输入.

```csharp
int key = 1;
TeUser user = context.SelectByKey<TeUser> (key);
```