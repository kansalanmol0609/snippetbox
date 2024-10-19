As I mentioned earlier, the `Exec()`, `Query()` and `QueryRow()` methods all use prepared statements behind the scenes to help prevent SQL injection attacks. They set up a prepared statement on the database connection, run it with the parameters provided, and then close the prepared statement.

This might feel rather inefficient because we are creating and recreating the same prepared statements every single time.

In theory, a better approach could be to make use of the DB.Prepare() method to create our own prepared statement once, and reuse that instead. This is particularly true for complex SQL statements (e.g. those which have multiple JOINS) and are repeated very often (e.g. a bulk insert of tens of thousands of records). In these instances, the cost of repreparing statements may have a noticeable effect on run time.

```go
// We need somewhere to store the prepared statement for the lifetime of our
 // web application. A neat way is to embed it in the model alongside the
// connection pool.
 type ExampleModel struct {
    DB         *sql.DB
    InsertStmt *sql.Stmt
 }
 // Create a constructor for the model, in which we set up the prepared
 // statement.
 func NewExampleModel(db *sql.DB) (*ExampleModel, error) {
    // Use the Prepare method to create a new prepared statement for the
    // current connection pool. This returns a sql.Stmt object which represents
    // the prepared statement.
    insertStmt, err := db.Prepare("INSERT INTO ...")
    if err != nil {
        return nil, err
    }
    // Store it in our ExampleModel struct, alongside the connection pool.
    return &ExampleModel{DB: db, InsertStmt: insertStmt}, nil
 }
 // Any methods implemented against the ExampleModel struct will have access to
 // the prepared statement.
 func (m *ExampleModel) Insert(args...) error {
    // We then need to call Exec directly against the prepared statement, rather
    // than against the connection pool. Prepared statements also support the
    // Query and QueryRow methods.
    _, err := m.InsertStmt.Exec(args...)
    return err
 }
 // In the web application's main function we will need to initialize a new
 // ExampleModel struct using the constructor function.
 func main() {
    db, err := sql.Open(...)
    if err != nil {
        logger.Error(err.Error())
        os.Exit(1)
    }
    defer db.Close()
    // Use the constructor function to create a new ExampleModel struct.
    exampleModel, err := NewExampleModel(db)
    if err != nil {
        logger.Error(err.Error())
        os.Exit(1)
    }
    // Defer a call to Close() on the prepared statement to ensure that it is
    // properly closed before our main function terminates.
    defer exampleModel.InsertStmt.Close()
 }
```

Prepared statements exist on database connections. So, because Go uses a pool of many database connections, what actually happens is that the first time a prepared statement (i.e. the `sql.Stmt` object) is used it gets created on a particular database connection.

`sql.Stmt` object then remembers which connection in the pool was used. The next time, the `sql.Stmt` object will attempt to use the same database connection again. If that connection is closed or in use (i.e. not idle) the statement will be re-prepared on another connection.

Under heavy load, it’s possible that a large number of prepared statements will be created on multiple connections. This can lead to statements being prepared and re-prepared more often than you would expect — or even running into server-side limits on the number of statements (in MySQL the default maximum is 16,382 prepared statements).
