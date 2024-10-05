How can we make any dependency available to our handlers?

- By putting the dependencies in global variables.
- By injecting dependencies into your handlers. This makes code easier to debug, less error prone and easier to test.

We will create `application` struct:

```go
type application struct {
	logger *slog.Logger
}
```

Then, we will bind all the handler functions with application so that they can access logger:

```go
func (app *application) home(w http.ResponseWriter, r *http.Request) {
   w.Header().Add("Server", "Go")

	files := []string{
		"./ui/html/base.tmpl",
		"./ui/html/partials/nav.tmpl",
		"./ui/html/pages/home.tmpl",
	}

	ts, err := template.ParseFiles(files...)

    if err != nil {
		app.logger.Error(err.Error(), "method", r.Method, "uri", r.URL.RequestURI())
		http.Error(w, "Internal Server Error", http.StatusInternalServerError)
		return
	}

}
```

Then in the `main` function, we can do following:

```go
app := &application{
    logger: logger,
}

...

mux.HandleFunc("GET /{$}", app.home)
mux.HandleFunc("GET /snippet/view/{id}", app.snippetView)
mux.HandleFunc("GET /snippet/create", app.snippetCreate)
mux.HandleFunc("POST /snippet/create", app.snippetCreatePost)
```

### Stack traces

You can use the `debug.Stack()` function to get a stack trace outlining the execution path of the application for the current goroutine.
