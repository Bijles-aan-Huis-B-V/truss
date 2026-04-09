# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**This is a deprecated fork.** The original Truss repo is no longer maintained. This fork will be archived once booking-service, user-service, and translation-service are migrated to Echo.

Truss is a Go CLI tool that generates gRPC microservices with go-kit from Protocol Buffer definitions. Users define services in `.proto` files (with HTTP annotations) and Truss generates a complete service scaffold including gRPC/HTTP transports, endpoints, clients, and handler stubs. A key feature is **smart handler preservation**: user-written business logic in `handlers/` survives regeneration via Go AST parsing.

## Build Commands

```bash
make                    # Build truss (runs gobindata + go install). Default target is `truss`.
make dependencies       # Install protoc plugins (protoc-gen-gogo, go-bindata)
make gobindata          # Recompile template files into Go binary data (required after template changes)
make truss              # Build CLI binary with version/date ldflags
```

`GO111MODULE=on` is required for tests; the Makefile sets this automatically.

## Test Commands

```bash
make test               # All tests (unit + integration)
make test-go            # Unit tests only (go test ./...)
make test-integration   # Integration tests (generates services, builds, verifies)
make testclean          # Remove generated integration test artifacts
```

Run a single test:
```bash
go test ./gengokit/handlers/ -run TestName
```

## Architecture

**Data flow**: `.proto` files → protoc → `.pb.go` → `svcdef.Svcdef` (parsed via Go AST) → `gengokit` templates → generated service

**Core packages:**

- **`cmd/truss/`** — CLI entry point. Orchestrates: proto validation → protoc execution → service definition parsing → code generation → file output.
- **`svcdef/`** — Parses `.pb.go` files (Go AST) and `.proto` files into structured service definitions (`Svcdef`, `Service`, `ServiceMethod`, `HTTPBinding`).
- **`deftree/`** — Parses protobuf AST for comment extraction and documentation generation.
- **`gengokit/`** — Code generation engine. `generator/gen.go` orchestrates template rendering. `ApplyTemplate()` renders and gofmts output.
- **`gengokit/handlers/`** — AST-based handler preservation. Parses existing handler files, adds/removes functions to match current service definition, preserves user implementations.
- **`gengokit/httptransport/`** — Generates HTTP transport layer from proto HTTP annotations (path parameters, query bindings, verbs).
- **`gengokit/template/`** — `.gotemplate` files embedded via go-bindata into `template.go`. Templates live under `template/NAME-service/`.
- **`truss/execprotoc/`** — Wrapper for protoc compilation.

**Generated service structure** (output):
```
NAME-service/
├── cmd/NAME/main.go           # Entry point (regenerated)
├── handlers/
│   ├── handlers.go            # User-modifiable handler stubs (preserved)
│   ├── hooks.go               # Lifecycle hooks (preserved)
│   └── middlewares.go         # Middleware definitions (preserved)
├── svc/
│   ├── endpoints.go           # go-kit endpoints (regenerated)
│   ├── server/run.go          # Server wiring (regenerated)
│   ├── transport_grpc.go      # gRPC transport (regenerated)
│   ├── transport_http.go      # HTTP transport (regenerated)
│   └── client/{grpc,http}/    # Client libraries (regenerated)
└── NAME.pb.go                 # Protobuf generated (regenerated)
```

## Key Patterns

- **Template changes** require running `make gobindata` to re-embed them before building.
- **Handler preservation** uses `go/ast` and `go/parser` to merge generated stubs with existing user code — edits to this logic require careful testing.
- **File strategy**: files in `handlers/` are user-owned (preserved across regeneration); everything else is regenerated.
- Templates use Go `text/template` with a custom `FuncMap` (see `gengokit/gengokit.go`).
- Error handling uses `github.com/pkg/errors` for wrapping with context.
- Go module is `github.com/metaverse/truss`; dependencies are vendored.

## CI

GitHub Actions matrix: Go 1.14/1.15 × protoc 3.6.1/3.13.0. Runs on push/PR to master.
