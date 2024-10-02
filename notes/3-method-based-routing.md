To restrict a route to a specific HTTP method, you can prefix the route pattern with the necessary HTTP method when declaring it, like so:

```go
mux.HandleFunc("GET /{$}", home)
```

- The HTTP methods in route patterns are case sensitive and should always be written in uppercase, followed by at least one whitespace character (both spaces and tabs are fine).

- You can only include one HTTP method in each route pattern.

- It’s totally OK to declare two (or more) separate routes that have different HTTP methods but otherwise have the same pattern, like we are doing here with `GET /snippet/create` and `POST /snippet/create`

- It’s important to be aware that a route pattern which doesn’t include a method — like `/article/{id}` will match incoming HTTP requests with any method.

There are some things which are not provided by Go for which you can use a third-party router package. The ones that I recommend are httprouter, chi, flow and gorilla/mux.
