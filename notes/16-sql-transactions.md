### Transactions

It’s important to realize that calls to `Exec()`, `Query()` and `QueryRow()` can use any connection from the `sql.DB` pool. Even if you have two calls to `Exec()` immediately next to each other in your code, there is no guarantee that they will use the same database connection.

Sometimes this isn’t acceptable. For instance, if you lock a table with MySQL’s `LOCK TABLES` command you must call `UNLOCK TABLES` on exactly the same connection to avoid a deadlock.

To guarantee that the same connection is used you can wrap multiple statements in a transaction. Here’s the basic pattern:

```go
type ExampleModel struct {
    DB *sql.DB
 }

 func (m *ExampleModel) ExampleTransaction() error {
    // Calling the Begin() method on the connection pool creates a new sql.Tx
    // object, which represents the in-progress database transaction.
    tx, err := m.DB.Begin()
    if err != nil {
        return err
    }
    // Defer a call to tx.Rollback() to ensure it is always called before the
    // function returns. If the transaction succeeds it will be already be
    // committed by the time tx.Rollback() is called, making tx.Rollback() a
    // no-op. Otherwise, in the event of an error, tx.Rollback() will rollback
    // the changes before the function returns.
    defer tx.Rollback()
    // Call Exec() on the transaction, passing in your statement and any
    // parameters. It's important to notice that tx.Exec() is called on the
    // transaction object just created, NOT the connection pool. Although we're
    // using tx.Exec() here you can also use tx.Query() and tx.QueryRow() in
    // exactly the same way.
    _, err = tx.Exec("INSERT INTO ...")
    if err != nil {
        return err
    }
    // Carry out another transaction in exactly the same way.
    _, err = tx.Exec("UPDATE ...")
    if err != nil {
        return err
    }
    // If there are no errors, the statements in the transaction can be committed
    // to the database with the tx.Commit() method.
    err = tx.Commit()
    return err
 }
```

Transactions are also super-useful if you want to execute multiple SQL statements as a single atomic action.
