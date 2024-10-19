We will encapsulate the code for working with MySQL in a separate package to the rest of our application.

Create `snippets.go` file -

```go
package models

import (
	"database/sql"
	"time"
)

// Define a snippet type to hold the data for an individual snippet. Notice how
// the fields of a the struct correspond to the fields in our MySQL snippets table?
type Snippet struct {
	ID      int
	Title   string
	Content string
	Created time.Time
	Expires time.Time
}

// Define a SnippetModel type which wraps a sql.DB connection pool.
type SnippetModel struct {
	DB *sql.DB
}

// This will insert a new snippet into the database.
func (m *SnippetModel) Insert(title string, content string, expires int) (int, error) {
	return 0, nil
}

// This will return a specific snippet based on its id.
func (m *SnippetModel) Get(id int) (Snippet, error) {
	return Snippet{}, nil
}

// This will return the 10 most recently created snippets.
func (m *SnippetModel) Latest() ([]Snippet, error) {
	return nil, nil
}
```

- There’s a clean separation of concerns. Our database logic won’t be tied to our handlers, which means that handler responsibilities are limited to HTTP stuff.
