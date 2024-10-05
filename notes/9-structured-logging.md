You may want to make the log entries easier to filter and work with. For example, you might want to distinguish between different severities of log entries (like informational and error entries), or to enforce a consistent structure for log entries so that they are easy for external programs or services to parse.

Go standard library includes the `log/slog` package for doing this. Each log entry includes following:

- A timestamp with millisecond precision.
- The severity level of the log entry (Debug, Info, Warn or Error).
- The log message (an arbitrary string value).
- Optionally, any number of key-value pairs (known as attributes).

### Creating a structured logger

```go
loggerHandler := slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{...})
logger := slog.New(loggerHandler)
```

The `slog.NewTextHandler()` function accepts two arguments -

- The first argument is the write destination for the log entries.
- The second argument is a pointer to a `slog.HandlerOptions` struct , which you can use to customize the behavior of the handler.

### Using a structured logger

You can then write a log entry at a specific severity level by calling the `Debug()`, `Info()`, `Warn()` or `Error()` methods on the logger -

```go
logger.Info("request received")
logger.Info("request received", "method", "GET", "path", "/")
```

It's output would be:

```
time=2024-03-18T11:29:23.000+00:00 level=INFO msg="request received
time=2024-03-18T11:29:23.000+00:00 level=INFO msg="request received" method=GET path=/
```

There is no structured logging equivalent to the `log.Fatal()` function that we can use to handle an error returned by `http.ListenAndServe()`. Instead, the closest we can get is logging a message at the Error severity level and then manually calling `os.Exit(1)` to terminate the application with the `exit code 1`, like we are in the code below:

```go
err := http.ListenAndServe(*addr, mux)

logger.Error(err.Error())
os.Exit(1)
```

### Safer attributes

To avoid missing adding a key or value and catch any problems at compile-time, you can use the `slog.Any()` function to create an attribute pair instead -

```go
logger.Info("starting server", slog.Any("addr", ":4000"))
```

You can go even further and introduce some additional type safety by using the `slog.String()`, `slog.Int()`, `slog.Bool()`, `slog.Time()` and `slog.Duration()` functions.

### JSON formatted logs

To create a handler that writes log entries as JSON objects, you can use the `slog.NewJSONHandler()` function -

```go
logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
```

When using the JSON handler, the log output will look similar to this:

```
{"time":"2024-03-18T11:29:23.00000000+00:00","level":"INFO","msg":"starting server","addr":":4000"}
{"time":"2024-03-18T11:29:23.00000000+00:00","level":"ERROR","msg":"listen tcp :4000: bind: address already in use"}
```

### Minimum log level

By default, the minimum log level for a structured logger is `Info`. That means that any log entries with a severity less than `Info` — i.e. `Debug` level entries — will be silently discarded.

You can use the `slog.HandlerOptions` struct to override this and set the minimum level to `Debug` (or any other level) if you want:

```go
logger := slog.New(slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{
    Level: slog.LevelDebug,
}))
```

### Caller location

You can also customize the handler so that it includes the filename and line number of the calling source code in the log entries, like so:

```go
logger := slog.New(slog.NewTextHandler(os.Stdout, &slog.HandlerOptions{
    AddSource: true,
}))
```

Output would then look like:

```
time=2024-03-18T11:29:23.000+00:00 level=INFO source=/home/alex/code/snippetbox/cmd/web/main.go:32 msg="starting server" addr=
```

### Decoupled logging

In staging or production environments, you can redirect the stream to a final destination for viewing and archival. This destination could be on-disk files, or a logging service such as Splunk. Either way, the final destination of the logs can be managed by your execution environment independently of the application.

For example, we could redirect the standard out stream to a on-disk file when starting the application like so:

```
 $ go run ./cmd/web >>/tmp/web.log
```

Using the double arrow `>>` will append to an existing file, instead of truncating it when starting the application.

### Concurrent logging

Custom loggers created by `slog.New()` are concurrency-safe. You can share a single logger and use it across multiple goroutines and in your HTTP handlers without needing to worry about race conditions.
