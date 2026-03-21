---
name: best-practices-cli
description: >
  Apply this skill whenever building, reviewing, scaffolding, or improving a CLI tool — regardless of language.
  Covers directory structure, coding standards, test patterns, and performance optimizations.
  Trigger when the user mentions CLI, command-line tool, terminal app, argparse, commander, cobra, click,
  bin scripts, or asks for help structuring or testing a CLI project.
---

# CLI Best Practices

Use this skill to build CLI tools that are consistent, testable, and production-ready.

## Directory Structure

Follow this layout unless the project already has a stronger local convention:

```
<cli-name>/
├── src/
│   ├── commands/     # one file per command — thin entry points only
│   ├── core/         # business logic, pure functions, no I/O coupling
│   └── utils/        # logger, spinner, formatting helpers
├── test/             # mirrors src/ structure
├── bin/              # entry-point script(s)
└── <config files>    # package.json / pyproject.toml / go.mod etc.
```

Keep commands thin — they parse args and delegate to core. Business logic never lives in command files.

## Coding Standards

- **Single responsibility** — each module does one thing.
- **Dependency injection** — pass I/O, config, and external services in; don't import them globally. Makes testing trivial.
- **Named exports** — avoid default exports for easier refactoring and tree-shaking.
- **Descriptive errors** — surface actionable messages to the user, never raw stack traces. Log verbose detail only in debug mode.
- **Validate inputs early** — check args/flags before doing any real work. Fail fast with a clear usage hint.
- **Exit codes** — `0` for success, non-zero for failures. Document them.

## Test Patterns

- Co-locate unit tests with source files (`.test.ts` / `_test.go` etc.) or mirror in `test/`.
- **Unit-test core logic** independently — no subprocess spawning needed.
- **Integration-test commands** by invoking the parsed handler directly, not the shell binary.
- Use temp directories for filesystem tests; clean up in `afterEach`/`defer`.
- Capture and assert on stdout/stderr output — don't just check exit codes.
- Test the unhappy paths: missing args, bad input, network timeout.

## Optimisations

- Lazy-load heavy dependencies — don't `import` large libs at the top level if only one subcommand needs them.
- Cache config reads; don't re-read from disk on every operation.
- Use a spinner or progress indicator for anything that takes > ~300 ms.
- Stream large outputs rather than buffering the whole result in memory.
- Prefer `async/await` (or goroutines, threads) only for genuinely concurrent I/O — don't add concurrency for its own sake.

## Review Checklist

Before finishing, verify the CLI:

- [ ] Commands are thin wrappers — no business logic inline
- [ ] All user-facing errors are descriptive, not stack traces
- [ ] Inputs validated before side-effects begin
- [ ] Tests cover both happy and error paths
- [ ] Exit codes are consistent and documented
- [ ] No unnecessary blocking or full-buffer I/O

## References

- Read `references/structure-examples.md` for concrete before/after directory layouts and why they work.
- Read `references/testing-patterns.md` for test fixtures, temp-dir helpers, and stdout-capture patterns across languages.
