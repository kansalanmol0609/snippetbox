Go’s `net/http` package ships with a built-in `http.FileServer` handler which you can use to serve files over HTTP from a specific directory -

```go
fileServer := http.FileServer(http.Dir("./ui/static/"))

mux.Handle("GET /static/", http.StripPrefix("/static", fileServer))
```

When this handler receives a request for a file, it will remove the leading slash from the request URL path and then search the `./ui/static` directory for the corresponding file to send to the user.

So, for this to work correctly, we must strip the leading `/static` from the URL path before passing it to `http.FileServer`. Otherwise it will be looking for a file which doesn’t exist and the user will receive a `404` page not found response.

### Serving a folder - http.FileServer

- It sanitizes all request paths by running them through the `path.Clean()` function before searching for a file. This removes any `.` and `..` elements from the URL path, which helps to stop directory traversal attacks.

- Range requests are fully supported. This is great if your application is serving large files and you want to support resumable downloads.

- The `Content-Type` is automatically set from the file extension using the `mime.TypeByExtension()` function.

- It will probably serve most frequently used files from RAM instead of hard drive because Windows or Mac stores them in RAM. So, it would be fast as well.

### Serving a single file - http.ServeFile

```go
func downloadHandler(w http.ResponseWriter, r *http.Request) {
    http.ServeFile(w, r, "./ui/static/file.zip")
}
```

It does not automatically sanitize the file path. If you’re constructing a file path from untrusted user input, to avoid directory traversal attacks you must sanitize the input with `filepath.Clean()` before using it

### Disabling directory listing

- The simplest way? Add a blank `index.html` file to the specific directory that you want to disable listings for. This will then be served instead of the directory listing, and the user will get a `200` OK response with no body.

- A more complicated (but arguably better) solution is to create a custom implementation of `http.FileSystem`, and have it return an `os.ErrNotExist` error for any directories.
