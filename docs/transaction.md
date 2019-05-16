# Transaction

## Transaction State

`DataContext` have two state: `Transactional State` and `Non-transactional State`

`Transactional State` is non-thread-safe, After the `DataContext` change to `Transactional State`, it performs a series of operations that need to be explicitly committed before the operation can be completed.

`Non-transactional State`is thread-safe, `DataContext` will commit automatically after each operation.

## Transaction Method

| Method | Introduce |
|:------|:------|
| BeginTrans | Start a transaction, which needs to be executed before the start of each transaction, The  `DataContext` change to `Transactional State` after execution |
| CommitTrans | Commit the transaction, which is completed after the transaction is committed, and cannot be used again after the transaction is committed |
| RollbackTrans | Roll back the transaction, and are automatically rolled back when an exception is thrown during execution,  and cannot be used again after the transaction is rolled back |
| ReleaseTrans | After executing the transaction, the transaction is explicitly released to a non-transactional `Non-transactional State`, `autoRelease` parameter is false |

## Transaction Introduce

### Transaction Execution

If you want to change to `Transactional State`, through `BeginTrans` method in `DataContext` to generate transaction Scope class `TransactionScope`, `DataContext` from `Non-transactional State` into `Transactional State`, finally through `CommitTrans` method commit the transaction, and the transaction can only be commit once.

### Release Transaction

Using parameter `autoRelease` set up automatic or manual release

When `autoRelease` is true, the transaction in the execution `CommitTrans` method or `RollbackTrans` method,  the transaction is automatically released, `DataContext` from `Transactional State` into `Non-transactional State`; 

When `autoRelease` is false, you need to explicitly use `ReleaseTrans` method release transaction,  `DataContext` from `Transactional State` into `Non-transactional State`. When executing `ReleaseTrans` method, but transaction has not been commit, the transaction is rolled back automatically.

### Transaction Safe-Level

Using parameter `safeLevel` set Transaction Safe-Level

* Default/None: Database default transaction level
* Low: ReadUncommitted transaction level
* Normal: ReadCommitted transaction level
* High: RepeatableRead transaction level
* Serializable: Serializable transaction level

## Method of Use

### Scope Mode

Standard mode, If leave `using  block has not been commit and the transaction rolled back automatically.

```csharp
// commit
using (TransactionScope trans = context.BeginTrans ()) {
       TeUser user1 = context.SelectById<TeUser> (3);
       user1.Account = "foo";
       context.Update(user1);
       TeUser user2 = context.SelectById<TeUser> (4);
       context.Delete(user2);
       trans.CommitTrans ();
}
// rollback
using (TransactionScope trans = context.BeginTrans ()) {
       TeUser user1 = new <TeUser> ();
       user1.Account = "foo";
       context.Insert (user1);
       if(user1.Id > 5){
           trans.RollbackTrans ();
       }
       else{
           trans.CommitTrans ();
       }
}
```

### Manual Mode

It is mainly used to split the transaction operation into different modules and finally ensure the transaction can be released.

```csharp
var autoRelease = false;
context.BeginTrans (autoRelease);
user1.Account = "foo";
context.Insert (user1);
trans.CommitTrans ();
trans.ReleaseTrans ();
```

## Exception

During transaction execution, the operation throws an exception

When `autoRelease` is true, When a transaction has an exception, the transaction is automatically rolled back, Automatic transaction release and `DataContext` from `Transactional State` into `Non-transactional State`.

When `autoRelease` is false, When a transaction has an exception, the transaction is automatically rolled back, Need to explicitly use ` ReleaseTrans ` method release transaction, `DataContext` from `Transactional State` into `Non-transactional State`.