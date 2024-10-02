Wildcard segments in a route pattern are denoted by an wildcard identifier inside {} brackets -

```go
mux.HandleFunc("/products/{category}/item/{itemID}", exampleHandler)
```

- The following requests would all match the route we defined above -

  ```
  /products/hammocks/item/sku123456789
  /products/seasonal-plants/item/pdt-1234-wxyz
  /products/experimental_foods/item/quantum%20bananas
  ```

- Patterns like `/products/c_{category}`, `/date/{y}-{m}-{d}` or `/{slug}.html` are not valid.

- Inside your handler, you can retrieve the corresponding value for a wildcard segment using its identifier and the `r.PathValue()` method -

  ```go
  func exampleHandler(w http.ResponseWriter, r *http.Request) {
      category := r.PathValue("category")
      itemID := r.PathValue("itemID")
      ...
  }
  ```

  The `r.PathValue()` method always returns a `string` value, this can be any value that the user includes in the URL — so you should validate or sanity check the value before doing anything important with it.

### Matches and Conflicts

When defining route patterns with wildcard segments, it’s possible that some of your patterns will ‘overlap’. When route patterns overlap, Go’s servemux needs to decide which pattern takes precedent so it can dispatch the request to the appropriate handler.

- The most specific route pattern wins. Formally, Go defines a pattern as more specific than another if it matches only a subset of requests that the other pattern matches.

- The order in which handlers are registered doesn't matter.

- There is a potential edge case where you have two overlapping route patterns but neither one is obviously more specific than the other. Go’s servemux considers the patterns to conflict, and will panic at runtime when initializing the routes.

### Subtree path patterns with wildcards

- If you have a subtree path pattern like "/user/{id}/" in your routes (note the trailing slash), this pattern will match requests like /user/1/, /user/2/a, /user/2/a/b/c and so on.

### Remainder wildcards

If a route pattern ends with a wildcard, and this final wildcard identifier ends in ..., then the wildcard will match any and all remaining segments of a request path.

For example, if you declare a route pattern like `/post/{path...}`, it will match requests like `/post/a`, `/post/a/b`, `/post/a/b/c` and so on.
