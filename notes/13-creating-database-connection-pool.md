### sql.Open() fn

We now need to connect to the database from our web application -

```go
// The sql.Open() function initializes a new sql.DB object, which is essentially a pool of database connections.
db, err := sql.Open("mysql", "web:pass@/snippetbox?parseTime=true")

if err != nil{
    ...
}
```

- The first parameter is the driver name and the second parameter is DSN string (data source name or connection string)

- The `parseTime=true` part of the DSN above is a driver-specific parameter which instructs our driver to convert SQL `TIME` and `DATE` fields to Go `time.Time` objects.

- The `sql.Open()` function returns a sql.DB object. This isn't a db connection - it's a pool of many connections. Go manages the connections in this pool as needed, automatically opening and closing connections to the database via the driver.

- The connection pool is safe for concurrent access, so you can use it from web application handlers safely.

- The connection pool is intended to be long-lived. In a web application, it's normal to initialize the connection pool in your `main()` function and then pass the pool to your handlers. You shouldn't call `sql.Open()` in a short-lived HTTP handler itself - it would be a waste of money and network resources.

- `sql.Open()` initialize the pool for future use. Actual connections to the database are established lazily, as and when needed for the first time. So, to verify that everything is set up correctly, we need to use the db.Ping() method to create a connection and check for any errors. If there is an error, we call `db.Close()` to close the connection pool and return the error.
