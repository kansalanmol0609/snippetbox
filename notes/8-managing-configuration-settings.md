In Go, a common way to manage configuration settings is to use command line flags when starting an application -

```go
$ go run ./cmd/web -addr=":80"
```

The easiest way to accept and parse a command-line flag in your application is with a line of code like this -

```go
addr := flag.String("addr", ":4000", "HTTP network address")
```

This essentially defines a new command-line flag with the name addr, a default value of ":4000" and some short help text explaining what the flag controls.

Ports `0-1023` are restricted and (typically) can only be used by services which have root privileges. If you try to use one of these ports you should get a `bind: permission` denied error message on start-up.

### Type conversions

`flag.String()` has the benefit of converting whatever value the user provides at runtime to a `string` type. If the value can’t be converted to a `string` then the application will print an `error` message and exit.

Go also has a range of other functions including `flag.Int()`, `flag.Bool()`, `flag.Float64()` and `flag.Duration()` for defining flags.

### Automated help

Another great feature is that you can use the `-help` flag to list all the available command line flags for an application and their accompanying help text -

```go
$ go run ./cmd/web -help
 Usage of /tmp/go-build3672328037/b001/exe/web:
    -addr string
        HTTP network address (default ":4000")
```

### Environment variables

If you want, you can store your configuration settings in environment variables and access them directly from your application by using the `os.Getenv()` function like so -

```go
 addr := os.Getenv("SNIPPETBOX_ADDR")
```

But it has following drawbacks:

- You can't specify a default value.
- You don't get the `-help` functionality.
- It always return string value and you don't get int, bool types etc.

### Pre-existing variables

It’s possible to parse command-line flag values into the memory addresses of pre-existing variables:

```go
type config struct {
    addr        string
    staticDir   string
}

var cfg config

flag.StringVar(&cfg.addr, "addr", ":4000", "HTML network address")
flag.StringVar(&cfg.staticDir, "static-dir", "./ui/static", "Path to static assets")

flag.Parse()
```
