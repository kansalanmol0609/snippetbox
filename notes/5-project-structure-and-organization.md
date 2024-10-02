There is no right or wrong way for doing this, but we will follow a popular approach.

The structure of your project repository should now look like this:

- cmd
  - web
    - handlers.go
    - main.go
- internal
- ui
  - html
    - pages
    - partials
    - base.tmpl
  - static
    - css
    - img
    - js
- go.mod

#### cmd

The cmd directory will contain the application-specific code for the executable applications in the project.

#### internal

The internal directory will contain the ancillary non-application-specific code used in the project.

The directory name `internal` carries a special meaning and behavior in Go: any packages which live under this directory can only be imported by code inside the parent of the internal directory.

This is useful because it prevents other codebases from importing and relying on the (potentially unversioned and unsupported) packages in our internal directory — even if the project code is publicly available somewhere like GitHub.

#### ui

The ui directory will contain the user-interface assets used by the web application. Specifically, the ui/html directory will contain HTML templates, and the ui/static directory will contain static files (like CSS and images).
