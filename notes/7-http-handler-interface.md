A handler is an object which satisfiesthe `http.Handler` interface -

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

In simple terms, this basically means that to be a handler an object must have a `ServeHTTP()` method with the exact signature -

```go
ServeHTTP(http.ResponseWriter, *http.Request)
```

So, a simplest handler could look like -

```go
type home struct {}

func (h *home) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("This is my home page"))
}
```

You could then register this with a servemux using the `Handle` method like so -

```go
mux := http.NewServeMux()
mux.Handle("/", &home{})
```

When this servemux receives a HTTP request for `"/"`, it will then call the `ServeHTTP()` method of the `home` struct — which in turn writes the HTTP response.

But in practice, we generally create handler functions -

```go
func home(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("This is my home page"))
}
```

But this home function is just a normal function; it doesn’t have a `ServeHTTP()` method. So in itself it isn’t a handler. Instead we can transform it into a handler using the `http.HandlerFunc()` adapter, like so -

```go
mux := http.NewServeMux()
mux.Handle("/", http.HandlerFunc(home))
```

The `http.HandlerFunc()` adapter works by automatically adding a `ServeHTTP()` method to the home function. When executed, this `ServeHTTP()` method then simply calls the code inside of the original home function.

Throughout this project so far we’ve been using the `HandleFunc()` method to register our handler functions with the servemux. This is just some syntactic sugar that transforms a function to a handler and registers it in one step, instead of having to do it manually -

```go
mux := http.NewServeMux()
mux.HandleFunc("/", home)
```

### Chaining handlers

The `http.ListenAndServe()` function takes a `http.Handler` object as the second parameter -

```go
func ListenAndServe(addr string, handler Handler) error
```

...but we’ve been passing in a servemux.

We were able to do this because the servemux also has a `ServeHTTP()` method, meaning that it too satisfies the `http.Handler` interface.

For me, it simplifies things to think of the servemux as just being a special kind of handler, which instead of providing a response itself passes the request on to a second handler. This isn’t as much of a leap as it might first sound. Chaining handlers together is a very common idiom in Go, and something that we’ll do a lot of later in this project.

### Requests are handled concurrently

All incoming HTTP requests are served in their own _goroutine_. For busy servers, this means it’s very likely that the code in or called by your handlers will be running concurrently. While this helps make Go blazingly fast, the downside is that you need to be aware of (and protect against) race conditions when accessing shared resources from your handlers.
