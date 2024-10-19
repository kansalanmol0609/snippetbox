To use MySQL from our Go web application, we need to install a database driver. This essentially acts as a middleman, translating commands between Go and the MySQL database itself.

We will use `go-sql-driver/mysql` driver. To download it, go to your project directory and run the `go get` command like so:

```bash
go get github.com/go-sql-driver/mysql@v1
```

The `go get` command will recursively download any dependencies that the package has. In this case, the `github.com/go-sql-driver/mysql` package itself uses the `filippo.io/edwards25519` package, so this will be downloaded too.

We're postfixing the package path with `@v1` to indicate that we want to download the latest available version with the major release number 1.

If you want to download the latest version -

```
go get github.com/go-sql-driver/mysql
```

If you want to download a particular version -

```
go get github.com/go-sql-driver/mysql@v1.0.3
```

After installing it, if you look at the `go.mod` file, you will see these 2 additional lines -

```go
 require (
    filippo.io/edwards25519 v1.1.0 // indirect
    github.com/go-sql-driver/mysql v1.8.1 // indirect
)
```

These lines in `go.mod` essentially tell the Go command exactly which version of a package should be used when you run a command like `go run`, `go test` or `go build` from your directory.

The `//indirect` annotation indicates that a package doesn't directly appear in any `import` statement in your codebase. This is because we haven't used these yet.

### go.sum file

You'll also see that a new file has been created in the root of your project directory called `go.sum`. This files stores the cryptographic checksums representing the content of the required packages. This file isn't designed to be human editable.

It serves 2 useful functions

- If you run the `go verify` command, this will verify that the checksums of the downloaded packages on your machine matches the entries in `go.sum` file, so that you can be confident that they haven't been altered.

- If someone else needs to download all the dependencies of the project - which they can do by running `go mod download` - they will get an error if there is any mismatch between the packages they are downloading and the checksums in the file.

### Upgrading packages

To upgrade to the latest available minor or patch release of a package, you can simply run `go get` with `-u` flag -

```bash
go get -u github.com/foo/bar
```

Or alternatively, if you want to upgrade to a specific version, then you should run the same command but with the appropriate `@version` suffix -

```bash
go get -u github.com/foo/bar@v2.0.0
```

### Removing unused packages

You could either run `go get` and postfix the package path with `@none` like -

```bash
go get github.com/foo/bar@none
```

or, if you've removed all the references to the package in your code, you can run `go mod tidy`, which will automatically remove any unused packages from your `go.mod` and `go.sum` files -

```bash
go mod tidy
```
