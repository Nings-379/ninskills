---
name: cmd-refactor-regression-test
description: Use this skill when validating cmd subpackage migrations, shared cmd/utils extraction, or Cobra command wiring changes in this repository.
user-invocable: true
metadata:
  version: 1.1.0
  author: GitHub Copilot
  tags: [testing, regression, cli, cobra, golang]
---

# Cmd Refactor Regression Test

Use this skill when:

- moving commands into subdirectories under `cmd/`
- extracting common helpers into `cmd/utils`
- updating command registration in `cmd/root.go`
- changing the behavior of `add`, `install`, `list`, `push`, `remove`, `search`, `status`, `update`, or `version`

## Goal

Run a consistent regression check for the command-layer refactor so that command registration, shared helpers, formatting, and smoke tests stay valid.

## Preconditions

- Run from the repository root.
- If `cmd/` files changed, format them before testing.
- Prefer validating the changed commands first, then run the full test suite.

## Standard Test Flow

### 1. Format changed Go files

Windows PowerShell:

```powershell
Get-ChildItem -Path cmd -Filter *.go -Recurse | ForEach-Object { gofmt -w $_.FullName }
gofmt -w main.go
```

### 2. Run the full Go test suite

```powershell
go test ./...
```

### 3. Run command smoke tests

Always run the commands that were touched. For command-package refactors, the baseline smoke tests are:

```powershell
go run . list --repo
go run . install -h
go run . update -h
go run . push -h
go run . add -h
go run . search -h
go run . remove -h
go run . status -h
go run . list --version
```

## Expected Results

- `gofmt` finishes without syntax errors.
- `go test ./...` exits successfully.
- Every help command exits with code `0` and prints usage.
- `go run . list --repo` exits with code `0` and either:
  - lists configured repositories, or
  - reports that no repositories are configured.

## Failure Handling

If a step fails:

1. Capture the exact command, exit code, and stderr.
2. Identify whether the failure is caused by:
   - missing imports
   - broken package registration in `cmd/root.go`
   - helper extraction regressions in `cmd/utils`
   - stale references to deleted helper files
   - syntax issues introduced during file moves
3. Fix the root cause.
4. Re-run the failed command and then re-run `go test ./...`.

## Reporting Template

When reporting the result, include:

- changed command packages
- whether formatting passed
- whether `go test ./...` passed
- which smoke tests were executed
- any remaining risk, such as untested runtime flows or missing integration coverage
