---
title: "Go Project Layout: What Actually Works"
description: "Practical patterns for structuring Go projects based on Helm, Hugo, and Prometheus, not the unofficial 'standard' layout."
date: 2026-01-14
lastmod: 2026-01-14
draft: true
image: cover.jpg
categories:
  - Best Practices
tags:
  - project-structure
  - modules
  - tooling
---

The Go community has no official project layout. This is intentional. Yet developers spend hours debating `pkg/` vs root packages, when the answer depends on what you're building.

Here's a practical take on structuring Go projects, based on patterns from Helm, Hugo, Prometheus, and smaller projects.

## The only rule that matters

Go enforces exactly one structural rule: `internal/` packages cannot be imported from outside the module. Everything else is convention.

The official documentation at [go.dev/doc/modules/layout](https://go.dev/doc/modules/layout) says: start simple. A `main.go` and `go.mod` is enough for small projects. Add structure when you need it, not before.

## Three patterns that work

### Library at root

For projects meant to be imported by others.

```
mylib/
├── go.mod
├── mylib.go            # Primary API
├── option.go           # Functional options
├── types.go            # Public types
└── internal/           # Private helpers
    └── util/
```

Users import directly:

```go
import "github.com/user/mylib"

client := mylib.New(mylib.WithTimeout(5 * time.Second))
```

Examples: [Cobra](https://github.com/spf13/cobra), [Viper](https://github.com/spf13/viper), [goldmark](https://github.com/yuin/goldmark).

### Library + CLI

For projects offering both a library and a command-line tool.

```
myproject/
├── go.mod
├── service.go          # Public API
├── types.go            # Public types
├── internal/
│   ├── config/         # Config parsing
│   └── util/           # Private helpers
└── cmd/
    └── mytool/         # CLI binary
        └── main.go
```

The CLI is a thin wrapper around the library. Users can:

```bash
# Import the library
go get github.com/user/myproject

# Install the CLI
go install github.com/user/myproject/cmd/mytool@latest
```

Examples: [Helm](https://github.com/helm/helm), [Hugo](https://github.com/gohugoio/hugo), [Prometheus](https://github.com/prometheus/prometheus).

### CLI only

For tools not meant to be imported as libraries.

```
mytool/
├── go.mod
├── main.go
├── internal/
│   ├── cmd/            # Command implementations
│   ├── config/         # Configuration
│   └── core/           # Business logic
└── testdata/
```

Everything in `internal/` signals: this is an application, not a library. [Terraform](https://github.com/hashicorp/terraform) uses this approach.

## Why not pkg/?

The `pkg/` directory adds a path segment for no benefit:

```go
// With pkg/
import "github.com/user/project/pkg/thing"

// Without pkg/
import "github.com/user/project/thing"
```

The second is shorter and clearer. The Go team explicitly does not recommend `pkg/`. [Russ Cox](https://github.com/golang-standards/project-layout/issues/117), Go's tech lead, called out the [golang-standards/project-layout](https://github.com/golang-standards/project-layout) repo (54k+ stars) for promoting patterns the Go team never endorsed.

Modern projects ([Hugo](https://github.com/gohugoio/hugo), [Prometheus](https://github.com/prometheus/prometheus), [Cobra](https://github.com/spf13/cobra)) skip `pkg/` entirely.

## When to use internal/

Use `internal/` for code that:

- Supports your public API but shouldn't be exposed
- May change without notice
- Has no stability guarantees

Common candidates:

- Configuration parsing
- Utility functions (file handling, string manipulation)
- Protocol implementations
- Test helpers

Do not use `internal/` for:

- Small projects where unexported functions suffice
- Code you might want to expose later (moving out of `internal/` is a breaking change for your imports)

## When to create a new package

Create a package when:

- Multiple files share a distinct responsibility
- The code has its own types and functions that form a cohesive unit
- You want to control visibility at package boundaries

Do not create a package for:

- A single file with a few helper functions
- "Organization" without functional benefit
- Matching some external project's structure

A 200-line file at root is fine. A `utils/` package with three functions is over-engineering.

## The decision process

When adding code:

| Question                           | Yes                                                 | No                       |
| ---------------------------------- | --------------------------------------------------- | ------------------------ |
| Should external users import this? | Root package                                        | `internal/`              |
| Is this CLI-specific?              | `cmd/appname/`                                      | Library code             |
| Does this need its own package?    | Only if multiple files with distinct responsibility | Keep in existing package |

## File organization at root

For library projects, split by responsibility:

```
mylib/
├── client.go           # Client type and methods
├── option.go           # Functional options
├── types.go            # Public types
├── errors.go           # Sentinel errors
├── request.go          # Request building
├── response.go         # Response parsing
└── mylib_test.go       # Tests
```

Each file has one job. Looking for error definitions? Check `errors.go`. Looking for configuration? Check `option.go`.

Avoid:

- `util.go` (name it by what it does)
- `helpers.go` (same problem)
- `misc.go` (where code goes to die)

## Mono-repo considerations

For projects with multiple binaries:

```
myproject/
├── go.mod              # Single module
├── lib/                # Shared library code
├── cmd/
│   ├── server/
│   ├── worker/
│   └── cli/
└── internal/
    └── shared/         # Internal shared code
```

Avoid multi-module repos (multiple `go.mod` files) unless you have a strong reason. They complicate:

- Local development (need `replace` directives)
- Versioning (tags must include module path)
- Testing (`go test ./...` doesn't work across modules)

The Go team recommends single-module repos for most projects.

## Real examples

| Project    | Structure                 | Library importable?                            |
| ---------- | ------------------------- | ---------------------------------------------- |
| Cobra      | Root package              | Yes, `github.com/spf13/cobra`                  |
| Helm       | `pkg/` + `cmd/`           | Yes, `helm.sh/helm/v3/pkg/action`              |
| Hugo       | Root + `hugolib/`         | Yes, `github.com/gohugoio/hugo/hugolib`        |
| Prometheus | Root packages             | Yes, `github.com/prometheus/prometheus/promql` |
| Terraform  | Everything in `internal/` | No, CLI only                                   |

## Common mistakes

- **Over-structuring early**: Starting with `pkg/`, `internal/`, `cmd/`, `api/`, `web/` for a 500-line project. Start flat, add structure when pain appears.
- **Copying [golang-standards/project-layout](https://github.com/golang-standards/project-layout)**: That repo is not endorsed by the Go team. It promotes patterns most Go projects don't use.
- **Empty packages for "future use"**: If a package has one file with two functions, it's not a package. It's a file.
- **internal/ for everything**: If nothing is importable, your project is an application, not a library. That's fine, but be intentional about it.

## Summary

- **Start simple**: `main.go` + `go.mod` is valid
- **Library code at root**: clean import paths
- **CLI in cmd/**: thin wrapper around library
- **Private code in internal/**: implementation details
- **Skip pkg/**: it adds nothing
- **One module**: avoid multi-module complexity

The best layout is the one you don't think about. If you're spending time on structure instead of code, you're over-engineering.
