By default, every response that your handlers send has the `HTTP` status code `200 OK`, plus three automatic system-generated headers: a `Date` header, and the `Content-Length` and `Content-Type` of the response body -

```bash
$ curl -i localhost:4000/

HTTP/1.1 200 OK
Date: Wed, 18 Mar 2024 11:29:23 GMT
Content-Length: 21
Content-Type: text/plain; charset=utf-8
Hello from Snippetbox
```

### Customizing status code

You can set the status code using `w.WriteHeader(201)` -

- It’s only possible to call `w.WriteHeader()` once per response, and after the status code has been written it can’t be changed.

- If you don’t call `w.WriteHeader()` explicitly, then the first call to `w.Write()` will automatically send a 200 status code to the user.

- The `net/http` package provides constants for HTTP status codes, which we can use instead of writing the status code number ourselves
  ```go
  w.WriteHeader(http.StatusCreated)
  ```

### Customizing headers

You can also customize the HTTP headers sent to a user by changing the response header map. Probably the most common thing you’ll want to do is include an additional header in the map, which you can do using the `w.Header().Add()` method -

```go
// Set a new cache-control header. If an existing "Cache-Control" header exists
// it will be overwritten.
w.Header().Set("Cache-Control", "public, max-age=31536000")

// In contrast, the Add() method appends a new "Cache-Control" header and can
// be called multiple times.
w.Header().Add("Cache-Control", "public")
w.Header().Add("Cache-Control", "max-age=31536000")

// Delete all values for the "Cache-Control" header.
w.Header().Del("Cache-Control")

// Retrieve the first value for the "Cache-Control" header.
w.Header().Get("Cache-Control")

// Retrieve a slice of all values for the "Cache-Control" header.
w.Header().Values("Cache-Control")
```

> You must make sure that your response header map contains all the headers you want before you call `w.WriteHeader()` or `w.Write()`. Any changes you make to the response header map after calling `w.WriteHeader()` or `w.Write()` will have no effect on the headers that the user receives.

### Writing response bodies

Because the `http.ResponseWriter` value in your handlers has a `Write()` method, it satisfies the `io.Writer` interface.

That means you can use standard library functions like `io.WriteString()` and the `fmt.Fprint*()` family (all of which accept an `io.Writer` parameter) to write plain-text response bodies too -

```go
// Instead of this...
w.Write([]byte("Hello world"))

// You can do this...
io.WriteString(w, "Hello world")
fmt.Fprint(w, "Hello world")

```

### Content sniffing

In order to automatically set the Content-Type header, Go content sniffs the response body with the `http.DetectContentType()` function.

The `http.DetectContentType()` function generally works quite well, but a common gotcha for web developers is that it can’t distinguish JSON from plain text. So, by default, JSON responses will be sent with a `Content-Type: text/plain; charset=utf-8` header.

```go
w.Header().Set("Content-Type", "application/json")
w.Write([]byte(`{"name":"Alex"}`))
```
